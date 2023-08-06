[블로그링크](https://velog.io/@hgo641/BigDecimal%EC%9D%84-%EC%95%8C%EC%95%84%EB%B3%B4%EC%9E%90)

돈과 같이 정밀한 계산이 필요한 숫자의 경우 `BigDecimal` 타입을 사용하는 게 좋다고 한다. `BigDeciaml`이 `float`, `double`같은 다른 타입과 무슨 차이가 있길래 그런걸까? 간단하게 알아보자!


# BigDecimal을 알아보자~~
## float, double의 한계

`float`과 `double`은  정확한 값(precise value)이 아닌 근삿값(approximate value)으로 계산하기에, 오차가 발생할 수 있다. 

>  근삿값으로 계산하는 이유는 `float`과 `double`이 부동 소수점 표현 방식을 사용하기 때문이다. 
>
> 부동 소수점은 정수를 2진수로 표현하기 위해 정규화 과정을 거치는 데 이 때 오차가 발생해서 근삿값을 저장한다. 자세한 설명은 [링크](https://gguguk.github.io/posts/fixed_point_and_floating_point/#%EA%B3%A0%EC%A0%95-%EC%86%8C%EC%88%98%EC%A0%90fixed-point)를 참고하자!

```java
final Double d1 = Double.valueOf(3.1);
final Double d2 = Double.valueOf(3.2);
System.out.println(d1 + d2);

// 6.300000000000001
```

```java
final Float d1 = Float.valueOf(0.1234567f);
final Float d2 = Float.valueOf(0.1f);
System.out.println(d1 + d2);

// 0.22345671
```





## BigDecimal의 특징

### 임의 정밀도 정수형

[공식문서](https://docs.oracle.com/javase/8/docs/api/java/math/BigDecimal.html)에 따르면, `BigDeciaml`은 임의 정밀도를 나타내는 `unscaled value`와 32bit의 `scale`로 이루어져있다고 한다. 

> `unscaled value`는 정수부를 표현하고, `scale`은 소수점 아래 자릿수를 표현한다.
>
> 예를 들어 `BigDecimal` 3.14의 경우 `unscaled value`는 314이고 `scale`은 2가 된다.

시작부터 어려운 말이 등장했다... 임의 정밀도가 뭔데!!!

간략하게 말하면 임의 정밀도 정수형이란 무제한 자릿수를 제공하는 정수열을 말한다고 한다... 정수를 자릿수 단위로 쪼개어 배열 형태로 표현해 아주 긴 정수라도 표현할 수 있다고 한다... 

[위키피디아](https://en.wikipedia.org/wiki/Arbitrary-precision_arithmetic)에 의하면 오직 host system의 memory 크기에 의해서만 정수 표현이 제한된다고 하는 것 같다... 더 알아보고 싶지 않으므로 생략하겠다... 대충 어마어마하게 긴 정수형을 표현할 수 있는 사기 기술이지만, 그만큼 계산하는데 많은 리소스가 든다고만 알고 넘어가자...



### 불변

`BigDecimal`은 불변의 성질을 가진다고 한다. `BigDecimal`은 같은 `BigDecimal`타입과의 연산 기능을 제공하는 데 이때 항상 새로운 `BigDecimal`객체를 만들어서 반환한다. 때문에 여러 스레드에서 동시에 접근해도 의도치 않은 동작을 막을 수 있다. 한 번 생성되면 값이 유지되는 것을 보장하기 때문에 캐싱 기능을 지원하기도 편리하다. 이외에도 불변성이 제공하는 모든 장점을 취한다.



### 10진수로 표현

앞서 `float`과 `double`은 정수를 2진수로 표현하기 위해 정규화 과정을 거쳤기에 소수점 계산에서 미세한 오차가 발생했다. 하지만 `BigDecimal`은 10진수로 표현하기 때문에 오차가 발생하지 않는다.



## java.math의 BigDecimal이 수를 표현하는 방식

java.math 패키지의 `BigDecimal`은 다음과 같이 네 개의 필드로 수를 표현하고 있다.

```java
public class BigDecimal extends Number implements Comparable<BigDecimal> {
    
    private final BigInteger intVal;

    private final int scale;

    private transient int precision;

    private final transient long intCompact;
    
}
```

* `intVal` : 정수부인 `unscaled value`를 표현한다. (숫자가 3.14일 때, `intVal`은 314)
* `scale` : 소수점 아래 자릿수를 표현한다. (숫자가 3.14일 때, `scale`은 2)
* `precision` : 총 자릿수를 표현한다.
* `intCompact` : `BigDecimal`연산의 성능 향상을 위한 필드로, `Long.max`보다 작은 수일 경우 정수부를 `intVal`대신  `intCompact`로 표현한다고 한다.



## BigDecimal 사용시 주의점

### 생성자 인자로 double을 넣으면 오차 발생

`BigDecimal` 생성자 인자로 `double`을 넣으면 오차가 발생한다. `double`은 앞서 말했듯이 근삿값으로 저장이 되는데, 이 값이 그대로 인자로 들어가기 때문이다.

```java
new BigDecimal(1234.56);
// 1234.56989898989…
```



오차를 막기 위해선 인자값으로 String타입을 넣어주거나, 정팩매인 `valueOf`를 사용하면 된다.

```java
new BigDecimal("1234.56");
BigDecimal.valueOf(1234.56);
```

> ```java
> public static BigDecimal valueOf(double val) {
>  // Reminder: a zero double returns '0.0', so we cannot fastpath
>  // to use the constant ZERO.  This might be important enough to
>  // justify a factory approach, a cache, or a few private
>  // constants, later.
>  return new BigDecimal(Double.toString(val));
> }
> ```
>
> `valueOf`는 인자로 받은 double을 String으로 바꾸는 로직을 수행한다.





### divide시 소수부 처리 전략 지정 필요

`BigDecimal`가 지원하는 나눗셈 메소드 `divide`는 기본적으로 소수점 처리(rounding)을 하지 않고 정확한 몫을 반환한다. 

그러나 나눈 몫인 무한대일 경우 ArithmeticException이 발생합니다. 예외를 막기 위해서는 무한대로 생기는 소수부를 어떻게 처리해줄 것인지 지정해주어야한다.

```java
BigDecimal seven = new BigDecimal("7");
BigDecimal three = new BigDecimal("3");

// 나누기 - 기본적으로 정확한 몫을 반환함
// 2.33333...
// java.lang.ArithmeticException: Non-terminating decimal expansion; no exact representable decimal result.
seven.divide(three);

// 나누기 - 소수점 아래 첫째 자리까지 반올림
// 2.3
seven.divide(three, 1, RoundingMode.HALF_UP);
```



보통 `RoundingMode`를 사용해 소수점 처리 전략을 지정한다. `RoundingMode`가 지원하는 전략은 다음과 같다.

```java
package java.math;

public enum RoundingMode {
    
    // 0에서 멀어지는 방향으로 올림 
    // 양수인 경우엔 올림, 음수인 경우엔 내림
    UP(BigDecimal.ROUND_UP),
    
    // 0과 가까운 방향으로 내림
    // 양수인 경우엔 내림, 음수인 경우엔 올림
    DOWN(BigDecimal.ROUND_DOWN),
    
    // 올림
    CEILING(BigDecimal.ROUND_CEILING),
    
    // 내림
    FLOOR(BigDecimal.ROUND_FLOOR),
    
    // 반올림
    // 5 이상이면 올림, 5 미만이면 내림
    HALF_UP(BigDecimal.ROUND_HALF_UP),
    
    // 반올림 (오사육입) 
    // 6 이상이면 올림, 6 미만이면 내림
    HALF_DOWN(BigDecimal.ROUND_HALF_DOWN),

    // 반올림 (오사오입, Bankers Rounding)
    // 5 초과면 올리고 5 미만이면 내림, 5일 경우 앞자리 숫자가 짝수면 버리고 홀수면 올림하여 짝수로 만듦
    HALF_EVEN(BigDecimal.ROUND_HALF_EVEN),
    
    // 소수점 처리를 하지 않음
    // 연산의 결과가 소수라면 ArithmeticException이 발생함
    UNNECESSARY(BigDecimal.ROUND_UNNECESSARY);
    
    ...
}
```



### equal()과 compareTo() 구분

```java
final BigDecimal b1 = new BigDecimal("7.10");
final BigDecimal b2 = new BigDecimal("7.1");

System.out.println(b1.equals(b2)); // false
System.out.println(b1.compareTo(b2)); // 0
```

`equals`는 `BigDecimal`의 `scale`값까지 같아야 true를 반환한다. `7.10`과 `7.1`은 논리적으로 같은 수일지라도, 소수점아래 자릿수가 다르므로 `equals`의 결과는 `false`가 된다.

`7.10`과 `7.1`을 같은 수로 보고 싶다면, `compareTo`를 사용하면 된다.



## 마무리

IEEE 754 표준에 의하면, `float` 값의 정밀도는 6 ~ 9 자리, `double` 값의 정밀도는 15 ~ 18 자리라고 한다. 해당 범위내의 수를 사용한다면 `float`과 `double`을 사용하는 것이 효율적이겠지만, 금융 도메인과 관련된 값은 정확한 값을 나타내는 게 가장 중요하므로 `BigDecimal`타입을 사용하는 게 좋을 것 같다.

## 참고
* https://docs.oracle.com/javase/8/docs/api/java/math/BigDecimal.html
* https://gguguk.github.io/posts/fixed_point_and_floating_point/#%EA%B3%A0%EC%A0%95-%EC%86%8C%EC%88%98%EC%A0%90fixed-point
* https://en.wikipedia.org/wiki/Arbitrary-precision_arithmetic
* https://dev.gmarket.com/75
