[TOC]

## 1.Base64

### 1.1 简介

[base64_百度百科 (baidu.com)](https://baike.baidu.com/item/Base64/8545775)

> Base64是网络上最常见的用于传输8Bit[字节码](https://baike.baidu.com/item/字节码/9953683?fromModule=lemma_inlink)的编码方式之一，Base64就是一种基于64个可打印字符来表示[二进制](https://baike.baidu.com/item/二进制/361457?fromModule=lemma_inlink)数据的方法。可查看RFC2045～RFC2049，上面有MIME的详细规范。
>
> Base64，就是包括小写字母a-z、大写字母A-Z、数字0-9、符号"+"、"/"一共64个字符的字符集，（任何符号都可以转换成这个字符集中的字符，这个转换过程就叫做base64编码。 [2]
>
> Base64编码是从二进制到字符的过程，可用于在[HTTP](https://baike.baidu.com/item/HTTP/0?fromModule=lemma_inlink)环境下传递较长的标识信息。采用Base64编码具有不可读性，需要解码后才能阅读。
>
> Base64由于以上优点被广泛应用于计算机的各个领域，然而由于输出内容中包括两个以上“符号类”字符（+, /, =)，不同的应用场景又分别研制了Base64的各种“变种”。为统一和规范化Base64的输出，Base62x被视为无符号化的改进版本。



从描述上可以清楚，Base64更多的是方便传输，并非是用来保护数据安全。

### 1.2 代码示例

```java
public class Base64Util {
    public static void main(String[] args) {
        String str = "Hello, World!";
        byte[] cipher = encryptByBASE64(str);
        byte[] plain = decryptByBase64(cipher);
        System.out.println(new String(plain));  // Hello, World!
    }

    /**
     * Base64原文
     * @param plain
     * @return
     */
    public static byte[] encryptByBASE64(String plain) {
        byte[] bytes = plain.getBytes();
        return Base64.getEncoder().encode(bytes);
    }

    /**
     * Base64解密
     * @param cipher
     * @return
     */
    public static byte[] decryptByBase64(byte[] cipher) {
        return Base64.getDecoder().decode(cipher);
    }
}
```



## 2.MD5  摘要算法

### 2.1 简介

[MD5_百度百科 (baidu.com)](https://baike.baidu.com/item/MD5/212708)

> MD5信息摘要算法（英语：MD5 Message-Digest Algorithm），一种被广泛使用的密码散列函数，可以产生出一个128位（16字节）的散列值（hash value），用于确保信息传输完整一致。MD5由美国密码学家罗纳德·李维斯特（Ronald Linn Rivest）设计，于1992年公开，用以替代MD4算法。这套算法的程序在RFC1321标准中被加以规范。1996年后该算法被证实存在弱点，可以被加以破解，对于需要高度安全性的数据，专家一般建议改用其他算法，如SHA-2。2004年，证实MD5算法无法防止碰撞（collision），因此不适用于安全性认证，如SSL公开密钥认证或是数字签名等用途。



### 2.2 代码示例

