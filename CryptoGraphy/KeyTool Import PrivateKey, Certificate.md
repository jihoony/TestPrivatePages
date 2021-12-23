# KeyTool Import PrivateKey, Certificate

## Files

### file list

```bash
% ls
certificate.crt         privateKey_private.pem  publicKey_public.pem
```



### show files

```bash
% file certificate.crt 
certificate.crt: PEM certificate

% cat certificate.crt 
-----BEGIN CERTIFICATE-----
MIIBqDCCAU6gAwIBAgIULvhjkKt1BonKn6rsqwt7cV0yUtAwCgYIKoZIzj0EAwIw
FjEUMBIGA1UEAwwLVEFZTy1ST09ULUUwIBcNMjAwMTAxMDAwMDAwWhgPOTk5OTEy
MzEyMzU5NTlaMBYxFDASBgNVBAMMC1RBWU8tUk9PVC1FMFkwEwYHKoZIzj0CAQYI
KoZIzj0DAQcDQgAEqZO7U8FhZ89jPVlKlsDyRedUwtYs8NH+mQBuC7jlOdxbKRoN
jVaLt8WWyDPTXE374OGZGdgt9roHyQ3MOJeGzqN4MHYwFgYKKwYBBAGCxGkFCQEB
/wQFMAMCAQEwDgYDVR0PAQH/BAQDAgEGMAwGA1UdEwEB/wQCMAAwHwYDVR0jBBgw
FoAU69xUcCMuwHNGGnTt1AYw5m/cTOYwHQYDVR0OBBYEFOvcVHAjLsBzRhp07dQG
MOZv3EzmMAoGCCqGSM49BAMCA0gAMEUCIELm69mYqefjQRI1b9L7DcCxo7U1yunP
Bisb7wGRb/qKAiEA6rk88xnoEh1o6q3sXO/mNmPWLqMLQQJ/02vAZ5JbAu0=
-----END CERTIFICATE-----                                                                                         

% file privateKey_private.pem 
privateKey_private.pem: PEM EC private key

% cat privateKey_private.pem 
-----BEGIN EC PRIVATE KEY-----
ME0CAQAwEwYHKoZIzj0CAQYIKoZIzj0DAQcEMzAxAgEBBCDjsACC7YNrNgk5VRAz
n6OpmqsIljfiePTAAsXcpvKd96AKBggqhkjOPQMBBw==
-----END EC PRIVATE KEY-----
```



## Import Certificate

### import cert

```bash
% keytool -importcert -keystore mykeystore.jks -storepass password -alias certificateAlias -file certificate.crt 
Owner: CN=TAYO-ROOT-E
Issuer: CN=TAYO-ROOT-E
Serial number: 2ef86390ab750689ca9faaecab0b7b715d3252d0
Valid from: Wed Jan 01 09:00:00 KST 2020 until: Sat Jan 01 08:59:59 KST 10000
Certificate fingerprints:
         SHA1: AA:41:D3:3D:C3:BC:17:DC:B2:A0:7A:C8:1B:6B:9C:35:43:A4:8F:00
         SHA256: 39:4D:58:BD:FB:8C:DB:4B:4A:C4:05:5D:48:A9:8A:14:2F:A4:F2:E8:1C:8F:F0:74:F1:51:AE:5D:05:5C:D2:3E
Signature algorithm name: SHA256withECDSA
Subject Public Key Algorithm: 256-bit EC (secp256r1) key
Version: 3

Extensions: 

#1: ObjectId: 1.3.6.1.4.1.41577.5.9 Criticality=true
0000: 30 03 02 01 01                                     0....


#2: ObjectId: 2.5.29.35 Criticality=false
AuthorityKeyIdentifier [
KeyIdentifier [
0000: EB DC 54 70 23 2E C0 73   46 1A 74 ED D4 06 30 E6  ..Tp#..sF.t...0.
0010: 6F DC 4C E6                                        o.L.
]
]

#3: ObjectId: 2.5.29.19 Criticality=true
BasicConstraints:[
  CA:false
  PathLen: undefined
]

#4: ObjectId: 2.5.29.15 Criticality=true
KeyUsage [
  Key_CertSign
  Crl_Sign
]

#5: ObjectId: 2.5.29.14 Criticality=false
SubjectKeyIdentifier [
KeyIdentifier [
0000: EB DC 54 70 23 2E C0 73   46 1A 74 ED D4 06 30 E6  ..Tp#..sF.t...0.
0010: 6F DC 4C E6                                        o.L.
]
]

Trust this certificate? [no]:  yes
Certificate was added to keystore
jihoon_yang@MacBook-Pro certificates % keytool -list -keystore mykeystore.jks 
Enter keystore password:  
Keystore type: PKCS12
Keystore provider: SUN

Your keystore contains 1 entry

certificatealias, 2021. 8. 6., trustedCertEntry, 
Certificate fingerprint (SHA-256): 39:4D:58:BD:FB:8C:DB:4B:4A:C4:05:5D:48:A9:8A:14:2F:A4:F2:E8:1C:8F:F0:74:F1:51:AE:5D:05:5C:D2:3E
```

### show import result

```bash
% ls             
certificate.crt         mykeystore.jks          privateKey_private.pem  publicKey_public.pem

% keytool -list -keystore mykeystore.jks 
Enter keystore password:  
Keystore type: PKCS12
Keystore provider: SUN

Your keystore contains 1 entry

certificatealias, 2021. 8. 6., trustedCertEntry, 
Certificate fingerprint (SHA-256): 39:4D:58:BD:FB:8C:DB:4B:4A:C4:05:5D:48:A9:8A:14:2F:A4:F2:E8:1C:8F:F0:74:F1:51:AE:5D:05:5C:D2:3E
```



## Import Private Key

### convert private key

```bash
% openssl pkcs12 -export -in certificate.crt -inkey privateKey_private.pem -out mykeystore.p12 -name "privatekeyAlias"
Enter Export Password:
Verifying - Enter Export Password:


% ls
certificate.crt         mykeystore.jks          mykeystore.p12          privateKey_private.pem  publicKey_public.pem
```

### import private key
```bash
% keytool -importkeystore -deststorepass password -destkeypass password -destkeystore mykeystore.jks -srckeystore mykeystore.p12 -srcstoretype PKCS12 -srcstorepass password -alias "privatekeyAlias"
Importing keystore mykeystore.p12 to mykeystore.jks...
```

### show import result
```bash
% keytool -list -keystore mykeystore.jks 
Enter keystore password:  
Keystore type: PKCS12
Keystore provider: SUN

Your keystore contains 2 entries

certificatealias, 2021. 8. 6., trustedCertEntry, 
Certificate fingerprint (SHA-256): 39:4D:58:BD:FB:8C:DB:4B:4A:C4:05:5D:48:A9:8A:14:2F:A4:F2:E8:1C:8F:F0:74:F1:51:AE:5D:05:5C:D2:3E
privatekeyalias, 2021. 8. 6., PrivateKeyEntry, 
Certificate fingerprint (SHA-256): 39:4D:58:BD:FB:8C:DB:4B:4A:C4:05:5D:48:A9:8A:14:2F:A4:F2:E8:1C:8F:F0:74:F1:51:AE:5D:05:5C:D2:3E
```

