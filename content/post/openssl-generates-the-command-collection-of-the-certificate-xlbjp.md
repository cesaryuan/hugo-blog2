---
title: OpenSSL生成证书的命令合集
slug: openssl-generates-the-command-collection-of-the-certificate-xlbjp
url: /post/openssl-generates-the-command-collection-of-the-certificate-xlbjp.html
date: '2024-06-21 17:44:39+08:00'
lastmod: '2024-06-21 22:17:57+08:00'
toc: true
isCJKLanguage: true
---

# OpenSSL生成证书的命令合集

> 参考：
>
> * [/docs/man3.0/man1/openssl-req.html](https://www.openssl.org/docs/man3.0/man1/openssl-req.html)
> * [/docs/man3.0/man1/openssl-genrsa.html](https://www.openssl.org/docs/man3.0/man1/openssl-genrsa.html)
> * [/docs/man3.0/man1/openssl-x509.html](https://www.openssl.org/docs/man3.0/man1/openssl-x509.html)
> * [/docs/man3.0/man1/openssl-s_client.html](https://www.openssl.org/docs/man3.0/man1/openssl-s_client.html)
> * [证书签发与 SubjectAltName 扩展项 - 张明丰的博客 | Mingfer&apos;s Blog](https://www.mingfer.cn/2020/06/13/altern-name/)
> * [How to Use OpenSSL to Generate Certificates](https://www.progress.com/blogs/how-to-use-openssl-to-generate-certificates)

---

查看`csr,crt,key`​文件的信息

```bash
openssl req -noout -text -in server.csr
openssl x509 -noout -text -in server.crt
openssl rsa -noout -text -in server.key
```

---

三种生成根证书的等效方法（如果需要带密码，请删除 `-nodes`​ 选项，或在 genrsa 中添加 `-aes256`​ 选项，然后手动输入密码）

```bash
# 1. 一步到位
openssl req -new -x509 -days 9999 -nodes -newkey rsa:2048 -keyout ca.key -out ca.crt -addext keyUsage=critical,keyCertSign -subj "/C=US/CN=mitmproxy"
# 2. 先生成私钥，再自签名证书
# openssl genrsa -traditional -out ca.key 2048
# openssl req -new -x509 -days 9999 -key ca.key -out ca.crt -addext keyUsage=critical,keyCertSign -subj "/C=US/CN=mitmproxy"
# 3. 先生成私钥，再生成签名请求，再自签名证书
# openssl genrsa -traditional -out ca.key 2048
# openssl req -new -key ca.key -out ca.csr -addext keyUsage=critical,keyCertSign -subj "/C=US/CN=mitmproxy"
# openssl x509 -req -days 9999 -in ca.csr -signkey ca.key -out ca.crt -copy_extensions copyall
cat ca.key ca.crt > ca.pem
```

---

自动获取域名证书的信息，并用自己的根证书签发

```bash
DOMAIN="music.163.com"
echo | openssl s_client -connect "${DOMAIN}:443" -servername "${DOMAIN}" 2>/dev/null | openssl x509 -text > original_cert.crt
openssl genrsa -traditional -out "${DOMAIN}.key" 2048
openssl x509 -x509toreq -in original_cert.crt -out original_cert.csr -key "${DOMAIN}.key" -copy_extensions copyall
openssl x509 -req -in original_cert.csr -CA ca.crt -CAkey ca.key -CAcreateserial -out "${DOMAIN}.crt" -days 365 -sha256 -copy_extensions copyall
cat "${DOMAIN}.crt" "${DOMAIN}.key" > "${DOMAIN}.pem"
rm original_cert.crt original_cert.csr
rm "${DOMAIN}.crt" "${DOMAIN}.key"
```

---

两种使用根证书为域名签发泛域名证书的方法

```bash
DOMAIN="music.163.com"
# 1. 先生成私钥和签名请求，再使用根证书签发证书
openssl req -new -nodes -newkey rsa:2048 -keyout "${DOMAIN}.key" -out "${DOMAIN}.csr" -subj "/C=CN/CN=${DOMAIN}" -addext "subjectAltName=DNS:${DOMAIN},DNS:*.${DOMAIN}" -addext "extendedKeyUsage=serverAuth"
openssl x509 -req -in "${DOMAIN}.csr" -CA ca.crt -CAkey ca.key -CAcreateserial -out "${DOMAIN}.crt" -days 365 -sha256 -copy_extensions copyall
cat "${DOMAIN}.crt" "${DOMAIN}.key" > "${DOMAIN}.pem"
# 2. 先生成私钥，再生成签名请求，再使用根证书签发证书
# openssl genrsa -traditional -out server.key 2048
# openssl req -new -key server.key -out server.csr -subj "/C=CN/CN=*.music.163.com" -addext "subjectAltName=DNS:music.163.com,DNS:*.music.163.com" -addext "extendedKeyUsage=serverAuth"
# openssl x509 -req -in server.csr -CA ca.crt -CAkey ca.key -CAcreateserial -out server.crt -days 365 -sha256 -copy_extensions copyall
# cat server.crt server.key > server.pem
```

---

使用根证书为IP签发证书的方法

```bash
DOMAIN="1.1.1.1"
# 1. 先生成私钥和签名请求，再使用根证书签发证书
openssl req -new -nodes -newkey rsa:2048 -keyout "${DOMAIN}.key" -out "${DOMAIN}.csr" -subj "/C=CN/CN=${DOMAIN}" -addext "subjectAltName=IP:${DOMAIN}" -addext "extendedKeyUsage=serverAuth"
openssl x509 -req -in "${DOMAIN}.csr" -CA ca.crt -CAkey ca.key -CAcreateserial -out "${DOMAIN}.crt" -days 365 -sha256 -copy_extensions copyall
cat "${DOMAIN}.crt" "${DOMAIN}.key" > "${DOMAIN}.pem"
```

---

自签发域名证书

```bash
DOMAIN="music.163.com"
openssl req -new -x509 -days 365 -nodes -newkey rsa:2048 -keyout "${DOMAIN}.key" -out "${DOMAIN}.csr" -subj "/C=CN/CN=${DOMAIN}" -addext "subjectAltName=DNS:${DOMAIN},DNS:*.${DOMAIN}" -addext "extendedKeyUsage=serverAuth"
```

---

自签发IP证书

```bash
DOMAIN="1.1.1.1"
openssl req -new -x509 -days 365 -nodes -newkey rsa:2048 -keyout "${DOMAIN}.key" -out "${DOMAIN}.csr" -subj "/C=CN/CN=${DOMAIN}" -addext "subjectAltName=IP:${DOMAIN}" -addext "extendedKeyUsage=serverAuth"
```

---

一些常见选项说明：

* ​`-nodes`​，`-newkey`​会创建未加密的 private key，从而跳过密码输入步骤
* ​`req`​，[req 命令主要创建和处理 PKCS#10 格式的证书请求，它还可以创建自签名证书用作根 CA。](https://www.openssl.org/docs/man1.1.1/man1/req.html)
* ​`-x509`​，告诉`req`​命令输出自签名证书，而不是证书请求
* ​`-newkey`​，告诉`req`​命令创建一个RSA私钥，从而跳过运行genrsa这一步
* ​`-addext`​，向`req`​生成的证书请求或自签证书中添加扩展属性，不能用于x509命令中
* ​`-subj`​，指定证书的拥有者信息
* ​`-addext "subjectAltName=DNS:test.com,DNS:*.test.com"`​，如果证书中包含 subjectAltName，在识别证书持有者时会忽略 Subject 子项，而是通过 subjectAltName 来识别证书持有者（[证书签发与 SubjectAltName 扩展项 - 张明丰的博客 | Mingfer&apos;s Blog](https://www.mingfer.cn/2020/06/13/altern-name/)）
* ​`-copy_extensions`​，`openssl x509 -req`​默认忽略csr中的扩展信息，所以需要`-copy_extensions copyall`​（[Why does the x509 command not copy extension in certificate request? · Issue #10458 · openssl/openssl](https://github.com/openssl/openssl/issues/10458)）

‍
