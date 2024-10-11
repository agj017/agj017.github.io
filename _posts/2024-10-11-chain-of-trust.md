---
layout: post
title: 2024-10-11-chain-of-trust
date: 2024-10-11
category: security
---

# 0. create a root certificate

## 0.1 generate a private key for the root certificate

```sh
openssl genrsa -out rootCA.key 2048
```

## 0.2 create a self-signed root certificate

```sh
openssl req -x509 -new -nodes -key rootCA.key -sha256 -days 3650 -out rootCA.crt -subj "/CN=MyRootCA"
```

## 0.3 print the root certificate

`

- Data session을 sha256로 hashing 후 private key(`rootCA.key`)로 RSAEncryption를 실행하여 Signature에 입력됨 (이를 self sign이라고 함)

- Issuer는 `CN=MyRootCA`로 입력됨

- Subject `CN=MyRootCA`로 입력됨

```sh
openssl x509 -in rootCA.crt -text -noout
Certificate:
    Data:
        Version: 3 (0x2)
        Serial Number:
            6a:94:bf:d5:af:c3:29:c0:2a:df:b9:9a:c2:f2:42:c8:f2:e8:88:3c
        Signature Algorithm: sha256WithRSAEncryption
        Issuer: CN=MyRootCA
        Validity
            Not Before: Oct 11 05:13:49 2024 GMT
            Not After : Oct  9 05:13:49 2034 GMT
        Subject: CN=MyRootCA
        Subject Public Key Info:
            Public Key Algorithm: rsaEncryption
                Public-Key: (2048 bit)
                Modulus:
                    00:9f:da:29:e6:9e:5b:e8:41:53:bc:51:17:13:8d:
                    35:a3:5a:20:7d:05:6d:a4:da:44:70:d9:2d:e8:14:
                    19:0c:fa:c1:13:0a:2f:fd:5d:59:8d:83:58:e6:ba:
                    d3:20:ac:66:9a:8f:2c:c0:63:15:57:ce:dc:08:6d:
                    b6:92:11:24:c9:8e:91:89:a6:87:01:ff:6a:5b:14:
                    28:c1:77:8c:ce:b9:ef:b6:fa:44:bb:c4:d5:16:5f:
                    1c:05:ba:0f:7a:aa:fd:2f:13:40:28:25:ff:d6:68:
                    20:a3:f2:ab:1a:9d:0f:ff:94:c8:a0:95:71:a2:f3:
                    26:15:42:b0:13:2e:da:9b:3c:d1:c1:19:8a:f6:0f:
                    2e:cd:7f:e2:4b:80:9a:c9:7d:89:91:95:b4:c9:61:
                    f3:5e:d5:47:e4:6e:0c:c2:30:c2:08:2e:2f:11:d5:
                    85:8c:3a:e8:ef:7e:d2:6d:ad:dd:68:9a:0a:25:bb:
                    0e:47:f3:49:43:34:d2:14:af:04:84:9c:74:e0:55:
                    18:f0:01:13:52:a9:40:ca:28:aa:a1:40:bf:b8:6d:
                    0a:9b:6d:1e:c1:d6:5b:43:2a:49:0c:a6:b8:40:e4:
                    e9:c0:1c:9f:43:47:27:d2:79:0a:34:ae:4f:b5:29:
                    cc:75:4d:b1:22:ca:91:dd:5e:ea:8e:a3:de:ff:6b:
                    a3:6f
                Exponent: 65537 (0x10001)
        X509v3 extensions:
            X509v3 Subject Key Identifier:
                EE:6E:54:13:0F:34:47:C4:6D:D7:4C:8F:E9:C2:20:A6:85:D2:CC:39
            X509v3 Authority Key Identifier:
                EE:6E:54:13:0F:34:47:C4:6D:D7:4C:8F:E9:C2:20:A6:85:D2:CC:39
            X509v3 Basic Constraints: critical
                CA:TRUE
    Signature Algorithm: sha256WithRSAEncryption
    Signature Value:
        29:46:34:b2:69:e8:9c:4e:de:ab:eb:d3:9b:4f:ec:6a:97:62:
        04:65:13:29:72:1b:44:15:76:3b:4c:ad:3b:19:3a:2f:ed:13:
        88:e4:fd:d0:07:bd:34:21:65:bd:c4:59:c9:4b:89:3c:d5:f7:
        f3:16:5d:cb:f3:39:b7:bd:58:44:69:30:35:a8:65:ef:93:b9:
        7e:60:30:27:cf:55:f9:96:d3:74:55:4e:f2:38:aa:5f:28:86:
        86:d9:5a:02:a8:62:0c:58:0e:b4:6c:0f:c1:d4:1f:d2:b0:86:
        cd:d3:00:ec:04:3b:30:bb:74:01:ae:2f:16:53:68:5d:a6:bf:
        b2:12:31:ad:2d:8e:fe:f7:e6:df:54:44:e7:b0:c0:d7:e8:a3:
        d2:7e:44:dc:2a:4e:18:d2:74:ab:a3:92:9e:24:cd:b7:a1:bf:
        13:ae:c4:da:25:cc:05:0c:c8:8e:38:9c:6e:94:ba:12:aa:14:
        97:46:85:4d:fc:35:d0:dd:b2:84:6c:32:28:fb:f6:33:23:7a:
        70:b4:0d:dc:1b:0d:26:4c:61:02:d9:0b:c1:2e:1a:b1:18:e1:
        0e:20:99:b5:03:02:25:b8:70:2a:c8:fa:e9:4b:71:c7:18:39:
        e4:76:bf:3f:38:63:47:28:82:f2:2c:db:a7:d6:68:6a:26:51:
        a5:22:05:08
