# PKCS, Public Key Cryptography Standard


## PKCS Intro

PKCS는 공개키 기반 구조(PKI, Public Key Infrastructure)에서 인터넷을 이용해 안전하게 정보를 교환하기 위한 제조사간 비공식 표준 프로토콜로 미국의 RSA가 개발한 암호 작성 시스템입니다. PKCS는 애플, 마이크로소프트, DEC 등의 회사에서 공동 개발하였지만, IETF에서 RFC로 받아드리면서 공식적인 표준으로 인정하고 있습니다.



## PKCS 세부 항목

PKCS에서 다루고 있는 Credential은 정보 시스템에서 암호화된 개인정보로 개인의 공개키 암호 알고리즘을 위한 공개키 및 개인키 쌍, 공개키 인증서, CA 관련 정보, 패스워드 등을 포함하는 암호 정보의 총합입니다. 이를 다루는 PKCS는 #1에서 #15까지 있으며 아래 표와 같은 내용을 담고 있습니다.

| No.  | PKCS title                                       | Comments                      |
| ---- | ------------------------------------------------ | ----------------------------- |
| 1    | RSA Cryptography Standard                        |                               |
| 2,4  |                                                  | incorporated into PKCS #1     |
| 3    | Diffie-Hellman Key Agreement Standard            | superseded by IEEE 1363a etc. |
| 5    | Password-Based Cryptography Standard             |                               |
| 6    | Extended-Certificate Syntax Standard             | never adopted                 |
| 7    | Cryptographic Message Syntax Standard            | superseded by RFC 3369 (CMS)  |
| 8    | Private-Key Information Syntax Standard          |                               |
| 9    | Selected Object Classes and Attribute Types      |                               |
| 10   | Certification Request Syntax Standard            |                               |
| 11   | Cryptographic Token Interface Standard           | referred to as CRYPTOKI       |
| 12   | Presonal Information Exchange Syntax Standard    |                               |
| 13   | *(reserved for ECC)*                             | never beed published          |
| 14   | *(reserved for pseudo random number generation)* | never beed published          |
| 15   | Cryptographic Token Information Syntax Standard  |                               |



- PKCS #1 RSA Cryptography Standard Version 1.5 (RFC 2313, 3447)

  RSA 알고리즘을 이용해 데이터를 암호화하고 전자 서명하는 방법에 대한 내용과 RSA 공개키의 구문을 설명합니다. PKCS #2와 #4는 폐기되고 주요 내용은 PKCS #1에 포함되었습니다.

- PKCS #3 Deffie-Hellman Key Agreement Standard 키 합의 표준

  디피헬먼 키분배 알고리즘의 구현을 위한 방법을 설명하였습니다. PKCS #3은 OSI의 전송 및 네트워크 계층에서이 안전한 전송 채널 구축을 위한 프로토콜에 사용되는 것이 목적입니다.

- PKCS #5 Password-based Cryptography Specification Version 2.0 (RFC 2898)

  개인 키정보를 사용자의 패스워드에 기반하여 암호화하는 방법을 정의한 패스워드 기반 암호표준입니다. PKCS #5는 외부에서 개인키를 생성하여 당사자에게 전송시킬 때 개인키를 암호화하는 것을 목적으로 합니다. 개인키를 특징 기기나 개인이 사용하기 위해 복호화하기 위해서는 패스워드가 필요합니다.

- PKCS #6 Extended-Certificate Syntax Standard

  X.509 인증서의 version 1을 확장했지만, 현재는 X.509 Version 3가 사용되므로 사용되지 않습니다.

- PKCS #7 Cryptographi Message Syntax Version 1.5 (RFC 3369)

  전자 서명이나 전자 봉투와 같은 암호 응요에 대한 일반적인 구문 표협입니다. PKCS #7은 PEM과 호환됩니다.

- PKCS #8 Private-Key Information Syntax Specification Version 1.2 (RFC 5208)

  개인키 정보 구문 표준으로 개인키와 속성 정보를 포함한 암호화된 개인키를 위한 구문을 정의합니다.

