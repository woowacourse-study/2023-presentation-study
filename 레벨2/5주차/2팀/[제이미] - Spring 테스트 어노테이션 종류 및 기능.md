## 주제 선정 이유

- 각 테스트 어노테이션 별로 어떤 느낌인지만 안다
- 설명하라고 하면 제대로 하지 못하며, 테스트 수행 시 어떤 어노테이션을 사용해야 할지 고민이 된다
- 어노테이션의 옵션들을 잘 모르겠다

## Spring 테스트 종류

- @SpringBootTest
- @WebMvcTest
- @JdbcTest
- @DataJdbcTest
- @DataJpaTest
- @RestClientTest
- @JsonTest
- 테스트에 자주 사용되는 어노테이션
    - @AutoConfigureTestDatabase
    - @ExtendWith
    - @DirtiesContext

- 위 테스트들은 아래 라이브러리를 통해 사용할 수 있다

```java
testImplementation 'org.springframework.boot:spring-boot-starter-test'
```

## @SpringBootTest

### 간단 설명

- SpringBoot 어플리케이션을 테스트하기 위해 사용
- 전체적인 **통합 테스트**를 제공하는 기본 테스트 어노테이션
- 애플리케이션의 모든 빈을 로드하므로 실제 운영환경과 유사한 테스트가 가능함
    - 애플리케이션과 동일하게 **ApplicationContext**를 로드하기 때문

- 단점
    - 애플리케이션에 설정된 모든 빈을 로드하는 만큼 규모가 클수록 속도가 느려짐
    - 모든 빈을 등록하는 만큼 테스트에서 빈을 등록하는 경우를 유의해야 함

### 테스트 옵션

- **`value`**, **`properties`**
    - 애플리케이션 실행에 필요한 프로퍼티를 key=value 형태로 추가할 수 있음
        - 예시 코드
            
            ```java
            // application.properties
            value = properties
            
            @SpringBootTest(value = "value=test")
            public class AnnotationTest {
            
                @Value("${value}")
                private String value;
            
                @Test
                public void contextLoads() {
                    assertThat(value).isEqualTo("test");
                }
            }
            ```
            
    - 테스트를 위한 properties 파일을 경로를 통해 직접 설정해줄 수 있음
        - `@TestPropertySource`로도 가능
        - 예시 코드
            
            ```java
            // application.properties
            value = properties
            
            // test-application.properties
            value = properties
            
            @SpringBootTest(value = "spring.config.additional-location=classpath:test-application.properties",)
            @TestPropertySource(locations = "classpath:test-application.properties")
            public class AnnotationTest {
            
                @Value("${value}")
                private String value;
            
                @Test
                public void contextLoads() {
                    assertThat(value).isEqualTo("test");
                }
            }
            ```
            
- ************`**classes**`************
    - 원하는 빈만 등록하기 위해선 classes로 class를 직접 등록할 수 있음
    - classes 설정이 없다면, 모든 빈을 등록함
        - 예시 코드
            
            주의) 현재 해당 코드는 NamedParameterJdbcTemplate 주입과 관련해 오류가 존재함
                    (현재 원인을 찾는 중)
            
            ```java
            @SpringBootTest(classes = {ItemService.class, ItemDao.class})
            public class AnnotationTest {
            
                @Autowired
                ItemService itemService;
            
                @Autowired
                ItemDao itemDao;
            
                @Test
                public void contextLoads() {
                    assertThat(itemService.loadItem(1L)).isEqualTo(
                            new Item.Builder()
                                    .id(1L)
                                    .name(new Name("위키드"))
                                    .imageUrl(new ImageUrl("https://image.yes24.com/themusical/upFiles/Themusical/Play/post_2013wicked.jpg"))
                                    .price(new Price(150000))
                    );
                }
            }
            ```
            
    