```

# 1. create a leaf certification

## 1.1 generate a private key for the leaf certificate

```sh
openssl genrsa -out leaf.key 2048
```

## 1.2 create a certificate signing request (CSR) for the leaf certificate

```sh
openssl req -new -key leaf.key -out leaf.csr -subj "/CN=MyLeafCertificate"
```

## 1.3 print the certificate signing request (CSR)

- Data session을 sha256로 hashing 후 private key(`leaf.key`)로 RSAEncryption를 실행하여 Signature에 입력됨

- Subject `CN=MyLeafCertificate`로 입력됨

```sh
openssl req -in leaf.csr -text -noout
Certificate Request:
    Data:
        Version: 1 (0x0)
        Subject: CN=MyLeafCertificate
        Subject Public Key Info:
            Public Key Algorithm: rsaEncryption
                Public-Key: (2048 bit)
                Modulus:
                    00:af:67:12:48:b4:3b:3d:48:9b:4a:fb:4e:89:db:
                    ce:36:65:5f:ac:a9:de:cd:43:54:e0:45:8f:8f:e7:
                    0a:ab:3d:cd:00:25:63:23:fa:5a:e9:6f:b0:29:e7:
                    85:30:2b:73:77:25:b0:07:c6:50:34:c9:d7:02:3f:
                    fa:8c:0f:e4:86:7e:01:cc:1d:ab:59:9c:62:e1:d5:
                    5c:8f:8a:46:65:c3:4f:fd:e7:dd:1f:06:f7:ab:d5:
                    cb:00:0c:73:05:3a:ce:99:f4:ac:41:d4:aa:87:f6:
                    ae:bc:d8:30:f7:3d:c3:cb:05:a4:2d:7e:08:26:b3:
                    b5:f1:86:b6:5c:64:9a:e6:37:b8:97:6a:3b:fd:bc:
                    4b:92:0b:e5:f0:ce:2d:9c:eb:dd:ed:31:f4:db:f4:
                    94:0f:91:d2:ec:51:2c:a4:21:c8:f4:f2:80:8f:3f:
                    d4:87:a6:46:84:32:05:d0:ec:14:f1:9f:a5:63:70:
                    82:9e:48:ab:a4:2e:df:b5:04:a7:0e:1c:3b:48:93:
                    d3:74:fc:cf:44:59:3f:eb:bc:1f:f4:9f:c1:63:19:
                    bb:99:d8:d7:07:d5:49:46:5b:68:58:c0:47:e2:0b:
                    6c:87:8b:ec:41:2c:15:cc:9a:28:e9:53:83:57:86:
                    71:d1:b7:cc:66:cc:94:01:ca:db:82:57:d8:6e:e9:
                    4f:0f
                Exponent: 65537 (0x10001)
        Attributes:
            (none)
            Requested Extensions:
    Signature Algorithm: sha256WithRSAEncryption
    Signature Value:
        a2:0a:08:fd:ae:bd:8c:93:27:c2:ff:b0:1b:42:7e:3f:71:b7:
        d9:b1:4d:d4:e5:80:7c:e7:b9:44:c8:78:86:2e:b7:75:cb:74:
        f3:fa:03:ec:f6:3b:d7:24:e1:1f:a0:3b:81:6c:f5:4b:99:35:
        c8:f4:01:f5:3e:2a:08:32:f8:39:cf:2b:f1:21:5a:eb:d2:74:
        e5:86:30:e3:8f:61:70:2f:b0:a8:2b:08:18:7c:5d:91:f3:b1:
        bd:2e:f8:83:28:85:74:6d:0e:4f:6e:f6:2c:f8:b7:df:84:d9:
        d3:23:61:9b:be:8b:10:30:86:95:72:03:31:26:3d:b6:f7:f7:
        af:da:9a:ae:df:c7:3e:e0:c7:7f:87:e5:a5:9c:e0:6c:e7:69:
        12:bd:9b:01:72:c8:3f:c2:45:71:73:b7:83:b2:41:e8:03:bb:
        de:d0:bf:a9:e5:3a:26:80:8e:3d:87:fd:d3:cf:20:fa:65:11:
        37:94:95:7d:eb:d1:7d:98:05:63:53:8e:3a:62:ac:cd:d8:08:
        b0:64:70:3f:b7:68:8b:98:89:cd:bb:ad:8b:32:0f:0c:a9:b8:
        b1:60:b8:b6:63:43:80:3a:93:59:53:41:9c:71:b5:8b:7d:a6:
        6d:ae:d6:94:77:e4:10:8b:f3:b3:60:6a:a5:55:25:3d:d8:af:
        48:91:13:5e
