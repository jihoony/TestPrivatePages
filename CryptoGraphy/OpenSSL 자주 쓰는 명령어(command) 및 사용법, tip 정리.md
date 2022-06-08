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

- 암호(pass phrase)는 asdfasdf 이며 입력창을 띄우지 않고 커맨드에서 바로 설정(`-passout` 옵션)
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
`/etc/pki/tls/private/examplelocalhost.key.enc` 라는 암호화된 개인키가 있을 경우 다음 openssl 명령어로 해독할 수 있습니다.

```bash
$ openssl rsa -in /etc/pki/tls/private/example.com.key.enc -out /etc/pki/tls/private/example.com.key

Enter pass phrase for /etc/pki/tls/private/example.com.key.enc:
```

Enter pass phrase에 개인키에 설정한 암호를 입력해 주면 `-out`에 지정한 경로에 복호화된 개인키가 저장됩니다.

보안때문에 권장하지는 않지만 `-passin pass:mypwd`옵션으로 명령행에서 바로 pass phrase를 입력할 수 있습니다. 

```bash
$ openssl rsa -in /etc/pki/tls/private/example.com.key.enc -out /etc/pki/tls/private/example.com.key -passin pass:secret123
```

> 사전에 copy 명령어로 개인키 백업을 권장합니다.

#### Pass phrase 설정

반대로 다음 openssl 명령어로 AES256 으로 개인키 파일인 `example.com.key` 를 암호화해서 `example.com.key.enc`로 저장할 수 있습니다.

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

PKCS#8 파일은 binary 형식(DER) 과 text 형식(PEM) 이 있을 수 있으며 에디터로 열었을 때 `-----BEGIN ENCRYPTED PRIVATE KEY-----` 로 시작하는 경우 PEM 이며 깨지는 문자가 있을 경우 DER 입니다.

**PKCS8 PEM 예제**

```PEM
-----BEGIN ENCRYPTED PRIVATE KEY-----
MIIFHzBJBgkqhkiG9w0BBQ0wPDAbBgkqhkiG9w0BBQwwDgQITJWLw/UHoM0CAggA
```

PEM 형식일 경우 `-inform der` 구문대신 `-inform pem` 을 사용하면 됩니다.

### pkcs#8 로 변환
개인키의 pass phrase 를 PKCS#8 형식으로 변환
```bash
$ openssl pkcs8 -topk8 -v2 aes128 -in aes-pri.pem -out aes-strong.key -outform der -passout pass:asdfasdf
```
- `-topk8` : output PKCS8 file
- `-v2 aes128` : PKCS#5 Ver 2.0 사용 및 aes128 사용

### HTTPS 연결의 인증서 디버깅
HTTPS 디버깅(httpd 의 SSLCertificateChainFile, SSLCACertificateFile 정상 설정 여부 확인 등)이나 curl 등의 ca bundle 에 등록하기 위한 목적으로 서버가 사용하는 SSL 인증서를 추출할 경우 아래 명령어 사용

```bash
$ openssl s_client -debug -connect ssl.example.com:443
```



## CMS(PKCS#7, S/MIME)

**cms(Cryptographic Message Syntax)** 명령어는 S/MIME v3.1 mail 이나 PKCS#7 형식의 데이타를 처리하는 명령어로 주요 옵션은 다음과 같다.

- `-verify` : 전자서명 검증 수행
- `-in` : 검증할 전자서명 데이타 파일
- `-certfile` : 검증에 사용할 인증서 파일(전자 서명 데이타내에 인증서가 없을 경우 필요 - 생성시 -nocerts 으로 생성했을 경우)
- `-out` : 검증후 원본을 저장할 파일명
- `-content` : 검증에 사용할 원본 파일
- `-CAfile` : 인증서 발급 체인(CA 인증서 묶음. PEM 형식으로 연접해서 작성해 주면 되며 예제는 curl 에 포함된 ca인증서 번들 파일 참고 - `/etc/pki/tls/certs/ca-bundle.crt`)

### signed data 검증
DER 로 인코딩된 cms signed-data 형식인 inputfile 을 검증하고 원본을 content 라는 파일로 저장
```bash
$ openssl cms -verify -in signedData.ber -inform DER  -out content
```

### 인증서 체인 지정
signed-data 안에 인증서 체인이 없을 경우 다음과 같은 에러가 발생한다
```bash
$ certificate verify error:cms_smime.c:304:Verify error:unable to get local issuer certificate
```

CA 인증서를 PEM 형식의 파일(Ex: ca-file) 으로 만든 후에  `-CAfile file` 옵션을 추가하면 검증시 사용할 CA 인증서를 지정해 줄 수 있다.

