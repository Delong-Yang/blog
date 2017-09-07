---
title: About-Kerberos-Principal-And-Key
date: 2017-08-24 17:09:57
tags: Kerberos
---

The original English blog is from [https://ssimo.org/blog/id_016.html](https://ssimo.org/blog/id_016.html)

## Kerberos Principal是什么？
简单来说principal作用等同用user name。Kerberos Princial 分为User Principal 和Service Princial。单纯只用username并不能很好的区分这两者的概念，因此Principal也就应运而生了。
## Principal 长啥样？
一个Principal由几个字符串组成，@后面的是Realm， 习惯上全部使用大写字母。@前面是Realm中的一个限定符(Identity), 可能由几个部分组成，用"/"分隔。

> Example: <br>
> compoennt1/component2@REALM

User Pricipal只有一个component和Realm, 比如*delong@EXAMPLE.COM*表示一个叫delong的用户属于EXAMPLE.COM这个Realm。
在AD(Active Directory)中，Realm就是domain name.

Service Principal有两个compoments，service和hostname
> Example: <br>
> HTTP/server.example.com@EXAMPLE.COM

当client向KDC(Key distribution centre)请求访问service的ticket时，就要用到principal。

## Keys and Keytabs
每个princial在KDC中都关联有一个key,用来加密发送给client的ticket，Service需要用相同的key解密ticket。所以Kerberos也称为shared key system。

User principal的key是user的password。 KDC中也存有user的password。

Service principal 就不用密码了，一般是随机生成的key，一份存放在KDC，一份存放在keytab文件里。

Create service principal name On the Windows Active Domain Controller:

> $ setspn -A HTTP/server.example.com serviceADUser

To create the keytab file use the following process:

> $ ktpass -out c:\dir\krb5.keytab -princ HTTP/server.example.com@EXAMPLE.COM -mapUser serviceADUser -mapOp set -pass serviceADUserPWD -crypto RC4-HMAC-NT -pType KRB5_NT_PRINCIPAL