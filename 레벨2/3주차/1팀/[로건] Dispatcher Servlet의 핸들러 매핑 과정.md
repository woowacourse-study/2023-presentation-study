# Dispatcher Servlet의 핸들러 매핑 과정

<img src="https://user-images.githubusercontent.com/79046106/236368581-e7e69543-539e-4488-aaf9-a4ddb635a5d4.png"  width="200" height="200" text-align="center"/>

제리가 궁금해 했지만, 제리 없이 발표함

`Controller` 클래스를 사용할 때, 클래스에 `@Controller`나 `@RequestMaaping`을 사용하지 않고, `@Bean`이나 `@Component`로 등록할 경우 경로가 인식되지 않음

```java
@Configuration
public class SpringConfig {

    private final DataSource dataSource;

    @Autowired
    public SpringConfig(final DataSource dataSource) {
        this.dataSource = dataSource;
    }

    @Bean
    public RacingGameController racingGameController() {
        return new RacingGameController(gameService());
    }

    @Bean
    public GameService gameService() {
        return new GameService(gameRepository());
    }

    @Bean
    public GameRepository gameRepository() {
        return new JdbcTemplateGameRepository(dataSource);
    }
}
```

프로그램 실행시

![image](https://user-images.githubusercontent.com/79046106/236368953-80579aa9-7f47-4c39-be4d-8e1ba937122a.png)

(다른 에러 로그는 보이지 않음)

404 에러이므로 Bean으로 등록되지 않은 것인지 확인함

```java
@SpringBootTest(webEnvironment = SpringBootTest.WebEnvironment.RANDOM_PORT)
class Test {
    @Autowired
    ApplicationContext ctx;

    @Test
    void test() {
        // Bean 등록된 내용을 출력
        String[] beanNames = ctx.getBeanDefinitionNames();
        for (String beanName : beanNames) {
            System.out.println(beanName);
        }
    }
} 
```

![image](https://user-images.githubusercontent.com/79046106/236369056-36baac39-d479-4567-aa06-cf5b907cbb67.png)

Bean으로 잘 등록되어 있는 것을 확인할 수 있음

진짜 문제는…

![image](https://user-images.githubusercontent.com/79046106/236369114-e526b6dc-b877-4a81-925e-4f1a9c0a593f.png)

DispatcherServlet에서는 컨트롤러 클래스만을 골라내 매핑하는 과정에서 발생함

[등록 과정]

1. 등록된 `Bean`을 모두 가져와서 for문을 돌림
2. `Bean`이 컨트롤러 클래스인지 확인함
(`@Controller`, `@RequestMapping`만 컨트롤러 클래스로 인정 - `RequestMappingHandlerMapping.java - isHandler` 참조)
3. 컨트롤러 클래스라면 해당 클래스가 가지고 있는 메서드를 모두 확인해봄
4. 이 과정에서 메서드가 `@RequestMapping`을 가지고 있는지 확인하여 `Map<Method, ?> methods`에 저장함
5. 이후 `methods` 에 저장된 모든 값을 확인하며 등록을 하는데, 핸들러를 등록하는 동안 다른 스레드에서 접근하지 못하도록 lock을 걸고 등록을 수행함

[등록 내용]

- `pathLookUp`: URL 경로와 핸들러 메서드 정보를 등록
- `nameLookUp`: 핸들러 메서드 정보와 사용자 정의 이름을 등록
- `corsLookUp`: 핸들러 메서드와 CORS 설정을 등록
- `registry`: 위에서 저장했던 모든 정보를 재등록
(사용처는 저장된 곳에 `@RequestMapping 경로 + HTTP Method`와 일치하는 값이 존재하는지 확인할 때 사용)

자동차 경주 미션에서는 `pathLookUp`만 사용함

```java
pathLookUp 데이터 예시 (LinkedMultiValueMap에 저장)
ArrayList로 저장(/plays로 다양한 HTTP Method 요청을 할 수 있으므로)

{
	key: "/plays",
	value: ["{POST [/plays]}", "{GET [/plays]}"]
}
```

ex) 클라이언트가 특정 URL로 POST 요청을 보내는 경우

(`AbstractHandlerMethodMapping.java - lookUpHandlerMethod)`

1. `lookUpPath = “/plays”`, `request = {GET, /plays, ...}`
2. `lookUpPath`를 통해 `pathLookUp`에 등록된 URL의 value 값을 가지고 옴 
`directPathMatches = ["{POST [/plays]}", "{GET [/plays]}"]`
3. `directPathMatches` 에서 `request` 정보와 일치하는 값들을 `matches` 로 저장한다 
4. `matches`에 값이 여러 개라면 최선의 값을 선택하는 로직이 동작하여 `bestMatch`에 저장함
5. `bestMatch`에 등록된 메서드를 실행함
6. 이후 `Dispatcher Servlet`에서 나머지 로직을 실행한다

- bestMatch 참고 데이터

![image](https://user-images.githubusercontent.com/79046106/236369157-2ce4cf66-6b07-4500-8875-e64b8b12b76b.png)

---

## 참고 코드

### ex) @Controller, @RequestMapping이 아닌 @Component, @Bean으로 등록

```java
@SpringBootTest(webEnvironment = SpringBootTest.WebEnvironment.RANDOM_PORT)
class Test {

    @Autowired
    RequestMappingHandlerMapping handlerMapping;

    @Test
    void test() {
        // 저장된 HTTP Method + URL 출력
        final Map<RequestMappingInfo, HandlerMethod> handlerMethods = handlerMapping.getHandlerMethods();

        for (RequestMappingInfo requestMappingInfo : handlerMethods.keySet()) {
            System.out.println(requestMappingInfo);
        }
    }
} 
```

![image](https://user-images.githubusercontent.com/79046106/236369226-29f36b79-fb31-4771-94ac-0b159a45036c.png)

`@Controller, @RestController`를 적용한 step1 코드라면 `{POST [/plays]}`가 추가되어 있는 것을 알 수 있음

![image](https://user-images.githubusercontent.com/79046106/236369268-bf741b6d-9569-4ecb-8adc-0b6617771b0f.png)

하지만 `@Bean`으로 등록하면 아예 컨트롤러 클래스로 인식하지 않음

`@RequestMapping, @Controller, @RestController`를 클래스에 붙이면 컨트롤러 클래스로 인식함

```java
@Override
protected boolean isHandler(Class<?> beanType) {
        return (AnnotatedElementUtils.hasAnnotation(beanType, Controller.class) ||
                AnnotatedElementUtils.hasAnnotation(beanType, RequestMapping.class));
}

// hasAnnotation: beanType이 해당 어노테이션을 가지고 있으면 true, 없으면 false를 반환
// @RestController도 내부에서는 @Controller를 가지고 있어서 인식함
// isAnnotation(beanType, Controller.class)는 false로 나옴
```

`@RestController` 데이터를 확인해보면 아래처럼 내부 어노테이션들이 등록되어 있는 것을 확인할 수 있음

![image](https://user-images.githubusercontent.com/79046106/236369298-e81e1427-947a-4e62-a155-e6b89133e3c2.png)
