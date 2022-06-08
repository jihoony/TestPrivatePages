# PEM 과 CER/DER/CRT/CSR 형식 파일이란?

PKI 나 전자서명 업무를 위해서 공인/공동 인증서(Digital Certificate)와 개인키(Private key)를 다뤄 보신 분은 PEM 이나 DER, CER 이란 단어와 이런 확장자를 갖는 파일을 보았을 겁니다.





## PEM

PEM (Privacy Enhanced Mail)은 Base64 로 인코딩한 텍스트 형식의 파일입니다.

Binary 형식의 파일을 전송할 때 손상될 수 있으므로 TEXT 로 변환하며 소스 파일은 모든 바이너리가 가능하지만 주로 인증서나 개인키가 됩니다.

AWS 에서 EC2 Instance 를 만들때 접속용으로 생성하는 개인키도 PEM 형식입니다.

어떤 바이너리 파일을 PEM 으로 변환했는지 구분하기 위해 파일의 맨 앞에 dash(-) 를 5 개 넣고 BEGIN 파일 유형을 넣고 다시 dash(-) 를 5개 뒤에 END 파일유형 구문을 사용합니다.





### OpenSSH Private Key

즉 아래는 OPENSSH Private Key 를 PEM 으로 변환한 예시로 BEGIN OPENSSH PRIVATE KEY 로 시작하는 것을 확인할 수 있습니다.

```pem
-----BEGIN OPENSSH PRIVATE KEY-----
b3BlbnNzaC1rZXktdjEAAAAACmFlczI1Ni1jdHIAAAAGYmNyeXB0AAAAGAAAABC/fXuBoa
WboQk95dZ4Udj3AAAAEAAAAAEAAAGXAAAAB3NzaC1yc2EAAAADAQABAAABgQC+dEvO8yL7
XgY1NlP/vro77yxiqq/hKe4QHMplS/LnMbaKtZP1ijyMSuTGoIA+Aw9CUDpWKXwekrBXpm
GvbDlmHQieRJPhh/3dW1xPKgRPAMiiA/9awSM0sFcjyH8NQcfweu93QMBZAg/WrsQz1l7j
...
-----END OPENSSH PRIVATE KEY-----
```







### 인증서

PKI 인증서(Certificate)는 BEGIN CERTIFICATE 구문으로 시작합니다.
```pem
-----BEGIN CERTIFICATE-----
MIIDczCCAlugAwIBAgIBBDANBgkqhkiG9w0BAQUFADBkMQswCQYDVQQGEwJLUjEN
MAsGA1UECgwES0lTQTEuMCwGA1UECwwlS29yZWEgQ2VydGlmaWNhdGlvbiBBdXRo 
...
UNkAAk/bg9ART6RCVmE6fhMy04Qfybo=
-----END CERTIFICATE-----
```







### 개인키

개인키는 BEGIN RSA PRIVATE KEY 로 시작하며 암호화되었을 경우 Proc-Type: 과 DEK-Info 헤더에 암호화 알고리즘 정보를 표시합니다.

```pem
-----BEGIN RSA PRIVATE KEY-----
Proc-Type: 4,ENCRYPTED
DEK-Info: AES-128-CBC,754D916B5B2D2AF9ACDEED19908F0336

oB2JARMSnliQOgu4V96xwuqqo9rP/XXJbYFT6XEriBpVX/qOwSpAkLXAL71SB0VS
iEDdxK96zctIgo0q3zpowO2cx6uAxk4BIDWxvpM4A8xCFJgskpXeXpI5pnpgDiLO
...
-----END RSA PRIVATE KEY-----
```







## CRT

인증서를 의미하는 CERT 의 약자로 보통 PEM 형식의 인증서를 의미하며 Linux 나 Unix 계열에서 .crt 확장자를 많이 사용합니다. 
에디터로 파일을 열어서 BEGIN CERTIFICATE 구문이 있는지 확인하면 됩니다.







## CER

Windows 에서 인증서를 내보내기 할때 사용하는 확장자로 보통 PEM 형식의 인증서를 의미합니다.







## DER

DER (Distinguished Encoding Rules)형식으로 인코딩된 바이너리 파일로 주로 인증서 확장자로 많이 사용합니다.
바이너리이므로 에디터로 열면 다 깨져보이므로 der 을 인식할 수 있는 프로그램(예: openssl 등) 으로 파싱하거나 ASN.1 파서를 이용해서 열어보면 됩니다.







## CSR

CSR (Certificate Signing Request) 은 인증기관(CA)에 인증서 발급 요청을 하는 특별한 ASN.1 형식의 파일이며 그 안에는 내 공개키 정보와 사용하는 알고리즘 정보등이 들어 있습니다.

CSR 생성시 보통 PEM 형식으로 인코딩해서 전달하며  다음과 같은 PEM 헤더가 붙어 있습니다.
```pem
-----BEGIN NEW CERTIFICATE REQUEST-----
MIIDZjCCAs8CAQAwajELMAkGA1UEBhMCS1IxETAPBgNVBAgMCHdvb3dhaGFuMREw
DwYDVQQHDAh3b293YWhhbjERMA8GA1UECgwId29vd2FoYW4xETAPBgNVBAsMCHdv
b3dhaGFuMQ8wDQYDVQQDDAZhZGJldGEwgZ8wDQYJKoZIhvcNAQEBBQADgY0AMIGJ
...
-----END NEW CERTIFICATE REQUEST-----
```







## CA-Bundle.crt

여러 인증기관(CA) 의 인증서를 미리 등록한 ca 파일 묶음으로 PEM 형식으로 구성되어 있습니다.

curl  이나 Mozilla Browser 등 HTTP Client 사이트에 가서 구할 수 있습니다.

- https://curl.se/ca/cacert.pem
- [CA - MozillaWiki](https://wiki.mozilla.org/CA)
