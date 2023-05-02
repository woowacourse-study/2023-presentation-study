## 서론

HTTP 의 요청에는 URI와 원하는 메서드가 포함된다. 

메서드 중 put 과 patch 메서드는 모두 엔티티를 수정하는 용도로 사용된다.

그렇다면, 미션에서 상품정보의 수정에 put과 patch 둘 중 어떤 method를 사용해야 할까?

---

## 정의와 사용법

### put 메서드

**리소스의 전체 수정**

1. 해당하는 자원이 없을시 → 새 엔티티를 **생성**
    
    생성 성공시, 201(created)  코드로 클라이언트에 응답한다
    
2. 해당하는 자원이 존재할 시 → 리소스로 **대체**
    
    엔티티를 업데이트 하고 200(ok), 204 (no content)로 클라이언트에 응답한다.
    

### patch 메서드

**리소스의 부분 수정**

요청된 변경사항만을 수정하고 204(no content) 코드를 반환한다.

---

## 사용예시

### put 메서드

좋아요 / 싫어요 기능 구현해보자

```java
PUT /likes
{
	"id" : 1234,
	"articleId" : 2345, // 원 게시물 번호
	"userId" : "kero", // 좋아요/ 싫어요 한 사람 Id
	"likeType" : "like" // 또는 "dislike" 
}

HTTP/1.1 200 OK
{
	"id" : 1234,
	"articleId" : 2345,
	"userId" : "kero", 
	"likeType" : "like" 
}
```

처음으로 누름 : 생성되어야 한다

기존에 눌렀음 : 다른 타입으로 수정되거나 취소 되어야 한다.

```java
// 서비스 단
@Transactional
public void update(Long articleId, Long userId, LikeType likeType) {
    Like like = likeRepository.findByArticleIdAndUserId(articleId, userId)
      .map(l -> l.setType(likeType))
      .orElse(new Like(articleId, userId, likeType));

    likeRepository.save(like);
}
```

특징 : 해당 자원의 모든 상태를 알고 있어야 한다.

왜냐하면, 요청 경로에 자원이 이미 존재하는 경우 해당 자원을 payload 정보와 교체하기 때문에

일부만 요청을 보낸 경우 null 로 업데이트 된다 
(모든 리소스를 입력해야하기 때문에)

```java
PUT /likes
{
	"id" : 1234,
	"articleId" : 2345, 
	"userId" : "kero", 
}

HTTP/1.1 200 OK
{
	"id" : 1234,
	"articleId" : 2345,
	"userId" : "kero", 
	"likeType" : null
}
```

일부만 업데이트 하고 싶더라도 모든 리소스를 전부 보내주어야 한다.

---

### patch 메서드

이름, 나이, 별명을 가진 회원 정보 중, 나이만 변경하고자 한다.

```java
PATCH /members/1234
{
	"age" : 15
}

HTTP/1.1 200 OK
{
	"name" : "kim",
	"ninkName" : "kero", 
	"age" : 15 
}
```

새롭게 바뀐 부분만 반영되고 나머지는 기존데이터가 유지된다 →  해당 자원의 상태 중 일부만 알고 있어도 된다.

이렇게 하기 위해서는 서버단에서 별도의 로직이 필요하다…

1. 변경되는 부분만을 받아주는 DTO 구현

```java
// DTO
public UpdateAge {
  
    private int age;
  
    public UpdateAge() {}
  
    public UpdateAge(int age) {
        this.age = age;
    }
}

// 서비스 단
@Transactional
public void updateMember(final Long id, final UpdateAge request) {
    Member member = memberRepository.findById(id)
        .orElseThrow(NoSuchElementException::new);
  
    member.update(request);
		memberRepository.save(member);
}
...
  
// member.java
public void update(final UpdateAge request) {
    this.age = request.getAge();
}
```

→ 일일이 dto를 만들어 주어야하는 번거로움이 있음

2. 모두 같은 DTO를 쓰도록 하되, null을 걸러주는 구현

```java
// 서비스 단
@Transactional
public void updateMember(final Long id, final UpdateMember request) {
    Member member = memberRepository.findById(id)
        .orElseThrow(NoSuchElementException::new);
  
		if(request.getName != null) {
			member.setName(request.getName);
		}    

		if(request.getNickName != null) {
			member.setNickName(request.getNickName);
		}  

		if(request.getAge != null) {
			member.setAge(request.getAge);
		}  

		memberRepository.save(member);
}
```

---

## 멱등성

같은 행위를 여러번 반복하더라도 같은 결과를 유지하는 성질

**멱등O 메서드**

- GET: 계속 조회해봤자 리소스는 변하지 않는다.
- PUT: 결과를 대체한다. 따라서 같은 요청을 여러번해도 최종결과는 같다.
- DELETE: 결과를 삭제한다. 같은 요청을 여러번 해도 삭제된 결과는 같다.

**멱등 X 메서드**

- POST : 같은 요청을 계속 호출하면 새로운 리소스가 생성되거나 리소스의 상태가 달라지기 때문에 호출 결과가 달라질 수 있다.

### put 메서드

**멱등성을 가지고 있다** 

리소스를 완전히 교체해버린다 → 같은 요청을 여러번 해도 최종 결과가 같다.

```java
PUT /members/1
{ "name" : "kim" ,
	"age" : 10}
```

### patch 메서드

**멱등으로 설계 할 수도 있고, 아니게 설계할 수도 있다.**

멱등으로 설계

```java
PATCH /members/1
{ "name" : "kero"}
```

이름을 “kero”로 변경하기만 할 뿐이라서 몇번을 반복해도 최종결과가 같음

멱등이 아니도록 설계

```
PATCH /members/1
{ "operation" : "add" ,
	"age" : 10}
```

operation 의 value가 add일 때, age만큼 더해주는 API를 patch로 설계한다면 동작을 할 때마다, 나이가 +10 이 되기 때문에 멱등이 아니다.
