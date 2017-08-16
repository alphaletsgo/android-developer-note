我们在fragment和adapter中经常会用到`LayoutInflater.inflate()`或`View.inflate()`来解析xml布局文件以得到View，那么这两者之间使用上有什么区别么？同时它们传参该如何理解呢？
首先我们来看一下LayoutInflater.inflate()是如何使用的：
```java
//首先我们会创建一个LayoutInflater对象
LayoutInflater inflater = LayoutInflater.from(getContext());
```
当我们在通过inflater对象调用inflate方法时，发现它存在多个重载方法：
```java
public View inflate(@LayoutRes int resource, @Nullable ViewGroup root);
public View inflate(XmlPullParser parser, @Nullable ViewGroup root);
public View inflate(@LayoutRes int resource, @Nullable ViewGroup root, boolean attachToRoot);
public View inflate(XmlPullParser parser, @Nullable ViewGroup root, boolean attachToRoot);
```


### LayoutInflater.inflate()


### LayoutInflater.inflate()和View.inflate()区别？
下面我们来看一下它们的源码，首先是View.inflate():
```java
  public static View inflate(Context context, @LayoutRes int resource, ViewGroup root) {
        LayoutInflater factory = LayoutInflater.from(context);
        return factory.inflate(resource, root);
    }
```




