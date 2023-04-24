## Bean 이란?

- Spring IoC 컨테이너가 관리하는 자바 객체

<br>

# Bean 등록 방법

## 1. Component Scanning을 통한 등록

- `**@ComonentScan**` : 어느 지점부터 컴포넌트를 찾으라고 알려주는 역할
- `**@Component**` : 해당 클래스의 인스턴스를 빈으로 등록

```java
@Component
public class RandomNumberGenerator implements NumberGenerator {
    private static final int MAX_RANDOM_NUMBER = 10;

    private final Random random = new Random();

    @Override
    public int generateNumber() {
        return random.nextInt(MAX_RANDOM_NUMBER);
    }
}
```

- Application 클래스의 @SpringBootApplication을 살펴보면 @ComponentScan 어노테이션을 확인할 수 있음
    - @ComponentScan이 붙어있는 클래스의 하위 패키지의 모든 클래스를 훑어보며 @Component가 붙어있는 클래스를 찾음 (빈 등록을 위해)

![image](https://user-images.githubusercontent.com/63184334/233943528-fcb7f325-72d8-4ea3-9245-73679a820e71.png)

- **`@Component`**는 **`@Controller`**, **`@Service`**, **`@Repository`** 등의 어노테이션들에도 포함되어 있어 해당 어노테이션 사용 시 **`@Component`**도 함께 수행되어 스프링 빈에 등록 됨

![image](https://user-images.githubusercontent.com/63184334/233943655-6f3f1156-1c51-4864-b085-96b2d243e89b.png)

![image](https://user-images.githubusercontent.com/63184334/233943332-f4cee058-4c95-4120-b6c5-c93a045e5c93.png)

![image](https://user-images.githubusercontent.com/63184334/233943730-36b16b04-5760-4c0b-ad90-c2eeff0042fa.png)

### `**@Componenet`의 특징**

- 클래스 레벨에서 선언
- 스프링이 런타임시에 컴포넌트스캔을 하여 자동으로 빈을 찾아 **클래스의 인스턴스**를 빈으로 등록
- 개발자가 직접 컨트롤이 가능한 내부 클래스에 사용

<br>

## 2. 자바 코드로 직접 빈 등록

- `**@Configuration**` : 해당 클래스에 @Bean을 통해 빈을 직접 등록하는 코드가 있음을 알림
- **`@Bean`** : 해당 메서드가 생성해 반환하는 객체를 스프링 빈으로 등록

```java
@Configuration
public class SpringConfig {
    
    @Bean
    public NumberGenerator NumberGenerator() {
        return new RandomNumberGenerator();
    }
}
```

### `@Bean`의 특징 및 용도

- 메소드 레벨에서 선언
- **메소드에서 반환되는 객체**를 개발자가 수동으로 빈으로 등록
- 개발자가 컨트롤이 불가능한 외부 라이브러리 사용시 사용

<br>

## Configuration과 TestConfiguration 충돌 시 해결 방법

### 기존 코드

```java
@Configuration
public class SpringConfig {

    @Bean
    public NumberGenerator NumberGenerator() {
        return new RandomNumberGenerator();
    }
}

@SpringBootTest(webEnvironment = SpringBootTest.WebEnvironment.RANDOM_PORT)
class RacingCarControllerTest {

    @TestConfiguration
    static class TestSpringConfig {

        @Bean
        public NumberGenerator NumberGenerator() {
            return new TestNumberGenerator(new ArrayList<>(List.of(7, 3, 7, 3, 7, 3, 7, 2, 7, 2, 7, 2)));
        }
    }
}
```

### 오류 메시지

```java
Description:

The bean 'NumberGenerator', defined in racingcar.controller.RacingCarControllerTest$TestSpringConfig, could not be registered. A bean with that name has already been defined in class path resource [racingcar/SpringConfig.class] and overriding is disabled.

Action:

Consider renaming one of the beans or enabling overriding by setting spring.main.allow-bean-definition-overriding=true
```

### 왜 충돌이 발생했는가?

- `@SpringBootTest`는 비즈니스 로직의 전체 Bean을 로드해 테스트하기에 SpringConfig 클래스의 NumberGenerator()를 통해 빈 객체를 반환하는 것 역시 등록됨
- `@TestConfiguration`에서 등록하는 빈이 동일한 빈 이름과 타입을 가진 빈이 반환되어 충돌이 발생함

<br>

### 1. Configuration 네이밍을 통한 오버라이딩

- `@Configuration`과 `@TestConfiguration`이 존재하고 두 빈 이름이 동일할 때
`@TestConfiguration`이 우선 시 되어 테스트가 잘 수행됨
- 주의) 클래스 명이 동일한 경우 각각 다른 어노테이션이 적용되어 있으므로 다른 클래스로 인식함
- @Configuration에도 @Componenet가 있으므로 빈으로 등록됨

![image](https://user-images.githubusercontent.com/63184334/233944103-122c0ad2-a38d-4668-8bd3-c84db57ba971.png)


```java
@Configuration
public class SpringConfig {

    @Bean
    public NumberGenerator NumberGenerator() {
        return new RandomNumberGenerator();
    }
}

@SpringBootTest(webEnvironment = SpringBootTest.WebEnvironment.RANDOM_PORT)
class RacingCarControllerTest {

    @TestConfiguration("springConfig")
    static class TestSpringConfig {

        @Bean
        public NumberGenerator NumberGenerator() {
            return new TestNumberGenerator(new ArrayList<>(List.of(7, 3, 7, 3, 7, 3, 7, 2, 7, 2, 7, 2)));
        }
    }
}
```

<br>

### 2. @Bean 네이밍을 통한 오버라이딩

- 두 메서드가 동일한 빈 이름을 갖는 경우, 해당 빈은 오버라이딩 됨
이때, 후자 빈 설정이 우선시 되어 사용되므로 테스트가 잘 수행 됨

```java
@Configuration
public class SpringConfig {

    @Bean
    public NumberGenerator NumberGenerator() {
        return new RandomNumberGenerator();
    }
}

@SpringBootTest(webEnvironment = SpringBootTest.WebEnvironment.RANDOM_PORT)
class RacingCarControllerTest {

    @TestConfiguration
    static class TestSpringConfig {

        @Bean("numberGenerator")
        public NumberGenerator NumberGenerator() {
            return new TestNumberGenerator(new ArrayList<>(List.of(7, 3, 7, 3, 7, 3, 7, 2, 7, 2, 7, 2)));
        }
    }
}
```

<br>

### 3. 하나는 Component로 바꾸기

- `@Bean`은 반환된 객체를 빈으로 등록하고, `@Component`는 클래스의 인스턴스를 빈으로 등록하기에 충돌이 발생하지 않음
- 또한, `@TestConfiguration`의 설정을 우선시 하기에 테스트가 잘 수행 됨

```java
@Component
public class RandomNumberGenerator implements NumberGenerator {
    private static final int MAX_RANDOM_NUMBER = 10;

    private final Random random = new Random();

    @Override
    public int generateNumber() {
        return random.nextInt(MAX_RANDOM_NUMBER);
    }
}

@SpringBootTest(webEnvironment = SpringBootTest.WebEnvironment.RANDOM_PORT)
class RacingCarControllerTest {

    @TestConfiguration
    static class TestSpringConfig {

        @Bean
        public NumberGenerator NumberGenerator() {
            return new TestNumberGenerator(new ArrayList<>(List.of(7, 3, 7, 3, 7, 3, 7, 2, 7, 2, 7, 2)));
        }
    }
}
```

---

### 참고자료

[[Spring] 스프링 빈(Bean)의 개념과 생성 원리](https://atoz-develop.tistory.com/entry/Spring-스프링-빈Bean의-개념과-생성-원리)

[Bean과 Component 차이](https://youngjinmo.github.io/2021/06/bean-component/)
