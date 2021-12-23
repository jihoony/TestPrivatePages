# Java Keytool 사용법


# 개요

Java 는 KetStore라는 인터페이스를 통해 Encryption/Decryption 및 Digital Signature에 사용되는 Private Key, Public Key 와 Certificate를 추상화하여 제공하고 있다.

KeyStore를 구현한 Provider에 따라 실제 개인키가 저장되는 곳이 로컬 디스크이든 HSM 같은 별도의 하드웨어이든 아니면 Windows의 CertStore나 OSX의 KeyChain 이든 상관없이 사용자는 소스 코드 수정없이 키와 인증서를 가져올 수 있고 이를 이용하여 데이타 암복호화, 전자서명을 수행할 수 있다.

keytool은 Keystore 기반으로 인증서와 키를 관리할 수 있는 커맨드 방식의 유틸리티로 JDK에 포함되어 있다. 커맨드 방식의 openssl과 비슷한 용도로 사용할 수 있는 프로그램이라 보면 될 것 같다.



# 용법

옵션 없이 keytool을 실행하면 다음과 같이 메인 command를 표시한다.

```bash
$ keytool
Key and Certificate Management Tool

Commands:

 -certreq            Generates a certificate request
 -changealias        Changes an entry's alias
 -delete             Deletes an entry
 -exportcert         Exports certificate
 -genkeypair         Generates a key pair
 -genseckey          Generates a secret key
 -gencert            Generates certificate from a certificate request
 -importcert         Imports a certificate or a certificate chain
 -importpass         Imports a password
 -importkeystore     Imports one or all entries from another keystore
 -keypasswd          Changes the key password of an entry
 -list               Lists entries in a keystore
 -printcert          Prints the content of a certificate
 -printcertreq       Prints the content of a certificate request
 -printcrl           Prints the content of a CRL file
 -storepasswd        Changes the store password of a keystore

Use "keytool -command_name -help" for usage of command_name 
```

커맨드마다 하위 옵션이 있으며 커맨드의 상세 설명을 보려면 -help 옵션을 커맨드 뒤에 추가하면 된다.



다음은 공개키와 개인키 두 개의 키쌍을 생성하는 -genkeypair 명령의 옵션을 표시하는 예제이다.

```bash
$ keytool -genkeypair -help
keytool -genkeypair [OPTION]...

Generates a key pair

Options:

 -alias <alias>                  alias name of the entry to process
 -keyalg <keyalg>                key algorithm name
 -keysize <keysize>              key bit size
 -sigalg <sigalg>                signature algorithm name
 -destalias <destalias>          destination alias
 -dname <dname>                  distinguished name
 -startdate <startdate>          certificate validity start date/time
 -ext <value>                    X.509 extension
 -validity <valDays>             validity number of days
 -keypass <arg>                  key password
 -keystore <keystore>            keystore name
 -storepass <arg>                keystore password
 -storetype <storetype>          keystore type
 -providername <providername>    provider name
 -providerclass <providerclass>  provider class name
 -providerarg <arg>              provider argument
 -providerpath <pathlist>        provider classpath
 -v                              verbose output
 -protected                      password through protected mechanism

Use "keytool -help" for all available commands
```





# KeyStore Type

keytool을 사용할 경우 명시적으로 *-keystore* 옵션으로 키스토어 파일의 경로를 지적하지 않으면 기본적으로 사용자의 홈디렉토리에서 .keystore 파일을 찾게 된다.

keystore는 여러 가지 타입을 지원하는데 기본적으로는 JKS(Java KeyStore) 라는 타입으로 처리된다.

다음은 *jks_keystore*라는 파일 이름으로 JKS 방식의 키스토어를 생성하는 명령어로 JKS는 기본 옵션이므로 *-storetype jks*은 생략 가능하다

```bash
$ keytool -genkeypair -keystore jks_keystore -storetype jks
```



인증서와 개인키를 저장하는 또 다른 표준인 PKCS12 타입을 사용할 경우 다음과 같이 *-storetype* 옵션을 추가하면 된다.

```bash
$ keytool -genkeypair -keystore pkcs12_keystore -storetype pkcs12
```



Windows와 Mac OSX는 OS에 개인키와 인증서를 저장하는 공간이 따로 있는데 keytool로 접근이 가능하다. **Windows-MY**는 사용자의 인증서와 개인키를 저장하는 공간이며 **Windows-ROOT**는 신뢰하는 루트 인증서를 저장하는 공간이다. OSX의 키체인(KeyChain) 에 접근시 **KeychainStore**를 타입으로 지정하면 된다. 그외 **Bouncy Castle**를 **JCE Provider**로 사용할 경우 **BKS** 타입을 사용할 수 있다.



