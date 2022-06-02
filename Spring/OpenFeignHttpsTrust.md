# Spring OpenFeign 사용시 https 신뢰하는 방법



OpenFeign 설정하기, 이 방법은 임시로만 사용하고 실제로는 해당 사이트의 인증서를 JVM에 등록해야 한다.

```java
package com.example.openfeignHttps;

import feign.Client;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

import javax.net.ssl.*;
import java.security.KeyManagementException;
import java.security.NoSuchAlgorithmException;
import java.security.SecureRandom;
import java.security.cert.CertificateException;
import java.security.cert.X509Certificate;

@Configuration
public class CustomFeignConfiguration {

    @Bean
    public feign.Client client() throws NoSuchAlgorithmException, KeyManagementException {
        return new Client.Default(sslContextFactory(), (hostname, session) -> true);
    }

    private SSLSocketFactory sslContextFactory() throws NoSuchAlgorithmException, KeyManagementException {
        SSLContext sslCtx = SSLContext.getInstance("TLS");
        TrustManager[] certs = new TrustManager[]{
                new X509TrustManager() {
                    @Override
                    public void checkClientTrusted(X509Certificate[] chain, String authType) throws CertificateException {

                    }

                    @Override
                    public void checkServerTrusted(X509Certificate[] chain, String authType) throws CertificateException {

                    }

                    @Override
                    public X509Certificate[] getAcceptedIssuers() {
                        return new X509Certificate[]{};
                    }
                }
        };
        sslCtx.init(null, certs, new SecureRandom());
        return sslCtx.getSocketFactory();
    }
}

```



Feign Interface에서 설정파일 지정하기

```java
package com.example.openfeignHttps;

import org.springframework.cloud.openfeign.FeignClient;
import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.PathVariable;
import org.springframework.web.bind.annotation.RequestParam;

@FeignClient(value = "targetName", url = "targetUrl", configuration = CustomFeignConfiguration.class)
public interface FeignClient {

    @GetMapping("users")
    ResponseEntity<String> getUserList();

    @GetMapping("user/{userId}")
    ResponseEntity<String> getUser(@PathVariable("userId") String userId);
}

```