```bash
$ openssl cms -verify -in signedData.ber -inform DER  -out content -CAfile ca-file
```

### detached signeddata 검증
서명에 사용된 컨텐츠가 CMS Signed Data 내에 없거나 또는 있어도 강제로 외부 파일을 사용할 경우 `-content file` 옵션으로 파일을 지정하면 된다.
```bash
$ openssl cms -verify -in signedData.ber -inform DER  -out content -CAfile ca-file -content origFile
```

### signeddata 생성
PEM 형식으로 된 인증서와 개인키를 사용하여 전자 서명 데이타 생성
```bash
$ openssl cms  -sign -in contents.pdf -aes128 -nosmimecap -signer sign-cert.pem -inkey sign-key.pem -outform DER -nodetach  -out signed-data.ber
```
- `-sign` : 전자 서명 데이타 생성
- `-in` : 전자서명할 원본 데이타
- `-nodetach`:  전자서명 데이타에 원본 첨부
- `-nosmimecap`:
- `-noattr`: 전자서명 데이타에 어떤 signed attributes 도 포함하지 않음.

### envelop  data 생성
CMS envelopedData data 생성(대칭키를 생성후 원본을 암호화한 후에 상대방 인증서의 공개키로 대칭키를 암호한 데이타 형식)
```bash
$ openssl cms -encrypt -in contents.pdf -aes256 -recip sign-cert.pem -outform DER -out enveloped-data.ber
```

- `-encrypt`: encrypt 데이타 생성
- `-in` : 암호화할 원본 데이타
- `-aes256` : AES256 으로 암호화(`-aes128`, `-seed`, `-camellia128` 등의 알고리즘 사용 가능)
- `-recip`: 데이타를 수신할 상대방의 인증서(이 안에 있는 공개키로 암호화하므로 상대방 인증서를 정확히 넣어주어야 함)

### envelop  data 해독
```bash
$ openssl cms -decrypt -in enveloped-data.ber -inform der -inkey kmpri.pem
```
- `-decrypt` : 1123
- `-in` : 해독할 enveloped data 파일
- `-inform` : 파일의 포맷. 기본값은 PEM 이며 der 인코딩되었을 경우 der 추가
- `-inkey` : 복호화할 개인키

> `-decrypt` 시 `-out` 옵션이 통하지 않으므로 `>` 로 원본 파일을 저장해야 함
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
- `-export` : PKCS#12 파일 생성
- `-in` : p12 에 들어갈 인증서
- `-inkey` : 포함시킬 개인키
- `-out` : 생성될 p12 파일명
- `-name` : FriendlyName 에 들어갈 이름이며 Java 에서 KeyStore 로 접근시 alias 항목이 되므로 필수로 입력해야 한다. openssl 은 입력되지 않았을 경우 인증서의 해시값을 설정하는 것 같다.
- `-descert` :  p12 내 인증서 항목을 Triple DES 로 암호화(기본값 RC2-40) - 인증서는 공개하는 용도이므로 크게 의미 없는 옵션
- `-des3` : encrypt private keys with triple DES (default)
- `-aes128` : 개인키를 AES128 로 암호화(권장)
- `-keypbe alg`: specify private key PBE algorithm (default 3DES)

#### 기타 인증서를 포함하여 p12 생성
```bash
$ openssl pkcs12 -export -in cert.pem -inkey pri-key.pem -out file.p12 -name "My Certificate" -certfile othercerts.pem
```
- `-certifile` : 포함시킬 추가 인증서


### Check a PKCS#12 file (.pfx or .p12)
PKCS#12 정보 출력
```bash
$ openssl pkcs12 -info -in keyStore.p12
```

PKCS#12 내 인증서를 파일로 저장(`-clcerts -nokeys`)
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

lesstif.cer 인증서를 검증하기 위한 OCSPRequest 를 생성하여 파일(ocsp-req.ber)로 저장. `-issuer` 옵션에는 인증기관 인증서를 입력
```bash
$ openssl ocsp -issuer myca.cer -cert lesstif.cer -reqout ocsp-req.ber 
```

### ocsp 로 인증서 검증

위에서 생성한 OCSPRequest 를 읽어서 `-url` 로 지정된 OCSP 서버에서 인증서 검증 요청
```bash
$ openssl ocsp -reqin ocsp-req.ber -text -url http://myocsp.server.com:8080/ocsp
```

검증할 인증서를 읽어서 검증 요청
```bash
$ openssl ocsp -issuer myca.cer -cert lesstif.cer  -text -url http://myocsp.server.com:8080/ocsp
```

### ocsp asn 파싱

`-reqin` 으로 지정된 파일로부터 OCSPRequest 형식의 데이타를 읽어서 출력

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
