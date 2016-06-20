---
title: keytool和openssl生成的证书转换
date: 2016-06-14 23:20:46
tags:
- 数字证书
categories:
- 技术
---
keytool和openssl生成的证书转换。
<!-- more -->
# keytool和openssl生成的证书转换
## keytool生成证书示例
生成私钥+证书：
`keytool -genkey -alias client -keysize 2048 -validity 3650 -keyalg RSA -dname "CN=localhost" -keypass $client_passwd -storepass $client_passwd -keystore ClientCert.jks`
生成文件文件ClientCert.jks。

------
导出证书：
```
~/tmp/cert# keytool -export -alias client -keystore ClientCert.jks -storepass $client_passwd -file ClientCert.crt
Certificate stored in file <ClientCert.crt>
~/tmp/cert# ll
total 8
-rw-r--r-- 1 root root  715 Jun 14 20:24 ClientCert.crt
-rw-r--r-- 1 root root 2066 Jun 14 20:21 ClientCert.jks
```
keytool工具不支持导出私钥。

## openssl生成证书示例
生成公钥私钥：
```
~/tmp/cert# openssl genrsa -des3 -out private.pem 1024
Generating RSA private key, 1024 bit long modulus
.............................++++++
...........++++++
e is 65537 (0x10001)
Enter pass phrase for private.pem:
Verifying - Enter pass phrase for private.pem:
~/tmp/cert# ll
total 4
-rw-r--r-- 1 root root 963 Jun 14 20:27 private.pem
```
创建证书请求：
```
~/tmp/cert# openssl req -subj "/C=CN/ST=BJ/L=BJ/O=HW/OU=HW/CN=CL/emailAddress=huawei@huawei.com" -new -out cert.csr -key private.pem
Enter pass phrase for private.pem:
~/tmp/cert# ll
total 8
-rw-r--r-- 1 root root 664 Jun 14 20:43 cert.csr
-rw-r--r-- 1 root root 963 Jun 14 20:27 private.pem
```
自签发证书：
```
~/tmp/cert# openssl x509 -req -in cert.csr -out public.crt -outform pem  -signkey private.pem -days 3650
Signature ok
subject=/C=CN/ST=BJ/L=BJ/O=HW/OU=HW/CN=CL/emailAddress=huawei@huawei.com
Getting Private key
Enter pass phrase for private.pem:
~/tmp/cert# ll
total 12
-rw-r--r-- 1 root root 664 Jun 14 20:43 cert.csr
-rw-r--r-- 1 root root 963 Jun 14 20:27 private.pem
-rw-r--r-- 1 root root 871 Jun 14 20:44 public.crt
```
## 转换
keytool和openssl生成的证书相互之间无法识别，keytool生成的为jsk文件，openssl默认生成的为PEM格式文件。需要先转换成pkcs12格式，然后再使用对方的命令转换成需要的格式。
### keytool生成的证书转换为PEM格式
```
~/tmp/cert# keytool -importkeystore -srcstoretype JKS -srckeystore ServerCert.jks -srcstorepass 123456 -srcalias server -srckeypass 123456 -deststoretype PKCS12 -destkeystore client.p12 -deststorepass 123456 -destalias client -destkeypass 123456 -noprompt
root@SZV1000101361:~/tmp/cert# ll
total 24
-rw-r--r-- 1 root root  664 Jun 14 20:43 cert.csr
-rw-r--r-- 1 root root 1708 Jun 14 21:01 client.p12
-rw-r--r-- 1 root root  963 Jun 14 20:27 private.pem
-rw-r--r-- 1 root root  871 Jun 14 20:44 public.crt
-rw-r--r-- 1 root root 1372 Jun 14 20:57 ServerCert.jks
-rw-r--r-- 1 root root 1682 Jun 14 20:55 server.p12
```
导出证书：
```
~/tmp/cert# openssl pkcs12 -in client.p12 -passin pass:$passwd -nokeys -out client.pem
MAC verified OK
```
导出私钥：
```
~/tmp/cert# openssl pkcs12 -in client.p12 -passin pass:$passwd -nocerts -out client.crt
MAC verified OK
Enter PEM pass phrase:
Verifying - Enter PEM pass phrase:
root@SZV1000101361:~/tmp/cert# ll
total 32
-rw-r--r-- 1 root root  664 Jun 14 20:43 cert.csr
-rw-r--r-- 1 root root 1184 Jun 14 21:10 client.crt
-rw-r--r-- 1 root root 1708 Jun 14 21:01 client.p12
-rw-r--r-- 1 root root 1127 Jun 14 21:07 client.pem
-rw-r--r-- 1 root root  963 Jun 14 20:27 private.pem
-rw-r--r-- 1 root root  871 Jun 14 20:44 public.crt
-rw-r--r-- 1 root root 1372 Jun 14 20:57 ServerCert.jks
-rw-r--r-- 1 root root 1682 Jun 14 20:55 server.p12
```

### PEM格式证书转换为jks文件
转换为pkcs12格式：
```
~/tmp/cert# openssl pkcs12 -export -in public.crt -inkey private.pem -out server.p12 -name server -passin pass:${passwd} -passout pass:${passwd}
~/tmp/cert# ll
total 16
-rw-r--r-- 1 root root  664 Jun 14 20:43 cert.csr
-rw-r--r-- 1 root root  963 Jun 14 20:27 private.pem
-rw-r--r-- 1 root root  871 Jun 14 20:44 public.crt
-rw-r--r-- 1 root root 1682 Jun 14 20:55 server.p12
```

导入到jks中：
```
~/tmp/cert# keytool -importkeystore -srckeystore server.p12 -srcstoretype PKCS12 -srcstorepass ${passwd} -alias server -deststorepass ${passwd} -destkeypass ${passwd} -destkeystore ServerCert.jks
~/tmp/cert# ll
total 20
-rw-r--r-- 1 root root  664 Jun 14 20:43 cert.csr
-rw-r--r-- 1 root root  963 Jun 14 20:27 private.pem
-rw-r--r-- 1 root root  871 Jun 14 20:44 public.crt
-rw-r--r-- 1 root root 1372 Jun 14 20:57 ServerCert.jks
-rw-r--r-- 1 root root 1682 Jun 14 20:55 server.p12
```




