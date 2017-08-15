# Android 使用【AIDL】调用外部服务

前面我们已经详细介绍了如何使用aidl应用。接下来我们接着第一个音乐播放器的例子扩展至两个应用之间使用aidl通信，也就是这里所说的调用外部服务，这里我们使用Android studio进行开发部署。

1) 介绍完背景下面我们继续，首先我们来创建服务端程序，创建一个项目再创建一个android server模块和一个client模块，整体结构如下图：

![img](/imgs/WX20170808-133504.png)

2) 接着我们在server模块的main目录下创建一个aidl文件夹，然后再创建一个包`cn.isif.android.server`，在该包下创建 IMusicControlService.aidl 文件，内容如下：（这里的包名必须和项目的包名一致）

```java 
// IMusicControlService.aidl
package cn.isif.android.server;
// Declare any non-default types here with import statements
import cn.isif.android.server.Music;

interface IMusicControlService {
    void playMusic();
	void stopMusic();
	void inLoadMusic(in Music m);
	void outLoadMusic(out Music m);
	void inOutLoadMusic(inout Music m);

}
```
3) 由上面代码可以看出来我们使用了自定义的类，使用自定义的对象这种情况处理起来会比较复杂点，大致分为如下步骤：
- 创建自定义类Music，该类必须实现Parcelable类，以及一些必须方法
```java
package cn.isif.android.server;

import android.os.Parcel;
import android.os.Parcelable;

/**
 * Created by ola on 2017/8/7.
 */

public class Music implements Parcelable{
    private int id;
    private String name;

    protected Music(Parcel in) {
        id = in.readInt();
        name = in.readString();
    }

    public Music(int id,String name){
        this.id = id;
        this.name = name;
    }
    
    public Music(){}
    
    //在编译时，编译器告诉我必须提供如上两种构造方法，为了不让编译器报错我这里实现了这两种构造方法

    public static final Creator<Music> CREATOR = new Creator<Music>() {
        @Override
        public Music createFromParcel(Parcel in) {
            return new Music(in);
        }

        @Override
        public Music[] newArray(int size) {
            return new Music[size];
        }
    };

    @Override
    public int describeContents() {
        return 0;
    }

    @Override
    public void writeToParcel(Parcel parcel, int i) {
        parcel.writeInt(id);
        parcel.writeString(name);
    }

    /**
     * 必须提供该方法，否则会抛异常
     * @param source
     */
    public void readFromParcel(Parcel source) {
        id = source.readInt();
        name = source.readString();
    }
}
```
- 创建Music.aidl文件，文件里只作声明
```java
// Music.aidl
package cn.isif.android.server;

parcelable Music;

```
- 如果需要传输自定义对象必须在IMusicControlService.aidl 文件中作导入操作，不然会报异常，同时需要做in和out标识;

![](/imgs/WechatIMG1469.jpeg)

> 关于AIDL中的定向tag表示了在跨进程通信中数据的流向，其中 in 表示数据只能由客户端流向服务端， out 表示数据只能由服务端流向客户端，而 inout 则表示数据可在服务端与客户端之间双向流通。其中，数据流向是针对在客户端中的那个传入方法的对象而言的。in 为定向 tag 的话表现为服务端将会接收到一个那个对象的完整数据，但是客户端的那个对象不会因为服务端对传参的修改而发生变动；out 的话表现为服务端将会接收到那个对象的参数为空的对象，但是在服务端对接收到的空对象有任何修改之后客户端将会同步变动；inout 为定向 tag 的情况下，服务端将会接收到客户端传来对象的完整信息，并且客户端将会同步服务端对该对象的任何变动。：

4) 创建一个service(TestService.class)
```java
package cn.isif.android.server;

import android.app.Service;
import android.content.Intent;
import android.os.IBinder;
import android.os.RemoteException;
import android.support.annotation.Nullable;
import android.util.Log;

/**
 * Created by ola on 2017/8/7.
 */

public class TestService extends Service {
    public static final String TAG = "TestService";
    public boolean isStop = false;


    private final IMusicControlService.Stub binder = new IMusicControlService.Stub() {
        @Override
        public void playMusic() throws RemoteException {
            Log.d(TAG,"play music ing");
        }

        @Override
        public void stopMusic() throws RemoteException {
            Log.d(TAG,"stop music end");
            isStop = true;
        }

        @Override
        public void inLoadMusic(Music m) throws RemoteException {

        }

        @Override
        public void outLoadMusic(Music m) throws RemoteException {

        }

        @Override
        public void inOutLoadMusic(Music m) throws RemoteException {

        }
    };
    @Nullable
    @Override
    public IBinder onBind(Intent intent) {
        return binder;
    }
}

```
5) 定义好service后我们还需要在manifest文件中配置一下，由于我们是外部调用所以我们必须配置intent-filter。