# KeyPair 내 Object 출력

## 인증서 목록 출력

다음 명령으로 KeyStore 내 인증서 목록을 출력할 수 있다.

```bash
$ keytool -list -keystore my-keystore.jks
```

JRE에 포함되어 있는 기본 인증기관(ca) 인증서 파일은 jre/lib/security/cacerts/cacerts 파일에 존재한다. 다음 명령은 기본 ca 목록을 출력한다.



```bash
$ keytool -list -keystore $JAVA_HOME/jre/lib/security/cacerts
```



# 인증서 Import

*-importcert* 명령어로 인증서를 임포트 할 수 있다. 만약 인증기관 인증서라면 *-trustcacerts* 옵션을 추가한다.

```bash
$ keytool -importcert -keystore my-keystore.jks -storepass changeit -trustcacerts -alias rootca -file "rootca.der"
```



# Private Key import

keytool은 외부에서 생성된 private key 를 keystore 에 import 하는 방법을 제공하지 않는다. 한 가지 방법은 JKD 6 이상부터 PKCS#12으로 된 인증서와 개인키를 keystore 에 import 하는게 가능하므로 openssl로 pkcs#12를 만들고 pics#12를 KeyStore로 임포트하면 된다.



1. 이미 외부에서 개인키(mycert.key)와 인증서(mycert.crt)는 생성 되었다고 가정한다.

2. 인증서와 개인키가 DER 방식으로 encoding 되어있으면 openssl에서 pkcs12로 변환하지 못하니 PEM 형식으로 변환해야 한다. 에디터로 열어서 다음과 같이 텍스트로 표시되면 PEM 이고 바이너리면 DER 이므로 변환해야 한다.

   < 인증서 예시 >

   ```pem
   -----BEGIN CERTIFICATE-----
   MIIFeDCCBGCgAwIBAgIBGTANBgkqhkiG9w0BAQsFADBpMQswCQYDVQQGEwJLUjEN
   MAsGA1UECgwES0lTQTEuMCwGA1UECwwlS29yZWEgQ2VydGlmaWNhdGlvbiBBdXRo
   b3JpdHkgQ2VudHJhbDEbMBkGA1UEAwwSS2lzYSBUZXN0IFJvb3RDQSA1MB4XDTEx
   ```

   < 개인키 예시 >

   ```PEM
   -----BEGIN RSA PRIVATE KEY-----
   MIIEpAIBAAKCAQEA6KLO6jGTx1NMZsN3QJh/YCrVgmZsHlaD8sSFIFUcc5wH0gy6
   oKggrOD7gE9CPRb3MQG53hx29c92ih/cFKrN1IoeSPj0ftxZPhKUczfot2CAH3GX
   BWh0OYeuCIv088aKhSMJJLP9ZruC6Zhb01HYJiWdpOMX53fSMRJZYgjlIHZMi76u
   ofLnvuP2Ry8VJntw2RFJeei0Z+6+YpHJlQbuzIrUjzNe6aCZMerbCtMoO7GnNN0y
   lYl2P+D9aDhypB4bH2tGQ4mYSSdCXslO46C9zeesStZkSanXbZCvP4dYmr2o7STe
   ```

   ```bash
   ## 인증서를 PEM 으로 변환
   $ openssl x509 -inform der -in mycert.der -out mycert.crt
   ## 개인키 변환
   $ openssl rsa -inform der -in mycert.key.der -out mycert.key
   ```

   

3. openssl로 PKCS12 생성

   ```bash
   $ openssl pkcs12 -export in mycert.crt -inkey mycert.key -out mykeystore.p12 -name "some alias"
   ```

   

4. Enter Export Password: 에 pkcs12 암호 입력(예: qwert123)

5. keytool로 PKCS12를 KeyStore로 변환

   ```bash
   $ keytool -importkeystore -deststorepass changeit -destkeypass changit -destkeystore my-keystore.jks -srckeystore mykeystore.p12 -srcstoretype PKCS12 -srcstorepass qwert123 -alias "some alias"
   ```

   



# alias 변경

```bash
$ keytool -changealias -keystore MY_KEYSTORE_2.jks -alias OLD_ALIAS -destalias NEW_ALIAS
```



# 암호 변경

Key Store에 저장된 개인키를 보호하기 위해 key store 자체에 대해서 암호를 걸 수 있고 특정 alias에 저장된 개인키에도 암호를 걸 수 있다.

## Keystore 암호 변경

Jus_keystore 라는 키스토어 파일의 암호를 변경한다.

```bash
$ keytool -storepasswd -keystore jks_keystore

Enter keystore password:
New keystore password:
Re-enter new keystore password:
```



