---
title: Fragment 
date: 2021-05-01 10:01:01 
toc: true 
tags: Android
---

## 来源

Fragment的概念来源于Android3.0，主要目的是为大屏幕（如平板电脑）上更加动态和灵活的界面设计提供支持。他可添加到Activity中，
所以多个Activity可添加同一个Fragment，提高了代码`复用性`。

## 生命周期

![](./fragment-view-lifecycle.png)

Fragment跟Activity的生命周期类似，不过只有在显示调用`addToBackStack()`时，系统才会将片段放入由宿主 Activity 管理的返回栈。

### 生命周期方法

***onCreate()***

系统会在创建片段时调用此方法。当片段经历暂停或停止状态继而恢复后，

***onCreateView()***

系统会在片段首次绘制其界面时调用此方法。如要为片段绘制界面，从此方法中返回的 View 必须是片段布局的根视图。如果片段未提供界面，可以返回 null。

我们创建视图的方法。

***onViewCreated***

onCrateView执行后立即执行，此时可以获取view

***onPause()***

系统会将此方法作为用户离开片段的第一个信号（但并不总是意味着此片段会被销毁）进行调用。

***onAttach()***

在片段已与 Activity 关联时进行调用（Activity 传递到此方法内）。

***onActivityCreated()***

当 Activity 的 onCreate() 方法已返回时进行调用。此时可以进行fragment与activity的交换操作。

***onDestroyView()***

在移除与片段关联的视图层次结构时进行调用。

***onDetach()***

在取消片段与 Activity 的关联时进行调用。

### Activity和Fragment的生命周期

![](./complete_android_fragment_lifecycle.png)

### Activity, Fragment和FragmentManager生命周期

![](./android-lifecycle-activity-to-fragments.png)

**注意点**

- 注意生命周期的对应，比如Eventbus的注册和取消注册

## 事务

为什么Fragment需要事务呢？我的理解是，由于需要对Fragment进行添加、移除、替换等操作，那么`FragmentManager`的出现是合适的，而`FragmentManager`
内部使用事务的方式来进行管理，就能保证各种操作的`原子性`了。

***基本用法***

```java
// 1.获取FragmentManager，在活动中可以直接通过调用getFragmentManager()方法得到
 fragmentManager = getSupportFragmentManager();
//        fragmentManager = getFragmentManager();
 // 2.开启一个事务，通过调用beginTransaction()方法开启
 transaction = fragmentManager.beginTransaction();
 // 3.向容器内添加或替换碎片，一般使用replace()方法实现，需要传入容器的id和待添加的碎片实例
 transaction.replace(R.id.testFragment, fragment);  //fr_container不能为fragment布局，可使用线性布局相对布局等。
 // 4.使用addToBackStack()方法，将事务添加到返回栈中，填入的是用于描述返回栈的一个名字
 transaction.addToBackStack(null);
 // 5.提交事物,调用commit()方法来完成
 transaction.commit();

```

有几个注意点：

- FragmentActivity和Fragment都有自己的FragmentManager

### 事务的操作

## 如何通信

可通过以下一些方式进行通信：

- Framgnet中直接获取Activity的引用
- Activity中通过FragmentManager获取对应Fragment的引用
- 通过函数回调  
- setArguments将参数传递给Fragment  
- 共享ViewModel
- Fragment Result API

### 直接获取引用

```java
//Fragment中获取Activity中的控件
View listView = getActivity().findViewById(R.id.list);

//Activity中获取某个Fragment
ExampleFragment fragment = (ExampleFragment) getSupportFragmentManager().findFragmentById(R.id.example_fragment);

```

这种方式虽然能方便的获取，但从代码设计的角度看是不合适的，`耦合性`太高了

### 函数回调

