# BigDecimal 사용시 주의 사항



java에서 실수 타입 데이터를 사용할 때 `float`와 `double`을 사용하여 표기한다.

각 타입의 지원 범위는 다음과 같다.

| Floating Point Type | Minimun Positive Value    | Default Value | Maximum Value            |
| ------------------- | ------------------------- | ------------- | ------------------------ |
| `float`             | 1.40239846e–45f           | 0             | 3.40282347e+38f          |
| `double`            | 4.94065645841246544e–324d | 0             | 1.7976931348623157e+308d |



이러한 실수 타입을 컴퓨터로 표기하기 위한 표준이 [IEEE Standard for Binary Floating-Point Arithmetic, ANSI/IEEE Std. 754-1985](https://en.wikipedia.org/wiki/IEEE_754-1985) 에 정의되어 있다.

binary 형태로 저장하는 과정에서 의도한 값을 정확하게 저장 할 수 없고 근사치로 저장하게 되며, 이로 인하여 오차가 발생한다.

따라서 정밀한 계산을 위해서는 `BigDecimal` 타입으로 선언하여 사용하도록 권고 된다.



`BigDecimal`을 사용해서 실수 타입 값을 사용할 때 아래 코드에서 보여 진 것처럼 주의해야 하는 사항이 있다.



테스트 코드

```java BigDecimal 테스트 코드
void bigDecimalTest(){
    double value = 48.33;

    BigDecimal floatValue = new BigDecimal(value);
    System.out.println(floatValue);

    BigDecimal stringValue = new BigDecimal(String.valueOf(value));
    System.out.println(stringValue);
}
```



실행 결과

```bash
48.3299999999999982946974341757595539093017578125
48.33
```

