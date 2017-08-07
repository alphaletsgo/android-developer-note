# 通过AIDL调用Service

我们通过控制音乐播放的小例子来初步认识一下AIDL,该例子只是简单介绍了如何使用aidl。

### 1.创建aidl文件

![img](/imgs/WX20170807-170603.png)

首先创建IMusicControlService.aidl文件，包名一定要和当前工程的包名一样哦（如上图）！

```java
package com.androidmanual.androidstud2.servic

interface IMusicControlService
{
	voidplayMusic();-------->播放音乐
	voidstopMusic();------->停止播放音乐
}
```
点击编译会自动生成IMusicControlService.java文件。

### 2.在res/layout目录下创建布局文件：

startserviceactivity.xml
```xml
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android
    android:layout_width="fill_parent
    android:layout_height="fill_parent
    android:orientation="vertical" >
    <TextView
        android:id="@+id/tv_main
        android:layout_width="fill_parent
        android:layout_height="wrap_content
        android:text="@string/hello
        android:textSize="18px" />
    <Button
        android:id="@+id/btn_play
        android:layout_width="wrap_content
        android:layout_height="wrap_content
        android:text="播放音乐" />
    <Button
        android:id="@+id/btn_stop
        android:layout_width="wrap_content
        android:layout_height="wrap_content
        android:text="停止播放" />
</LinearLayout>
```
### 3.创建一个TestService类，在该类的内部实例化IMusicControlService中的`playMusic()`和`stopMusic()`接口

```java
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
    };
    @Nullable
    @Override
    public IBinder onBind(Intent intent) {
        return binder;
    }
}
```
在该类的`onBind()`方法中返回上面实例的binder，即`return binder`;

### 4.创建StartServiceActivity类继承Activity类，在该类中通过ServiceConnection和后台的service连接

```java
private final ServiceConnection serviceConnection = new ServiceConnection() {
	// 第一次连接service时会调用这个方法
	@Override
	public void onServiceConnected(ComponentName name, IBinder service) {
		iMusicControlService = IMusicControlService.Stub
				.asInterface(service);
	}
	// service断开的时候会调用这个方法
	@Override
	public void onServiceDisconnected(ComponentName name) {
		// TODO Auto-generated method stub
		System.out.println("service unconntection");
		iMusicControlService = null;
	}
};
```
在oncreate方法中绑定service：
```java
Intent intent= new Intent();
intent.setClass(StartServiceActivity.this,ControlMusicService.class);
bindService(intent,serviceConnection,Context.BIND_AUTO_CREATE);
```
在点击playmusic按钮被点击时，执行如下代码：
```java
iMusicControlService.playMusic();
```
在点击stopmusic按钮被点击时，执行如下代码：
```java
iMusicControlService.stopMusic();
unbindService(serviceConnection);
```
好了这样就通过在Activity中通过aidl控制service了。