```java
public static class FragmentA extends ListFragment {
    ...
    // Container Activity must implement this interface
    //定义回调接口
    public interface OnArticleSelectedListener {
        public void onArticleSelected(Uri articleUri);
    }
    ...
}


public static class FragmentA extends ListFragment {
    OnArticleSelectedListener listener;
    ...
    @Override
    public void onAttach(Context context) {
        super.onAttach(context);
        try {
        //宿主实现该接口，context就是宿主Activity
            listener = (OnArticleSelectedListener) context;
        } catch (ClassCastException e) {
            throw new ClassCastException(context.toString() + " must implement OnArticleSelectedListener");
        }
    }
    
    @Override
    public void onListItemClick(ListView l, View v, int position, long id) {
        // Append the clicked item's row ID with the content provider Uri
        Uri noteUri = ContentUris.withAppendedId(ArticleColumns.CONTENT_URI, id);
        // Send the event and Uri to the host activity
        // 将数据传给Activity，宿主中操作其他Fragment
        listener.onArticleSelected(noteUri);
    }

    ...
}


```

### 共享ViewModel

#### 与宿主Activity通信

```Java
public class ItemViewModel extends ViewModel {
    private final MutableLiveData<Item> selectedItem = new MutableLiveData<Item>();
    public void selectItem(Item item) {
        selectedItem.setValue(item);
    }
    public LiveData<Item> getSelectedItem() {
        return selectedItem;
    }
}

public class MainActivity extends AppCompatActivity {
    private ItemViewModel viewModel;

    @Override
    public void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        //注意ViewModelProvider的参数
        viewModel = new ViewModelProvider(this).get(ItemViewModel.class);
        viewModel.getSelectedItem().observe(this, item -> {
            // Perform an action with the latest item data
        });
    }
}

public class ListFragment extends Fragment {
    private ItemViewModel viewModel;

    @Override
    public void onViewCreated(@NonNull View view, Bundle savedInstanceState) {
        super.onViewCreated(view, savedInstanceState);
        //注意requireActivity
        viewModel = new ViewModelProvider(requireActivity()).get(ItemViewModel.class);

        ...

        items.setOnClickListener(item -> {
            // Set a new item
            viewModel.select(item);
        });
    }
}

```

将ViewModel放到同一范围，这样返回的是同一个ViewModel。

#### 与其他Fragment通信

```java
public class ListViewModel extends ViewModel {
    private final MutableLiveData<Set<Filter>> filters = new MutableLiveData<>();

    private final LiveData<List<Item>> originalList = ...;
    private final LiveData<List<Item>> filteredList = ...;

    public LiveData<List<Item>> getFilteredList() {
        return filteredList;
    }

    public LiveData<Set<Filter>> getFilters() {
        return filters;
    }

    public void addFilter(Filter filter) { ... }

    public void removeFilter(Filter filter) { ... }
}

public class ListFragment extends Fragment {
    private ListViewModel viewModel;

    @Override
    public void onViewCreated(@NonNull View view, Bundle savedInstanceState) {
        super.onViewCreated(view, savedInstanceState);
        //保证在同一个范围
        viewModel = new ViewModelProvider(requireActivity()).get(ListViewModel.class);
        viewModel.getFilteredList().observe(getViewLifecycleOwner(), list -> {
            // Update the list UI
        });
    }
}

public class FilterFragment extends Fragment {
    private ListViewModel viewModel;

    @Override
    public void onViewCreated(@NonNull View view, Bundle savedInstanceState) {
    //保证在同一个范围
        viewModel = new ViewModelProvider(requireActivity()).get(ListViewModel.class);
        viewModel.getFilters().observe(getViewLifecycleOwner(), set -> {
            // Update the selected filters UI
        });
    }

    public void onFilterSelected(Filter filter) {
        viewModel.addFilter(filter);
    }

    public void onFilterDeselected(Filter filter) {
        viewModel.removeFilter(filter);
    }
}

```

#### 父子Fragment通信

