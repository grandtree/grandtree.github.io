---
layout: post
title:  "Get self signed root certificate"
tags: Network Security 
---

It is common that in daily work of a devloper we encounter internal web server
using self signed certificate, because our web browser and OS used don't accept
it for obvious reason, we have to do one more two clicks to accept the risk or
tell script code to ignore certificate check. One soltuion is that we broadcast the root
certificate (of course the public one) used to sign these certificates, but in
fact there is no need for this, according to the implementation of PKI, a full
chain of certificates involved in signing are sent from server, we only need to
get the root with some tools. In this article, I will show how-to with 2 tools. 

In the examples above, I am using one web server of mine (https://vultr.quyi.buzz:2320) 
and 2 certificates:

- the certificate of root CA: of course, this CA is myself, so the issuer and
  subject are the same entity.  
- the certificate of web server: it is signed by the root CA above. 

At the beginning, this certificate casues trouble because it is not signed by a
well-known CA:

```
qy@yinspiron:~/tmp$ curl https://vultr.quyi.buzz:2320/index.html -o index.html
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
  0     0    0     0    0     0      0      0 --:--:--  0:00:01 --:--:--     0
curl: (60) SSL certificate problem: unable to get local issuer certificate
More details here: https://curl.se/docs/sslcerts.html

curl failed to verify the legitimacy of the server and therefore could not
establish a secure connection to it. To learn more about this situation and
how to fix it, please visit the web page mentioned above.
```

# Get root cerficate

## With openssl  

'--showcerts' of openssl can print out the content of certificate chain:

```
qy@yinspiron:~/tmp$ openssl s_client -showcerts -connect  vultr.quyi.buzz:2320
CONNECTED(00000003)
depth=1 C = CN, ST = Beijing, L = Beijing, O = quyi, CN = quyi.buzz, emailAddress = grandtree2005@gmail.com
verify error:num=19:self signed certificate in certificate chain
verify return:1
depth=1 C = CN, ST = Beijing, L = Beijing, O = quyi, CN = quyi.buzz, emailAddress = grandtree2005@gmail.com
verify return:1
depth=0 C = CN, ST = Beijing, O = quyi, CN = vultr.quyi.buzz
verify return:1
---
Certificate chain
 0 s:C = CN, ST = Beijing, O = quyi, CN = vultr.quyi.buzz
   i:C = CN, ST = Beijing, L = Beijing, O = quyi, CN = quyi.buzz, emailAddress = grandtree2005@gmail.com
-----BEGIN CERTIFICATE-----
MIIDSzCCAjMCFCvCL9U2rfDbldoUWDC/+L3bHGiKMA0GCSqGSIb3DQEBCwUAMHwx
CzAJBgNVBAYTAkNOMRAwDgYDVQQIDAdCZWlqaW5nMRAwDgYDVQQHDAdCZWlqaW5n
MQ0wCwYDVQQKDARxdXlpMRIwEAYDVQQDDAlxdXlpLmJ1enoxJjAkBgkqhkiG9w0B
CQEWF2dyYW5kdHJlZTIwMDVAZ21haWwuY29tMB4XDTIzMDkxNzA0MDQxN1oXDTMz
MDkxNDA0MDQxN1owSDELMAkGA1UEBhMCQ04xEDAOBgNVBAgMB0JlaWppbmcxDTAL
BgNVBAoMBHF1eWkxGDAWBgNVBAMMD3Z1bHRyLnF1eWkuYnV6ejCCASIwDQYJKoZI
hvcNAQEBBQADggEPADCCAQoCggEBAJ4XHjrLkOSxq/29IE3RBBSQee5y+jASBSa4
I93moUmlZOgNwITlXazLqhzRR5R6HuQucSEpAWKCmYo7ZfKEiff99trDZ+zKFeKV
FkizIGakm14Sp0lWTbUKJOuIrub6cJygN+kCIiz/gZjKsS2iPujhJuNfeNT9+nVG
3JxvFouqdtD2tx5ZucYrZcBc//cAEuPLSWmf4vF4Rh3gCwUKE9glTDqdN4j6emys
sIRmE8Ef19PyASlqB0u6neJDpu5wrtV+N3DCmRrfea6Pmd7jbruDZ4zy1Ayw3//5
QEb8/OTXlx7CqMjIXxH2gR2rXS9aTIo4NLajFxZpzjAFpHZrauUCAwEAATANBgkq
hkiG9w0BAQsFAAOCAQEAb37XLDdJRyrhZ/hVhXVnsI4beQts+dGB5J3vRYqCjCoI
Q6Gwh8nPciZvPHBS4EF1vLcFAHFoV3DWvhVPUm82Aa40opXeua0AWhFr07MuUXi0
T+VvAOoUhH3E41VIAG1ELqPjNHIiL8Lue+uys6Nzy8DMhVYm8Zqr5Pwpw2qCAU53
HQ9S1mIbgrzdnOpMTz56sBy7nV5hOr7wyX1497HNeEHcFrdJwuz7cqeRnpa3A28u
J1Hj/rHOkXWMckyYj9de5QxyKuibOGfyUX9s4O44lJtVQO10r7A/RRwQLFoTR/vD
/8f7ygiIat5nhR6eC0DB2XXaXv5cbaMg7TmSTzagSw==
-----END CERTIFICATE-----
 1 s:C = CN, ST = Beijing, L = Beijing, O = quyi, CN = quyi.buzz, emailAddress = grandtree2005@gmail.com
   i:C = CN, ST = Beijing, L = Beijing, O = quyi, CN = quyi.buzz, emailAddress = grandtree2005@gmail.com
-----BEGIN CERTIFICATE-----
MIID2TCCAsGgAwIBAgIUeSf5sNDxwgzUSogWM07mv2JBUJ8wDQYJKoZIhvcNAQEL
BQAwfDELMAkGA1UEBhMCQ04xEDAOBgNVBAgMB0JlaWppbmcxEDAOBgNVBAcMB0Jl
aWppbmcxDTALBgNVBAoMBHF1eWkxEjAQBgNVBAMMCXF1eWkuYnV6ejEmMCQGCSqG
SIb3DQEJARYXZ3JhbmR0cmVlMjAwNUBnbWFpbC5jb20wHhcNMjMwOTE3MDMwNjUw
WhcNMzMwOTE0MDMwNjUwWjB8MQswCQYDVQQGEwJDTjEQMA4GA1UECAwHQmVpamlu
ZzEQMA4GA1UEBwwHQmVpamluZzENMAsGA1UECgwEcXV5aTESMBAGA1UEAwwJcXV5
aS5idXp6MSYwJAYJKoZIhvcNAQkBFhdncmFuZHRyZWUyMDA1QGdtYWlsLmNvbTCC
ASIwDQYJKoZIhvcNAQEBBQADggEPADCCAQoCggEBAM498v6cJZAcZM9J4eQWtGUB
4nTDMcDtfVfxJxZ9AzADB4A4CNv2HYVS0nKPrdtX+SpV5o5BefXms4tudD2yZzsf
RaN3cb0vp3bc3TfwjNK2LYEe8VjMw1kLKj+ReBtmtTNhAwTsIzMC7GzWE/P7ngiL
pCqkIXs4wGzKqshk2udDDoxjsXCJ/GC12iPzZIty8FkZHEV3LoTIrYT1wPR0hRZ4
xgQMKJ2QqHooIXF9oMOJc6nf2Wz1ckGkVyZCA6IL7OfKn/0QT8n+ZhlqldgdymaW
HhXdmS+MrJ722C2kRI9BI3NvkZIuF9SiiXFF065pLDqsFnkhDlJPN5isD9zExm0C
AwEAAaNTMFEwHQYDVR0OBBYEFK7HCyABB9ZO8gqx58ORJElF6XmRMB8GA1UdIwQY
MBaAFK7HCyABB9ZO8gqx58ORJElF6XmRMA8GA1UdEwEB/wQFMAMBAf8wDQYJKoZI
hvcNAQELBQADggEBAExskJHlM3yUzy9m7kThYAYV2pqHSdB8p9YPJ8b5cd+VdLQZ
Toj5Tlv/Kcu3eiH+IHc9/J4O2KFPNGOkIySlU3GPiEr4Fw03srV2HPgjqImgz674
v68b4OJ/KwUvAMNHwjY/hcu6ykTTj8ZFoyDxdyQ7mjj83tPBxo+VDC5NMW/5uV1x
/QezAi33Jx9l+vBCf36Zd3zcnHYvU3RnxO3GmgpuWPBUdpQiWZbuI6jVn+QCV22g
euEaZy85XMDvWBsmuNR89OrBN7NXmZN8XETYPorj8fGRUEe++alGo4f3DXrZmQnp
Q6COPmZt+ckQQX/HiRX3A/vU+6W8++YOJjHXWi4=
-----END CERTIFICATE-----
---
Server certificate
subject=C = CN, ST = Beijing, O = quyi, CN = vultr.quyi.buzz

issuer=C = CN, ST = Beijing, L = Beijing, O = quyi, CN = quyi.buzz, emailAddress = grandtree2005@gmail.com

---
No client certificate CA names sent
Peer signing digest: SHA256
Peer signature type: RSA-PSS
Server Temp Key: X25519, 253 bits
---
SSL handshake has read 2500 bytes and written 418 bytes
Verification error: self signed certificate in certificate chain
---
New, TLSv1.2, Cipher is ECDHE-RSA-AES256-GCM-SHA384
Server public key is 2048 bit
Secure Renegotiation IS supported
Compression: NONE
Expansion: NONE
No ALPN negotiated
SSL-Session:
    Protocol  : TLSv1.2
    Cipher    : ECDHE-RSA-AES256-GCM-SHA384
    Session-ID: 55DA6ED93656C9EE47DC2C2CF972B65163F20568BDCD37596334E11A8A1B946A
    Session-ID-ctx:
    Master-Key: A8B39793EA26F3D0414253A640A3DC039E94CA987FF39CCAC2A4A29A174A3320E6D66EECA685F019E814C8C8F86F681E
    PSK identity: None
    PSK identity hint: None
    SRP username: None
    TLS session ticket lifetime hint: 300 (seconds)
    TLS session ticket:
    0000 - d0 88 10 f7 13 e3 eb 4a-e0 75 19 83 1a 9c 0e b6   .......J.u......
    0010 - b9 10 c9 c2 61 fc 40 26-a8 ea 19 f0 50 cd a6 ed   ....a.@&....P...
    0020 - 0d 36 24 0d 20 1c 79 fb-9d f3 b7 e5 79 21 85 c0   .6$. .y.....y!..
    0030 - bc 38 c2 d1 22 ce 5d 33-6d e8 79 cf 3f b4 e0 85   .8..".]3m.y.?...
    0040 - e9 f6 3b 05 3d 40 7b e2-2c ae 76 a2 e6 91 7a d8   ..;.=@{.,.v...z.
    0050 - 21 d5 a1 0e ba 09 e8 3a-e4 38 06 35 ce 08 f9 f8   !......:.8.5....
    0060 - 0e fe 4a 6a 97 43 1b f1-5f 61 f3 ee 4c af fe 7e   ..Jj.C.._a..L..~
    0070 - 2c ef e1 8a a8 fb 09 cd-5a a9 74 ac 7b 6b 23 ea   ,.......Z.t.{k#.
    0080 - ba c7 41 b0 05 59 77 89-93 a1 c8 32 0d 52 f4 b8   ..A..Yw....2.R..
    0090 - 58 ec b8 16 69 7c 66 11-8a 8d 5c c9 0d f0 aa 33   X...i|f...\....3
    00a0 - d9 81 83 b4 72 5c 7b 4d-09 90 89 29 72 47 36 a3   ....r\{M...)rG6.
    00b0 - bb ca 5d 68 c9 18 59 41-5d 04 9b 93 e1 b9 17 bf   ..]h..YA].......

    Start Time: 1746683936
    Timeout   : 7200 (sec)
    Verify return code: 19 (self signed certificate in certificate chain)
    Extended master secret: yes
---
closed
------------------------------------------------------------------------------------------------------------------------------------------- 13:59:56

```

We need to get the root certificate, which is the last one in the chain, becasue
it can cover all the certificates signed by it. Remember to also include the
lines of 'BEGIN' and "END". 

```
-----BEGIN CERTIFICATE-----
MIID2TCCAsGgAwIBAgIUeSf5sNDxwgzUSogWM07mv2JBUJ8wDQYJKoZIhvcNAQEL
BQAwfDELMAkGA1UEBhMCQ04xEDAOBgNVBAgMB0JlaWppbmcxEDAOBgNVBAcMB0Jl
aWppbmcxDTALBgNVBAoMBHF1eWkxEjAQBgNVBAMMCXF1eWkuYnV6ejEmMCQGCSqG
SIb3DQEJARYXZ3JhbmR0cmVlMjAwNUBnbWFpbC5jb20wHhcNMjMwOTE3MDMwNjUw
WhcNMzMwOTE0MDMwNjUwWjB8MQswCQYDVQQGEwJDTjEQMA4GA1UECAwHQmVpamlu
ZzEQMA4GA1UEBwwHQmVpamluZzENMAsGA1UECgwEcXV5aTESMBAGA1UEAwwJcXV5
aS5idXp6MSYwJAYJKoZIhvcNAQkBFhdncmFuZHRyZWUyMDA1QGdtYWlsLmNvbTCC
ASIwDQYJKoZIhvcNAQEBBQADggEPADCCAQoCggEBAM498v6cJZAcZM9J4eQWtGUB
4nTDMcDtfVfxJxZ9AzADB4A4CNv2HYVS0nKPrdtX+SpV5o5BefXms4tudD2yZzsf
RaN3cb0vp3bc3TfwjNK2LYEe8VjMw1kLKj+ReBtmtTNhAwTsIzMC7GzWE/P7ngiL
pCqkIXs4wGzKqshk2udDDoxjsXCJ/GC12iPzZIty8FkZHEV3LoTIrYT1wPR0hRZ4
xgQMKJ2QqHooIXF9oMOJc6nf2Wz1ckGkVyZCA6IL7OfKn/0QT8n+ZhlqldgdymaW
HhXdmS+MrJ722C2kRI9BI3NvkZIuF9SiiXFF065pLDqsFnkhDlJPN5isD9zExm0C
AwEAAaNTMFEwHQYDVR0OBBYEFK7HCyABB9ZO8gqx58ORJElF6XmRMB8GA1UdIwQY
MBaAFK7HCyABB9ZO8gqx58ORJElF6XmRMA8GA1UdEwEB/wQFMAMBAf8wDQYJKoZI
hvcNAQELBQADggEBAExskJHlM3yUzy9m7kThYAYV2pqHSdB8p9YPJ8b5cd+VdLQZ
Toj5Tlv/Kcu3eiH+IHc9/J4O2KFPNGOkIySlU3GPiEr4Fw03srV2HPgjqImgz674
v68b4OJ/KwUvAMNHwjY/hcu6ykTTj8ZFoyDxdyQ7mjj83tPBxo+VDC5NMW/5uV1x
/QezAi33Jx9l+vBCf36Zd3zcnHYvU3RnxO3GmgpuWPBUdpQiWZbuI6jVn+QCV22g
euEaZy85XMDvWBsmuNR89OrBN7NXmZN8XETYPorj8fGRUEe++alGo4f3DXrZmQnp
Q6COPmZt+ckQQX/HiRX3A/vU+6W8++YOJjHXWi4=
-----END CERTIFICATE-----
```

## With curl  

Versatile curl doesn't miss this case, it has '-w %{certs}':

```
localhost:~ # curl --insecure -w %{certs}  https://vultr.quyi.buzz:2320/index.html
<!DOCTYPE HTML PUBLIC "-//W3C//DTD HTML 4.01//EN" "http://www.w3.org/TR/html4/strict.dtd">
<HTML>
        <HEAD>
                <TITLE>My first HTML document</TITLE>
        </HEAD>
        <BODY>
                <P>Hello world!
        </BODY>
</HTML>
Subject:C = CN, ST = Beijing, O = quyi, CN = vultr.quyi.buzz
Issuer:C = CN, ST = Beijing, L = Beijing, O = quyi, CN = quyi.buzz, emailAddress = grandtree2005@gmail.com
Version:0
Serial Number:2bc22fd536adf0db95da145830bff8bddb1c688a
Signature Algorithm:sha256WithRSAEncryption
Public Key Algorithm:rsaEncryption
Start date:Sep 17 04:04:17 2023 GMT
Expire date:Sep 14 04:04:17 2033 GMT
RSA Public Key:2048
rsa(n):9E171E3ACB90E4B1ABFDBD204DD104149079EE72FA30120526B823DDE6A149A564E80DC084E55DACCBAA1CD147947A1EE42E712129016282998A3B65F28489F7FDF6DAC367ECCA15E2951648B32066A49B5E12A749564DB50A24EB88AEE6FA709CA037E902222CFF8198CAB12DA23EE8E126E35F78D4FDFA7546DC9C6F168BAA76D0F6B71E59B9C62B65C05CFFF70012E3CB49699FE2F178461DE00B050A13D8254C3A9D3788FA7A6CACB0846613C11FD7D3F201296A074BBA9DE243A6EE70AED57E3770C2991ADF79AE8F99DEE36EBB83678CF2D40CB0DFFFF94046FCFCE4D7971EC2A8C8C85F11F6811DAB5D2F5A4C8A3834B6A3171669CE3005A4766B6AE5
rsa(e):10001
Signature:6f:7e:d7:2c:37:49:47:2a:e1:67:f8:55:85:75:67:b0:8e:1b:79:0b:6c:f9:d1:81:e4:9d:ef:45:8a:82:8c:2a:08:43:a1:b0:87:c9:cf:72:26:6f:3c:70:52:e0:41:75:bc:b7:05:00:71:68:57:70:d6:be:15:4f:52:6f:36:01:ae:34:a2:95:de:b9:ad:00:5a:11:6b:d3:b3:2e:51:78:b4:4f:e5:6f:00:ea:14:84:7d:c4:e3:55:48:00:6d:44:2e:a3:e3:34:72:22:2f:c2:ee:7b:eb:b2:b3:a3:73:cb:c0:cc:85:56:26:f1:9a:ab:e4:fc:29:c3:6a:82:01:4e:77:1d:0f:52:d6:62:1b:82:bc:dd:9c:ea:4c:4f:3e:7a:b0:1c:bb:9d:5e:61:3a:be:f0:c9:7d:78:f7:b1:cd:78:41:dc:16:b7:49:c2:ec:fb:72:a7:91:9e:96:b7:03:6f:2e:27:51:e3:fe:b1:ce:91:75:8c:72:4c:98:8f:d7:5e:e5:0c:72:2a:e8:9b:38:67:f2:51:7f:6c:e0:ee:38:94:9b:55:40:ed:74:af:b0:3f:45:1c:10:2c:5a:13:47:fb:c3:ff:c7:fb:ca:08:88:6a:de:67:85:1e:9e:0b:40:c1:d9:75:da:5e:fe:5c:6d:a3:20:ed:39:92:4f:36:a0:4b:
-----BEGIN CERTIFICATE-----
MIIDSzCCAjMCFCvCL9U2rfDbldoUWDC/+L3bHGiKMA0GCSqGSIb3DQEBCwUAMHwx
CzAJBgNVBAYTAkNOMRAwDgYDVQQIDAdCZWlqaW5nMRAwDgYDVQQHDAdCZWlqaW5n
MQ0wCwYDVQQKDARxdXlpMRIwEAYDVQQDDAlxdXlpLmJ1enoxJjAkBgkqhkiG9w0B
CQEWF2dyYW5kdHJlZTIwMDVAZ21haWwuY29tMB4XDTIzMDkxNzA0MDQxN1oXDTMz
MDkxNDA0MDQxN1owSDELMAkGA1UEBhMCQ04xEDAOBgNVBAgMB0JlaWppbmcxDTAL
BgNVBAoMBHF1eWkxGDAWBgNVBAMMD3Z1bHRyLnF1eWkuYnV6ejCCASIwDQYJKoZI
hvcNAQEBBQADggEPADCCAQoCggEBAJ4XHjrLkOSxq/29IE3RBBSQee5y+jASBSa4
I93moUmlZOgNwITlXazLqhzRR5R6HuQucSEpAWKCmYo7ZfKEiff99trDZ+zKFeKV
FkizIGakm14Sp0lWTbUKJOuIrub6cJygN+kCIiz/gZjKsS2iPujhJuNfeNT9+nVG
3JxvFouqdtD2tx5ZucYrZcBc//cAEuPLSWmf4vF4Rh3gCwUKE9glTDqdN4j6emys
sIRmE8Ef19PyASlqB0u6neJDpu5wrtV+N3DCmRrfea6Pmd7jbruDZ4zy1Ayw3//5
QEb8/OTXlx7CqMjIXxH2gR2rXS9aTIo4NLajFxZpzjAFpHZrauUCAwEAATANBgkq
hkiG9w0BAQsFAAOCAQEAb37XLDdJRyrhZ/hVhXVnsI4beQts+dGB5J3vRYqCjCoI
Q6Gwh8nPciZvPHBS4EF1vLcFAHFoV3DWvhVPUm82Aa40opXeua0AWhFr07MuUXi0
T+VvAOoUhH3E41VIAG1ELqPjNHIiL8Lue+uys6Nzy8DMhVYm8Zqr5Pwpw2qCAU53  
HQ9S1mIbgrzdnOpMTz56sBy7nV5hOr7wyX1497HNeEHcFrdJwuz7cqeRnpa3A28u  
J1Hj/rHOkXWMckyYj9de5QxyKuibOGfyUX9s4O44lJtVQO10r7A/RRwQLFoTR/vD
/8f7ygiIat5nhR6eC0DB2XXaXv5cbaMg7TmSTzagSw==
-----END CERTIFICATE-----
Subject:C = CN, ST = Beijing, L = Beijing, O = quyi, CN = quyi.buzz, emailAddress = grandtree2005@gmail.com
Issuer:C = CN, ST = Beijing, L = Beijing, O = quyi, CN = quyi.buzz, emailAddress = grandtree2005@gmail.com
Version:2       
Serial Number:7927f9b0d0f1c20cd44a8816334ee6bf6241509f
Signature Algorithm:sha256WithRSAEncryption
Public Key Algorithm:rsaEncryption
X509v3 Subject Key Identifier:AE:C7:0B:20:01:07:D6:4E:F2:0A:B1:E7:C3:91:24:49:45:E9:79:91
X509v3 Authority Key Identifier:keyid:AE:C7:0B:20:01:07:D6:4E:F2:0A:B1:E7:C3:91:24:49:45:E9:79:91
X509v3 Basic Constraints:CA:TRUE
Start date:Sep 17 03:06:50 2023 GMT
Expire date:Sep 14 03:06:50 2033 GMT
RSA Public Key:2048
rsa(n):CE3DF2FE9C25901C64CF49E1E416B46501E274C331C0ED7D57F127167D03300307803808DBF61D8552D2728FADDB57F92A55E68E4179F5E6B38B6E743DB2673B1F45A37771BD2FA776DCDD37F08CD2B62D811EF158CCC3590B2A3F91781B66B533610304EC233302EC6CD613F3FB9E088BA42AA4217B38C06CCAAAC864DAE7430E8C63B17089FC60B5DA23F3648B72F059191C45772E84C8AD84F5C0F474851678C6040C289D90A87A2821717DA0C38973A9DFD96CF57241A457264203A20BECE7CA9FFD104FC9FE66196A95D81DCA66961E15DD992F8CAC9EF6D82DA4448F4123736F91922E17D4A2897145D3AE692C3AAC1679210E524F3798AC0FDCC4C66D
rsa(e):10001
Signature:4c:6c:90:91:e5:33:7c:94:cf:2f:66:ee:44:e1:60:06:15:da:9a:87:49:d0:7c:a7:d6:0f:27:c6:f9:71:df:95:74:b4:19:4e:88:f9:4e:5b:ff:29:cb:b7:7a:21:fe:20:77:3d:fc:9e:0e:d8:a1:4f:34:63:a4:23:24:a5:53:71:8f:88:4a:f8:17:0d:37:b2:b5:76:1c:f8:23:a8:89:a0:cf:ae:f8:bf:af:1b:e0:e2:7f:2b:05:2f:00:c3:47:c2:36:3f:85:cb:ba:ca:44:d3:8f:c6:45:a3:20:f1:77:24:3b:9a:38:fc:de:d3:c1:c6:8f:95:0c:2e:4d:31:6f:f9:b9:5d:71:fd:07:b3:02:2d:f7:27:1f:65:fa:f0:42:7f:7e:99:77:7c:dc:9c:76:2f:53:74:67:c4:ed:c6:9a:0a:6e:58:f0:54:76:94:22:59:96:ee:23:a8:d5:9f:e4:02:57:6d:a0:7a:e1:1a:67:2f:39:5c:c0:ef:58:1b:26:b8:d4:7c:f4:ea:c1:37:b3:57:99:93:7c:5c:44:d8:3e:8a:e3:f1:f1:91:50:47:be:f9:a9:46:a3:87:f7:0d:7a:d9:99:09:e9:43:a0:8e:3e:66:6d:f9:c9:10:41:7f:c7:89:15:f7:03:fb:d4:fb:a5:bc:fb:e6:0e:26:31:d7:5a:2e:
-----BEGIN CERTIFICATE-----
MIID2TCCAsGgAwIBAgIUeSf5sNDxwgzUSogWM07mv2JBUJ8wDQYJKoZIhvcNAQEL
BQAwfDELMAkGA1UEBhMCQ04xEDAOBgNVBAgMB0JlaWppbmcxEDAOBgNVBAcMB0Jl
aWppbmcxDTALBgNVBAoMBHF1eWkxEjAQBgNVBAMMCXF1eWkuYnV6ejEmMCQGCSqG
SIb3DQEJARYXZ3JhbmR0cmVlMjAwNUBnbWFpbC5jb20wHhcNMjMwOTE3MDMwNjUw
WhcNMzMwOTE0MDMwNjUwWjB8MQswCQYDVQQGEwJDTjEQMA4GA1UECAwHQmVpamlu
ZzEQMA4GA1UEBwwHQmVpamluZzENMAsGA1UECgwEcXV5aTESMBAGA1UEAwwJcXV5
aS5idXp6MSYwJAYJKoZIhvcNAQkBFhdncmFuZHRyZWUyMDA1QGdtYWlsLmNvbTCC
ASIwDQYJKoZIhvcNAQEBBQADggEPADCCAQoCggEBAM498v6cJZAcZM9J4eQWtGUB
4nTDMcDtfVfxJxZ9AzADB4A4CNv2HYVS0nKPrdtX+SpV5o5BefXms4tudD2yZzsf
RaN3cb0vp3bc3TfwjNK2LYEe8VjMw1kLKj+ReBtmtTNhAwTsIzMC7GzWE/P7ngiL
pCqkIXs4wGzKqshk2udDDoxjsXCJ/GC12iPzZIty8FkZHEV3LoTIrYT1wPR0hRZ4
xgQMKJ2QqHooIXF9oMOJc6nf2Wz1ckGkVyZCA6IL7OfKn/0QT8n+ZhlqldgdymaW
HhXdmS+MrJ722C2kRI9BI3NvkZIuF9SiiXFF065pLDqsFnkhDlJPN5isD9zExm0C
AwEAAaNTMFEwHQYDVR0OBBYEFK7HCyABB9ZO8gqx58ORJElF6XmRMB8GA1UdIwQY
MBaAFK7HCyABB9ZO8gqx58ORJElF6XmRMA8GA1UdEwEB/wQFMAMBAf8wDQYJKoZI
hvcNAQELBQADggEBAExskJHlM3yUzy9m7kThYAYV2pqHSdB8p9YPJ8b5cd+VdLQZ
Toj5Tlv/Kcu3eiH+IHc9/J4O2KFPNGOkIySlU3GPiEr4Fw03srV2HPgjqImgz674
v68b4OJ/KwUvAMNHwjY/hcu6ykTTj8ZFoyDxdyQ7mjj83tPBxo+VDC5NMW/5uV1x
/QezAi33Jx9l+vBCf36Zd3zcnHYvU3RnxO3GmgpuWPBUdpQiWZbuI6jVn+QCV22g
euEaZy85XMDvWBsmuNR89OrBN7NXmZN8XETYPorj8fGRUEe++alGo4f3DXrZmQnp
Q6COPmZt+ckQQX/HiRX3A/vU+6W8++YOJjHXWi4=
-----END CERTIFICATE-----
```

# Add root certificate to certificate store of OS

Command for this purpose differs on various distros, below is how I do it on
SUSE Linux, rootCA_20230917.crt has content copied and cop from output of
utilities above:

```
yinspiron:~ # cp /home/qy/smb/smb_qy_pikvm/Downloads/rootCA_20230917.crt
/usr/share/pki/trust/anchors/
yinspiron:~ # update-ca-certificates
yinspiron:~ #
```

Now curl can succeed:

```
qy@yinspiron:~/tmp$ curl https://vultr.quyi.buzz:2320/index.html -o index.html
% Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
Dload  Upload   Total   Spent    Left  Speed
100   198  100   198    0     0    144      0  0:00:01  0:00:01 --:--:--   145
qy@yinspiron:~/tmp$ cat index.html
Hello world!
qy@yinspiron:~/tmp
```
