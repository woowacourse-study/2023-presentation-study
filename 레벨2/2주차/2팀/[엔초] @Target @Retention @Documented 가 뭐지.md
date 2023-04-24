# @Target @Retention @Documented 가 대체 뭐야

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/6e0dedbe-7f8a-4843-a2d4-b4647f691d67/Untitled.png)

`@RestController` 는 `@Controller` 와 `@ResponseBody` 가 합쳐진 것이다. 

실제로 `@RestController` 에 들어가보면 아래와 같이 되어있다.

`@Controller` , `@ResponseBody` , `@Target` , `@Retention` , `@Documented` 를 가지고 있다.

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/09fc2c49-1210-4001-9632-df7386ebc0a0/Untitled.png)

`@Controller` 에 들어가보면 또 아래와 같이 나온다.

`@Controller` 는 `@Component` , `@Target` , `@Retention` , `@Documented` 를 가지고 있다.

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/7afc4d87-24b4-4243-a00d-b84f2ed67397/Untitled.png)

`@Component` 에 들어가보면 아래와 같이 나온다.

`@Index` , `@Target` , `@Retention` , `@Documented` 를 가지고 있다.

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/9f358bf4-76b4-4bb9-b045-08d8521ba01f/Untitled.png)

`@Index` 에 들어가보면 아래와 같이 나온다.

 `@Target` , `@Retention` , `@Documented` 를 가지고 있다.

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/0b415c5b-7ef0-4db2-b075-03b0dcd11be0/Untitled.png)

이쯤되면 이상하다. 모든 어노테이션이 `@Target` , `@Retention` , `@Documented` 를 가지고 있다.

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/cd8b2f77-7e7e-490e-950b-188c72b901fe/Untitled.png)

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/6929020f-7e1f-4962-ac0e-ec73973dda96/Untitled.png)

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/5cf8cb6c-63b7-4027-8040-f610584d6281/Untitled.png)

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/d947b204-dad1-49bf-8fe5-d4c8fc8594c9/Untitled.png)

심지어는 `@Target` 에 들어가도 `@Target` , `@Retention` , `@Documented` 이 있다.

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/ffb4f052-691e-4658-896f-2adc295d57ea/Untitled.png)

 `@Target` , `@Retention` , `@Documented`  이 어노테이션들은 자바에서 제공하는 어노테이션이다.

---

# 어노테이션

소스코드에 추가해서 사용할 수 있는 메타 데이터의 일종

***메타데이터** : 애플리케이션이 처리해야 할 데이터가 아니라, 컴파일 과정과 실행 과정에서 코드를 어떻게 처리해야 하는지를 알려주기 위한 추가 정보*

## 어노테이션의 기능

- 컴파일러에게 코드 작성 문법 에러를 체크하도록 정보 제공
- 소프트웨어 개발 환경이 빌드나 배포 시 코드를 자동으로 생성할 수 있도록 정보 제공
- 런타임에 특정 기능을 실행하도록 정보를 제공

## 어노테이션의 종류

- Built-in Annotation : 자바에서 기본적으로 제공하는 어노테이션
- Meta Annotation: 커스텀 어노테이션을 만들 수 있게 제공된 어노테이션
- Custom Annotation : Meta Annotation을 통해 만든 커스텀 어노테이션

## **Built-in Annotation**

| @Override | 선언한 메서드가 오버라이드 되었다는 것을 나타냄 
(부모 클래스 또는 인터페이스에 해당 메서드가 없다면 컴파일 에러 발생) |
| --- | --- |
| @Deprecated | 해당 메서드가 더이상 사용되지 않음을 나타냄
(사용할 경우 컴파일 경고 발생) |
| @SuppressWarnings | 선언한 곳의 컴파일 경고 무시
 - @SuppressWarnings("all") : 모든 경고를 억제
 - @SuppressWarnings("cast") : 타입 캐스트 관련 경고 억제
 - @SuppressWarnings("dep-ann") : 사용하지 말아야 할 주석 관련 경고 억제
 - @SuppressWarnings("deprecation") : Deprecated 메서드를 사용한 경우 발생하는 경고 억제
 - @SuppressWarnings("fallthrough") : switch 문에서 break 구문 누락 관련 경고 억제
 - @SuppressWarnings("finally") : finally 블럭 관련 경고 억제
 - @SuppressWarnings("null") : null 관련 경고 억제
 - @SuppressWarnings("rawtypes") : 제네릭을 사용하는 클래스 매개 변수가 특정되지 않았을 때의 경고 억제
 - @SuppressWarnings("unchecked") : 검증되지 않은 연산자 관련 경고 억제
 - @SuppressWarnings("unused") : 사용하지 않는 코드 관련 경고 억제 |
