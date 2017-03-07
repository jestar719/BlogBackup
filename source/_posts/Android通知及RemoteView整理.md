---
title: Android通知及RemoteViews整理
date: 2017-02-27 16:53:59
tags: Android
---
前两天看了官方的教学视频,讲的是使用`NotificationCompact`来使用通知.后来网上搜索了关与通知的文章,发现示例还是使用的`Notification`.从4.4到AndroidN都有关与通知的更新.所以通知最好还是用官方提供的兼容工具来创建和使用.
<!-- more -->
## 通知
### 通知创建和使用的流程
* 创建Notification.Builder的实例
* 通过Builder来设置通知的相关属性,并通过builder()方法来获取设置好的通知
* 获取NotificationManager来发送通知.

```java
Notification.Builder builder = new Notification.Builder(getApplicationContext());
builder.setContentTitle("这是标题");
Notification build = builder.build();
NotificationManager nm = (NotificationManager) getSystemService(Context.NOTIFICATION_SERVICE);
nm.notify(0,build);
```

官方的兼容包中提供了兼容工具类,用来在不同版本下创建和使用兼容的通知
```java
NotificationCompat.Builder builder = new NotificationCompat.Builder(getApplicationContext());
builder.setContentTitle("这是标题");
Notification nofitication = builder.build();
NotificationManagerCompat.from(getApplicationContext()).notify(0,Notification);
```

使用`NotificationCompact`和`NotificationManagerCompat`两个类来提供兼容性.实际上底层的实现还是一样的.
`NotificationManagerCompat`通过静态方法from(Context Context)创建了一个自身的实例.实例中通过参数Context获取了一个`NotificationManager`,实际上通知还是通过它来发送.
实际上这两个工具类内部会根据应用运行机器的版本来创建对应的实现类来提供兼容性.

### 通知的设置
通知是通过Builder来设置的.常见的设置如下
* 设置标题 setContentTitle(CharSequence)
* 设置文字内容 setContentText(CharSequence)
* 设置副文字 setSubContextText(CharSequence)
* 设置大图标 setLargeIcon(Bitmap) 不设置时默认为应用图标
* 设置小图标 setSmallIcon(int)  参数为图片资源Id
* 设置颜色 setColor(int color) 参数为RGB值
* 设置声音 setSound
* 设置震动 setVibrate
* 设置闪灯 setLight(int,int,int) 第一个int为颜色的资源id,第二个和第三个是闪灯开启和关闭的时间,单位为毫秒
* 设置点击后是否自动取消 setAutoCancel(boolean)

#### 进度条
从4.1开始通知里自带进度条.通过`setProgress(in max,int progress,boolean indeterminate)`来设置进度
* max 进度最大值
* progress 当前进度
* indeterminate 是否是无限循环 true表示无限循环进度条,不需要进度及最大值.false反之

当进度条加载完毕后可以能过`setProgress(0,0,false)`来隐藏进度条.

### 消息发送
发送通知需要一个id和一个Notification.通过NotificationManager发送到远程的NotificationService上.用Notification里的数据更新远程的UI.
使用notify(int id,Notification notify)可以不断的更新同一个通知.另外此方法可以有个重载的方法可以设置一个Tag.
#### ID
id需要自己设置,一个Id对应一个通知.NotificationService收到Notification后会根据Id去查找对应的View.如果不存在则创建一个.如果存在则更新UI
也就是说如果id相同时,后续的通知会把之前的通知替换掉,如果id不同时会弹出多个通知.

### PendingIntent
PendingIntent用与处理通知的交互事件(也就是点击事件).PendingIntent表示的是一个待定的意图.
可以用来开启一个Activity,Service或广播接收者,它是对Intent的包装.对外部提供这个Intent.通知被点击时系统会按照PendingIntent设置的方法及Intent设置的目标去开启对应的Activity,Service或广播接收者.
创建PendingIntent的方法如下
```java
PendingIntent.startActivity(Context context,int requestCode,Intent intent,int flag);
PendingIntent.startService(Context context,int requestCode,Intent intent,int flag);
PendingIntent.startBroadcast(Context context,int requestCode,Intent intent,int flag);
```

