# Android加密技术运用

> 数据加密是过程通过对文件或数据进行算法计算后得到结果一种处理过程，让数据变成不能够阅读的形式，也可以根据不同的算法来解析生成的数据，让加密数据还原成原来的数据。通过加密来实现保护数据，让数据不被别人窃取并阅读的目的。加密的分类分为两种非对称加密和对称加密，下面介绍三种常见的加密使用。

按我自己的理解：

- 数据在网络传输的过程中可能会被修改，被窃取，在前后台数据交换的时候就需要数据加密。

### 1. 安全哈希算法(SHA、MD5等等)

安全哈希算法(Secure Hash Algorithm，缩写为SHA)是一个密码散列函数家族。能计算出一个数字消息所对应到的，长度固定的字符串的算法。安全哈希算法包含很多种算法其中包括(SHA系列  MD5 等等很多)，安全哈希算法是一种单向加密的过程，也就是说在加密之后没有办法解密，虽然无法解密，但是将源文件和加密后文件打包发送，在后台接收后，对源文件加密得到加密后的文件，两者一对比就能查看文件内容是否被修改。

##### 流程

发送端数据加密	==》	加密数据和原本数据一同发送 	==》	接收端接收数据	==》	接收端对原数据加密得到加密数据	==》	接收端加密数据和发送端发送的加密数据比较	==》	可判断数据正确、修改与否

### 2. AES算法(高级加密标准)

AES算法(高级加密标准)是一种对称加密算法一个密码对应一个密钥的形式，能够加密同时也能解密。AES算法包括AES、AESWRAP、DES、ARC4等等，与SHA算法比起来AES算法加密同时可以解密，Https连接支付宝时也是靠AES进行保护，下面是AES算法加密使用的代码：

```java
/**
     * 加密
     * @param content  加密内容
     * @param password 加密的密码 用来生成密钥
     * @return
     * @throws Exception  会抛出很多异常这里（NoSuchAlgorithmException 没找到指定算法异常, InvalidKeyException 无效的key异常  等等）
     */
    public byte[] encrypt(String content , String password) throws Exception {
        
        //得到key生成器
        KeyGenerator kgen = KeyGenerator.getInstance(ALGORITHM);
        //初始化生成器
        kgen.init(128,new SecureRandom(password.getBytes()));
        //获取秘钥
        SecretKey secretKey = kgen.generateKey();
        //获取ecode 编码格式
        byte[] enCodeFormat = secretKey.getEncoded();
        //转换成AES专用密钥
        SecretKeySpec key = new SecretKeySpec(enCodeFormat, ALGORITHM);
        //将加密内容转为数组
        byte[] c = content.getBytes();
        //获取密码器
        Cipher cipher = Cipher.getInstance(ALGORITHM);
        //初始化密码器  Cipher.ENCRYPT_MODE（加密模式）
        cipher.init(Cipher.ENCRYPT_MODE,key);
        //执行doFinal获取加密后结果
        byte[] result = cipher.doFinal(c);

        return  result;
    }
```

```java
/**
     * 解密
     * @param content  解密内容
     * @param password 解密的密码 用来生成密钥进行解密
     * @return
     * @throws Exception  会抛出很多异常这里（NoSuchAlgorithmException 没找到指定算法异常, InvalidKeyException 无效的key异常  等等）
     */
    public byte[] decrypt(byte[] content , String password) throws Exception {

        //等到key生成器
        KeyGenerator keyGenerator = KeyGenerator.getInstance(ALGORITHM);
        //初始化key生成器
        keyGenerator.init(128,new SecureRandom(password.getBytes()));
        //根据key生成器获取秘钥
        SecretKey secretKey = keyGenerator.generateKey();
        //根据秘钥获取ecode(编码格式)
        byte[] encoded = secretKey.getEncoded();
        //生成AES指定的秘钥
        SecretKeySpec key = new SecretKeySpec(encoded,ALGORITHM);
        //生成密码器
        Cipher cipher = Cipher.getInstance(ALGORITHM);
        //初始化加密器  Cipher.DECRYPT_MODE(解密模式)
        cipher.init(Cipher.DECRYPT_MODE,key);
        //解密
        byte[] result = cipher.doFinal(content);

        return  result;
    }
```

- 步骤如下

1. 获取`KeyGenerator`
2. 获取秘钥
3. 获取密码器
4. 初始化密码器给出相应的操作模式
5. 执行doFinal执行加密

##### 流程：

发送端将数据通过加密算法得到加密数据	==》	加密后数据发送给接收端	==》	接收端接收到加密数据	==》	通过对应的解密方法解密得到真实数据

- 注意：AES是对称加密，也就是说加密端和解密端用的加密密钥和解密密钥是同一个。

### 3. RSA算法

RSA算法是一种非对称加密算法，RSA加密可靠性非常高，目前只有短的RSA要是才能被强力破解，随着加密位数的增加破解难度也会增加就更难破解。RSA加密需要一对钥匙(公钥、私钥)，使用一把钥匙加密再使用另一把解密。

一般来说将公钥公布给接收端的人用于解密，将私钥保留，这样导致加安全性相比于对称加密更高。

获取公钥私钥的代码如下：