```

## 1.4 sign the leaf certificate using the root certificate

```sh
openssl x509 -req -in leaf.csr -CA rootCA.crt -CAkey rootCA.key -CAcreateserial -out leaf.crt -days 365 -sha256
Certificate request self-signature ok
subject=CN=MyLeafCertificate
```

## 1.5 print the rootCA.srl

```sh
cat rootCA.srl
6658072D9D3E969AD1D6EEED18EA35ACE8B6CCDD
```

## 1.6 print the leaf certificate

- Data session을 sha256로 hashing 후 private key(`rootCA.key`)로 RSAEncryption를 실행하여 Signature에 입력됨

- Issuer는 `CN=MyRootCA`로 입력됨

- Subject는 `CN=MyLeafCertificate`로 입력됨

-- Serial Number는 `rootCA.srl`의 값이 입력됨

```sh
openssl x509 -in leaf.crt -text -noout
Certificate:
    Data:
        Version: 3 (0x2)
        Serial Number:
            66:58:07:2d:9d:3e:96:9a:d1:d6:ee:ed:18:ea:35:ac:e8:b6:cc:dd
        Signature Algorithm: sha256WithRSAEncryption
        Issuer: CN=MyRootCA
        Validity
            Not Before: Oct 11 05:30:27 2024 GMT
            Not After : Oct 11 05:30:27 2025 GMT
        Subject: CN=MyLeafCertificate
        Subject Public Key Info:
            Public Key Algorithm: rsaEncryption
                Public-Key: (2048 bit)
                Modulus:
                    00:af:67:12:48:b4:3b:3d:48:9b:4a:fb:4e:89:db:
                    ce:36:65:5f:ac:a9:de:cd:43:54:e0:45:8f:8f:e7:
                    0a:ab:3d:cd:00:25:63:23:fa:5a:e9:6f:b0:29:e7:
                    85:30:2b:73:77:25:b0:07:c6:50:34:c9:d7:02:3f:
                    fa:8c:0f:e4:86:7e:01:cc:1d:ab:59:9c:62:e1:d5:
                    5c:8f:8a:46:65:c3:4f:fd:e7:dd:1f:06:f7:ab:d5:
                    cb:00:0c:73:05:3a:ce:99:f4:ac:41:d4:aa:87:f6:
                    ae:bc:d8:30:f7:3d:c3:cb:05:a4:2d:7e:08:26:b3:
                    b5:f1:86:b6:5c:64:9a:e6:37:b8:97:6a:3b:fd:bc:
                    4b:92:0b:e5:f0:ce:2d:9c:eb:dd:ed:31:f4:db:f4:
                    94:0f:91:d2:ec:51:2c:a4:21:c8:f4:f2:80:8f:3f:
                    d4:87:a6:46:84:32:05:d0:ec:14:f1:9f:a5:63:70:
                    82:9e:48:ab:a4:2e:df:b5:04:a7:0e:1c:3b:48:93:
                    d3:74:fc:cf:44:59:3f:eb:bc:1f:f4:9f:c1:63:19:
                    bb:99:d8:d7:07:d5:49:46:5b:68:58:c0:47:e2:0b:
                    6c:87:8b:ec:41:2c:15:cc:9a:28:e9:53:83:57:86:
                    71:d1:b7:cc:66:cc:94:01:ca:db:82:57:d8:6e:e9:
                    4f:0f
                Exponent: 65537 (0x10001)
        X509v3 extensions:
            X509v3 Subject Key Identifier:
                07:A6:16:5F:1F:E5:BA:70:F4:E6:F8:56:AA:69:82:B3:C0:26:DD:71
            X509v3 Authority Key Identifier:
                EE:6E:54:13:0F:34:47:C4:6D:D7:4C:8F:E9:C2:20:A6:85:D2:CC:39
    Signature Algorithm: sha256WithRSAEncryption
    Signature Value:
        9a:99:3e:5b:0e:7e:22:cb:90:e6:06:23:75:71:58:9e:63:84:
        b8:e0:19:52:cc:ef:f0:73:40:e6:7f:95:24:6f:0d:c3:8d:09:
        7f:28:dc:38:85:d5:bd:72:80:94:a5:27:22:ca:12:6a:42:5a:
        6b:83:17:af:71:c5:8f:55:8e:84:a6:75:74:99:21:66:f0:cb:
        fe:6c:48:9a:93:13:a4:16:bd:61:cf:d3:c2:a4:ae:37:f7:c0:
        8f:8f:cc:6e:18:45:d5:dd:97:eb:e1:f6:0c:f9:56:b0:f8:dc:
        3e:b6:af:1d:c9:2e:ed:47:5d:23:61:39:6b:f1:5a:a2:00:f8:
        a7:48:0e:43:37:1e:c3:82:0c:d1:71:77:e7:2f:b2:89:b4:66:
        9e:39:b4:02:b9:bf:9f:9d:53:dd:be:90:88:ba:f4:65:fc:7e:
        83:a9:8d:19:44:9e:ac:9b:c1:df:99:9e:34:6a:57:4d:03:25:
        cd:46:f1:21:d3:29:63:62:73:12:33:ea:79:b7:71:cb:fe:43:
        f8:5f:86:e7:03:30:2d:29:2f:e3:25:ec:81:ba:dd:5a:4f:78:
        e8:9c:86:95:86:52:65:56:38:2a:3d:e7:e3:89:3c:7b:f4:40:
        62:fe:79:e3:dc:9e:ec:66:a0:4c:73:ff:85:f9:0c:b5:64:df:
```

# 3. verify the leaf certificate

- leaf.crt 의 `Data` session를 sha256 hashing

- leaf.crt 의 `Signature Value`를 `rootCA.crt`의 public key로 복호화

- 첫번째 값과 두번째 값이 같으면 veridation 통과

```sh
openssl verify -CAfile rootCA.crt leaf.crt
leaf.crt: OK
```