所需要的参数都一样,startActivity还可以额外带一个bundle参数.
* Context 启动所需上下文.实际上传递到外部的是应用的Application.所以在点击通知时哪怕应用没启动,也可以开启对应的组件.
* requestCode 请求码,自定义
* intent 启动所需组件的intent,因为Context是Application.所以启动Activity需要NEW_TASK标识.
* flag PendingIntent的标识

对与更新同一个id的通知而言,这两个没有作用,不管PendingIntent是否有变化,始终都是新的通知替换掉原来的.
当每次都使用不同的id来弹出多个通知时,这些flag才会发生作用.这里需要涉及PendingIntent的匹配问题.
两个PendingIntent匹配指的是requestCode相同并且PendingIntent内部的intent都匹配.
当两个intent内部的ComponentName和intent-filter相同,这两个intent就算是匹配的.ComponentName相同指的是这个intent希望开启的组件是相同的.
多个通知的PendingIntent如果不匹配是互不干扰的.如果是匹配的.则根据flag有不同的处理方式.
* FLAG_ONE_SHOT
    PendingIntent只执行一次,也就是说多个含有匹配的PendingIntent的通知,只要有一个被点击了,那么其它的就点击无效.通知里如果还含有其它的PendingIntent,还是可以被点击的.
* FLAG_NO_CREAT
    如果PendingIntent不存在时并不创建新的实例,而是返回null.这个很少用.
* FLAG_CANCEL_CURRENT
    如果PendingIntent已经存在,则取消前者,创建新的实例以保持数据为最新.多个含有匹配PendingIntent的通知,只有最新的可以被点击,其它的点击无效.
* FLAG_UPDATE_CURRENT
    多个含有PendingIntent的通知都会更新其Intent中的Extra数据与最新的通知中保持一至.

### Action
通知的点击交互被称为Action.
通知自身的点击交互设置方法是setContentIntent(PendingIntent).
在新版本中通知可以添加不同的按钮来对应不同的Action,添加Action的方法有两种
```java
builer.addAction(Action)
builer.addAction(int iconId,CharSequence title,PendingIntent intent)
```
实际上都是添加Action实例.区别在与Action是否自己来构造.Action在通知上显示为一个带图标的文字按钮.必须的三个参数如下
* IconId 图标的资源id
* Title 按钮的文字
* PendingIntent 按钮的点击处理

### Style通知样式
在4.0版本后通知可以被拉伸,通知可以设置不同的扩展样式.
```java
builer.setStyle(NotificationCompat.style);
```
系统提供了四种默认样式
* MediaStyle  媒体播放器样式.可以最多提供5个Action,用与后台播放媒体文件
```java
builer.setStyle(new NotificationCompat.MediaStyle()
                .setMediaSession(MediaSession.Token));
```
* BigPictureStyle 大图样式.可以附加一张图片,扩展显示
```java
buider.setStyle(new NotificationCompat.BigPictureStyle()
                .bigPicture(Bitmap));
```
* InboxStyle 文字式表样式,可以以列表的形式显示最多5行的文字
```java
builer.setStyle(new NotificationCompat.InboxStyle()
                .setLine(CharSequence)
                .setLine(CharSequence));
```
* BigTextStyle 多文字内容.下拉显示全部文字
```java
buider.setStyle(new NotificationCompat.BigTextStyle()
                .bitText(CharSequence));
```

## RemoteViews
RemoteViews是Android提供的一种远程服务,可以跨进程显示及更新UI,通知及桌面小部件就是基与它封装的.
其核心是跨进程传递数据.把UI的布局及对其View的操作封装到RemoteViews里,传递给其它进程.其它进程收到RemoteViews后调用RemoteViews的方法创建或更新UI并显示

### RemoteViews的原理
RemoteViews本身并不是个View.它是个Pacelable,是个可以跨进程传递的一个数据.它携带了UI的布局及对应的数据
其它进程收到这个RemoteViews后会在其进程中根据这个布局inflat出对应的View.然后根据RemoteViews里的属性反射设置到对应的View上,然后根据设置点击监听.监听的处理就是调用对应的PendingIntent.
因为是跨进程,所以无法直接操作View,所以系统把对view的一个操作定义为Action对象.Action对象本身也是个Pacelable,所以可以跨进程.其封装了操作View的数据.远程进程遍历所有的Action并执行其apply方法通过反射更新View
向先前说的设置view的属性,设置点击监听,都是一个Action.

