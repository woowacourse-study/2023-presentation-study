## Spring application 테스트


- Spring을 사용하여 application을 구성 했다면, 테스트 실행 환경에서도 동일한 구성이 갖추어져 있어야 한다.
-> 직접 구성하지 않은 서블릿이나 WAS, DataSource, jdbcTemplate 및 실행 환경등

Java로만 구성된 code Test

```java
public class Dao {
	private final CustomConnceton customConnceton = new CustomConnection();

	public void do() {
		customConnceton.query(...)
	}
}

@Test
void saveTest() {
	private Dao dao = new Dao();

	dao.do()

	//then
}
```

Spring framework를 통해 구성된 code Test

```java
@Component
public class Dao {

	private final JdbcTemplate jdbcTemplate;

	public Dao (JdbcTemplate jdbcTemplate) {
		this.jdbcTemplate = jdbcTemplate;
	}

	public void do() {
		jdbcTemplate.query(...)
	}
}

@Test
void saveTest() {
	Dao dao = new Dao(new JdbcTemplate());

	dao.do() //fail
}
```

`DataSource`를 구성하여 `JdbcTemplate`을 생성해주는 Spring의 도움을 받던 `Dao`는 Test환경에서 실행 시, Spring context가 생성되지 않아 실행에 실패하게 된다.

## Test 구성 범위

Production code와 동일하게 Spring application을 테스트 하기 위해선 테스트 실행 환경에서도 비슷하거나 동일한 구성 또는 환경이 제공되어야 한다.
테스트 역시 Spring framework에서 제공하는 기능들을 통해 간편하게 구성할 수 있다.

## @SpringBootTest

> SpringBoot 기반의 테스트에서 사용 되는 어노테이션

