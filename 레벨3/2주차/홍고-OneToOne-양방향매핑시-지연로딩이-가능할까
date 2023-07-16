# OneToOne 양방향매핑시 지연로딩이 가능할까?
## TargetEntity에서 지연로딩 불가능
결론만 먼저 말하자면 `SourceEntity`를 조회할 때는 지연로딩이 가능하지만, `TargetEntity`를 조회할 때는 지연로딩이 불가능하다.

`User`와 `UserInfo`가 OneToOne 양방향으로 연결되어있으며, `User`가 외래키를 관리하고 있다. 이 때 외래키 관리자인 `User`를 `SourceEntity`, `User`에게 참조되고 있는 `UserInfo`를 `TargetEntity`라고 해보자. fetch는 지연 로딩으로 설정해두었다.

```java
@Entity
@Table(name = "user")
public class User {
    @Id
    @GeneratedValue
    private Long id;

    @Column(nullable = false)
    private String name;

    @OneToOne(cascade = CascadeType.PERSIST, fetch = FetchType.LAZY)
    @JoinColumn(nullable = false)
    private UserInfo userInfo;
    
    public void setUserInfo(final UserInfo userInfo) {
        this.userInfo = userInfo;
    }
}
```
```java
@Entity
@Table(name = "user_info")
public class UserInfo {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @OneToOne(mappedBy = "userInfo", fetch = FetchType.LAZY)
    private User user;
    
    public UserInfo(User user) {
        this.user = user;
        user.setUserInfo(this);
    }
}
```

## User(SourceEntity) 조회시 수행되는 쿼리
```java
@SpringBootTest
public class UserRepositoryTest {
    
    ...
        
    @Test
    void findUserById() {
        User user = new User("홍고");
        UserInfo userInfo = new UserInfo(user);
        userRepository.save(user);

        userRepository.findById(user.getId());
    }
}
```

```
Hibernate: 
    select
        user0_.id as id1_2_0_,
        user0_.name as name2_2_0_,
        user0_.user_info_id as user_inf3_2_0_ 
    from
        user user0_ 
    where
        user0_.id=?
```
`SourceEntity`인 `User`는 지연 로딩이 수행된다.

## UserInfo(TargetEntity) 조회시
```java
@SpringBootTest
public class UserInfoRepositoryTest {
    
    ...
        
    @Test
    void findUserInfo() {
        User user = new User("홍고");
        UserInfo userInfo = new UserInfo(user);
        userRepository.save(user);

        userInfoRepository.findById(userInfo.getId()).get();
    }
}
```

```
Hibernate: 
    select
        userinfo0_.id as id1_2_0_ 
    from
        user_info userinfo0_ 
    where
        userinfo0_.id=?
2023-07-16 18:18:06.477 TRACE 12448 --- [    Test worker] o.h.type.descriptor.sql.BasicBinder      : binding parameter [1] as [BIGINT] - [1]
Hibernate: 
    select
        user0_.id as id1_1_0_,
        user0_.name as name2_1_0_,
        user0_.user_info_id as user_inf3_1_0_ 
    from
        user user0_ 
    where
        user0_.user_info_id=?
```
`TargetEntity`인 `UserInfo`는 fetch를 Lazy로 설정해도 즉시 로딩이 되는 것을 볼 수 있다.

### 왜 무조건 즉시로딩이 될까?
지연로딩시 jpa는 연관 객체를 null 또는 프록시 객체로 할당해둔다. 연관 객체가 존재한다면 프록시 객체를, 존재하지 않는다면 null을 할당한다.

하지만 OTO 양방향 관계일 경우 DB 관점에서 `TargetEntity`는 `SourceEntity`에 대한 컬럼이 없으므로, 연관 객체의 존재 여부를 판단할 수 없다. 때문에 `SourceEntity`의 존재 여부를 알기 위한 select 쿼리가 필요핟고 한다.

이는 `@OneToOne(optional = false)`, `@JoinColumn(nullable = false)`로 설정해도 무조건 즉시 로딩으로 수행된다. 지연 로딩으로 불러오고 싶다면, 단방향 매핑으로 바꾸거나 `@OneToMany`, `@ManyToOne`으로 매핑해주어야 한다.

> `@OneToMany`는 왜 지연로딩이 가능할까?

> `@OneToMany`일 때 외래키 관리자가 `Many`측이라면, `@OneToOne`과 마찬가지로 `One`인 DB Entity 쪽에서 연관 객체의 존재여부를 알 수 없다. 그러나 `@OneToOne`과 달리 `@OneToMany`의 `One`객체는 `Many`를 참조하는 필드를 컬렉션으로 관리한다.

