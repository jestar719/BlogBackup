---
title: 详解Lambda
date: 2017-02-25 16:20:58
tags: Java
---


Java8已经更新了好久了。变化很大，但感觉有用的不多。其中最广为人知的就是`Lambda`表达式。看起来比较蛋疼，感觉`Java`越来越`C`化了。

当初以为`Lambda`的作用就是为了简化匿名内部类的输写，最近看了些文章才发现是我自己肤浅了。`Java`更新一个大版本是又怎么会只是这样小小的功能
<!-- more -->

#### Lambda表示式

`Lambda`的基本格式为`()->内容`,以箭头为分隔，左边为参数区，右边为代码区．

可以把它看成是一个临时定义的方法，没有方法名，默认为`public`,根据代码区里的有没有`return`来决定返回值．如果没有`return`则是`void`.
1. 单行代码可以省略return
2. 多行代码使用`{}`来包裹.
3. 代码内可以引用外部局部变量，实际上是隐式的把变量变成`final`

网上用得最多的例子就是`(x,y)->x+y;`,很蛋疼的例子，当初看`Junit`单元测试时也是这样的例子．

来个实际点的例子吧．

```java
public boolean isEmpty(String text){
    return text==null||"".eqauls(text)
}
//用Lambda是
(text)->return text==null||"".eqauls(text);
```


看到这里估计很多人会觉得更蛋疼了．明明`Lambda`使用是的匿名内部类的简写，为什么要拿方法来做比较，另外明明有方法可以调用，何必要用`Lambda`

#### Lambda的起源

曾经无数次的羡慕`js`有闭包，可以把方法当做参数来传递．在项目中有很多类都存在着很类似的方法

```java
public String getIds(List<T> list){
    if(list==null||list.isEmpty){
        return null;
    }
    StringBuilder builder = new StringBuilder();
    for(T t:list){
        String id=doSomething(t)
        builder.append(",").append(id);
    }
    return builder.deleteCharAt(0).toString();
}
```

想抽取一下,奈何每个类里`T`不同，`doSomething`的实现也不同．`T`可以用泛型，`doSomething（T t）`这个方法怎么传呢？？

面向接口吧．我们可以定义一个接口来干这个事．

```java
public interface GetId<T>{
    String getId(T t);
}
```

上面的方法就可以抽到工具类中了

```java
public static <T> String getIds(List<T> list,GetId<T> interf){
    if(list==null||list.isEmpty){
            return null;
        }
        StringBuilder builder = new StringBuilder();
        for(T t:list){
            String id=interf.getId(t);
            builder.append(",").append(id);
        }
        return builder.deleteCharAt(0).toString();
}
```

说来说去好像不关`Lambda`什么的事呀．实际上上面的例子就是Lambda的起源．

随着函数式编程广为程序员喜爱，Java也在考虑加入函数式编程．但有两个问题，一　做为一个强类型语言，类型转换有点蛋疼，二　不能把方法当成参数\(有返回值的还好，`void`方法当参数是相当的无语的\)．

`Java`考虑了半天，使用了上面的例子的逻辑来处理，用接口来包装方法，把接口当参数来代替．

上面的例子中．在类中调用如下

```java
String ids=Utils.getIds(List<XXX> list,new GetId<XXX>(){
    getId(XXX x){
        return ooxx(x);
    }
});
```

这是一个参数，如果是多个参数那不要吓死人．

```java
String ids=Utils.getIds(List<XXX> list,
    new GetId<XXX>(){
        getId(XXX x){
            return ooxx(x);
            }
        },
    new GetId<XXX>(){getId(XXX x){
        return xxoo(x);
        }
        },
    new GetId<XXX>(){getId(XXX x){
        return oxox(x);
        }
    }
);
```

所以`Lambda`第一个作用就体现出来了简化书写，好写好看。如

```java
String ids=Utils.getIds(List<XXX> list,(x)->ooxx(x));
```

这里有点奇怪`x`是什么的鬼？？？，其实完整的应该是

```java
String ids=Utils.getIds(List<XXX> list,(XXX x)->ooxx(x));
```

