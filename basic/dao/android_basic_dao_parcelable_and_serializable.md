### 概念

- Serializable

Serializable的作用是为了保存对象的属性到本地文件、数据库、网络流、rmi以方便数据传输，这种传输可以是程序内的也可以是两个程序间的。

- Parcelable

Parcelable的设计初衷是因为Serializable效率过慢，为了在程序内不同组件间以及不同Android程序间(AIDL)高效的传输数据而设计，这些数据仅在内存中存在。Parcelable是IBinder通信消息的载体。

### 选择
Parcelable的性能比Serializable好，在内存开销方面较小，所以在内存间数据传输时推荐使用Parcelable，如activity间传输数据，而Serializable可将数据持久化方便保存，所以在需要保存或网络传输数据时选择Serializable，因为android不同版本Parcelable可能不同，所以不推荐使用Parcelable进行数据持久化。

### Parcelable使用案例

相对于Serializable的使用Parcelable要复杂一些，实现Parcelable时需要实现writeToParcel、describeContents函数以及静态的CREATOR变量，实际上就是将如何打包和解包的工作自己来定义，而序列化的这些操作完全由底层实现。
```java
public class MyParcelable implements Parcelable {
    private int mData;
    private String mStr;

    public int describeContents() {
        return 0;
    }

    // 写数据进行保存
    public void writeToParcel(Parcel out, int flags) {
        out.writeInt(mData);
        out.writeString(mStr);
    }

    // 用来创建自定义的Parcelable的对象
    public static final Parcelable.Creator<MyParcelable> CREATOR= new Parcelable.Creator<MyParcelable>() {
        public MyParcelable createFromParcel(Parcel in) {
            return new MyParcelable(in);
        }

        public MyParcelable[] newArray(int size) {
            return new MyParcelable[size];
        }
    };

    // 读数据进行恢复
    private MyParcelable(Parcel in) {
        mData = in.readInt();
        mStr = in.readString();
    }
}
```