# Aspect with Annotation



## 목적

Annotation을 정의하고 해당 Annotation이 붙은 method를 Aspect 처리하도록 한다.

이때 Annotation 값을 파라미터로 전달하도록 하는 방법



## 참고

> AOP를 사용하기 위해서는 spring-aop dependency가 포함되어야 한다.
>
> spring-aop는 dependency에 spring-web의 필수 요소이기 때문에, spring-web이 추가되어 있다면 따로 추가할 필요가 없다.



## 설정 방법



**`MyAnnotation` Annotation 정의**

```java
@Target({ElementType.METHOD})
@Retention(RetentionPolicy.RUNTIME)
@Documented
public @interface MyAnnotation{
    String name() default "";
}
```



**Enable Aspect**

```java
@Configuration
@EnableAspectJAutoProxy
public class AspectConfiguration {
}
```



**`MyAnnotationAdvisor` Configuration**

```java
@Sl4j
@Aspect
@Component
public class MyAnnotationAdvisor {
    
    @PointCut("@annotation(annotation)")
    public void pointCutMyAnnoration(MyAnnotation annotation){};
    
    @Around(value="pointCutMyAnnoration(annotation)")
    public Object proceedingJoinPointMyAnnotation(ProceedingJoinPoint joinPoint, MyAnnotation annotation){

        log.debug(annotation.toString());
        
        try{
            
            Object object = joinPoint.proceed(args);
            
        } catch (Throwable e) {
            
            log.error("Exception occurred [" + e.getLocalizedMessage() + "]", e);
            
        }
    }
}
```

