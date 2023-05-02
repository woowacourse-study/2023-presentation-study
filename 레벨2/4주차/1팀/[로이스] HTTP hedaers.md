# HTTP Headers

### Header

HTTP header는 HTTP의 요청과 응답에 필요한 부가적인 정보들을 담고있는 필드이다.

Message나 Body의 데이터에 대한 의미를 변경하거나 조정하는데 사용될 수 있다.

헤더는 대소문자를 구별하지 않으며, 줄의 처음에서 시작하여 바로 다음에 `:`과 헤더에 해당하는 값이 따라온다.

### Header의 종류

Header는 사용되는 목적과 진영에 따라 다양한 방법으로 분류할 수 있다.

RFC에서 정의한 명확한 구분 기준은 없지만, 편의상 분류할 수 있는데 다음과 같이 4개의 헤더로 자주 분류한다.

- **General header** : 요청과 응답 모두에 적용되지만 바디에서 최종적으로 전송되는 데이터와는 관련이 없는 헤더
- **Request  header** : 요청하는 리소스 또는 클라이언트 자체에 대한 정보가 포함되는 헤더
- **Response header** : 서버의 이름, 버전, 위치 등과 같은 서버에 대한 자체정보 또는 응답에 대한 추가 정보가 포함되는 헤더
- **Representation(Entity) header** : MIME 유형 또는 적용된 인코딩/압축과 같은 리소스 본문에 대한 정보가 포함되는 헤더

### General Header

- **Date**
  HTTP 메시지를 생성한 일시 (RFC 1123에서 규정)
  `Date: Tue, 2 May 2023 09:05:12 GMT`

- **Connection**
  클라이언트와 서버 간 연결에 대한 옵션 설정

  `Connection: close`
  `Connection: Keep-Alive`

- **Content-Type**

  HTTP 메세지에 담긴 데이터의 형식을 명시

- **Cache-Control**
  쿠키/캐시 관련된 설정

  ```http
  HTTP/1.1 200 OK 
  Date: Fri, 12 Mar 2021 03:52:32 GMT 
  Server: WSGIServer/0.2 CPython/3.8.0 
  Content-Type: application/json 
  Cache-Control: max-age=15 
  
  {"firstName": "\uad11\ubbfc", "lastName": "\uae40"} 
  ```

  

- **Transfer-Encoding**
  전송되는 메세지의 인코딩 정보 설정

  `Transfer-Encoding: chunked`

  `Transfer-Encoding: compress`
  `Transfer-Encoding: gzip`

- **Trailer**

  청크(chunk) 인코딩된 메세지에 대한 지시어.

  ```http
  HTTP/1.1 200 OK
  Content-Type: text/plain
  Transfer-Encoding: chunked
  Trailer: Expires
  
  7\r\n
  Mozilla\r\n
  9\r\n
  Developer\r\n
  7\r\n
  Network\r\n
  0\r\n
  Expires: Tue, 2 May 2023 09:05:12 GMT\r\n
  \r\n
  ```



### Request Header

- **Host**

  요청하는 호스트와 포트 번호(HTTP/1.1 이후부턴 필수 항목, IP가 아닌 도메인 정보가 담겨야 한다)

  `Host: www.example.com:80`

- **User-agent**
  요청 측의 브라우저 혹은 OS와 관련된 정보를 포함

  `Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_7) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/112.0.0.0 Safari/537.36`

- **Authorization**
  인증 토큰을 보낼 때 사용하는 헤더
  `Authorization: Basic YWxhZGRpbjpvcGVuc2VzYW1l`

  `Authorization: <schem> <credentials>`

- **Cookie**

  이전에 서버로 부터 받은 쿠키를 담는 헤더

  `Cookie: session_id=12345; user_id=67890`

- **Referer**

  현재 요청 직전에 머물고 있던 웹 링크 주소를 담는 헤더

  `Referer: https://www.example.com/previous-page.html`

- **Origin**
  요청의 출처를 담는 헤더
  URI 정보는 포함하지 않고 서버 이름만 포함된다.
  `Origin: https://developer.mozilla.org`
  ***CORS** 에 사용되는 header*

- **Accept**

  요청 측에서 기대하는 데이터 타입을 명시
  `ex. Accept : application/json` -> 서버의 응답 결과가 `application/json` 인 경우만 받겠다는 의미

  \+ `Accept-Charset`, `Accept-Encoding`, `Accept-Language`

  ```http
  Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/ *;q=0.8
  Accept-Language: en-US,en;q=0.5
  Accept-Encoding: gzip, deflate, br
  ```

  ​	q는 우선순위(가중치)

### Response header

- **Server**
  요청을 처리한 웹서버 정보
  `Server: nginx`

  `Server: cloudflare`

- **Location**

  생성된 리소스 주소를 포함하여 응답

  `Location: /product/1`

  \+ Content-Location -> 컨텐츠 협상이 끝난 뒤의 경로 (`/product.html`,  `/product.xml`)

- **ETag**
  리소스의 특정 버전의 식별값을 담는 헤더
  웹 캐시 유효성 검증 -> 컨텐츠가 변경되지 않은 경우 서버는 전체 응답을 다시 전송할 필요가 없다.
  `ETag: W/"6345748b-1f101"`

- **WWW-Authenticate**
  인증을 요구하는 401상태 응답과 함께 사용
  서버에서 각각 다른 인증값을 사용하는 다양한 영역들이 있는데, 이를 표기하는 헤더
  `WWW-Authenticate: Basic realm="Corporate Financials"`

- **Allow**
  `405 Method Not Allowed` 상태 응답과 함께 사용
  요청된 리소스에 대해 허용된 HTTP 메서드를  표기
  `Allow: GET, HEAD, POST`

- **Access-Control-Allow-Origin**
  요청에 대한 자원을 공유할 수 있는 출처(Origin)에 대해 표기
  `Access-Control-Allow-Origin: https://developer.mozilla.org`
  `Access-Control-Allow-Origin: *`

- **Vary**
  서버에서 응답을 생성할 때 고려된 요청 헤더를 표시하는데 사용

  `Vary: Accept-Language, Accept-Encoding, Origin`

- **Retry-After**
  재요청을 하기 전까지 기다려야 하는 시간

  `Retry-After: Wed, 21 Oct 2015 07:28:00 GMT`
  `Retry-After: 120` (120 seconds)

  3가지 case에 사용됨. (리다이렉트, 429 status(Too many request), 503 status(Service Unavailalbe))

### Entity header

- **Content-Type**
  데이터의 형식

  `Content-Type: text/html; charset=utf-8`

- **Content-Encoding**

  데이터 인코딩 방식

  `Content-Encoding: gzip`

- **Content-Language**

  데이터의 자연 언어

  `Content-Languauge: ko`

- **Content-Length**
  데이터의 길이(10진수)
  `Content-Length: 5`

- **Content-Security-Policy**
  컨텐츠 보안 정책 설정
  컨텐츠를 불러올 소스를 명시
  `Content-Security-Policy: default-src 'self' example.com *.example.com; image-src: *`

  => 컨텐츠 불러올 소스 명시(`example.com`, `example.com`의 모든 서브 도메인, origin 으로 부터 오는 모든 컨텐츠 허용. + 이미지(컨텐츠)는 모든 소스 허용)

  

### Header에 표기하는 정보를 Body에 담으면 안되나?

1. 여러 자원에서 공통으로 일관되게 사용되는 정보 (ex 인증)
2. 보안과 관련된 정보
3. 비즈니스 로직에 직접적으로 활용되지 않는 정보 (메타데이터)
   => Header에서 사용하도록 하자