- **`webEnvironment`**
    - 웹 테스트 환경을 설정할 수 있음
    - 기본값 : Mock
    
    - **`WebEnvironment.MOCK`**
        - 내장 서블릿 컨테이너 대신 mock 서블릿 환경으로 내장톰캣이 구동되지 않음
            
            즉, 브라우저에서 접속되지 않음을 의미함
            
        - 서버에 연결이 불가능 하기 때문에 MockMvc를 사용해 테스트를 진행한다
        - 웹 서버를 실행하지 않고 테스트를 수행할 때 유용함
        
        - 오류 코드
            
            서버에 연결할 수 없으므로 오류가 발생할 것
            
            ```java
            @SpringBootTest(webEnvironment = SpringBootTest.WebEnvironment.MOCK)
            public class AnnotationTest {
            
                @Autowired
                ObjectMapper objectMapper;
                
                @Test
                public void contextLoads() throws Exception {
                    String content = objectMapper.writeValueAsString(new ItemRequest("레드북", 150000, "url"));
                    RestAssured.given()
                               .body(content)
                               .contentType(MediaType.APPLICATION_JSON_VALUE)
                               .when()
                               .post("/items")
                               .then()
                               .statusCode(HttpStatus.CREATED.value())
                               .header("Location", "/");
                }
            }
            ```
            
        - 예시 코드
            
            > **MockMVC란?**
            개발한 웹 프로그램을 실제 서버에 구동하지 않고도 테스트를 위한 Spring MVC 동작 재현을 제공하는 수단
            `@AutoConfigureMockMvc`을 사용해 보다 편하게 MockMvc 객체를 주입받을 수 있음
            > 
            
            주의) 스터빙할 객체(ex. ItemService)의 경우 ㅊ이 아닌 **`@MockBean`**으로 설정해주어야 한다
            
            ```java
            @SpringBootTest(webEnvironment = SpringBootTest.WebEnvironment.MOCK)
            @AutoConfigureMockMvc
            public class AnnotationTest {
            
                @Autowired
                MockMvc mockMvc;
            
                @MockBean
                ItemService itemService;
            
                @Test
                public void contextLoads() throws Exception {
                    BDDMockito.given(itemService.loadAllItem()).willReturn(List.of(ItemResponse.from(new Item.Builder().id(1L)
                                                                                                                       .name(new Name("위키드"))
                                                                                                                       .imageUrl(new ImageUrl("https://image.yes24.com/themusical/upFiles/Themusical/Play/post_2013wicked.jpg"))
                                                                                                                       .price(new Price(150000))
                                                                                                                       .build())));
            
                    mockMvc.perform(get("/"))
                           .andExpect(model().attribute("products", EXPECTED_PRODUCTS))
                           .andExpect(view().name("index"))
                           .andExpect(status().isOk());
            		}
            }
            ```
            
    
    - **`WebEnvironment.RANDOM_PORT`**
        - EmbeddedWebApplicationContext를 로드하며 실제 서블릿 환경을 구성
        - 내장톰캣이 구동되나 랜덤 포트로 구동됨
            - 즉, 브라우저에서 접속함을 의미함
        - 실제 서버를 사용해 테스트를 수행할 수 있어 실제 서버와 상호작용이 가능함
            
            이를 통해환경에서 발생할 수 있는 환경에서 발생할 수 있는 문제에 대한 테스트를 할 수 있음
            
        - 예시 코드
            
            주의) `@BeforeEach`을 통해 `RestAssured.port = port;` 꼭 수행해주어야 함
            
            ```java
            @SpringBootTest(webEnvironment = SpringBootTest.WebEnvironment.RANDOM_PORT)
            public class AnnotationTest {
            
                @Autowired
                ObjectMapper objectMapper;
            
                @LocalServerPort
                int port;
            
                @BeforeEach
                void setUp() {
                    RestAssured.port = port;
                }
            
                @Test
                public void contextLoads() throws Exception {
                    String content = objectMapper.writeValueAsString(new ItemRequest("레드북", 150000, "url"));
                    RestAssured.given()
                               .body(content)
                               .contentType(MediaType.APPLICATION_JSON_VALUE)
                               .when()
                               .post("/items")
                               .then()
                               .statusCode(HttpStatus.CREATED.value())
                               .header("Location", "/");
                }
            }
            ```
            
    
    - **`WebEnvironment.DEFINE_PORT`**
        - **`WebEnvironment.RANDOM_PORT`**와 동일하게 구동되나 설정된 포트로 구동됨
            - 만약 설정을 하지 않는다면 기본값인 8080으로 구동
        - 특정 포트 혹은 특정 상황에 대한 테스트를 수행해야 하는 경우에 유용함
            
            
        - 예시 코드
            - server.port 설정 방법은 프로퍼티 파일 혹은 value로 지정할 수 있음
            
            ```java
            // application.properties
            server.port=1234
            
            @SpringBootTest(value = "server.port=1111", webEnvironment = SpringBootTest.WebEnvironment.DEFINED_PORT)
            public class AnnotationTest {
            
                @Autowired
                ObjectMapper objectMapper;
            
                @Value("${server.port}")
                @LocalServerPort
                int port;
            
                @BeforeEach
                void setUp() {
                    System.out.println(port);
                    RestAssured.port = port;
                }
            
                @Test
                public void contextLoads() throws Exception {
                    String content = objectMapper.writeValueAsString(new ItemRequest("레드북", 150000, "url"));
                    RestAssured.given()
                               .body(content)
                               .contentType(MediaType.APPLICATION_JSON_VALUE)
                               .when()
                               .post("/items")
                               .then()
                               .statusCode(HttpStatus.CREATED.value())
                               .header("Location", "/");
                }
            }
            ```
            
    
    - **`WebEnvironment.NONE`**
        - 웹이 아닌 자바 애플리케이션을 수행할 때 사용
        - 웹이 아닌 경우 HTTP 요청이 필요하지 않으므로 다른 환경보다 보다 가볍게 테스트를 수행할 수 있음
        - 우리는 웹 환경이므로 예시 코드는 없습니다.



<br>

***추후 나머지 테스트 어노테이션에 대해서도 추가해두도록 하겠습니다.***