- PKCS #9 Selected Object Classes and Attribute Types

  선택된 속성 형식 표준으로 PKCS #6 확장 인증서 구문 표준의 확장 인증서. PKCS #7의 전자서명 메세지와 PKCS #8 개인키 정보에서 사용되는 속성 유형들을 정의합니다.

- PKCS #10 Certification Request Syntax Specification v1.7 (RFC 2986)

  인증 요청 구문 표준으로 인증서 발급 요청서를 위한 구문 정의합니다. 인증서 발급 요청서는 사용자 식별 명칭, 공개키, 옵션인 속성들로 구성되어 사용자가 인증서 발생을 요구하기 위하여 인증 기관에 요청하는 구문입니다.

- PKCS #11 Cryptographi Token Interface Standard 암호 토큰 인터페이스 표준

  스마트카드와 같은 암호화 장비를 위한 기술 독립 프로그래밍 인터페이스 (CAPI)를 설명합니다.

- PKCS #12 Personal Information Exchange Syntax Standard 개인 정보 교환 표준

  사용자의 개인키, 인증 등의 저장과 교환을 위한 포맷을 설명합니다.

- PKCS #13 ECC 타원 곡선 암호 표준 ECC

  타원 곡선 암호에 기반한 공개키를 암호화를 하고 서명하는 매커니즘을 설명합니다.

- PKCS #14 의사 난수 생성 표준 pseudo random number generation

  블록 암호, 해시 함수 등 다양한 방법의 의사 난수 생성을 설명합니다.

- PKCS #15 Cryptographic Token Information Syntax Standard 암호 토큰 정보 형식 표준

  암호화 토큰에 저장된 암호화 보증서의 포맷을 위한 표준을 설명합니다.



### PKCS 표준 분류

| 종류                      | PKCS 문서 번호                        |
| ------------------------- | ------------------------------------- |
| 암호화 방법 정의          | #1, #3, #5, #13                       |
| 다양한 데이터 포맷 정의   | #1, #3, #6, #7, #8, #9, #10, #12, #13 |
| 암호화 관련 API 규격 정의 | #11                                   |



### PKCS #5, #10 (인증서 발행 방식의 차이)

우리가 일반적으로 사용하는 것을 PKCS #5와 PKCS #10 입니다.

이 두가지는 개인키와 공개키를 생성하는 방식에서 차이가 있습니다. 

PKCS #10은 단말이나 사용자가 직접 키를 생성한 후에 공개키를 CA 서버로부터 인증받아 인증서를 서명받은 후에 배포합니다. 개인키가 기기나 사용자를 벗어나지 않음으로 안정성을 보장받는 방법입니다. 

PKCS #5는 외부 CA 서버로 부터 단말이나 사용자의 개인키와 공인키를 생성한 후에 패스워드로 안전하게 암호화하여 기기나 사용자에게 주입하는 방식입니다. 

따라서, PKCS #5 방식을 지원하면 보통 PKCS #10을 지원하지 않고, 그 역도 마찬가지 입니다.





### PKCS #12 (Keystore)

- 사용자의 개인정보(사용자의 개인키, 인증 등)의 저장과 교환을 위한 포맷
- 암호화를 사용하는 디지털 인증서가 포함 된 파일. 개인용 키 또는 기타 중요한 정보를 전송하기 위한 휴대용 형식으로 사용
- 보통 .pfx, .p12 등의 확장자로 저장
- 바이너리 형식으로 저장되며 PKCS#12 포맷의 파일은 인증서, 개인키 내용을 파일 하나에 모두 담고 있다.
- 백업 또는 이동용으로 주로 사용
- P12 파일을 생성하기 전에 개인키(예: key.pem), 인증 기관에서 서명한 인증서(예: certificate.pem), CA 기관의 하나 이상의 인증서(중간 CA 인증서라고도 함)가 있어야 한다.
- P12파일은 PKCS#12 형식으로 하나 또는 그 이상의 certificate(public)과 그에 대응하는 private key를 포함하고 있는 keystore 파일이며 패스워드로 암호화 되어 있다. 열어서 내용을 확인하여면 패스워드가 필요하다.



