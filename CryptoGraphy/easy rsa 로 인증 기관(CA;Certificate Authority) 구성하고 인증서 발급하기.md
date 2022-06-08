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
