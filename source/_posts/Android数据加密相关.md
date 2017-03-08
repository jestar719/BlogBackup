---
title: Android数据加密相关
date: 2017-02-28 18:18:39
tags: [Android,Java]
---
最近在看HTTPS,其核心还是基与Rsa的SSL/TSL加密,所以整理一下在Android开发中常见的几种加密方式
* MD5
* BASE64
* AES
* DES
* RSA
* SSL/TSL
<!-- more -->

## MD5加密
MD5英文全称`Message-Digest Algorithm 5`,全称消息算法摘要,是一种不可逆加密方式

### 特点
* 压缩性：任意长度的数据，算出的MD5值长度都是固定的。
* 容易计算：从原数据计算出MD5值很容易。
* 抗修改性：对原数据进行任何改动，哪怕只修改1个字节，所得到的MD5值都有很大区别。
* 强抗碰撞：已知原数据和其MD5值，想找到一个具有相同MD5值的数据（即伪造数据）是非常困难的。

因为其特点,MD5加密通常用来验证文件的完整性,防串改.

### 实现
在Java中提供了MessageDigest类来加密数据.
```java
byte[] resource; //需要加密的数据
MessageDigest md5=MessageDigest.getInstance("MD5");
byte[] bytes=md5.digest(resource);
StringBuilder builder=new StringBuilder(bytes.lenght);
for (byte byte :bytes ) {
 String temp = Integer.toHexString(b & 0xff);
 if (temp.lenght==1) {
     builder.append(0);
 }
    builder.append(temp);
}
String MD5=builder.toString();
```
MD5加密是对字节数组进行操作的.所以无论加密文件或是其它都需要先转换为字节数组.加密后的结果也是字节数组.
为了方便查看,需要按规定转换为字节数组.

## BASE64
BASE64并不是一种加密方式,而是对字节数组的一种编码方式.实际上是让字节数据转换为文本的方案.

### 原理
BASE64是把字节数据转换为ASC码中对应的64个字符(英文大小写,+及/).
转换方式为每6位字节对应一个字符.不足位的后面补0.6位字节转换为十进制数值.范围是0-63,对应64个字符.如果有不足位的,在编码后会补上数量对应的=

### 用处
* 统一编码方式,不需要考虑字符集对数据的影响
* 可以把任何文件或数据都以纺一的文本方式保存

### 实现
在Java中提供了Base64类来实现编码转换
```java
//编码
String encode= Base64.encode(byte[] bytes,int flag);
//解码
byte[] decode=Base64.decode(byte[] bytes,int flag);
```

flag表示编码方式,通常使用`Base64.DEFAULT`
* `Base64.DEFAULT` 默认的编码方式
* `Base64.NO_PADDING` 忽略后缀的=号
* `Base64.NO_WRAP` 省略换行符
* `Base64.CTRL` 使用微软的换行符,而不是Liunux的
* `Base64.URL_SAFE` 使用`-和_`代替`+和/`以保证在URL的安全.通常用与网络传输

## AES
`Advanced Encryption Standard`,全称为高级加密标准,又称为 **Rijndael** 加密法,使用的是区块加密方案.
区块加密方案是把一定数量的字节划分为一个区块,把区块内的数据加密成相同位数的数据
AES是一种对称加密方案,所以需要加密端和解密端都使用相同的密钥.

### 特点
* 区块 AES是对字节数组按区块划分,对区块进行加密.AES区块大小固定为128位.
* 密钥 AES的密钥长度有三种,128位/192位/256位 位数越多安全性越高.
* 矩阵 AES每次对区块中的16个字节进行操作,这4个字节会组成一个4x4的矩阵.
* 回合密钥 AES对每个矩阵的操作时都会根据密钥生成一个16位的回合密钥.对应矩阵上的每一个字节

### 实现
在Java中通过MessageDigest来进行AES加解密
```java
//密钥原数据字符串,两端必须一样,128位的密钥字符串长度必须为16
private static final String KEY=""
//加密方式,AES表示AES加密,CBC表示区块的处理方式,PKC5Padding表示区块的填充方式.
private static final String MODE="AES/CBC/PKCS5Padding"

//生成密钥
KeyGenerator keyGenerator = KeyGenerator.getInstance(KEY);
SecretKey secretKey = keyGenerator.generateKey();
//初始化密码处理类
Cipher cipher = Cipher.getInstance(MODE);
cipher.init(Cipher.ENCRIPT_MODE,secretKey);  //第一个参数为模式,ENCRIPT_MODE为加密, DECRIPT为解密
//cipher.init(Cipher.DECRIPT_MODE,secretKey,"BC");  使用`PKCS7Padding`方式时需要加载bouncycastle包,BC为提供该填充方式的Provicer
byte[] date=cipher.doFinal(byte[] data); //加密和解密都是同一个方法,由初始化的模式决定.对源字节数组进行处理.生成新的字节数组.
```
填充方式有几种.Java6没有实现所有的填充方式.
* `NoPadding,PKCS5Padding,ISO10126Padding` Java6实现
* `PKCS7Padding,ZeroBytePadding` 需要加载`bouncycastle`包

生成密钥时,`getInstance()`有个重载的方法可以添加一个provier.为密钥添加一个随机数.
AES加密安全性高,速度快.

## DES
`Data Encryption Standard`,标准加密算法.使用64位密钥的对称加密.使用的也是区块加密方案.
因为是64位密钥,安全性不高,现已经被AES加密替代.
实现方法同AES加密,只需要把MODE中的`AES`改为`DES`

