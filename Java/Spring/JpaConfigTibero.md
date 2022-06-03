# Spring JPA에서 Tibero를 사용하기 위한 설정

Tibero를 JPA를 사용하기 위해서 반드시 지정해 주어야 하는 항목은 JPA Dialect 이다.

Tibero는 기본적으로 Oracle을 따라가기 때문에 Oracle Dialect를 상속받아 사용하면 된다.



pom.xml에 spring jpa와 tibero dependency를 추가한다.

```xml
<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-data-jpa</artifactId>
    </dependency>
	<dependency>
		<groupId>tibero6</groupId>
		<artifactId>tibero6-custom</artifactId>
		<version>6.0</version>
		<scope>system</scope>
		<systemPath>${project.basedir}/lib/tibero6-jdbc.jar</systemPath>
	</dependency>
</dependencies>  
```



Configuration에서 JPA 설정을 한다.

```java
package com.example.tiberojpa;

import org.hibernate.dialect.Oracle10gDialect;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.orm.jpa.JpaTransactionManager;
import org.springframework.orm.jpa.LocalContainerEntityManagerFactoryBean;
import org.springframework.orm.jpa.vendor.HibernateJpaVendorAdapter;
import org.springframework.transaction.PlatformTransactionManager;

import javax.persistence.EntityManagerFactory;
import javax.sql.DataSource;

@Configuration
public class JPAConfig {

    @Autowired
    private DataSource dataSource;

    public static class TiberoDialect extends Oracle10gDialect { }

    @Bean
    public EntityManagerFactory entityManagerFactory(){
        HibernateJpaVendorAdapter vendorAdapter = new HibernateJpaVendorAdapter();
        vendorAdapter.setGenerateDdl(false);

        LocalContainerEntityManagerFactoryBean factory = new LocalContainerEntityManagerFactoryBean();
        factory.setJpaVendorAdapter(vendorAdapter);
        factory.setPackagesToScan("com.example.tiberojpa");
        factory.setDataSource(this.dataSource);

        factory.getJpaPropertyMap().put("show-sql", true);
        factory.getJpaPropertyMap().put("hibernate.dialect", TiberoDialect.class);
        factory.getJpaPropertyMap().put("hibernate.physical_naming_strategy", "org.springframework.boot.orm.jpa.hibernate.SpringPhysicalNamingStrategy");

        factory.afterPropertiesSet();

        return factory.getObject();
    }

    @Bean
    public PlatformTransactionManager transactionManager(){
        JpaTransactionManager transactionManager = new JpaTransactionManager();
        transactionManager.setEntityManagerFactory(entityManagerFactory());
        return transactionManager;
    }
}

```