| @SafeVarargs | 제네릭 가변인자를 매개변수로 사용할 때의 경고를 무시 |
| @FunctionalInterface | 함수형 인터페이스임을 나타냄. 람다 함수를 위한 인터페이스 지정.
(메서드가 없거나 두 개 이상이면 컴파일 에러 발생) |

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/3ee7dff4-112e-4edb-a34c-214b5f61e8da/Untitled.png)

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/c72b7d24-937a-42ec-aebf-e7c2a46bbb27/Untitled.png)

## **Meta Annotation**

| @Retention | 어노테이션이 유지되는 기간을 지정하는 데 사용. 어떤 시점까지 어노테이션이 영향을 미치는지 결정 |
| --- | --- |
| @Documented | 해당 어노테이션을 javadoc에 포함시킴 |
| @Target | 어노테이션이 적용할 위치 지정 |
| @Inherited | 어노테이션의 상속을 가능하게 함 |
| @Repeatable | 연속적으로 어노테이션을 사용할 수 있게 함 |

### `@Target`

- 생성할 어노테이션이 적용될 수 있는 위치 나열
    
    
    ![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/6929020f-7e1f-4962-ac0e-ec73973dda96/Untitled.png)
    
    `@Target` 에는 
    ElementType 이넘값이 사용된다.
    
    ![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/97c885ae-ff1c-4b5e-9712-27f7fe69c755/Untitled.png)
    
    - ElementType.Type : 타입 선언 (클래스, 인터페이스, enum)
    - ElementType.Field : 멤버 변수 선언
    - ElementType.METHOD : 메서드 선언
    - ElementType.PARAMETER : 매개변수 선언
    - ElementType.CONSTRUCTOR : 생성자 선언
    - ElementType.LOCAL_VARIABLE : 지역 변수 선언
    - ElementType.ANNOTATION_TYPE : 어노테이션 선언
    - ElementType.PACKAGE : 패키지 선언
    - ElementType.TYPE_PARAMETER : 매개변수 타입 선언
    - ElementType.TYPE_USE : 타입 사용
    - ElementType.MODULE : 모듈 선언

### `@Retention`

- 자바 컴파일러가 어노테이션을 다루는 방법을 기술하며, 어느 시점까지 영향을 미치는지를 결정
    
    ![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/6929020f-7e1f-4962-ac0e-ec73973dda96/Untitled.png)
    
    `@Retention` 에는 
    RetentionPolicy 이넘값이 사용된다.
    
    ![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/c1e96ae2-9892-4867-bd41-e63ae8cc9e96/Untitled.png)
    
    - RetentionPolicy.SOURCE : 컴파일 전까지만 유효
    - RetentionPolicy.CLASS : 컴파일러가 클래스를 참조할 때까지 유효
    - RetentionPolicy.RUNTIME : 컴파일 이후에도 JVM에 의해 계속 참조가 가능
    
    ### 사용 예시
    
    ```java
    @Target(ElementType.TYPE)
    @Retention(RetentionPolicy.RUNTIME)
    public @interface Greeting {
        String greeting() default "안녕하세요";
    }
    ```
    
    ```java
    public class Person {
    
        private final String name;
        private final int age;
    
        public Person(String name, int age) {
            this.name = name;
            this.age = age;
        }
    
        public String getName() {
            return name;
        }
    
        public int getAge() {
            return age;
        }
    }
    ```
    
    ```java
    @Greeting(greeting = "반가워요~")
    public class KindPerson extends Person {
        public KindPerson(String name, int age) {
            super(name, age);
        }
    }
    ```
    
    ```java
    @Greeting(greeting = "뭘 봐")
    public class BadPerson extends Person {
        public BadPerson(String name, int age) {
            super(name, age);
        }
    }
    ```
    
    ```java
    @Greeting(greeting = "멍멍")
    public class Dog {
    
        private final String name;
        private final int age;
    
        public Dog(String name, int age) {
            this.name = name;
            this.age = age;
        }
    
        public String getName() {
            return name;
        }
    
        public int getAge() {
            return age;
        }
    }
    ```
    
    ```java
    public class Main {
        public static void main(String[] args) {
            Person kindPerson = new KindPerson("제이미", 24);
            Person badPerson = new BadPerson("싸가지", 30);
            Dog dog = new Dog("두리", 3);
    
            System.out.printf("%s(%d세)가 하는 말 : ", kindPerson.getName(), kindPerson.getAge());
            printMent(kindPerson);
    
            System.out.printf("%s(%d세)가 하는 말 : ", badPerson.getName(), badPerson.getAge());
            printMent(badPerson);
    
            System.out.printf("%s(%d세)가 하는 말 : ", dog.getName(), dog.getAge());
            printMent(dog);
        }
    
        public static void printMent(Object person) {
            Annotation[] annotations = person.getClass().getDeclaredAnnotations();
            for (Annotation annotation : annotations) {
                if (annotation instanceof Greeting) {
                    Greeting greeting = (Greeting) annotation;
                    System.out.println(greeting.greeting());
                }
            }
       
    ```
    
    실행결과
    ![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/bd2c1cad-ad31-40e2-be73-3d6f72b20b39/Untitled.png)