![](https://velog.velcdn.com/images/roycewon/post/36cb1092-c327-4cbd-882d-b9c0ff5b8875/image.png)

SpringBootConfiguration(스프링 부트 구성)을 찾아서 실행한다. → Spring boot 환경과 유사하게 사용 가능

→ DispatcherServlet, Tomcat 등 stand-alone을 위한 구성정보

SpringBootConfiguration과 더불어 `TestRestTemplate`, `WebTestClient` 등 Web환경에서 활용되는 테스트용 빈도 등록한다.

`org.springframework.boot.test.context.SpringBootTestContextBootstrapper.class` 에 등록하는 구성정보를 확인할 수 있다.

- 대략 196개의 bean이 등록된다.

```java
@SpringBootTest
public class SimpleTest {

    @Autowired
    ApplicationContext applicationContext;

    @Test
    void test() {
        System.out.println(applicationContext.getBeanDefinitionCount());
			// 196
    }
}
//Started SimpleTest in 1.288 seconds
```

## @WebMvcTest

>Web 환경과 관련된 Bean들로만 구성한다.

![](https://velog.velcdn.com/images/roycewon/post/5711b29b-bed7-4942-abd6-f708c0b66b35/image.png)


- `@Controller`, *`@ControllerAdvice`, `@JsonComponent`, `@Converter`, `@Filter`,`@WebMvcConfigurer` ,`@HandlerMethodArgumentResolver`* 가 등록된다.
  

***`@Component`, `@Service`, `@Repository` 는 등록되지 않는다.***
    
- 103개의 빈이 등록된다. (*Started SimpleTest in 0.992 seconds*)

SpringBootTest와는 다르게 모든 빈을 등록하지 않는다. 

@Component 어노테이션을 meta-data로 가지는 @Service, @Repository 뿐만 아니라 프로덕션 코드에서 직접 등록한 빈 객체들도 등록되지 않는다.

따라서, Controller → Service → Repository 의 의존관계가 구성되어 있다면,
Bean에 등록될 Controller는 필요한 의존관계가 형성되지 않아 실행 시, 실패한다.

Ex).

```java
@Controller
class ControllerForTest {
    private final ServiceForTest serviceForTest;

    public ControllerForTest(ServiceForTest serviceForTest) {
        this.serviceForTest = serviceForTest;
    }
}

@Service
class ServiceForTest {
}
```

@WebMvcTest

```java
@WebMvcTest(ControllerForTest.class) // controllers 필드에 Controller를 명시하면, 해당 Controller만 등록된다.
public class SimpleTest {

    @Autowired
    ApplicationContext applicationContext;

    @Test
    void test() {
			applicationContext.getBean(ServiceForTest.class);
    }
}
```
![](https://velog.velcdn.com/images/roycewon/post/651a7009-2ef7-4e02-a54f-8a8990c51ba3/image.png)


→ `ServiceForTest` 는 등록되지 않는다.

빈으로 등록해야 할 `ServiceForTest` 하나를 위해 SpringBootTest로 구성 범위를 변경하지 않고도 필요한 Bean을 간편하게 등록하는 방법을 @WebMvcTest에서는 다음과 같은 어노테이션을 권장한다.

> Typically {***@code*** @WebMvcTest} is used in combination with{***@link*** org.springframework.boot.test.mock.mockito.MockBean @MockBean} or{***@link*** Import @Import} to create any collaborators required by your {***@code*** @Controller}beans.
> \- *@WebMvcTest docs*

정리하면, Test하고자 하는 Controller와 협력하는 빈들을 `@MockBean` , `@Import` 를 통해 등록할 수 있다.

```java
@Import(ServiceForTest.class)
@WebMvcTest
public class SimpleTest {

    @Autowired
    ApplicationContext applicationContext;

    @Test
    void test() {
        assertThat(applicationContext.getBean(ServiceForTest.class)).isNotNull();
    }
}
```

```java
@WebMvcTest(ControllerForTest.class)
public class SimpleTest {

    @Autowired
    ApplicationContext applicationContext;

	@MockBean
    ServiceForTest serviceForTest;

    @Test
    void test() {
        assertThat(applicationContext.getBean(ServiceForTest.class)).isNotNull();
    }
}
```


@Import로 필요한 협력 객체를 Bean으로 등록하면 실제 프로덕션 코드를 검증할 수 있다. 하지만, 해당 Bean이 필요로 하는 모든 객체 역시 Import를 해야한다.
`Controller` -> `Service` -> `Bean1`, `Bean2` -> `Bean3` 의 의존관계가 필요하다면, 
`@Import({Service.class, Bean1.class, Bean2.class, ...})` 처럼 모두 선언해야 한다.

이와 달리`@MockBean`을 통해 구성한다면 복잡한 의존관계로부터 독립적인 환경을 구성할 수 있지만, Stub을 활용한 코드만 수행 될 뿐, 의존하는 객체의 실제 동작을 테스트 해볼 수 없다는 단점이 있다.

내가 하고자 하는 검증이 어떤 검증인가에 대해 잘 판단하고 사용하도록 하자.

## @JdbcTest

![](https://velog.velcdn.com/images/roycewon/post/b04af543-2dc5-45ff-b9e2-7dd7f851f373/image.png)


- Jdbc 환경과 관련된 Bean들로만 구성한다. (JdbcTemplate, DataSource, …)
- @Repository는 등록되지 않는다.
- ***embedded in-memory database를 구성한다.***
- 53개의 빈이 등록된다. (*Started SimpleTest in 0.749 seconds*)

`@JdbcTest`에 `@AutoConfigureTestDatabase`를 메타어노테이션으로 가지고 있다.
이는, 따로 DB를 구성하지 않아도 embedded in-memory database(h2)를 구성해주는 기능을 한다.
![](https://velog.velcdn.com/images/roycewon/post/5a77db99-04d9-41f5-b651-dda75bc32ac2/image.png)

만약, application.properties(혹은 .yml)을 통해 datasource의 옵션을 지정해 두었어도, @JdbcTest의 embedded in-memory database로 대체(replace)된다.
`@AutoConfigureTestDatabase(replace = Replace.NONE)`를 추가하여 테스트 실행시 일반적인 방법으로 실행 프로필을 찾아 `dataSource`가 생성되도록 설정할 수 있다.


## 정리
모든 SpringApplication 테스트 환경에서 구성 범위를 모두 포함하는 `@SpringBootTest` 를 사용하면 안될까? 해도 된다.
하지만, 성능 상의 이점이 필요하다면 테스트에 사용 되지 않을 불필요한 빈이 등록되는 것에 대해 고민해볼 필요가 있을 것이다.

위 예시에서, 테스트를 위해 단순히 기본값을 통해서만 테스트를 수행 하였지만 `@SpringBootTest`와 `@JdbcTest`는 Context가 Load되는데,  `1.3s` <---> `0.75s` 의 차이를 보였다.
실제 프로덕트에선 더 많은 Bean들이 등록될 것이고, 더 많은 코드와 테스트가 생겨날 것이다. 빠른 테스트는 빠른 피드백으로 이어지고, 이는 생산성과 관련될 수 있다. 또, 풍부한 자원을 가진 개발자의 컴퓨팅 자원이 아닌 배포 환경에서 빌드를 위해 테스트가 수행될 때도 더 많은 차이를 불러올 수 있다.
이러한 점들을 고려하여 내가 하고자 하는 테스트의 단위에 맞게 필요한 Context를 구성해보면 좋을 것 같다.