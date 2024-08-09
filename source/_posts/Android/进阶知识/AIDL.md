---
title: AIDL
tags: Android
toc: true
---



Binder是一个工作在Linux层面的驱动，这一段驱动运行在内核态。Binder本身又是一种架构，这种架构提供了服务端、Binder驱动和客户端三个模块。

![](./aidl_1.png)

## 服务端

Binder服务端实际上就是一个Binder类的对象，当我们创建一个Binder对象的时候，Binder内部就会启动一个隐藏线程，该线程的主要作用就是接收Binder驱动发送
来的消息，那么Binder驱动为 什么会给Binder服务端的线程发送消息呢?原因很简单，我们在客户端调用服务端的时候并不能直接调用服务端相应的类和方法，
只能通过Binder驱动来调用。当服务端的隐藏线程收到Binder 驱动发来的消息之后，就会回调服务端的onTransact方法，我们来看看这个方法的方法头:

```java
/**
@param code 指定客户端要调 用服务端的哪一个方法
@param data 客户端传来的参数
@param reply    表示服务端返回的参数
@param flags    表示客户端的调用是否有返回值，0表示服务端执行完成之后有返回值，1表示服务 端执行完后没有返回值。
*/
protected boolean onTransact(int code, Parcel data, Parcel reply, int flags) throws RemoteException


public class MyAddBinder extends Binder {
    private final static int ADD = 1;

    @Override
    protected boolean onTransact(int code, Parcel data, Parcel reply, int flags) throws RemoteException {
        switch (code) {
            case ADD:
                data.enforceInterface("MyAddBinder");
                int a = data.readInt();
                int b = data.readInt();
                int add = add(a, b);
                reply.writeInt(add);
                return true;
        }
        return super.onTransact(code, data, reply, flags);
    }
    
    public int add(int a, int b) {
        return a + b;
    }
}

public class MyService extends Service {
    @Nullable
    @Override
    public IBinder onBind(Intent intent) {
        return new MyAddBinder();
    }
}
```

## Binder驱动

Binder驱动是Binder服务端和Binder客户端之间连接的一个桥梁，当一个服务端Binder被创建出来的时候，系统同时会在Binder驱动中创建另外一个Binder对象，
当客户端想要访问远程的Binder服务端的时候，都是通过这个Binder对象来完成的。那么Binder驱动中的这个对象要怎么样获取呢?其实很简单，
这个BInder对象就是我们用绑定的方式启动一个Service服务时，在绑定成功时所获取的那个IBinder对象。

```java
 boolean b = bindService(intent, new ServiceConnection() {
        @Override
        public void onServiceConnected(ComponentName name, IBinder service) {
        //这个service就是Binder驱动中创建的Binder对象
                mRemote = service;
       }
       @Override
       public void onServiceDisconnected(ComponentName name) {
       }
   
   }, Service.BIND_AUTO_CREATE);                
```

## 客户端

在客户端获取Binder驱动中的Binder对象，然后调用该对象中的 transact方法进行数据传递。客户端在向服务端发送消息的时候是以线程间通信的模式来进行的，
而且调用服务端代码是同步进行的，也就是说线程会阻塞。

```java
int code = 1;
//向服务端发送的数据
Parcel data = Parcel.obtain();
//接收服务端返回的数据
Parcel reply = Parcel.obtain();
data.writeInterfaceToken("MyAddBinder");
data.writeInt(10);
data.writeInt(9);
try {
    mRemote.transact(code, data, reply, 0);
    int i = reply.readInt();
    Log.d("google.sang", "add: " + i);
    reply.recycle();
    data.recycle();
} catch (RemoteException e) {
    e.printStackTrace();
}
```
