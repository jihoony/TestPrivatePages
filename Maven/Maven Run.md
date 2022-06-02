# Maven run option

## java run

add dependency in pom.xml

```xml
<build>
    <plugins>
	    <plugin>
            <groupId>org.codehaus.mojo</groupId>
            <artifactId>exec-maven-plugin</artifactId>
            <version>3.0.0</version>
            <configuration>
                <mainClass>com.test.Main</mainClass>
            </configuration>
        </plugin>
    </plugins>
</build>
```



run maven command

```bash
mvn compile exec:java
```



## Springboot run

run maven command

```bash
mvn spring-boot:run
```

