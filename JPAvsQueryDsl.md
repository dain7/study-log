# Spring Data JAP와 QueryDsl
## Spring Data JPA
- 지루하게 반복되는 CRUD 문제를 세련된 방법으로 해결
- 개발자는 인터페이스만 작성한다.
- 스프링 데이터 JPA가 구현 객체를 동적으로 생성해서 주입

### 쿼리 메서드 기능
- 메서드 이름으로 쿼리를 생성하는 기능

#### 메서드 이름으로 쿼리 생성
- 메서드 이름만으로 JPQL 쿼리를 생성한다.
- 선언된 메서드에 대해서는 애플리케이션 로딩 시점에 쿼리를 다 만들어 버린다 => 에러를 사전에 잡을 수 있다
```java
public interface MemberRepository extends JpaRepository<Member, Long> {
   List<Member> findByName(String username);
}
```
```SQL
SELECT * FROM MEMBER M WHERE M.NAME = "hello"
```
#### 이름으로 검색 + 정렬
- 자주 쓰는 기능 정렬을 위해 sort를 넣어줄 수 있다. 스프링 데이터에서 만들어놨다.
```java
public interface MemberRepository extends JpaRepository<Member, Long> {
   List<Member> findByName(String username, Sort sort);
}
```
#### 이름으로 검색 + 정렬 + 페이징
- 추가로 페이징을 위한 기능도 만들어놨다.
- 실제로 사용할 땐 꼭 DTO로 변환해서 뿌리자.
- Page 객체의 content에 데이터가 나오고, Pageable, sort, first, empty 등이 정보를 얻을 수 있다.
```java
public interface MemberRepository extends JpaRepository<Member, Long> {
   Page<Member> findByName(String username, Pageable pageable);
}
```
#### @Query, JPQL 정의
- @Query 어노테이션을 사용해서 직접 JPQL을 지정할 수 있다.
- 마찬가지로 Named 쿼리도 애플리케이션 로딩 시점에 파싱을 하므로, 이로인해 런타임 에러를 내지 않을 수 있다.
```java
public interface MemberRepository extends JpaRepository<Member, Long> {
   
   @Query("select m from Member m where m.username = ?1")
   Member findByUsername(String username, Pageable pageable);
}
```

## QueryDSL
- SQL, JPQL을 코드로 작성할 수 있도록 도와주는 빌더 api
### SQL, JPQL의 문제점
- SQL, JPAL은 문자열이다. Type-Check가 불가능하다.
- 컴파일 시점에 알 수 있는 방법이 없다 (문자열이기 때문에)
- 해당 로직 실행 전까지 작동여부 확인을 할 수 없다.
- 해당 쿼리 실행 시점에 오류를 발견한다.
### QueryDSL 장점
- 문자가 아닌 코드로 작성하자.
- 컴파일 시점에 문법 오류를 발견하자.
- 코드 자동완성
- 단순하고 쉽다. 코드 모양이 JPQL과 거의 비슷하다.
- 동적 쿼리
### QueryDSL는 동적쿼리다
- 동적쿼리 : 특정 조건들이나 상황에 따라 변경되면 동적쿼리
- 정적쿼리 : 어떤 조건 또는 상황에도 변경되지 않으면 정적쿼리
- JPQL은 정적쿼리이다. 
- QueryDSL은 BooleanBuilder에 조건을 쭉쭉 넣고 쿼리를 실행시키면 된다.
- 원하는 필드만 추려서 DTO로 변환도 가능하다.
```java
String name = "member";
int age = 9;
​
QMember m = QMember.member;
​
BooleanBuilder builder = new BooleanBuilder();
if(name != null) {
   builder.and(m.name.contains(name));
}
if(age != 0) {
   builder.and(m.age.gt(age));
}
​
List<Member> list =
   query.selectFrom(m)
        .where(builder)
        .fetch();
```
### QueryDSL은 자바다.
- 객체지향적인 관점에서 가장 중요하다.
- 중복코드를 함수로 추출해서 재사용할 수 있다. 
```java
return query.selectFrom(coupon)
          .where(
                coupon.type.eq(typeParam),
                isServiceable()
      )
      .fetch();
​
private BooleanExpression isServiceable() {
   return coupon.status.wq("LIVE")
      .and(marketing.viewCount.lt(markting.maxCount));
}
```
