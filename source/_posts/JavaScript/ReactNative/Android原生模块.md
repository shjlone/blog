---
title: Android原生模块
tags: React Native 
---



## 模块的定义


要实现通信，需要先定义ReactContexBaseJavaModule，并将ReactPackage注册到ReactInstanceManager中


## ReactMethod



注意，@ReactMethod 注解的方法必须满足以下条件：

- 方法必须是公开的（public）。
- 方法不能是静态的（static）。
- 方法的返回类型必须是 void。
- 方法的参数类型必须是 React 支持的类型，例如 String、ReadableMap、Callback、Promise 等。

### 用法

在module中定义可以被JS调用的函数

#### 1. 使用`集成到现有Android应用`中的IndexModule文件，添加一个ReactMethod

```Java


public class IndexModule extends ReactContextBaseJavaModule {

    @NonNull
    @Override
    public String getName() {
        return "IndexModule";
    }

    @ReactMethod
    public void show(String message, int duration) {
        Toast.makeText(getReactApplicationContext(), message, duration).show();
    }

    private static final String DURATION_SHORT_KEY = "SHORT";
    private static final String DURATION_LONG_KEY = "LONG";

    //返回的常量可供JS使用
    @Nullable
    @Override
    public Map<String, Object> getConstants() {
        final Map<String, Object> constants = new HashMap<>();
        constants.put(DURATION_SHORT_KEY, Toast.LENGTH_SHORT);
        constants.put(DURATION_LONG_KEY, Toast.LENGTH_LONG);
        return constants;
    }
}

```

**ReactMethod支持的参数类型映射关系**

- Boolean -> Bool
- Integer -> Number
- Double -> Number
- Float -> Number
- String -> String
- Callback -> function
- ReadableMap -> Object
- ReadableArray -> Array



#### 2. js调用

```JavaScript

//ToastExample.js

import {NativeModules} from 'react-native';

//index对于IndexModule中getName的返回值
export default NativeModules.index;



//其他地方使用
import ToastExample from './ToastExample';
ToastExample.show('hoho', ToastExample.SHORT);

```

### 同步方法

如果需要同步返回结果，可以使用`isBlockingSynchronousMethod`参数，该方法会阻塞JS线程，直到返回结果

```Java
@ReactMethod(isBlockingSynchronousMethod = true)
public String getUniqueIdSync() {
  // 这个方法可以同步返回结果
  return "uniqueId";
}
```


```JavaScript
import { NativeModules } from 'react-native';

let uniqueId = NativeModules.MyModule.getUniqueIdSync();
```


## 回调函数Callback

Callback可以在JS调用原生的方法里回调JS方法

```Java
    //Callback测试
    @ReactMethod
    public void testCallback(String param1, Callback successCallback, Callback errorCallback) {
        try {
            successCallback.invoke(param1);
        } catch (Exception e) {
            errorCallback.invoke("error:" + e.getMessage());
        }
    }
```

```JavaScript

<Button onPress={()=>{
    NativeModules.IndexModule.testCallback("测试回调", (msg)=>{
        NativeModules.IndexModule.show(msg, NativeModules.IndexModule.SHORT);
    }, (error) => {
        NativeModules.IndexModule.show(error, NativeModules.IndexModule.SHORT);
    });
}} title={"Callback测试"}></Button>

```


## Promise

使用Promise，可以在声明了async的异步函数内使用await关键字来调用



```Java
    @ReactMethod
    public void testPromise(String param1, Promise promise) {
        WritableMap map = Arguments.createMap();
        map.putDouble("relativeX",1);
        map.putDouble("relativeY", 1);
        map.putDouble("width", 2);
        map.putDouble("height",3);

        promise.resolve(map);
    }
```

```JavaScript
<Button onPress={
async ()=> {
    var {
        relativeX,
        relativeY,
        width,
        height,
    } = await NativeModules.IndexModule.testPromise('Awesome');
    NativeModules.IndexModule.show(result, NativeModules.IndexModule.SHORT);

}} title={"测试Promise"}></Button>

```




## 发送事件

原生模块可以在没有被调用的情况下通过RCTDeviceEventEmitter往JavaScript发送事件通知，

```Java


    @ReactMethod
    public void testSendEvent(String param1) {
        WritableMap params = Arguments.createMap();
        params.putString("eventProperty", param1);
        setEvent("EventReminder", params);
    }


    /**
     * 往JS发送事件
     * @param eventName
     * @param params
     */
    public void setEvent(String eventName, WritableMap params) {
        getReactApplicationContext().getJSModule(DeviceEventManagerModule.RCTDeviceEventEmitter.class)
                .emit(eventName, params);
    }

```

