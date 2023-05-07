## Axios란?

> 브라우저, Node.js를 위한 Promise API를 활용하는 HTTP **비동기** 통신 라이브러리
> 

*API 호출을 위한 다른 라이브러리로는 **AJAX**가 있고, JavaScipt에서 기본으로 **Fetch API**를 제공해준다.

***Node.js** : 가장 널리 사용되는 JavaScript 프레임워크

## Axios 사용하기

### 의존성 추가하기

- jsDeliver
    
    ```jsx
    <script src="https://cdn.jsdelivr.net/npm/axios/dist/axios.min.js"></script>
    ```
    
- unpkg CDN
    
    ```jsx
    <script src="https://unpkg.com/axios/dist/axios.min.js"></script>
    ```
    

### 요청하기

```jsx
axios({
  url: 'https://test/api/cafe/list/today', // 통신할 웹문서
  method: 'get', // 통신할 방식
  data: { // 인자로 보낼 데이터
    foo: 'diary'
  }
});
```

- 요청(Request) 파라미터 옵션
    - **method** : 요청 방식(별도로 설정해주지 않을 시 GET이 기본으로 설정됨)
    - **url** : 요청을 보내고자 하는 **서버 주소**
    - **headers** : 요청 헤더 객체(`@RequestHeader`로 받아올 수 있다)
    - **data** : 요청 방식이 PUT, POST, PATCH일 때 body에 보내는 데이터 객체(`@RequestBody`로 받아올 수 있다)
    - **params** : URL 파라미터 객체 (`@RequestParam`로 받아올 수 있다.)
    - **responseType** : 서버가 응답해주는 데이터의 타입 지정(json, text, stream, blob 등이 있음)

### 응답하기(Response 데이터 처리)

- axios를 통해 서버로 요청을 보내면 서버에서 요청에 대한 처리를 하고 데이터를 다시 클라이언트에게 전송한다.
- `.then()` 메서드에 대한 파라미터로 응답에 대한 **콜백 함수**를 넘겨주면, 응답받은 데이터를 처리한다.
    
    ```jsx
    axios({
        method: "get", // 통신 방식
        url: "www.naver.com", // 서버
    })
    .then(function(response) {
      console.log(response.data)
      console.log(response.status)
      console.log(response.statusText)
      console.log(response.headers)
      console.log(response.config)
    })
    ```
    
    ```jsx
    response.data: {}, // 서버가 제공한 응답 객체(데이터)
    response.status: 200, // 서버 응답의 HTTP 상태 코드
    response.statusText: 'OK',  // 서버 응답의 HTTP 상태 메시지
    response.headers: {},  // *서버가 응답 한 헤더는 모든 헤더 이름이 소문자로 제공된다.
    response.config: {}, // 요청에 대해 `axios`에 설정된 구성(config)
    response.request: {}
    ```
    
    만약 요청-응답에 대한 예외를 처리하고 싶은 경우, `.then()`에 이어 `.catch()` 메서드를 호출해주면 된다.
    
    ```jsx
    axios.get('/user?ID=12345')
      .then(function (response) {
        // 성공했을 때
        console.log(response);
      })
      .catch(function (error) {
        // 에러가 났을 때
        console.log(error);
      })
      .finally(function () {
        // 항상 실행되는 함수
      });
    ```
    

### +) axios 단축 메소드

```jsx
axios.request(config)
axios.get(url[, config])
axios.delete(url[, config])
axios.head(url[, config])
axios.options(url[, config])
axios.post(url[, data[, config]])
axios.put(url[, data[, config]])
axios.patch(url[, data[, config]])
```

### +) Axios 전역 설정(Config Default)

Axios로 보내는 모든 요청에 적용되는 설정의 Default 값을 전역으로 명시할 수 있다.

주로 서버에서 서버로 axios를 사용할 때 **요청 헤더를 명시하는 데** 자주 쓰인다.

```jsx
axios.defaults.baseURL = 'https://api.example.com';
axios.defaults.headers.common['Authorization'] = AUTH_TOKEN;
axios.defaults.headers.post['Content-Type'] = 'application/x-www-form-urlencoded';
```

### 사용 예시

- admin.js의 코드 중 일부

```jsx
const createProduct = (product) => {
    axios.request({
        url: '/products',
        method: 'POST',
        data: product,
    }).then((response) => {
        window.location.reload();
    }).catch((error) => {
        alert(error.response.data.message);
    });
};
```

