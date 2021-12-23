# OpenSSL

# OpenSSL 자주 쓰는 명령어(command) 및 사용법, tip 정리

## 설치
RHEL/CentOS Linux는 기본 패키지에 포함되어 있으므로 별도 설치를 안해도 됩니다.

설치된 openssl의 version은 다음 명령어로 확인할수 있습니다.

```bash
$ openssl version

OpenSSL 1.0.2o 27 Mac 2018
```



## 인증서 정보 보기

openssl로 x.509 certificate를 parsing 하는 방법

### 파싱 및 정보 보기

PEM 인코딩된 x.509 인증서를 파싱해서 정보 출력

```bash
$ openssl x509 -text -noout -in localhost.crt
```

텍스트가 아닌 바이너리(DER 인코딩)일 경우 *-inform der* 옵션 추가

```bash
$ openssl x509 -inform der -text -noout -in localhost.crt
```



### 인코딩 변환
#### DER -> PEM

DER 인코딩된 인증서를 PEM으로 변경

```bash
$ openssl x509 -inform der -outform pem -out mycert.pem -in mycert.der
```



#### PEM -> DER

PEM 형식 인증서를 DER로 인코딩해서 저장

```bash
$ openssl x509 -inform pem -outform der -out mycert.der -in mycert.pem
```




## 개인키(PrivateKey)

### RSA2048 키 생성 및 개인키를 AES256으로 암호화

- 암호(pass phrase)는 asdfasdf 이며 입력창을 띄우지 않고 커맨드에서 바로 설정(*-passout* 옵션)
```bash
$ openssl genrsa -aes257 -passout pass:asdfasdf -out aes-pri.pem 2048
```

#### 위에서 생성한 개인키 복호화 하여 RSA Private Key 추출
```bash
$ openssl rsa -outform der -in aes-pri.pem -passin pass:asdfasdf -out aes-pri.key
```

#### pass phrase와 암호화 알고리즘 변경
- 알고리즘: Triple DES -> AES256
- Pass phrase: asdfasdf -> new-password
```bash
$ openssl rsa -aes256 -in aes-pri.pem -passin pass:asdfasdf -passout pass:new-password -out aes-pri.key
```

### 개인키(PrivateKey) pass phrase 해독 및 설정

보안에 취약할 수 있지만 어쩔수 없이 개인키를 암호(pass phrase)없이 사용해야 하는 경우가 종종 있습니다.

예로 비용때문에 AWS CloudFront나 Load Balancer를 사용하지 않고 직접 EC2나 On-Premise 서버에서 웹 서버를 설치하고 SSL/TLS를 설정할 경우 개인키 암호가 있으면 웹 서버 재구동시마다 입력이 필요하므로 서버 리부팅등시 문제가 됩니다.

이럴 경우 다음 openssl 명령어로 개인키를 해독해서 저장하면 pass phrase 입력없이 개인키를 사용할 수 있습니다.
#### Pass phrase 해독
/etc/pki/tls/private/examplelocalhost.key/enc 라는 암호화된 개인키가 있을 경우 다음 openssl 명령어로 해독할 수 있습니다.

```bash
$ openssl rsa -in /etc/pki/tls/private/example.com.key.enc -out /etc/pki/tls/private/example.com.key

Enter pass phrase for /etc/pki/tls/private/example.com.key.enc:
```

Enter pass phrase에 개인키에 설정한 암호를 입력해 주면 -out에 지정한 경로에 복호화된 개인키가 저장됩니다.

보안때문에 권장하지는 않지만 -passin pass:mypwd옵션으로 명령행에서 바로 pass phrase를 입력할 수 있습니다. 

```bash
$ openssl rsa -in /etc/pki/tls/private/example.com.key.enc -out /etc/pki/tls/private/example.com.key -passin pass:secret123
```

> 사전에 copy 명령어로 개인키 백업을 권장합니다.

#### Pass phrase 설정

반대로 다음 openssl 명령어로 AES256 으로 개인키 파일인 example.com.key 를 암호화해서 example.com.key.enc로 저장할 수 있습니다.

```bash
$ openssl rsa -aes256 -in example.com.key  -out example.com.key.enc
```

#### Pass phrase 설정 여부 확인

개인키가 암호화 되었는지 여부는 간단하게 사용하는 에디터로 개인키 파일을 열고 다음과 같이 Proc-Type 구분이 앞에 있는지 확인하면 됩니다.