```java
public static final String ALGORITHM_RSA = "RSA" ;

    public static final String PUBLIC_KEY = "public_key.dat";

    public static final String PRIVATE_KEY = "private_key.dat";

    public static final int KEY_SIZE = 1024 ;

    /***
     * 生成密钥对
     * @throws NoSuchAlgorithmException
     * @throws IOException
     */
    public void generatorKeyPair() throws NoSuchAlgorithmException, IOException {
        //获取秘钥对生成器
        KeyPairGenerator kp = KeyPairGenerator.getInstance(ALGORITHM_RSA);
        //初始化生成器 KEY_SIZE是1024 也可以是其他长度，长度越长越难破解也会更加密耗时
        kp.initialize(KEY_SIZE,new SecureRandom());
        //生成密钥对
        KeyPair keyPair = kp.generateKeyPair();
        //获取公钥私钥
        PublicKey aPublic = keyPair.getPublic();
        PrivateKey aPrivate = keyPair.getPrivate();
        //把公钥私钥保存到文件中 
        ObjectOutputStream publicOs = new ObjectOutputStream(new FileOutputStream(PUBLIC_KEY));
        ObjectOutputStream privateOS = new ObjectOutputStream(new FileOutputStream(PRIVATE_KEY));

        //对象写入文件
        publicOs.writeObject(aPublic);
        privateOS.writeObject(aPrivate);
        //关流
        privateOS.close();
        publicOs.close();

    }
```

获取密钥对之后可以进行加密解密操作，代码如下

```dart
/**
     * 加密
     * @param content 加密内容
     * @return
     * @throws Exception IO异常加密异常等
     */
    public String encrypt(String content) throws Exception {

        //获取公钥
        ObjectInputStream is = new ObjectInputStream(new FileInputStream(PUBLIC_KEY));
        //读对象流
        Key key = (Key) is.readObject();
        //关流
        is.close();
        //获取密码器
        Cipher cipher = Cipher.getInstance(ALGORITHM_RSA);
        //初始化密码器 设置加密模式
        cipher.init(Cipher.ENCRYPT_MODE,key);
        //加密内容转数组
        byte[] bytes = content.getBytes();
        //加密
        byte[] bytes1 = cipher.doFinal(bytes);
        //方便观看使用BASE64Encoder编码一下
        BASE64Encoder base64Encoder = new BASE64Encoder();
        String encode = base64Encoder.encode(bytes1);

        return encode;

    }

    /**
     * 解密
     * @param content 解密内容
     * @return
     * @throws Exception IO异常加密异常等
     */
    public String decrypt(String content)throws Exception {

        //获取公钥
        ObjectInputStream is = new ObjectInputStream(new FileInputStream(PRIVATE_KEY));
        //读对象流
        Key key = (Key) is.readObject();
        is.close();
        //获取密码器
        Cipher cipher = Cipher.getInstance(ALGORITHM_RSA);
        //初始化密码器 设置加密模式
        cipher.init(Cipher.DECRYPT_MODE,key);
        //加密时返回的字符串编码过所以这里解码下那都原始的加密数据
        BASE64Decoder base64Decoder = new BASE64Decoder() ;        
        byte[] bytes = base64Decoder.decodeBuffer(content);
        //解密
        byte[] bytes1 = cipher.doFinal(bytes);

        return new String(bytes1);

    }
```

##### 弊端：

一般情况下，对称加密算法相比非对称加密算法速度更快，非对称加密算法虽然可以验证来源消息是否真实，且安全性上有着很高的水准，但是加解密速度远远慢于对称加密，在某些极端情况下，甚至能比对称算法慢上1000倍。

##### 流程：

发送端将数据通过私钥进行加密	==》	私钥自我保留，公钥交给接收端	==》	接收端接收到公钥，用发送端公钥加密接收端的公钥后发还给发送端	==》	发送端解析收到的返还，拿到了接收端的公钥。

详细：

1. A要向B发送信息，A和B都要产生一对用于加密和解密的公钥和私钥。

2. A的私钥保密，A的公钥告诉B；B的私钥保密，B的公钥告诉A。

3. A要给B发送信息时，A用B的公钥加密信息，因为A知道B的公钥。

4. A将这个消息发给B（已经用B的公钥加密消息）。

5. B收到这个消息后，B用自己的私钥解密A的消息。其他所有收到这个报文的人都无法解密，因为只有B才有B的私钥。



### 举例

- https请求

在网络请求时像https请求时，会将数据进行AES算法加密再用RSA算法加密，最后在使用SHA加密(举例说明实际不一定非要这样加密过程，算法一类都可以自己定制，三种算法组合使用)。这样使https请求变成两层加密还有一层校验形式传输。

- 国家加密算法SIM2（非对称加密）

SIM2采用的就是非对称加密（手机号中的SIM传输算法）

- 支付宝开放平台的支付业务（非对称加密）

支付宝会让你生成`公私钥(openssl可以直接生成)`，`私钥`放在自己的`服务端`(切记)，`公钥`上传到支付宝的`商户平台`，
 拿到`订单信息`的时候，请求服务端`通过私钥签名后的订单信息`，
 然后调用支付宝的sdk，支付宝会拿`公钥`来`验签`，验证成功之后才会进入支付选项。

