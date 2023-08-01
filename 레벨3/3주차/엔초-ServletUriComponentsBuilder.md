# ServletUriComponentsBuilder

# ServletUriComponentsBuilder란?

- Spring Framework에서 제공하는 유틸리티 클래스
- URI(Uniform Resource Identifier)를 생성하고 조작하는 데 사용
    - URI: 인터넷에서 리소스를 고유하게 식별하는 표준화된 문자열
- 주로 웹 애플리케이션에서 동적인 URI를 생성해야 할 때 사용
    - 컨트롤러에서 클라이언트의 요청을 처리한 후, 해당 요청과 관련된 URI를 생성하는 데 사용
    - 즉, 현재 요청에 대한 정보를 바탕으로 URI를 생성하고 수정할 수 있음
    - ex) 특정 리소스에 대한 URI를 생성하거나, 쿼리 매개변수를 추가하여 동적인 URI를 생성하는 경우

## ServletUriComponentsBuilder 사용법

### 자주 사용하는 메서드

- `**fromCurrentContextPath()`**
    - 현재 **컨텍스트 경로**를 기반으로 `ServletUriComponentsBuilder` 생성
- `**fromCurrentRequest()**`
    - 현재 **요청**을 기반으로 `ServletUriComponentsBuilder` 객체 생성
- `**path(String path)**`
    - 만들어진 uri에 경로 추가
    - 경로 변수를 사용하여 동적인 경로 생성 가능
- `**queryParam(String name, Object... values)**`
    - 만들어진 uri에 쿼리 매개변수 추가

### 사용법

1. `fromCurrentRequest()` 또는 `fromCurrentContextPath()` 메서드를 사용하여 `ServletUriComponentsBuilder` 객체를 생성
    - HttpServletRequest를 파라미터로 받아 생성하기도 함
2. 필요한 경우, `path()` 메서드를 사용하여 경로 추가
3. 필요한 경우, `queryParam()` 메서드를 사용하여 Query Parameter 추가
4. build() 메서드를 호출하여 생성된 URI를 완성합니다.

## 주요 메서드 활용 예시

### `**fromCurrentContextPath()**`

```java
@RestController
@RequestMapping("/members")
@RequiredArgsConstructor
public class MemberController {

    private final MemberService memberService;

    @PostMapping
    public ResponseEntity<Void> create(@RequestBody MemberRequest request) {
        memberService.create(request);

        URI uri = ServletUriComponentsBuilder.fromCurrentContextPath()
                                             .build()
                                             .toUri();

        return ResponseEntity.created(uri).build();
    }
}
```