```PEM
-----BEGIN RSA PRIVATE KEY-----
Proc-Type: 4,ENCRYPTED
DEK-Info: AES-256-CBC,9BF3ACA724D1187B19BDDB1585687E8A
```

- Proc-Type: 개인키가 암호화 되었음을 나타냅니다.
- DEK-Info: 암호화 알고리즘을 표시하며 AES 방식의 256 bit key 를 사용하며 CBC 운영 모드를 사용합니다.

암호화되지 않은 개인키를 바로 아래와 같은 구문으로 시작되므로 구분할 수 있습니다.

```PEM
-----BEGIN RSA PRIVATE KEY-----
MIIEogIBAAKCAQEAtHyN+f/vS6mb
```

#### Pass phrase 란?
일반적으로 어떤 시스템이나 자원에 접근하기 위한 패스워드(password)를 비밀번호로 번역합니다. 숫자만 암호로 사용했던 예전에는 맞지만 알파벳과 특수문자를 조합할 수 있는 지금은 비밀번호보다는 pass phrase라는 단어가 더 정확합니다.



### pkcs#8 방식의 개인키 해독

[Private-Key Information Syntax Specification](http://www.ietf.org/rfc/rfc5208.txt) 방식으로 암호화된 RSA PrivateKey 를 해독하려면 아래 명령을 사용합니다.

```bash
$ openssl pkcs8 -inform der -in pkcs8-pri.key -out rsa-pri.key
```

PKCS#8 파일은 binary 형식(DER) 과 text 형식(PEM) 이 있을 수 있으며 에디터로 열었을 때 -----BEGIN ENCRYPTED PRIVATE KEY----- 로 시작하는 경우 PEM 이며 깨지는 문자가 있을 경우 DER 입니다.

< PKCS8 PEM 예제 >

```PEM
-----BEGIN ENCRYPTED PRIVATE KEY-----
MIIFHzBJBgkqhkiG9w0BBQ0wPDAbBgkqhkiG9w0BBQwwDgQITJWLw/UHoM0CAggA
```

PEM 형식일 경우 *-inform der* 구문대신 *-inform pem* 을 사용하면 됩니다.

### pkcs#8 로 변환
개인키의 pass phrase 를 PKCS#8 형식으로 변환
```bash
$ openssl pkcs8 -topk8 -v2 aes128 -in aes-pri.pem -out aes-strong.key -outform der -passout pass:asdfasdf
```
- -topk8 : output PKCS8 file
- -v2 aes128 : PKCS#5 Ver 2.0 사용 및 aes128 사용

### HTTPS 연결의 인증서 디버깅
HTTPS 디버깅(httpd 의 SSLCertificateChainFile, SSLCACertificateFile 정상 설정 여부 확인등)이나 curl 등의 ca bundle 에 등록하기 위한 목적으로 서버가 사용하는 SSL 인증서를 추출할 경우 아래 명령어 사용

```bash
$ openssl s_client -debug -connect ssl.example.com:443
```



## CMS(PKCS#7, S/MIME)

**cms(Cryptographic Message Syntax)** 명령어는 S/MIME v3.1 mail 이나 PKCS#7 형식의 데이타를 처리하는 명령어로 주요 옵션은 다음과 같다.

- **-verify** : 전자서명 검증 수행
- **-in** : 검증할 전자서명 데이타 파일
- **-certfile** : 검증에 사용할 인증서 파일(전자 서명 데이타내에 인증서가 없을 경우 필요 - 생성시 -nocerts 으로 생성했을 경우)
- **-out** : 검증후 원본을 저장할 파일명
- **-content** : 검증에 사용할 원본 파일
- **-CAfile** : 인증서 발급 체인(CA 인증서 묶음. PEM 형식으로 연접해서 작성해 주면 되며 예제는 curl 에 포함된 ca인증서 번들 파일 참고 - /etc/pki/tls/certs/ca-bundle.crt)

### signeddata 검증
DER 로 인코딩된 cms signed-data 형식인 inputfile 을 검증하고 원본을 content 라는 파일로 저장
```bash
$ openssl cms -verify -in signedData.ber -inform DER  -out content
```

### 인증서 체인 지정
signed-data 안에 인증서 체인이 없을 경우 다음과 같은 에러가 발생한다
```bash
$ certificate verify error:cms_smime.c:304:Verify error:unable to get local issuer certificate
```

CA 인증서를 PEM 형식의 파일(Ex: ca-file) 으로 만든 후에  **-CAfile file** 옵션을 추가하면 검증시 사용할 CA 인증서를 지정해 줄 수 있다.

```bash
$ openssl cms -verify -in signedData.ber -inform DER  -out content -CAfile ca-file
```

### detached signeddata 검증
서명에 사용된 컨텐츠가 CMS Signed Data 내에 없거나 또는 있어도 강제로 외부 파일을 사용할 경우 -content file 옵션으로 파일을 지정하면 된다.
```bash
$ openssl cms -verify -in signedData.ber -inform DER  -out content -CAfile ca-file -content origFile
```

### signeddata 생성
PEM 형식으로 된 인증서와 개인키를 사용하여 전자 서명 데이타 생성
```bash
$ openssl cms  -sign -in contents.pdf -aes128 -nosmimecap -signer sign-cert.pem -inkey sign-key.pem -outform DER -nodetach  -out signed-data.ber
```
- **-sign** : 전자 서명 데이타 생성
- **-in** : 전자서명할 원본 데이타
- **-nodetach**:  전자서명 데이타에 원본 첨부
- **-nosmimecap**:
- **-noattr**: 전자서명 데이타에 어떤 signed attributes 도 포함하지 않음.

### envelop  data 생성
CMS envelopedData data 생성(대칭키를 생성후 원본을 암호화한 후에 상대방 인증서의 공개키로 대칭키를 암호한 데이타 형식)
```bash
$ openssl cms -encrypt -in contents.pdf -aes256 -recip sign-cert.pem -outform DER -out enveloped-data.ber
```

- **-encrypt**: encrypt 데이타 생성
- **-in** : 암호화할 원본 데이타
- **-aes256** : AES256 으로 암호화(-aes128, -seed, -camellia128 등의 알고리즘 사용 가능)
- **-recip**: 데이타를 수신할 상대방의 인증서(이 안에 있는 공개키로 암호화하므로 상대방 인증서를 정확히 넣어주어야 함)

### envelop  data 해독
```bash
$ openssl cms -decrypt -in enveloped-data.ber -inform der -inkey kmpri.pem
```
- -decrypt : 1123
- -in : 해독할 enveloped data 파일
- -inform : 파일의 포맷. 기본값은 PEM 이며 der 인코딩되었을 경우 der 추가
- -inkey: 복호화할 개인키

> -decrypt 시 -out 옵션이 통하지 않으므로 > 로 원본 파일을 저장해야 함
>
> openssl cms -decrypt -in enveloped-data.ber -inform der -inkey kmpri.pem > contents



## PKCS#12

### Check a Certificate Signing Request (CSR) - PKCS#10

```bash
$ openssl req -text -noout -verify -in CSR.csr
```

### pkcs12 생성
#### pkcs12 생성
```bash
$ openssl pkcs12 -export -in cert.pem -inkey pri-key.pem -out file.p12 -name "My Certificate"
```
- -export : PKCS#12 파일 생성
- -in : p12 에 들어갈 인증서
- -inkey: 포함시킬 개인키
- -out : 생성될 p12 파일명
- -name: riendlyName 에 들어갈 이름이며 Java 에서 KeyStore 로 접근시 alias 항목이 되므로 필수로 입력해야 한다. openssl 은 입력되지 않았을 경우 인증서의 해시값을 설정하는 것 같다.
- -descert :  p12 내 인증서 항목을 Triple DES 로 암호화(기본값 RC2-40) - 인증서는 공개하는 용도이므로 크게 의미 없는 옵션
- -des3 : encrypt private keys with triple DES (default)
- -aes128 : 개인키를 AES128 로 암호화(권장)
- -keypbe alg: specify private key PBE algorithm (default 3DES)

#### 기타 인증서를 포함하여 p12 생성
```bash
$ openssl pkcs12 -export -in cert.pem -inkey pri-key.pem -out file.p12 -name "My Certificate" -certfile othercerts.pem
```
- -certifile: 포함시킬 추가 인증서


### Check a PKCS#12 file (.pfx or .p12)
PKCS#12 정보 출력
```bash
$ openssl pkcs12 -info -in keyStore.p12
```

PKCS#12 내 인증서를 파일로 저장(-clcerts -nokeys)
```bash
$ openssl pkcs12 -in file.p12 -clcerts -nokeys -out file.crt
```

PKCS#12 내 개인키를 파일로 저장
```bash
$ openssl pkcs12 -in file.p12 -nocerts -out file.key
```

PKCS#12 내 개인키에 pass phrase 를 적용하지 않고 파일로 저장
```bash
$ openssl pkcs12 -in file.p12 -out file.pem -nodes
```



## OCSP

> 인증서는 PEM 형식이어야 함.

### OCSPRequest 생성

lesstif.cer 인증서를 검증하기 위한 OCSPRequest 를 생성하여 파일(ocsp-req.ber)로 저장. -issuer 옵션에는 인증기관 인증서를 입력
```bash
$ openssl ocsp -issuer myca.cer -cert lesstif.cer -reqout ocsp-req.ber 
```

### ocsp 로 인증서 검증

위에서 생성한 OCSPRequest 를 읽어서 -url 로 지정된 OCSP 서버에서 인증서 검증 요청
```bash
$ openssl ocsp -reqin ocsp-req.ber -text -url http://myocsp.server.com:8080/ocsp
```

검증할 인증서를 읽어서 검증 요청
```bash
$ openssl ocsp -issuer myca.cer -cert lesstif.cer  -text -url http://myocsp.server.com:8080/ocsp
```

### ocsp asn 파싱

-reqin 으로 지정된 파일로부터 OCSPRequest 형식의 데이타를 읽어서 출력

```bash
$ openssl ocsp -reqin ocsp-req.ber -text

OCSP Request Data:
    Version: 1 (0x0)
    Requestor List:
        Certificate ID:
          Hash Algorithm: sha1
          Issuer Name Hash: D530654290FA7C42771A7566518BB1420AB04CE0
          Issuer Key Hash: 4D5D560A0703DF83CAF3D56D8F19FC12AC90A28A
          Serial Number: 598E19F6
    Request Extensions:
        OCSP Nonce: 
            0410D8F5A2A55605873CBEBB043FCA79022A
```



## TSA(Time Stamp Authority)

### ts 생성
```bash
$ openssl ts -query -data mydata.txt -no_nonce -sha1 -out design1.tsq  
```

### print
```bash
$ openssl ts -query -in design1.tsq -text
```



## ASN1Parse

### UTF8String 생성
UTF8String 을 생성해서 utf8string.der 파일로 저장

```bash
$ openssl asn1parse -genstr "UTF8:헬로 World" -out utf8string.der
```

### UTF8String file 로 부터 파싱
생성된 ASN1 파일로 부터 파싱
```bash
$ openssl asn1parse -inform DER -in utf8string.der
```

### UTCTime 생성
```bash
$ openssl asn1parse -genstr "UTCTIME:970909034126Z" -out utctime.der
```

### UTCTime  파싱
```bash
$ openssl asn1parse -inform DER -in utctime.der
```



# OpenSSL로 ROOT CA 생성 및 SSL 인증서 발급

> 간단하게 CA 를 구성하려면 openssl 을 래핑한 easy rsa 를 사용하세요.



## 개요

웹서비스에 https 를 적용할 경우 SSL 인증서를 VeriSign 이나 Thawte, GeoTrust 등에서 인증서를 발급받아야 하지만 비용이 발생하므로 실제 운영 서버가 아니면 발급 받는데 부담이 될 수 있다.
이럴때 OpenSSL 을 이용하여 인증기관을 만들고 Self signed certificate 를 생성하고 SSL 인증서를 발급하는 법을 정리해 본다.
발급된 SSL 인증서는 apache httpd 등의 Web Server 에 설치하여 손쉽게 https 서비스를 제공할 수 있다.



### Self Signed Certificate ?

인증서(digital certificate)는 개인키 소유자의 공개키(public key)에 인증기관의 개인키로 전자서명한 데이타다.
모든 인증서는 발급기관(CA) 이 있어야 하나 최상위에 있는 인증기관(root ca)은 서명해줄 상위 인증기관이 없으므로 root ca의 개인키로 스스로의 인증서에 서명하여 최상위 인증기관 인증서를 만든다.
이렇게 스스로 서명한 ROOT CA 인증서를 Self Signed Certificate(SSC) 라고 부른다.
IE, FireFox, Chrome 등의 Web Browser 제작사는 VeriSign 이나 comodo 같은 유명 ROOT CA 들의 인증서를 신뢰하는 CA로 브라우저에 미리 탑재해 놓는다.
저런 기관에서 발급된 SSL 인증서를 사용해야 browser 에서는 해당 SSL 인증서를 신뢰할수 있는데 OpenSSL 로 만든 ROOT CA와 SSL 인증서는 Browser가 모르는 기관이 발급한 인증서이므로 보안 경고를 발생시킬 것이나 테스트 사용에는 지장이 없다.
ROOT CA 인증서를 Browser에 추가하여 보안 경고를 발생시키지 않으려면 Browser 에 SSL 인증서 발급기관 추가하기 를 참고하자.



### Certificate Signing Request?

공개키 기반(PKI)은 private key(개인키)와 public key(공개키)로 이루어져 있다.
인증서라고 하는 것은 내 공개키가 맞다고 인증기관(CA)이 전자서명하여 주는 것이며 나와 보안 통신을 하려는 당사자는 내 인증서를 구해서 그 안에 있는 공개키를 이용하여 보안 통신을 할 수 있다.
CSR(Certificate Signing Request) 은 인증기관에 인증서 발급 요청을 하는 특별한 ASN.1 형식의 파일이며(PKCS#10 - RFC2986)  그 안에는 내 공개키 정보와 사용하는 알고리즘 정보등이 들어 있다.
개인키는 외부에 유출되면 안 되므로 저런 특별한 형식의 파일을 만들어서 인증기관에 전달하여 인증서를 발급 받는다.
SSL 인증서 발급시 CSR 생성은 Web Server 에서 이루어지는데 Web Server 마다 방식이 상이하여 사용자들이 CSR 생성등을 어려워하니 인증서 발급 대행 기관에서 개인키까지 생성해서 보내주고는 한다.



## ROOT CA 인증서 생성Link to ROOT CA 인증서 생성

openssl 로 root ca 의 개인키와 인증서를 만들어 보자

1. CA 가 사용할 RSA  key pair(public, private key) 생성

   ```bash
   $ openssl genrsa -aes256 -out /etc/pki/tls/private/lesstif-rootca.key 2048
   ```
   > 개인키 분실에 대비해 AES 256bit 로 암호화한다. AES 이므로 암호(pass phrase)를 분실하면 개인키를 얻을수 없으니 꼭 기억해야 한다.

2. 개인키 권한 설정

   > 보안 경고
   > 개인키의 유출 방지를 위해 group 과 other의 permission 을 모두 제거한다.
   > chmod 600  /etc/pki/tls/private/lesstif-rootca.key

3. CSR(Certificate Signing Request) 생성을 위한 rootca_openssl.conf 로 저장

   ```properties
   [ req ]
   default_bits            = 2048
   default_md              = sha1
   default_keyfile         = lesstif-rootca.key
   distinguished_name      = req_distinguished_name
   extensions             = v3_ca
   req_extensions = v3_ca
    
   [ v3_ca ]
   basicConstraints       = critical, CA:TRUE, pathlen:0
   subjectKeyIdentifier   = hash
   ##authorityKeyIdentifier = keyid:always, issuer:always
   keyUsage               = keyCertSign, cRLSign
   nsCertType             = sslCA, emailCA, objCA
   [req_distinguished_name ]
   countryName                     = Country Name (2 letter code)
   countryName_default             = KR
   countryName_min                 = 2
   countryName_max                 = 2
   
   # 회사명 입력
   organizationName              = Organization Name (eg, company)
   organizationName_default      = lesstif Inc.
    
   # 부서 입력
   #organizationalUnitName          = Organizational Unit Name (eg, section)
   #organizationalUnitName_default  = Condor Project
    
   # SSL 서비스할 domain 명 입력
   commonName                      = Common Name (eg, your name or your server's hostname)
   commonName_default             = lesstif's Self Signed CA
   commonName_max                  = 64 
   ```

   ```bash
   $ openssl req -new -key /etc/pki/tls/private/lesstif-rootca.key -out /etc/pki/tls/certs/lesstif-rootca.csr -config rootca_openssl.conf
   ```
   아래는 OpenSSL 의 프롬프트
   ```bash
   You are about to be asked to enter information that will be incorporated
   into your certificate request.
   What you are about to enter is what is called a Distinguished Name or a DN.
   There are quite a few fields but you can leave some blank
   For some fields there will be a default value,
   If you enter '.', the field will be left blank.
   Country Name (2 letter code) [KR]:
   Organization Name (eg, company) [lesstif Inc]:lesstif Inc.
   Common Name (eg, your name or your servers hostname) [lesstif's Self Signed CA]:lesstif's Self Signed C
   ```


4. 10년짜리 self-signed 인증서 생성

   > -extensions v3_ca 옵션을 추가해야 한다.
   
   ```bash
   $ openssl x509 -req \
-days 3650 \
-extensions v3_ca \
-set_serial 1 \
-in /etc/pki/tls/certs/lesstif-rootca.csr \
-signkey /etc/pki/tls/private/lesstif-rootca.key \
-out /etc/pki/tls/certs/lesstif-rootca.crt \
-extfile rootca_openssl.conf
   ```
   > 서명에 사용할 해시 알고리즘을 변경하려면 -sha256, -sha384, -sha512 처럼 해시를 지정하는 옵션을 전달해 준다.
   > 기본값은 -sha256 이며 openssl 1.0.2 이상이 필요

5. 제대로 생성되었는지 확인을 위해 인증서의 정보를 출력해 본다.

   ```bash
   $ openssl x509 -text -in /etc/pki/tls/certs/lesstif-rootca.crt
   ```

   
## SSL 인증서 발급
위에서 생성한 root ca 서명키로 SSL 인증서를 발급해 보자



#### 키 쌍 생성

1. SSL 호스트에서 사용할 RSA  key pair(public, private key) 생성

   ```bash
   $ openssl genrsa -aes256 -out /etc/pki/tls/private/lesstif.com.key 2048
   ```

   

2. Remove Passphrase from key

   > 개인키를 보호하기 위해 Key-Derived Function 으로 개인키 자체가 암호화되어 있다. 인터넷 뱅킹등에 사용되는 개인용 인증서는 당연히 저렇게 보호되어야 하지만 SSL 에 사용하려는 키가 암호가 걸려있으면 httpd 구동때마다 pass phrase 를 입력해야 하므로 암호를 제거한다.

   ```bash
   $ cp  /etc/pki/tls/private/lesstif.com.key  /etc/pki/tls/private/lesstif.com.key.enc
   $ openssl rsa -in  /etc/pki/tls/private/lesstif.com.key.enc -out  /etc/pki/tls/private/lesstif.com.key
   ```

   > 보안 경고
   >
   > 개인키의 유출 방지를 위해 group 과 other의 permission 을 모두 제거한다.
   >
   > chmod 600  /etc/pki/tls/private/lesstif.com.key*



#### CSR 생성

CSR(Certificate Signing Request) 생성을 위한 openssl config 파일을 host_openssl.conf 로 저장

```properties
[ req ]
default_bits            = 2048
default_md              = sha1
default_keyfile         = lesstif-rootca.key
distinguished_name      = req_distinguished_name
extensions             = v3_user
## 인증서 요청시에도 extension 이 들어가면 authorityKeyIdentifier 를 찾지 못해 에러가 나므로 막아둔다.
## req_extensions = v3_user

[ v3_user ]
# Extensions to add to a certificate request
basicConstraints = CA:FALSE
authorityKeyIdentifier = keyid,issuer
subjectKeyIdentifier = hash
keyUsage = nonRepudiation, digitalSignature, keyEncipherment
## SSL 용 확장키 필드
extendedKeyUsage = serverAuth,clientAuth
subjectAltName          = @alt_names
[ alt_names]
## Subject AltName의 DNSName field에 SSL Host 의 도메인 이름을 적어준다.
## 멀티 도메인일 경우 *.lesstif.com 처럼 쓸 수 있다.
DNS.1   = www.lesstif.com
DNS.2   = lesstif.com
DNS.3   = *.lesstif.com

[req_distinguished_name ]
countryName                     = Country Name (2 letter code)
countryName_default             = KR
countryName_min                 = 2
countryName_max                 = 2

# 회사명 입력
organizationName              = Organization Name (eg, company)
organizationName_default      = lesstif Inc.
 
# 부서 입력
organizationalUnitName          = Organizational Unit Name (eg, section)
organizationalUnitName_default  = lesstif SSL Project
 
# SSL 서비스할 domain 명 입력
commonName                      = Common Name (eg, your name or your server's hostname)
commonName_default             = lesstif.com
commonName_max                  = 64
```

인증서 발급 요청(CSR) 생성 

```bash
$ openssl req -new -key /etc/pki/tls/private/lesstif.com.key -out /etc/pki/tls/certs/lesstif.com.csr -config host_openssl.conf
```

```bash
You are about to be asked to enter information that will be incorporated
into your certificate request.
What you are about to enter is what is called a Distinguished Name or a DN.
There are quite a few fields but you can leave some blank
For some fields there will be a default value,
If you enter '.', the field will be left blank.

Country Name (2 letter code) [KR]:
Organization Name (eg, company) [lesstif Inc]:lesstif's Self Signed CA
Common Name (eg, your name or your servers hostname) [lesstif.com]:*.lesstif.com
```

5년짜리 lesstif.com 용 SSL 인증서 발급 (서명시 ROOT CA 개인키로 서명)

```bash
$ openssl x509 -req -days 1825 -extensions v3_user -in /etc/pki/tls/certs/lesstif.com.csr \
-CA /etc/pki/tls/certs/lesstif-rootca.crt -CAcreateserial \
-CAkey  /etc/pki/tls/private/lesstif-rootca.key \
-out /etc/pki/tls/certs/lesstif.com.crt  -extfile host_openssl.conf
```

제대로 생성되었는지 확인을 위해 인증서의 정보를 출력해 본다.

```bash
$ openssl x509 -text -in /etc/pki/tls/certs/lesstif.com.crt
```



# easy rsa 로 인증 기관(CA;Certificate Authority) 구성하고 인증서 발급하기

Easy RSA 는 OpenVPN 프로젝트에서 사용하기 위해 만든 하위 프로젝트로 인증 기관 구축(CA; Certificate Authority) 유틸리티입니다. 

OpenSSL 로 CA 를 구성하려면 복잡한 여러 설정이 필요하지만 easy rsa 를 사용하면 간단하게 CA 를 구성하고 인증서를 발급할 수 있습니다.



## 설치

easy rsa 는 OpenSSL 을 쉽게 사용하기 위한 script 이므로 OpenSSL 이 설치되어 있어야 합니다. 일반적인 리눅스 배포판이라면 기본적으로 openssl 이 설치되어 있지만 혹시 없는 경우 다음 명령어로 설치하면 됩니다.

```bash
$ sudo yum install openssl
```
```bash
$ sudo apt install openssl
```

OpenSSL 이 준비되었으면 [github 의 easy rsa 프로젝트에 연결](https://github.com/OpenVPN/easy-rsa/releases) 한 후에 Release 를 클릭합니다.

패키지 목록에서 OS 에 맞는 패키지를 다운로드 받고 압축을 풀면 되며 또는 커맨드에서 다음 명령을 실행해도 됩니다.

```bash
$ wget https://github.com/OpenVPN/easy-rsa/releases/download/v3.0.8/EasyRSA-3.0.8.tgz
```



## 사용

### CA 초기화

먼저 init-pki 명령으로 PKI 초기화를 해줍니다. easy-rsa 하위에 pki 폴더가 생기고 이 안에 새로운 인증서가 생성됩니다.

```bash
$./easyrsa init-pki

init-pki complete; you may now create a CA or requests.
Your newly created PKI dir is: /home/lesstif/Downloads/EasyRSA-3.0.8/pki
```
build-ca 명령으로 CA 를 구성합니다. 먼저 CA 개인키의 pass phrase 를 묻는데 잊어버리면 CA 를 새로 만들어야 하므로 입력해 주고 잊지 않게 기억해 둡니다. 
```bash
$ ./easyrsa build-ca

Using SSL: openssl OpenSSL 1.1.1j  FIPS 16 Feb 2021

Enter New CA Key Passphrase: 
Re-Enter New CA Key Passphrase: 
Generating RSA private key, 2048 bit long modulus (2 primes)
```
pass phrase 입력이 끝나면 2048 비트의 RSA 키를 생성합니다. 
```bash
e is 65537 (0x010001)
You are about to be asked to enter information that will be incorporated
into your certificate request.
What you are about to enter is what is called a Distinguished Name or a DN.
There are quite a few fields but you can leave some blank
For some fields there will be a default value,
If you enter '.', the field will be left blank.

Common Name (eg: your user, host, or server name) [Easy-RSA CA]:lesstif CA      
```
이제 CA 인증서의 common name 을 설정해야 하는데 서버 이름이나 host 이름등을 입력해 주면 됩니다. 저는 "lesstif CA" 를 입력했습니다.

입력이 끝나면 CA 개인키와 인증서가 pki 폴더밑에 생성되고 인증서 발급 준비가 완료됩니다.

```bash
CA creation complete and you may now import and sign cert requests.
Your new CA certificate file for publishing is at:
/home/lesstif/Downloads/EasyRSA-3.0.8/pki/ca.crt
```



### 인증서 서명  요청 생성(Client)

인증서가 필요한 client 에서는 인증서 서명 요청(CSR; Certificate signing request)을 생성한후에 CA 에 전달해 주면 CA 가 개인키(private key) 로 전자서명한 것이 인증서입니다.

요청 측에서도 공개 키쌍 생성을 해야 하므로 init-pki 로 초기화를 해줍니다.
```bash
$ ./easyrsa init-pki
```
이제 gen-req 명령뒤에 서버 이름을 옵션으로 전달해서 CSR 을 생성합니다. 

```bash
$ ./easyrsa gen-req LesstifWebServer

Using SSL: openssl OpenSSL 1.1.1j  FIPS 16 Feb 2021
Generating a RSA private key
......+++++
...............................+++++
writing new private key to '/home/user/EasyRSA-3.0.8/pki/easy-rsa-34188.6cwDro/tmp.Gp67YQ'
Enter PEM pass phrase:
Verifying - Enter PEM pass phrase:
```
pass phrase 를 입력하면 개인 키와 공개 키가 생성되고 발급 요청자의 common name 을 입력하라고 하며 자동으로 gen-req 뒤에 준 옵션으로 설정됩니다.

보통 서버 이름을 입력하며 저는 lesstifWebServer 로 설정했습니다.

```bash

You are about to be asked to enter information that will be incorporated
into your certificate request.
What you are about to enter is what is called a Distinguished Name or a DN.
There are quite a few fields but you can leave some blank
For some fields there will be a default value,
If you enter '.', the field will be left blank.

Common Name (eg: your user, host, or server name) [LesstifWebServer]: LesstifWebServer
```

키 쌍과 req가 생성되었다는 메시지가 출력되는데 req 는 CSR 로 인증기관(CA) 에 전달해 주면 CA 가 CSR 을 받아서 요청자의 공개 키를 꺼내고 여러 정보를 조합한 후에 CA의 개인키로 전자서명해 주는데 이게 바로 인증서입니다.

```bash
Keypair and certificate request completed. Your files are:
req: /home/user1/EasyRSA-3.0.8/pki/reqs/LesstifWebServer.req
key: /home/user1/EasyRSA-3.0.8/pki/private/LesstifWebServer.key
```
그러므로 생성된 req 인 /home/user1/EasyRSA-3.0.8/pki/reqs/LesstifWebServer.req 를 인증기관에 전달해 주면 됩니다



### 인증서 발급(CA)

CA 는 req 파일을 받아서 import-req 명령으로 인증서를 발급해 주면 됩니다. 이때 Client 가 생성한 req 파일의 절대 경로를 입력해야 합니다.

```bash
$./easyrsa import-req /home/user1/EasyRSA-3.0.8/pki/reqs/LesstifWebServer.req LesstifWebServer

Using SSL: openssl OpenSSL 1.1.1j  FIPS 16 Feb 2021

The request has been successfully imported with a short name of: LesstifWebServer
You may now use this name to perform signing operations on this request.
```

성공적으로 import 가 되었다는 메시지가 표시되면 이제 CA 개인키로 서명해서 인증서를 발급해 주면 됩니다.

명령은 sign-req 이며 뒤에 옵션으로 client 와 위에서 전달받은 CSR 내 common name을 입력합니다.

```bash
$ ./easyrsa sign-req client LesstifWebServer
Using SSL: openssl OpenSSL 1.1.1j  FIPS 16 Feb 2021


You are about to sign the following certificate.
Please check over the details shown below for accuracy. Note that this request
has not been cryptographically verified. Please be sure it came from a trusted
source or that you have verified the request checksum with the sender.

Request subject, to be signed as a client certificate for 825 days:

subject=
    commonName                = LesstifWebServer


Type the word 'yes' to continue, or any other input to abort.
  Confirm request details: 
```

  commonName 확인 창이 뜨면 yes 를 입력합니다.
  ```bash
Using configuration from /home/lesstif/Downloads/EasyRSA-3.0.8/pki/easy-rsa-36037.BzVS1I/tmp.egTrHW
Enter pass phrase for /home/lesstif/Downloads/EasyRSA-3.0.8/pki/private/ca.key:
  ```
CA 개인 키의 pass phrase 를 묻는 창에 제대로 입력이 되면 pki/issued 폴더에 common name 이름이 붙은 인증서가 발급됩니다. 발급된 인증서를 CSR 을 보낸 client 에 전달해 주면 됩니다. 
```bash
Check that the request matches the signature
Signature ok
The Subject's Distinguished Name is as follows
commonName            :ASN.1 12:'LesstifWebServer'
Certificate is to be certified until Jun 17 06:49:41 2023 GMT (825 days)

Write out database with 1 new entries
Data Base Updated

Certificate created at: /home/lesstif/Downloads/EasyRSA-3.0.8/pki/issued/LesstifWebServer.crt
```