```JavaScript

<Button onPress={
()=> {
    NativeModules.IndexModule.testSendEvent('emitttt');
}} title={"测试Event Emitter"}></Button>



//监听原生事件
function compnentDidMount() {
    const eventEmitter = new NativeEventEmitter(NativeModules.IndexModule);
    this.listener = eventEmitter.addListener('EventReminder', (reminder) => {
        console.log(reminder.name);
        console.log(reminder.location);
        console.log(reminder.date);
        //toast原生侧发送的消息
        NativeModules.IndexModule.show(reminder.eventProperty, NativeModules.IndexModule.SHORT);
    });
}

function componentWillUnmount() {
    this.listener && this.listener.remove();
}

compnentDidMount();
```



## 从startActivityResult中获取结果

运用Promise，当JS调用某个原生方法，原生打开另一个原生界面，原生界面返回的数据通过Promise再返回给JS。通过以下代码来监听回调

```Java
reactContext.addActivityEventListener(mActivityResultListener);
```


### 具体实例

创建一个ImagePickerModule

```Java

package com.lqd.androidpractice.rn.modules;

import android.util.Log;

import androidx.annotation.NonNull;

import com.facebook.react.bridge.ActivityEventListener;
import com.facebook.react.bridge.BaseActivityEventListener;
import com.facebook.react.bridge.Promise;
import com.facebook.react.bridge.ReactApplicationContext;
import com.facebook.react.bridge.ReactContextBaseJavaModule;
import com.facebook.react.bridge.ReactMethod;

public class ImagePickerModule extends ReactContextBaseJavaModule {

    private static final int IMAGE_PICKER_REQUEST = 467081;
    private static final String E_ACTIVITY_DOES_NOT_EXIST = "E_ACTIVITY_DOES_NOT_EXIST";
    private static final String E_PICKER_CANCELLED = "E_PICKER_CANCELLED";
    private static final String E_FAILED_TO_SHOW_PICKER = "E_FAILED_TO_SHOW_PICKER";
    private static final String E_NO_IMAGE_DATA_FOUND = "E_NO_IMAGE_DATA_FOUND";

    @NonNull
    @Override
    public String getName() {
        return "ImagePickerModule";
    }

    private Promise mPickerPromise;

    private final ActivityEventListener mActivityEventListener = new BaseActivityEventListener() {
        @Override
        public void onActivityResult(int requestCode, int resultCode, android.content.Intent data) {
            if (requestCode == IMAGE_PICKER_REQUEST) {
                if (mPickerPromise != null) {
                    if (resultCode == android.app.Activity.RESULT_CANCELED) {
                        mPickerPromise.reject(E_PICKER_CANCELLED, "Image picker was cancelled");
                    } else if (resultCode == android.app.Activity.RESULT_OK) {
                        android.net.Uri uri = data.getData();
                        if (uri == null) {
                            mPickerPromise.reject(E_NO_IMAGE_DATA_FOUND, "No image data found");
                        } else {
                            mPickerPromise.resolve(uri.toString());
                        }
                    }
                    mPickerPromise = null;
                }
            }
        }
    };

    public ImagePickerModule(ReactApplicationContext reactContext) {
        super(reactContext);
        reactContext.addActivityEventListener(mActivityEventListener);
    }

    /**
     * JS调用选择图片，并回调给JS
     * @param promise
     */
    @ReactMethod
    public void pickImage(final Promise promise) {
        Log.d("ImagePickerModule", "pickImage>>>>>>>>>>>>>>>>>>>>>>>>>>>");
        android.app.Activity currentActivity = getCurrentActivity();
        if (currentActivity == null) {
            promise.reject(E_ACTIVITY_DOES_NOT_EXIST, "Activity doesn't exist");
            return;
        }

        // Store the promise to resolve/reject when picker returns data
        mPickerPromise = promise;

        try {
            final android.content.Intent galleryIntent = new android.content.Intent();
            galleryIntent.setType("image/*");
            galleryIntent.setAction(android.content.Intent.ACTION_GET_CONTENT);
            final android.content.Intent chooserIntent = android.content.Intent.createChooser(galleryIntent, "Pick an image");
            currentActivity.startActivityForResult(chooserIntent, IMAGE_PICKER_REQUEST);
        } catch (android.content.ActivityNotFoundException e) {
            mPickerPromise.reject(E_FAILED_TO_SHOW_PICKER, e);
            mPickerPromise = null;
        }
    }

}


```

JS测试

```JavaScript

<Button onPress={ async ()=>{
    var result = await NativeModules.ImagePickerModule.pickImage();
    console.log(result);
}} title={"测试Activity Result"}></Button>



```


## 监听生命周期事件

```Java

//监听生命周期回调
reactContext.addLifecycleEventListener(new LifecycleEventListener() {
    @Override
    public void onHostResume() {
        
    }

    @Override
    public void onHostPause() {

    }

    @Override
    public void onHostDestroy() {

    }
});

```