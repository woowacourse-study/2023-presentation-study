### 목차

1. `@SpringBootTest(webEnvironment=RANOM_PORT)` 와 `RestAssured` 의 조합 ⇒ 테스트 와르르
2. 발생한 문제와 해결 방안
    1. 테이블이 이미 생성되어 있다는 SQL 오류
        - CREATE IF NOT EXISTS
    2. 데이터가 사라지지 않아서 검증에 실패하는 문제 (저장은 2개만 했는데 WHY 조회 4개?)
        1. `@Transactional` 사용 → 여전히 실패
        2. RANDOM_PORT, DEFINED_PORT 외의 다른 옵션 사용 → 사용 불가
        3. TRUNCATE 사용 + test용 DB 분리
        4. `@AutoConfigureTestDatabase`
    3. 만약 한 통합 테스트 클래스 내에서 테스트들끼리 서로 영향을 미친다면? 
        1. TRUNCATE 사용

## `@SpringBootTest(webEnvironment=RANOM_PORT)` 와 `RestAssured` 
⇒ 테스트 와르르

![image](https://user-images.githubusercontent.com/70891072/235097004-03cda6f1-01f6-4e9f-bf77-71055e7ca809.png)

`@SpringBootTest` 가 2개 있는 구조

**AdminControllerIntegrationTest**: 
RestAssured를 사용해 실제 웹 서버를 띄우는 통합 테스트

**CartServiceTest**: 
Service단의 통합 테스트

- 개별 테스트를 실행시킬 때는 성공하지만, 전체 테스트를 돌리면 실패하는 상황
    - 테스트는 순서가 정해져 있지 않다.
    - AdminControllerIntegrationTest → CartServiceTest : CartServiceTest 실패
    - CartServiceTest → AdminControllerIntegrationTest : 테스트 모두 성공

## 1. 테이블이 이미 생성되어 있다는 SQL 오류

![image](https://user-images.githubusercontent.com/70891072/235097069-f94d38ce-25fd-4864-b18c-8b0080ec251e.png)
- 문제
    - 테이블이 이미 생성되어 있다는 에러가 발생했다.
- 원인
    - 하나의 task에서 수행되는 테스트들은 같은 데이터베이스를 공유하기 때문에, 처음 실행되는 DDL문에서 테이블이 생성되고 나면 다음 DDL문이 실행될 때는 “이미 테이블이 존재한다는 오류”가 발생하는 것
- 해결
    - DDL문을 `CREATE TABLE IF NOT EXISTS` 로 변경 → 테이블이 없을 때만 생성되도록 추가

## 2. 데이터가 사라지지 않아서 검증에 실패하는 문제

- 문제
    - AdminControllerIntegrationTest가 먼저 실행되고, CartServiceTest가 실행될 때 저장하지 않은 데이터가 조회 되어 검증에 실패
    
![image](https://user-images.githubusercontent.com/70891072/235097140-cf7f216a-88ad-49ec-ab51-e3ca0a87c7f3.png)
    
![image](https://user-images.githubusercontent.com/70891072/235097189-75275c8f-8184-4701-a57f-dbe0fbcc9cfc.png)
    
    - 저장한 데이터는 2개인데, 4개의 데이터가 조회 된다.  ⇒ 검증 실패

- 원인
    - API 통합 테스트가 진행된 후에 서비스 통합 테스트가 진행되는 경우일 때, 데이터가 사라지지 않은 것
    - 즉 API 통합 테스트의 데이터가 rollback 되지 않고 데이터베이스에 남아있다는 것이 원인
    
![image](https://user-images.githubusercontent.com/70891072/235097245-a0244696-24e9-4ff3-95fd-acd8455f272b.png)
    

### 시도한 해결 방안 1.  `@Transactional` 사용

- 트랜잭션
    - 거래 라는 뜻
    - 돈 출금 → 상대의 계좌에 입금 ⇒ 이 하나의 과정이 모두 성공적으로 이루어져야 “돈 거래”가 완성되는 것!
    - 만약 중간에 하나라도 실패한다면? 되돌려야 한다 → 없던 거래로 만들어야 한다.
    
    ```
    모든 작업들이 성공적으로 완료되어야 작업 묶음의 결과를 적용하고, 
    어떤 작업에서 오류가 발생했을 때는 이전에 있던 모든 작업들이 성공적이었더라도 없었던 일처럼 완전히 되돌리는 것
    ```
    
    - 데이터베이스에 적용해보자면, A의 계좌에 update(-10000)와 B의 계좌에 update(+10000)을 하고 read했을 때 결과까지 모두 성공했을 때에만 완성 처리
- 테스트에 붙는 `@Transcational` 어노테이션
    - 해당 테스트 메서드 내의 작업들이 하나의 트랜잭션
    - 모든 작업이 성공적으로 완료된 후에 변경사항들을 commit(완전 저장) 하지 않고, rollback(변경되기 전의 상태로 데이터베이스를 되돌린다)
    - 데이터베이스에 변경이 없었던 것처럼 만들어준다.
- 결과
    - 실패

### `@Transactional` 이 ROLLBACK 되지 않는 이유

> If your test is `@Transactional`, it will rollback the transaction at the end of each test method by default. However, as using this arrangement with either `RANDOM_PORT`or `DEFINED_PORT` implicitly provides a real servlet environment, HTTP client and server will run in separate threads, thus separate transactions. Any transaction initiated on the server won’t rollback in this case.
> 

- SpringBootTest의 webEnvironment 중, `RANDOM_PORT` 와 `DEFINED_PORT` 는 테스트를 실행시켰을 때 실제 웹 서버가 띄워진다.
- 그리고 이 웹 서버 (요청을 받고 작업을 처리하는 역할)은 다른 스레드로 동작한다.
- Transactional이 설정된 곳: 실제 테스트 코드가 작성되어 있는 스레드 (즉 post, get 등 API요청을 보내는 스레드)
- 하지만, 실제 DB에 대한 비즈니스 로직이 수행되는 역할은 다른 스레드에서 수행됨
- 따라서, 하나의 테스트가 완료되어도 Transactional은 **실제 작업이 이루어진 스레드가 아닌 다른 스레드에 걸려있기 때문에** DB가 rollback 되지 않는다. (자신 외의 스레드에 영향을 주지 않는다.)

![image](https://user-images.githubusercontent.com/70891072/235097304-deb2073a-b0d6-46f1-bb44-dd5ff272fad0.png)
### 시도한 해결 방안 2. `NONE`, `MOCK` 옵션 사용

- `MOCK`, `NONE` 은 실제 서버를 띄우지 않기 때문에 현재 발생하는 테스트 문제를 해결할 수 있을 것이라 생각
- **하지만 `MOCK` 과 `NONE` 은 둘 다 실제 서버를 띄우지 않기 때문에 RestAssured 테스트가 불가능하다!**
    - Mock 설정: 가짜 서블릿 컨테이너를 띄워준다.
    - 그냥 테스트 가능한 가짜 환경을 만들어준다고 이해
    - 실제 웹 서버가 아니기 때문에 실제 서버로 요청을 보내는 RestAssured를 사용하면 `Connection Refused Exception` 이 발생한다.
    - 따라서, Mock Environment를 사용할거면 가짜 환경으로 요청을 보낼 수 있게 해주는 `MockMVC` 를 사용해야 한다.
    - `NONE` 의 경우, 아예 서버를 띄우지 않기 때문에 RestAssured가 불가능하다.

### 시도한 해결 방안 3. `TRUNCATE FROM TABLE 테이블명`

- `@AfterEach`를 사용하여 테스트가 실행된 후에 매번 테이블을 초기화한다.
- `TRUNCATE` 의 문제점: 현재 테스트에서는 프로덕션과 동일한 데이터베이스를 사용한다.
    - 지금은 in-memory DB이지만, 만약 외부 DB와 연결하게 된다면 실제 데이터가 모두 삭제될 수 있다.
    - `test/resources/application.properties` 로 테스트 DB 환경 설정을 하면, 자동으로 테스트 환경에서는 해당 DB로 실행이 된다.

### 시도한 해결 방안 4. `@AutoConfigureTestDatabase`

> Annotation that can be applied to a test class to **configure a test database to use instead of the application-defined or auto-configured** `[DataSource](https://docs.oracle.com/en/java/javase/17/docs/api/java.sql/javax/sql/DataSource.html)`
> 
- 해당 어노테이션이 붙은 테스트는 랜덤하게 설정된 것 외의 in-memory 데이터베이스를 사용한다.
![image](https://user-images.githubusercontent.com/70891072/235097352-6075bf8b-3b6b-499c-8d7f-b4f2942035d1.png)

- `AdminControllerIntegrationTest` 에서만 다른 데이터베이스를 사용 → 해당 테스트의 DB 반영 사항이 다른 테스트에 영향을 미치지 않는다.
- `TRUNCATE` 를 하지 않아도 된다.

### cf) 만약 하나의 `@SpringBootTest(RANDOM_PORT)` 테스트 내에서 테스트 결과가 다른 테스트에 영향을 미친다면?

```java
// 응답이 JSON으로 온다고 가정
@SpringBootTest(webEnvironment = RANDOM_PORT) {

	@Test
	void saveAndFindAll1() {
		
	// post 데이터 1개
	
	// get -> 데이터가 1개만 포함되어 있음을 검증
	
	}
	
	@Test
	void saveAndFindAll2() {
		// post 데이터 1개
	  
	  // get -> 데이터가 1개만 포함되어 있기를 원함, 실제 응답: 2개의 데이터 조회
	}
}
```

- 위의 경우에는 랜덤 DB를 사용해도 같은 DB 내에서 rollback이 되지 않기 때문에 `saveAndFindAll2()` 가 앞 테스트의 영향을 받아 검증에 실패
- 이런 경우, `@AfterEach` 로 `TRUNCATE` 를 하는 방법 뿐이라 생각한다.
