---
title: 责任链模式及OkHttp中的实现
date: 2017-02-25 16:37:13
tags: 设计模式
---

#### 责任链模式

责任链模式是对一个事件的处理方法,所有能对事件进行处理的对象按顺序形成一个链表.事件经过链表中每个处理对象轮流处理.如果有返回值.则返回也是顺着这条链表反向返回.这个链表是先进后出模式.

- 在现实中的责任链模型之一就是网络连接.对与程序猿而言,七层或五层的网络连接模型是肯定知道的.

当一个网络请求发出时,需要经过应用层->传输层->网络层->连接层->物理层

收到响应后正好反过来,物理层->连接层->网络层->传输层->应用层

在请求经过各层时,由每层轮流处理.每层都可以对请求或响应进行处理.并可以中断链接,以自身为终点返回响应
<!-- more -->

- 另一个现实模型就是Web服务器的请求缓存

一个请示从客户端发送到Web服务器需要经过客户端->中间服务器->反向代理->Web端缓存(如Redis)->Web数据库

除了Web数据库是请求的点外,中间所有层都可以缓存数据.如果有缓存时会终止请求,以自身为终点返回数据.

如果途经的层没有缓存,则会在收到下一级的返回的数据时对数据进行缓存.然后返回上一层.

在设计模式中,负责链模式就是对这种顺序处理事件的行为的抽象,通过接口来定义处理事件的方法.顺序分发/处理事件.
- 每个责任人实现相同的接口,处理一个事件对象
- 让事件对象责任人之间顺序传递
- 事件的处理结果的返回是逆序的
- 责任链中的每个责任人都可以有权不继续传递事件,以自身为终点处理事件返回结果

#### OkHttp中的责任链模式

在`Okhttp`中,`Intercepter`就是典型的责任链械的实现.它可以设置任意数量的`Intercepter`来对网络请求及其响应做任何中间处理.比如设置缓存,`Https`的证书验证,统一对请求加密/防串改,打印自定义Log,过滤请求等.

```java
interface Intercepter{
    Response chian(Chian chian);
}
```

这个接口很简单,拿到`Chain`对象.最后返回个`Response`,那这个对象是什么鬼??

它有两个重要的方法
`public Request getQuest()`及`public Response process(Request quest)`
拿到请求及设置请求拿到响应.
一般的实现是先拿到请求.然后对请求做一番蹂躏,然后`process`一下,拿到`Response`,折腾一下.然后`return`掉
```java
public Response chain(Chain chain){
    Requset quest = chian.getRequset()
    ooxx(request);
    Response response = chain.process(quest);
    xxoo(response);
    return response;
}
```
或许它不知道,`Resquse`其实是被上家ooxx过的,`Respnse`也是被下家xxoo过的.

具体的实现可以去看一下源码,我这里就写一下自己理解的超简单的实现

```java
public class Okhttp {
private List<Intercepter> mIntercepters=new LinkedList<>();
private Request mRequest;
private int mIndex;
private Callback mCallback;
private Chain mChain=new Chain();
private Intercepter mNetIntercepter =new Intercepter() {
@Override
public Response chain (Chain chain) {
    //这里是真实的发送网络请求
    return 网络请示的响应;
    }
};

/**
* 添加拦截器,后加的放最前面
* @param intercper 拦截器
*/
public void addIntercepter(Intercepter intercper){
    mIntercepters.add(0, intercper);
}

/**
* OkHttp请求的入口
* @param request 请示
* @param callback 回调
*/
public void execute(Request request ,Callback callback){
    mCallback=callback;
    mIndex=0;
    mRequest=request;
    new Thread(new Runnable() {
        @Override
        public void run () {
            Response response=mChain.process(mRequest);
            mCallback.onResponse(response);
        }
    }).start();
}

/**
* 定义的一个类用与递归的方式完成责任链发送请求获取响应
*/
private class Chain {
public Request getRequest(){
    return mRequest;
}

/**
* 顺序获取拦截器,传递chain对象给拦截器,获取响应
* @return 请求的响应
*/
private Response getResponse () {
    Intercepter intercepter = mIntercepters.get(mIndex);
    mIndex++;
    return intercepter.chain(this);
}

/**
* 如果责任链没走完,则顺序从责任链中获取拦截器,处理请求
* 否则由真正的负责网络请求的拦截器处理请求
* @param request 请求
* @return 响应
*/
Response process(Request request){
    mRequest=request;
    if (mIndex>=mIntercepters.size()){
        return Okhttp.this.mNetIntercepter.chain(this);
        }else{
            return getResponse();
        }
    }
}
}

```
不得不说程序猿写文章真是容易骗字数.我已经极力在简化了.代码还是差不多上百行.

应该写得够简单吧.重点也就几个
1. 内部有个处理真实网络请求的`Intercepter`,做为最后的接盘侠.
2. 返回响应的是`intercepter.chain(this)`,如果这个拦截器调用了`chain`的`process()`方法就形成递归,否则就是终断了请求的传递
3. 整个责任链的传递都是同步的.整体在子线程运行,最后通过回调返回响应.

还有一个很经典的实现就是Android的事件分发机制.这里我就不贴代码骗字数了.网上随便一找一堆.

#### 责任链模式的用途

当业务逻辑需要形成一个事件处理流时,就可以考虑使用责任链模式.通过接口来规范中间环节的行为,专注与事件流的传递.可以随意扩展及调整中间环节.

我在实际业务中的有一个场景中使用了责任链模式.(当时使用的时候其实并不清楚)

1. 在一个列表中可以弹出一个筛选菜单,菜单项是不定的.某些项选择后可以增加更多的选项条目
2. 选项条目类型不同.有单选,多选,输入等.
3. 点击完成时才把所有筛选条目形成对应的网络请求参数

实际处理起来很简易.定义了两个类
```Java
interface ParamsSetAble{
    void setParams(NetParams params);
}

public class NetParams{}
```
每个条目实现`ParamsSetAble`接口用与设置参数.

`NetParams`类用与收集参数,最后转换成网络请求所用的格式

当点击完成时,遍历所有的条目,传递`NetParams`对象.最后把这个对象传递给网络请求的类.

在传递时进行可以进行参数检查,错误时可以中断传递,提示错误.我这里是通过手动抛异常来中断的.在外部统一`catch`处理.如果用`Rxjava`可以不用`catch`,在`onError`里统一管理.如`Observer.error(new 自定义异常(中断提示信息))`

[原文链接](http://www.jianshu.com/p/8b9f45a79ee6)