## RSA
目前使用最广泛的非对称加密.生成一对密钥--公钥和私钥.公钥对来对外提供.私钥只有密钥生成者自己拥有
非对称算法需要指定密钥长度,越长安全性越好,但加解密的速度就越慢.通常指定1024或2048.
一次加密的的密文长度为密钥长度/8-11,所以1024长度的密钥一次只能加密117字节.2048能加密245字节.
所以非对称加密通常只用与短数据加密,如签名或对称加密的密钥.
RSA加密都是一用与一对多的场景.
RSA有两种使用方式
* 加密算法
    公钥加密,只有私钥才能解密
* 签名算法
    私钥签名,只有公钥才能验证.
    如Github使用的SSH登录就是使用的RSA加密算法.用户把公钥保存到服务器,
    通过SSH登录时服务器发一个随机数给客户端,客户端使用私钥加密发送到服务器,服务器用保存的公钥解密.

### RSA的密钥的产生
* 随机获取两个大质数,得其积N
* 获取N的欧拉函数值->整数R
* 随机获取一个小与R并与之互质的整数E,计算出E的反模元素D
* 公钥是(N,E),私钥是(N,D)

### RSA的原理
* 欧拉定理
    两个互质的正整数,A和N,N的欧拉函数为P,则A的P次方除以N余1
* 费马小定理(RSA算法核心)
    因为质数P的欧拉函数为P-1,所以一个整数A和一个质数N互质时,A的N-1次方除以N余1
* 反模元素
    两个互质的正整数.A和N,则一定存在一个整数B,使得A乘B除以N余数为1        

### 实现
#### 定义常量
```java
private static final KEY_SIZE=1024;
private static final RSA="RSA";
private static final MODE="RSA/ECB/PCKS1Padding"
```
这里注意RSA的加密填充方式,需要两端保持一至.
在Adrioid中默认使用的是`RSA/None/NoPadding`,在Java中使用的是`RSA/None/PCKS1Padding`.

#### 创建密钥
```java
KeyPairGenerator rsa = KeyPairGenerator.getInstance(RSA);
rsa.initialize(KEY_SIZE);
KeyPair keyPair = rsa.generateKeyPair();
Key publicKey = keyPair.getPublic();
byte[] publicKeyEncoded = publicKey.getEncoded();
Key privateKey = keyPair.getPrivate();
byte[] privateKeyEncoded = privateKey.getEncoded();
```

#### 把字节数组转换为Key
公钥会以字节数组的形式公开.接收方需要把字节数据转化为公钥
```java
//公钥公开的方式为X509
KeySpec KeySpec = new X509EncodedKeySpec(publicKeyEncoded);
PublicKey publicKey = keyFactory.generatePublic(keySpec);
//私钥公开的方式为PKCS8
KeySpec keySpec = new PKCS8EncodedKeySpec(publicKeyEncoded);
PrivateKey privateKey = keyFactory.generatePrivate(keySpec);
```

#### 加解密
通常使用公钥加密,使用私钥解密.还是使用`Cipher`类
```java
Cipher cipher=Cipher.instance(MODE);
//使用公钥加密
cipher.init(Cipher.ENCRIPT_MODE,publicKey);
cipher.doFinal();
//使用私钥解密
cipher.init(Cipher.DECRIPT_MODE,privateKey);
cipher.doFinal();
```

## SSL
安全套接字（Secure Socket Layer，SSL）协议是Web浏览器与Web服务器之间安全交换信息的协议，提供两个基本的安全服务：鉴别与保密。
* 鉴别 可选的客户端认证及强制的服务端认证
* 保密 双方在连接时定义好加密方式,所有传递内容都会加密.

SSL是间与应用层与TCP层.应用数据经过SSL层加密并加上SSL头传输给TCP层.

### SSL通信流程
* 握手
    握手是在两端连接后数据传输前的协议行为,通过三次握手确定双方的身份,双方的加密方式,以及确定密钥.这个握手过程是通过RAS加密及身份证书完成的.
* 加密通信
    完成握手协议手,双方就按确定的加密方式以对称加密的方式对数据进行加密

### 握手协议
* 客户端发送至服务器 一个会话ID,自身SSL版本.一个32位随机数.一自身支持的密码套件列表.一个hello.
* 服务器接收后会根据客户端提供的列表选择一个密码套件,确定与客户端之间的加密方式
* 服务器发送至客户端 会话ID,SSL版本,一个32位随机数,一个密码套件.一个servieHello.及自己的证书.
* 客户端收到证书后可以对证书进行验证.然后生成一个32位随机数.这样总共就有三个随机数了.根据服务端确定的加密方式用这三个随机数生成密钥.然后从证书中获得服务端的公钥对第三个随机数加密
* 客户端发送至服务器 加密的第三个随机数.
* 服务器收到加密的随机数后使用私钥解密,然后使用三个随机数生成密钥.
* 服务器发送至客户端 准备完成,可以开始加密通信

### 证书
证书是一台服务器对外提供的一个身份证明.需要通过可靠的第三方认证机构(CA)来发布.
一个数字证书通常有以下几项
* 证书持有者的公钥
* 证书的发布机构
* CA的签名
* 签名摘要的算法

### 证书的验证
一般的浏览器都会有CA根证书,含有所有CA的公钥
* CA验证 CA根证书中找到不证书的发布机构
* CA签名摘要验证 使用CA的公钥来解密签名摘要,如果解不开说明证书不对
* CA签名验证 使用公钥解密签名.然后使用签名摘要算法进行签名摘要,比对解密手的签名摘要.如果不同说明签名被更改
* 证书过期验证 实现了在线证书状态协议(OCSP)的客户端可以在线查询证书是否过期

## TSL
TLS 1.0是IETF（Internet Engineering Task Force，Internet工程任务组）制定的一种新的协议，它建立在SSL 3.0协议规范之上，是SSL 3.0的后续版本，可以理解为SSL 3.1
