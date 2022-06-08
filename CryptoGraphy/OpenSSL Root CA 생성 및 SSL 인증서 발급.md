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
   > 서명에 사용할 해시 알고리즘을 변경하려면 `-sha256`, `-sha384`, `-sha512` 처럼 해시를 지정하는 옵션을 전달해 준다.
   > 기본값은 `-sha256` 이며 openssl 1.0.2 이상이 필요

5. 제대로 생성되었는지 확인을 위해 인증서의 정보를 출력해 본다.

   ```bash
   $ openssl x509 -text -in /etc/pki/tls/certs/lesstif-rootca.crt
   ```

   
## SSL 인증서 발급
위에서 생성한 root ca 서명키로 SSL 인증서를 발급해 보자



### 키 쌍 생성

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



### CSR 생성

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