## Key 암호 변경

jks_keystore 라는 키스토어 파일내의 mykey 라는 alias 를 가진 개인키의 암호를 변경한다.

```bash
$ keytool -keypasswd -alias mykey -keystore jks_keystore

Enter keystore password:
Enter key password for <mykey>
New key password for <mykey>:
Re-enter new key password for <mykey>:
```



# Ref

## KeyStore Entries

| Type        | Description                                                  |
| ----------- | ------------------------------------------------------------ |
| PrivateKey  | This is a type of keys which are used in asymmetric cryptography. It is usually protected with password because of its sensitivity. It can also be used to sign a digital signature. |
| Certificate | A certificate contains a public key which can identify the subject claimed in the certificate. It is usually used to verify the identity of a server. Sometimes it is also used to identify a client when requested. |
| SecretKey   | A key entry which is sued in symmetric cryptography.         |



## KeyStore Type

| Type       | Description                                                  |
| ---------- | ------------------------------------------------------------ |
| JKS        | Java Key Store. You can find this file at sun.security.provider.JavaKeyStore. This keystore is Java specific, it usually has an extension of jks. This type of keystore can contain private keys and certificates, but it cannot be used to store secret keys. Sing it's a Java specific keystore, so it cannot be used in other programming languages. The private keys stored in JKS cannot be extracted in Java. |
| JCEKS      | JCE key store(Java Cryptography Extension KeyStore). It is a super set of JKS with more algorithms supported. It is an enhanced standard added later by Sun. You can find this file at com.sun.crypto.provider.JceKeyStore. This keystore has an extension of jceks. The entries which can be put in the JCEKS keystore are private keys, secret keys and certificates. This keystore provides much stronger protection for stored private keys by using Triple DES encryption.<br />The provider of JCEKS is SunJCE, it was introduced in Java 1.4. Hence prior to Java 1.4, only JKS can be used. |
| PCKS12     | this is a standard keystore type which can be used in Java and other languages. You can find this keystore implementation at sun.security.pkcs12.PKCS12KeyStore. It usually has an extension of p12 or pfx. You can store private keys, secret keys and certificates on this type. Unlike JKS, the private keys on PKCS12 keystore can be extracted in Java. This type is portable and can be operated with other libraries written in other languages such as C, C++ or C#.<br />Currently the default keystore type in Java is JKS, i.e the keystore format will be JKS if you don't specify the -storetype while creating keystore with keytool. However, [the default keystore type will be changed to PKCS12 in Java 9](http://openjdk.java.net/jeps/229) because its enhanced compatibility compared to JKS. You can check the default keystore type at **$JRE/lib/security/java.security** file: |
| PKCS11     | this is a hardware keystore type. It provides an interface for the Java library to connect with hardware keystore devices such as SafeNet's Luna, nCipher or Smart cards. You can find this implementation at sun.security.pkcs11.P11KeyStore. When you load the keystore, you no need to create a specific provider with specific configuration. This keystore can store private keys, secret keys and certificates. When loading the keystore, the entries will be retrieved from the keystore and then converted into software entries. |
| DKS        | Domain KeyStore is a keystore of keystore. It abstracts a collection of keystores that are presented as a single logical keystore. Itself is actually not a keystore. This new keystore type is introduced in [Java 8](http://docs.oracle.com/javase/8/docs/technotes/guides/security/enhancements-8.html). There is a new class [DomainLoadStoreParameter](http://docs.oracle.com/javase/8/docs/api/java/security/DomainLoadStoreParameter.html) which closely relates to DKS.<br />This keystore is located at **sun.security.provider.DomainKeyStore.java**. |
| Windows-MY | this is a type of keystore on Windows which is managed by the Windows operating system. It stores the user private keys and certificates which can be used to perform cryptographic operations such as signature verification, data encryption etc. Since it's a kind of native keystore, Java doesn't have a general API to access it. Oracle provides a separate API to access the Windows-MY keystore -- SunMSCAPI. The provider class for this API is **sun.security.mscapi.SunMSCAPI**. |
| BKS        | BoucyCastle keystore, is a keystore format provided the popular third party Java cryptographic library provider -- [BouncyCastle](https://bouncycastle.org/specifications.html). It is a keystore similar to the [JKS](https://www.pixelstech.net/article/1409966488-Different-types-of-keystore-in-Java----JKS) provided by Oracle JDK. But it supports storing secret key, private key and certificate. It is frequently used in mobile application developments. |



## Links

- https://docs.oracle.com/javase/7/docs/technotes/guides/security/StandardNames.html#KeyStore
- https://docs.oracle.com/javase/6/docs/technotes/tools/solaris/keytool.html