## 기타 JavaScript 지식


### 비동기 처리 키워드(Async, Await)

JavaScript는 타 언어들과 마찬가지로 **동기 처리를 기본으로 한다.**

- 동기 처리란?
    
    > 이전 실행의 결과가 나올 때까지 **기다렸다가** 결과가 나오면 순차적으로 그 다음 실행을 하는 것.
    > 
- 비동기 처리란?
    
    > 특정 코드의 연산이 끝날 때까지 **코드의 실행을 멈추지 않고** 다음 코드를 먼저 실행하는 것
    > 

**Promise 객체 :** 간단히 말해 **비동기 처리를 지원하는 형태의 객체**라고 할 수 있다.

axios도 내부적으로 promise 객체를 사용하고 있다.

⇒**axios는 기본적으로 비동기적으로 처리된다.**

```jsx
var example="강아지";
const response = await axios(options); //{animal: "고양이"}를 반환하는 api라고 가정
example=response?.data?.animal;
console.log(example);    //null이 출력된다.
```

⚠️axios(….)**.then()** 메서드을 통해 **콜백 함수**를 정의해주는 것으로 해결할 수 있겠지만, 모든 동작을 콜백함수를 통해 처리하게 되면 수많은 콜백함수의 계층이 생긴다(이를 **콜백 지옥**이라고 함)

<aside>
  💡 <b>만약 비동기 작업을 동기적으로 처리하고 싶다면?</b>

⇒`async`, `await` 키워드를 사용하면 된다.

- `async` : 이 동작(함수)가 promise 동작을 한다는 것을 명시한다.
- `await` : 이 동작이 끝날 때까지 기다리겠다.

```jsx
async function getStudies({ token, year, season }) {
  const options = {
    method: 'GET',
    url: API_URL + '/v1/study/list',
    headers: {
      Authorization: token,
    },
    params: { year: year, season: season },
  };
  try {
    const response = await axios(options);
    return response.data;
  } catch (error) {
    return error.response.data;
  }
}
```

</aside>

### JavaScript(ES6)에서 지원하는 간단한 객체 표현 문법

- 전개 연산자(Spread Operator)
    1. 배열의 경우
        
        ```jsx
        const array = [1,2,3]
        ```
        
        ```jsx
        [...array, 4, 5, 6] == [1, 2, 3, 4, 5, 6]
        [-1, 0, ...array] == [ -1, 0, 1, 2, 3]
        [-1, 0, ...array, 4, 5] == [-1, 0, 1, 2, 3, 4, 5]
        ```
        
    2. 객체의 경우
        
        ```jsx
        const object = { fruit : "사과" }
        ```
        
        ```jsx
        {...object, animal : "강아지" } == {fruit : "사과", animal : "강아지" }
        ```
        
- 프로퍼티 축약(Property Shorthand)
    
    아래와 같이 key의 이름과 value로 들어가는 변수의 이름이 같을 때, key:value 형식을 생략할 수 있다.
    
    ```jsx
    const cat="고양이";
    const dog="강아지";
    
    const object1={
    	cat : cat,
    	dog : dog,
    }
    
    const object2={
    	cat,
    	dog,
    }
    
    //object1과 object2는 같다.
    ```
    
- 구조분해 할당(destructuring assignment)
    
    객체가 가지고 있는 요소를 동일한 이름을 가진 변수에 할당할 수 있음.
    
    ```jsx
    const object={
    	cat : "고양이",
    	dog : "강아지",
    	chicken : "닭",
    	hamster : "햄스터",
    }
    
    //변수 cat과 hamster은 각각 "고양이", "햄스터"라는 값으로 초기화 됨.
    const [cat, hamster] = object
    
    //전개 연산자를 활용할 수 있다.
    //cat = "고양이", object2 = {dog : "강아지, chicken : "닭",	hamster : "햄스터"}
    const [cat, ...object2] = obejct
    ```
    

## 참고 자료

[Axios란?  / Axios  사용 및 서버 통신 해보기!](https://velog.io/@zofqofhtltm8015/Axios-사용법-서버-통신-해보기)

[비동기방식 promise, await, async, axios 정리](https://sudo-minz.tistory.com/28)

[[JS] Axios 라이브러리를 통한 비동기처리 방식](https://velog.io/@unani92/Axios-라이브러리를-통한-비동기처리-방식)