- http://localhost:8080/members 로 POST 요청을 했다면
uri = “http://localhost:8080”
    
    ![fromCurrentContextPath](https://rapid-browser-8fb.notion.site/image/https%3A%2F%2Fs3-us-west-2.amazonaws.com%2Fsecure.notion-static.com%2Fb0f106db-ff02-4dfa-9aed-07de3a70e952%2F%25E1%2584%2589%25E1%2585%25B3%25E1%2584%258F%25E1%2585%25B3%25E1%2584%2585%25E1%2585%25B5%25E1%2586%25AB%25E1%2584%2589%25E1%2585%25A3%25E1%2586%25BA_2023-07-31_%25E1%2584%258B%25E1%2585%25A9%25E1%2584%2592%25E1%2585%25AE_12.14.42.png?table=block&id=73970f1b-7592-4789-9d68-cfe5e243e73e&spaceId=46a9d753-8b1d-4083-bd7d-4b5f6daf3c1f&width=2000&userId=&cache=v2)
    

### `fromCurrentRequest()`

```java
@RestController
@RequestMapping("/members")
@RequiredArgsConstructor
public class MemberController {

    private final MemberService memberService;

    @PostMapping
    public ResponseEntity<Void> create(@RequestBody MemberRequest request) {
        memberService.create(request);

        URI uri = ServletUriComponentsBuilder.fromCurrentRequest()
                                             .build()
                                             .toUri();

        return ResponseEntity.created(uri).build();
    }
}
```

- http://localhost:8080/members 로 POST 요청을 했다면
uri = “http://localhost:8080/members”
    
    ![fromCurrentRequest](https://rapid-browser-8fb.notion.site/image/https%3A%2F%2Fs3-us-west-2.amazonaws.com%2Fsecure.notion-static.com%2F12b1b872-d0d8-4e3f-89ef-bf8e00e61c08%2F%25E1%2584%2589%25E1%2585%25B3%25E1%2584%258F%25E1%2585%25B3%25E1%2584%2585%25E1%2585%25B5%25E1%2586%25AB%25E1%2584%2589%25E1%2585%25A3%25E1%2586%25BA_2023-07-31_%25E1%2584%258B%25E1%2585%25A9%25E1%2584%2592%25E1%2585%25AE_12.15.44.png?table=block&id=9f33552f-8fa9-4dc0-9062-4a7b6e0ae13d&spaceId=46a9d753-8b1d-4083-bd7d-4b5f6daf3c1f&width=2000&userId=&cache=v2)
    

### `path(String path)`

```java
@RestController
@RequestMapping("/members")
@RequiredArgsConstructor
public class MemberController {

    private final MemberService memberService;

    @PostMapping
    public ResponseEntity<Void> create(@RequestBody final MemberRequest request) {
        final Long memberId = memberService.create(request);

//        URI uri = ServletUriComponentsBuilder.fromCurrentRequest()
//                                             .path("/" + memberId)
//                                             .build()
//                                             .toUri();

        URI uri = ServletUriComponentsBuilder.fromCurrentRequest()
                                              .path("/{memberId}")
                                              .buildAndExpand(memberId)
                                              .toUri();

        return ResponseEntity.created(uri).build();
    }
}
```

- http://localhost:8080/members 로 POST 요청을 했다면
uri = “http://localhost:8080/members/{memberId}”
    
    ![path](https://rapid-browser-8fb.notion.site/image/https%3A%2F%2Fs3-us-west-2.amazonaws.com%2Fsecure.notion-static.com%2Fcdc155c9-d0f2-4c6a-92f3-0f784efcce70%2F%25E1%2584%2589%25E1%2585%25B3%25E1%2584%258F%25E1%2585%25B3%25E1%2584%2585%25E1%2585%25B5%25E1%2586%25AB%25E1%2584%2589%25E1%2585%25A3%25E1%2586%25BA_2023-07-31_%25E1%2584%258B%25E1%2585%25A9%25E1%2584%2592%25E1%2585%25AE_1.09.45.png?table=block&id=94fcce8a-0844-40b4-8f95-494f1d2a447d&spaceId=46a9d753-8b1d-4083-bd7d-4b5f6daf3c1f&width=2000&userId=&cache=v2)
    

### `queryParam(String name, Object... values)`

```java
@RestController
@RequestMapping("/products")
@RequiredArgsConstructor
public class ProductController {

    private final ProductService productService;

    @PostMapping
    public ResponseEntity<Void> create() {
        productService.create(request);

        URI uri = ServletUriComponentsBuilder
                            .fromCurrentRequest()
                            .queryParam("category", "electronics", "fashion")
														.queryParam("sort", "time")
                            .build()
                            .toUri();

        return ResponseEntity.created(uri).build();
    }
}
```

```java
@RestController
@RequestMapping("/products")
@RequiredArgsConstructor
public class ProductController {

    private final ProductService productService;

    @PostMapping
    public ResponseEntity<Void> create() {
        productService.create(request);

        String[] categories = {"electronics", "fashion"};
        URI uri = ServletUriComponentsBuilder.fromCurrentRequest()
                                             .queryParam("category", categories)
                                             .build()
                                             .toUri();

        return ResponseEntity.created(uri).build();
    }
}
```

- http://localhost:8080/products 로 요청했다면
uri = “http://localhost:8080/products/category=electronics&category=fashion”
    
    ![queryParam](https://rapid-browser-8fb.notion.site/image/https%3A%2F%2Fs3-us-west-2.amazonaws.com%2Fsecure.notion-static.com%2F68e95b32-728d-43d3-bc67-2face7be4caa%2FUntitled.png?table=block&id=fa595dde-6f6f-467b-8f64-2b90454afbe2&spaceId=46a9d753-8b1d-4083-bd7d-4b5f6daf3c1f&width=2000&userId=&cache=v2)