```java
public class ListFragment extends Fragment {
    private ListViewModel viewModel;

    @Override
    public void onViewCreated(@NonNull View view, Bundle savedInstanceState) {
        viewModel = new ViewModelProvider(this).get(ListViewModel.class);
        viewModel.getFilteredList().observe(getViewLifecycleOwner(), list -> {
            // Update the list UI
        }
    }
}

public class ChildFragment extends Fragment {
    private ListViewModel viewModel;
    @Override
    public void onViewCreated(@NonNull View view, Bundle savedInstanceState) {
    //注意参数
        viewModel = new ViewModelProvider(requireParentFragment()).get(ListViewModel.class);
        ...
    }
}

```

### Fragment Result API

在某些情况下，您可能要在 Fragment 之间或 Fragment 与其宿主 Activity 之间传递一次性值。则可以使用setFragmentResultListener。
它的原理是在getParentFragmentManager实现了观察者模式。

#### 与其他Fragment通信

![](./fragment-a-to-b.png)

一旦 Fragment A 处于 STARTED 状态，它就会收到结果并执行监听器回调。

对于给定的键，只能有一个监听器和结果。如果您对同一个键多次调用 setFragmentResult()，并且监听器未处于 STARTED 状态，则系统会将所有待处理的结果替换为更新后的结果。如果您设置的结果没有相应的监听器来接收，则结果会存储在 FragmentManager 中，直到您设置一个具有相同键的监听器。监听器收到结果并触发 onFragmentResult() 回调后，结果会被清除。这种行为有两个主要影响：

- 返回堆栈上的 Fragment 只有在被弹出且处于 STARTED 状态之后才会收到结果。
- 如果在设置结果时监听结果的 Fragment 处于 STARTED 状态，则会立即触发监听器的回调。

```java
@Override
public void onCreate(@Nullable Bundle savedInstanceState) {
    super.onCreate(savedInstanceState);
    //监听某个事件
    getParentFragmentManager().setFragmentResultListener("requestKey", this, new FragmentResultListener() {
        @Override
        public void onFragmentResult(@NonNull String requestKey, @NonNull Bundle bundle) {
            // We use a String here, but any type that can be put in a Bundle is supported
            String result = bundle.getString("bundleKey");
            // Do something with the result
        }
    });
}

//在Fragment中触发该事件
button.setOnClickListener(new View.OnClickListener() {
    @Override
    public void onClick(View v) {
        Bundle result = new Bundle();
        result.putString("bundleKey", "result");
        getParentFragmentManager().setFragmentResult("requestKey", result);
    }
});

```

#### 父子Fragment通信

![](./pass-parent-child.png)

```java
@Override
public void onCreate(@Nullable Bundle savedInstanceState) {
    super.onCreate(savedInstanceState);
    // We set the listener on the child fragmentManager
    getChildFragmentManager()
        .setFragmentResultListener("requestKey", this, new FragmentResultListener() {
            @Override
            public void onFragmentResult(@NonNull String requestKey, @NonNull Bundle bundle) {
                String result = bundle.getString("bundleKey");
                // Do something with the result
            }
        });
}

button.setOnClickListener(new View.OnClickListener() {
    @Override
    public void onClick(View v) {
        Bundle result = new Bundle();
        result.putString("bundleKey", "result");
        // The child fragment needs to still set the result on its parent fragment manager
        getParentFragmentManager().setFragmentResult("requestKey", result);
    }
});


```

#### 与宿主Activity通信

```java
class MainActivity extends AppCompatActivity {
    @Override
    public void onCreate(@Nullable Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        getSupportFragmentManager().setFragmentResultListener("requestKey", this, new FragmentResultListener() {
            @Override
            public void onFragmentResult(@NonNull String requestKey, @NonNull Bundle bundle) {
                // We use a String here, but any type that can be put in a Bundle is supported
                String result = bundle.getString("bundleKey");
                // Do something with the result
            }
        });
    }
}

```

## 参考资料

- [https://developer.android.com/guide/components/fragments?hl=zh-cn](https://developer.android.com/guide/components/fragments?hl=zh-cn)
- [https://developer.android.com/guide/fragments?hl=zh-cn](https://developer.android.com/guide/fragments?hl=zh-cn)
- [https://github.com/xxv/android-lifecycle](https://github.com/xxv/android-lifecycle)