```xml
<?xml version="1.0" encoding="utf-8"?>
<manifest xmlns:android="http://schemas.android.com/apk/res/android"
    package="cn.isif.android.server">

    <application
        android:allowBackup="true"
        android:icon="@mipmap/ic_launcher"
        android:label="@string/app_name"
        android:supportsRtl="true"
        android:theme="@style/AppTheme">
        <activity android:name=".MainActivity">
            <intent-filter>
                <action android:name="android.intent.action.MAIN" />

                <category android:name="android.intent.category.LAUNCHER" />
            </intent-filter>
        </activity>
        <service android:name=".TestService">
            <intent-filter>
                <action android:name="cn.isif.TestServer" />
            </intent-filter>
        </service>
    </application>

</manifest>
```
好了，在该例子server中我们需要做的就这些，这个时候点击编译可能会报如下错误：
![](/imgs/WX20170808-140528.png)

这时，需要我们在grade文件中加入如下配置：
```
sourceSets {
        main {
//            manifest.srcFile 'src/main/AndroidManifest.xml'
            java.srcDirs = ['src/main/java', 'src/main/aidl']
//            resources.srcDirs = ['src/main/java', 'src/main/aidl']
//            aidl.srcDirs = ['src/main/aidl']
//            res.srcDirs = ['src/main/res']
//            assets.srcDirs = ['src/main/assets']
        }
    }
```

最后，再来一张server端概览图：
![](/imgs/WX20170808-141007.png)

6) 我们接着些client端，拷贝server端aidl包下所有内容到client main文件夹下,不做任何改变，编译时同样可能出现找不到Music类，解决办法参考第5步：
![](/imgs/WX20170808-141405.png)

7) 编写客户端调用demo，创建xml文件,为了简单起见我这里只定义了两个按钮：
```xml
<?xml version="1.0" encoding="utf-8"?>
<RelativeLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools" android:id="@+id/activity_main"
    android:layout_width="match_parent" android:layout_height="match_parent"
    android:paddingBottom="@dimen/activity_vertical_margin"
    android:paddingLeft="@dimen/activity_horizontal_margin"
    android:paddingRight="@dimen/activity_horizontal_margin"
    android:paddingTop="@dimen/activity_vertical_margin"
    tools:context="cn.isif.android.client.MainActivity">

    <Button
        android:text="start"
        android:onClick="onStart"
        android:clickable="true"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content" />

    <Button
        android:text="stop"
        android:onClick="onStop"
        android:clickable="true"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_alignParentTop="true"
        android:layout_alignParentRight="true"
        android:layout_alignParentEnd="true" />
</RelativeLayout>

```
8) 在activity中实现调用

```java
package cn.isif.android.client;

import android.content.ComponentName;
import android.content.Context;
import android.content.Intent;
import android.content.ServiceConnection;
import android.os.IBinder;
import android.os.RemoteException;
import android.support.v7.app.AppCompatActivity;
import android.os.Bundle;
import android.util.Log;
import android.view.View;

import cn.isif.android.server.IMusicControlService;

public class MainActivity extends AppCompatActivity {
    public static final String TAG = "MainActivity";
    private IMusicControlService iMusicControlService = null;
    private final ServiceConnection serviceConnection = new ServiceConnection() {
        @Override
        public void onServiceConnected(ComponentName componentName, IBinder iBinder) {
            iMusicControlService = IMusicControlService.Stub.asInterface(iBinder);
            Log.d(TAG,"连接成功！！！");
        }

        @Override
        public void onServiceDisconnected(ComponentName componentName) {
            iMusicControlService = null;
            Log.d(TAG,"已断开连接");
        }
    };

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        //发起连接远程service
        Intent intent = new Intent();
        intent.setAction("cn.isif.TestServer");
        intent.setComponent(new ComponentName("cn.isif.android.server", "cn.isif.android.server.TestService"));
        bindService(intent,serviceConnection, Context.BIND_AUTO_CREATE);
    }

    /**
     * 调用远程方法
     *
     * @param view
     */
    public void onStart(View view){
        try {
            iMusicControlService.playMusic();
        } catch (RemoteException e) {
            e.printStackTrace();
        }
    }

    /**
     * 调用远程方法
     * 
     * @param view
     */
    public void onStop(View view){
        try {
            iMusicControlService.stopMusic();
        } catch (RemoteException e) {
            e.printStackTrace();
        }
    }

}

```

客户端概览图：

![](/imgs/WX20170808-142113.png)

至此该案例已完成！