> 때문에 연관 객체를 null 또는 프록시로 할당하지 않고, 무조건 프록시 객체(컬렉션)을 만든 후 만약 연관된 객체가 없다면 빈 컬렉션을, 연관된 객체가 있다면 실제 DB에서 조회해 온 객체를 사용한다고 한다.

### 즉시 로딩이 나빠?
그냥 즉시 로딩하면 안되는 건가? 즉시 로딩을 하면 어떤 단점이 있을까?

위 예시에서는 하나의 `UserInfo`만 조회를 했었다. 이번에는 여러 명의 `UserInfo`를 조회해보자.

```java
@Test
void findAll() {
    User user1 = new User("홍고");
    UserInfo userInfo1 = new UserInfo(user1);
    userRepository.save(user1);

    User user2 = new User("엔초");
    UserInfo userInfo2 = new UserInfo(user2);
    userRepository.save(user2);

    userInfoRepository.findAll();
}
```
쿼리 결과는?
```
Hibernate: 
    select
        userinfo0_.id as id1_2_ 
    from
        user_info userinfo0_
Hibernate: 
    select
        user0_.id as id1_1_0_,
        user0_.name as name2_1_0_,
        user0_.user_info_id as user_inf3_1_0_ 
    from
        user user0_ 
    where
        user0_.user_info_id=?
2023-07-16 21:55:25.445 TRACE 8316 --- [    Test worker] o.h.type.descriptor.sql.BasicBinder      : binding parameter [1] as [BIGINT] - [1]
Hibernate: 
    select
        user0_.id as id1_1_0_,
        user0_.name as name2_1_0_,
        user0_.user_info_id as user_inf3_1_0_ 
    from
        user user0_ 
    where
        user0_.user_info_id=?
```
두 번의 select문이 수행되었다! 각 `UserInfo`를 찾아올때마다 해당 `UserInfo`의 `User`가 있는지 알기 위해 `UserInfo`개수만큼의 select문이 추가로 실행된다. `UserInfo`가 10만명이라면 10만번의 쿼리가 추가적으로 실행되는 셈으로 성능에 큰 영향을 끼칠 것이다.

## 해결?
### fetch join 사용
즉시 로딩으로 인해 쿼리가 두 번 날아가는 게 싫다면, `fetch join`을 고려해볼 수 있다.

`fetch join`은 JPQL에서 성능 최적화를 위해 제공하는 기능이다. `@Query`어노테이션을 사용해서 `fetch join`을 적용할 수 있다.

```java
public interface UserInfoRepository extends JpaRepository<UserInfo, Long> {
    @Query("select distinct userinfo from UserInfo userinfo left join fetch userinfo.user")
    @Override
    Optional<UserInfo> findById(Long aLong);
}
```

```
Hibernate: 
    select
        distinct userinfo0_.id as id1_2_0_,
        user1_.id as id1_1_1_,
        user1_.name as name2_1_1_,
        user1_.user_info_id as user_inf3_1_1_ 
    from
        user_info userinfo0_ 
    left outer join
        user user1_ 
            on userinfo0_.id=user1_.user_info_id
```
`fetch join`을 적용하기 이전에는 `UserInfo`개수만큼의 쿼리가 추가적으로 실행되었지만, 이번에는 `join`을 사용해 하나의 쿼리만 발생한 것을 볼 수 있다.

## 내 생각
`@OneToOne` 양방향 매핑일 때 `SourceEntity`를 조회할 때는 지연로딩이 가능하지만, `TargetEntity`를 조회할 때는 무조건 즉시 로딩이 수행되었다.
그리고 즉시 로딩을 하면 여러 개의 `TargetEntity`를 조회할 때 그 개수만큼 추가적으로 select쿼리를 날려야한다는 것을 알게되었다.

`fetch join`을 사용하면 즉시 로딩때 여러 번의 select쿼리가 발생하는 것을 막을 수 있다. 그러나 `UserInfo`를 조회해올 때 `User`의 정보까지 한 번에 가져오는 것은 변함이 없으므로, 지연 로딩처럼 `User`가 필요한 시점에서 `User`의 정보를 조회해올수는 없다.

`@OneToOne` 양방향 매핑을 사용하면 양쪽 객체에서 바로 서로를 참조할 수 있기 때문에 편해보이지만, 무조건 즉시 로딩을 해야하며 양 객체가 생성되는 시점이 같을 경우 매핑이 어렵다는 단점이 있다.

때문에 `@OneToOne` 양방향 매핑은 지양하는 게 좋을 것 같다. 뚜렷한 해결책을 찾지못한 것 같아 기분이 찝찝하다! `@OneToOne` 양방향 매핑 관계를 표현하고 싶을 때 어떻게 보안할 것인지 더 학습해봐야겠다.

## 참고
https://jeong-pro.tistory.com/249
https://kihwan95.tistory.com/12