这里就体现了`lambda`的牛逼之处**类型推导**\(准确来说是Java8的特性\),编译时推导参数类型和返回类型.

最上面那个坑爹的例子估计很多人都会像我当初一样蒙Ｂ。`(x,y)->x+y;`,这ｘ和ｙ是什么的东东？？

实际上这个例子应该是`(int x,int y)->return x+y;`,这样看就清楚多了．传两个int,返回其和．

再给个实际点的例子吧

```java
String[] strings={"xx","oo","xxoo"};
//传统的排序是这样的
Arrays.sort(strings,new Comparetor<String>(){
    public int compareTo(String s1,String s2){
            return s1.length-s2.length;
        }
})
//使用lambda是这样的
Arrays.sort(strings,(String s1,String s2)->return s1.length-s2.length);
//或者
Arrays.sort(strings,(s1,s2)->s1.length-s2.length);
//或者
Arrays.sort(strings,(s1,s2)->{
    int x= s1.length;
    int y=s2.length;
    return x-y;
});
```

#### Lambda的实现

`lambda`的原理就是以把接口当参数传递的方式来形成闭包．所以这个接口只能定义一个方法，这种接口叫`函数接口`.这种接口可以隐式转为`lambda`

为了防止该接口的单方法性被破坏，`Java8`定义了一个注解`FunctionInterface`，Java库中所有这种接口都已经添加了这个注解。如

```java
@FunctionInterface
public Interface Runable{
    void run();
}
```

当然，自己也可以定义函数接口，很简单，只要是单方法都可以。最好是加上注解，防止自己或别人去添加新的方法.

另外Java8中提供了很多常用的接口，免得自己去创建。这些接口放在`java.utils.function`包里．

默认的接口可接收和返回的基本数据只有`int`,`long`,`double`三种，`String`视为对象。

1. Custmer一家，提供了`void`类的接口，可以接收单个或两个的基本数据类型或对象的参数\(无参的有现成的`Runable`\)。如

```java
interface Custmer<T>{
    void apply(T t)
}

interface BiConsumer<T,U>{
    void apply(T t,U u);
}
```

另外还支持对象+基本数据类型的参数

2. Predicate一家，提供了可接收1-2个基本数据类型或对象的参数，返回`boolean`的接口\(无参的是`booleanSupplier`接口\)
3. Function,Oprator,Supplier一家，提供了接收0-2个参数，返回基本数据类型或对象的接口
    * Function一家可接收1-2个参数，返回的是对象
    * Supplier一家不接收参数，返回基本数据类型和对象
    * Operator一家接收1-2个参数，返回同类型的数据。`unary`前缀的是接收一个参数，`binary`前缀的是接收两个参数。那个坑爹的`(x,y)->x+y`的例子就是由`IntBinaryOperator`接口来实现的．

```java
interface IntBinaryOperator{
    int applyAsInt(int left,int right);
}
```

默认提供的接口，如果是两个基本数据类型的参数，则参数的类型都必须是相同的.也就是说两个以内的参数基本上不用自己定义函数接口了．上面的例子就可以这样写
```java
public static <T> String getIds(List<T> list,Function<T,String> interf){
    if(list==null||list.isEmpty){
        return null;
    }
    StringBuilder builder = new StringBuilder();
    for(T t:list){
            String id=interf.apply(t);
            builder.append(",").append(id);
        }
    return builder.deleteCharAt(0).toString();
}
//使用时如下　
Utils.getIds(list,t->ooxx(t);)
```
#### 函数式编程
函数式编程需要可以把函数当成参数，在`Java`中所有的东西都是需要有类型的，只有函数自身是没有的。`Lambda`以间接的方式提供了函数的类型，及类型的推导，为函数式编程清扫了最大的障碍。再配合新提供的`Stream`，在`Java`中终与可以愉快的的使用函数式编程了。
然而....没有Lambda,没有Java8,我用`Rxjava`一样可以愉快的函数式编程．

[原文地址](http://www.jianshu.com/p/c40eacfa85ef)