### RemoteViews的创建
```java
RemoteViews remoteview=new RemoteViews(String packageName,int layoutId);
如果通知需要使用自定义UI可以通过创建RemoteView实现
builer.setCustomContentView(RemoteViews);
或
builer.setCustomBigContentView(RemoteViews);
```
packageName表示的是客户端的包名,layoutId表示布局Id.

### RemoteViews的设置
RemoteViews对View的操作是在远程进行的.所以客户端只是封装了操作.通过RemoteViews提供的一系列set方法,RemoteViews把所有设置都封装成Action保存到集合中.
* 设置属性参数为view的id及属性值如
    ```java
    setText(int id,CharSequence text)
    setColor(int Id,int color)
    ```
* 设置属性参数为view的id,方法名及属性值如
    ```java
    setBitmap(int id,String methodName,Bitmap bitmap)
    setBoolean(int id,String methodName,boolean value)
    ```
* 单个View设置点击事件
    ```java
    setOnClickPendingIntent(int id,PendingIntent intent);
    ```
* 集合View设置点击事件 需要 `setPendingIntentTemplate`及`setOnClickFillIntent`两个方法结合使用.

因为Action操作View是通过反射的.所以不能使用自定义控件,只能用有限的系统控件

#### Layout
* FramgLayout
* LinearLayout
* RelativeLayout
* GridLayout

#### View
* Button
* ImageView
* TextView
* ProgressBar
* ImageButton
* ListView
* GridView
* Chronometer
* AnalogClock
* ViewFlipper
* StackView
* AdapterViewFlipper
* ViewStub

### 发送RemoteViews到其它进程
可以使用任何IPC方式把本地设置好的RemoteViews发送到其它进程
是通过NotificationManager发送到系统进程,小部件是通过AppWidgetManager发送到系统进程.都是通过Binder机制.
因为RemoteViews是个Parcelable,所以可以通过很多系统提供的方式来传递,如Intent,Message,或其它IPC方式

### 远程的UI操作
远程收到RemoteViews后通过调用其 `apply()`或`reApply()` 方法创建或更新UI

#### Apply
```java
public View apply(Context context,ViewGroup parent,OnClickHandler handler);
```
其中parent是父布局,这个方法会创建一个View并根据Action设置属性,返回这个设置好的View
* 创建布局
    从RemoteViews里获取布局文件的id,根据id填充出对应的View.因为是根据ID来填充布局的,所以必须保证远程进程能根据id获取到布局文件.
    系统是根据RemoteViews里的包名来获取包内的布局文件的,同一个应用内不同的线程也可以根据id来获取布局文件.但如果是两个不同的应用是不能根据id来获取布局文件的
* 设置属性
    RemoteViews会调用`perfomApply(View view,ViewGroup parent,OnClickHandler handler)`来设置填充好的View
    其实就是遍历Action.调用其 `apply()` 方法操作View.反射设置属性,或通过OnClickHandler设置点击交互
* 把填充并设置好的View添加到父布局中

#### ReApply
```java
public void reapply(Context context, View v, OnClickHandler handler);
```       
**reApply()** 方法只是更新UI,也是调用 **perfomApply()** 方法.
如果是两个不同的应用间使用RemoteViews,是不能使用 **apply()** 方法来创建UI的.
这时需要两边约定好布局文件,远程端在需要创建UI时在使用本地的布局文件填充View.然后调用 **reApply()** 方法来更新UI,最后再添加到父布局        

## 总结
通知是由RemoteViews来实现的,通过Notification类来封装一个RemoteViews.
设置不同的style实际上是设置RemoteViews里的layoutId.
通过NotificationManager的 **notify()** 方法发送到系统的NotificationService中.
系统的NotificationService里维护着各种Notification,实际上一个id对应着一个RemoteViews.
系统在自己的进程里创建和更新通知.当通知被点击时,在系统进程中会根据PendingIntent来启动指定的Activity,Service,及BroadcasterReceiver.
