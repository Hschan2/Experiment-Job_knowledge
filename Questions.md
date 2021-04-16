## 유튜브 개발바닥에서 물어보는 질문들이란? [Link](https://www.youtube.com/watch?v=3ArYMq5AomI&t=6s)

### HttpSession으로 개발했을 때, Key 값이 같다면 어떻게 분리하는가?
보통 DB 같은 경우 Query Parameter가 계속 바뀌어 전송되어 다른 값을 구분해서 가져오지만 Session은 이미 같은 Key 값으로 고정되어 있는 상태에서 사용자에 따라 구분한다.   

스프링을 공부하면서 <b>HttpSession</b>으로 개발을 경험해보았지만 <b>Key값이 같을 때</b>, 분리하여 사용하는 방법을 생각해 본 적이 없었다. 그래서 한 번 검색해 보았다.   

모든 BackEnd 언어의 Session의 <b>동작 방식이 동일</b>하다고 한다. 이는 스프링의 HttpSession을 비롯하여 모든 BackEnd 언어의 동작 방식이 같다는 것이다.   

서버에서 Session을 사용을 설정할 경우에는 클라이언트가 해당 서버에 접속했을 때, 서버에는 Session 파일을 하나 생성한다. 그리고 서버는 브라우저에 JSESSIONID라는 <b>쿠키</b>를 전송한다. (쿠키명 변경 가능)   

이제 브라우저는 다른 페이지로 이동할 때마다 JSESSIONID를 서버로 <b>함께 전송</b>한다. 그러면 서버는 JSESSIONID 쿠키 값을 보고 A 브라우저는 A Session이 매칭되었다는 것을 인지하고 <b>사용자를 확인</b>할 수 있다.   

브라우저마다 Session 값이 겹치지 않는 이유는 브라우저 별로 별도의 쿠키를 관리하고 쿠키를 별도로 관리하기 때문에 서버에서는 같은 사용자로 인식하지 못한다. 예를 들어 로그인 기능을 만들 때, 다른 브라우저에서 같은 아이디로 접속하더라도 쿠키명이 달라지게 된다.   

국내에서는 이렇게 쿠키를 사용하여 인증 방식을 채택하였지만, 보안 문제가 발생하였다. 그래서 Session을 이용해서 중요한 데이터는 서버가 관리하고, 해당 Session의 ID (기본키)를 클라이언트의 브라우저에 전달하여 인증을 구현한다.   

위의 내용은 <b>Servlet</b>에서 지원한다.   
서버는 HttpServletRequest에서 getSession을 통해 HttpSession을 가져다 사용하면 되며 클라이언트에는 브라우저가 서버의 자원을 요청할 시, 항상 쿠키를 Header에 함께 포함시키기 때문에 별도의 작업은 필요하지 않으나 Cookie 사용 안함 시 Session은 유지되지 않는다.   

### Shell Script로 배포 스크립트를 만들었을 때, 다른 언오와의 return의 차이점은?   
-

### JPA N + 1이 발생했을 때, 원인과 해결 방법이 무엇이며 Join 쿼리를 이용하면 어떻게 되는가?
JPQL을 실행하면 JPA는 이를 분석하여 SQL를 생성한다. JPQL 입장에서는 즉시 로딩, 자연 로딩과 같은 글로벌 패치 전략을 무시하고 JPQL만 사용해서 SQL을 생성한다.   

JPQL은 특징이 있다. findById()같은 경우에는 엔티티를 영속성 컨텍스트에서 먼저 찾고 영속성 컨텍스트에 없는 경우 DB에서 찾지만 JPQL은 항상 DB에 SQL을 실행해서 결과를 조회한다. 그리고 다음과 같은 작업을 진행한다.   
1. JPQL을 호출하면 DB에 우선적으로 조회   
2. 조회한 값을 영속성 컨텍스트에 저장   
3. 영속성 컨텍스트에 조회할 때 이미 존재하는 데이터가 있다면 데이터를 버림   

JPA N + 1이 발생하는 이유는 JPQL에서 글로벌 패치 전략을 완전히 무시하고 SQL을 생선한다.   

findAll()의 경우 아래처럼 호출한다.
```
SELECT * FROM MEMBER
```
즉시 로딩인 경우
```
val members = memberRepository.findAll()
```
JPQL에서 동작한 쿼리를 통해 members에 데이터가 바인딩된다. 그 이후 JPA에서는 글로벌 패치 전략(즉시 로딩)을 받아 들여 해당 member에 대해 추가적인 LAZY 로딩으로 N + 1을 발생시킨다.   

지연 로딩인 경우
```
val members = memberRepository.findAll()
```
동일하게 members에 데이터가 바인딩되지만 JPA가 글로벌 패치 전략을 받아들이지만 지연 로딩이기 때문에 추가적인 SQL을 발생시키지 않는다. 하지만 LAZY 로딩으로 추가 작업을 진행하면 N + 1 문제가 발생한다.   

#### 해결 방법
1. Batch Size   
@BatchSize(size = 크기)를 지정한다. size 만큼 데이터를 미리 로딩하며 연관된 엔티티를 조회할 때 size만큼 WHERE IN 쿼리를 통해 조회하게 되고 size를 넘어가게 되면 WHERE IN 쿼리를 진행한다. 그러나 글로벌 패치 전략을 변경해야 하며, 정해진 Batch Size만큼 조회되는 것도 고정되기 때문에 권장하지 않는다.   

2. Fetch Join   
쿼리 조회를 위한 Interface를 생성한 후
```
interface MemberRepository : JpaRepository<Member, Long> {
    @Query(
            "select m from Member m left join fetch m.orders"
    )
    
    fun findAllWithFetch(): List<Member>
}

혹은

@Repository
public interface PostRepository extends JpaRepository<Post, Long> {
    @Query("select p from Post p left join fetch p.commentList")
    List<Post> findAllWithFetchJoin(); 
    // findAllWithFetchJoin을 사용함으로써 N + 1 문제를 발생하지 않게 만들 수 있다.
}
```
Fetch Join을 사용한다.
```
@Test
    internal fun `페치 조인 사용`() {
        val members = memberRepository.findAllWithFetch()
        // 조회한 모든 회원에 대해서 조회하는 경우에도 N+1 문제가 발생하지 않음
        for (member in members) {
            println("order size: ${member.orders.size}")
        }
    }
```
가장 많이 사용하는 방법으로 Fetch를 통해 Join 쿼리를 진행한다. Fetch 키워드를 사용하면 연관된 엔티티나 컬렉션을 한 번에 동시 조회할 수 있다. 즉, Fetch Join을 사용하게 되면 연관된 엔티티는 프록시가 아닌 실제 엔티티를 조회하게 되므로 연관 관계 객체까지 한 번의 쿼리로 가져올 수 있다.   

#### 일반 Join의 경우
```
interface MemberRepository : JpaRepository<Member, Long> {
    @Query(
        "select m from Member m join m.orders" // fetch를 제거
    )
    
    fun findAllWithFetch(): List<Member>
}

@Test
internal fun `페치 조인 키워드 제거`() {
    val members = memberRepository.findAllWithFetch() // 패치 타입 Lazy 경우
    // 패치 조인하지 않은 상태에서는 N+1 문제 발생
    for (member in members) {
        println("order size: ${member.orders.size}")
    }
}
```
조인을 통해 연관 관계 컬렉션까지 함께 조회되는 것처럼 보이지만 JPQL은 결과를 반환할 때, 연관 관계까지 고려하지 않고 SELECT 절에 지정한 엔티티만 조회한다. 따라서 컬렉션은 초기화하지 않은 컬렉션 레퍼를 반환하게 되고 컬렉션이 없기 때문에 LAZY 로딩이 발생하게 되고 결과적으로 N + 1 문제가 발생한다.   

#### Fetch Join의 한계
1. 컬렉션을 Fetch Join을 하면 페이징 API를 사용할 수 없다. (메모리 문제)
2. 둘 이상 컬렉션을 Fetch할 수 없다.

### 단방향과 양방향 바인딩의 차이점

### 로그인 페이지 구현
아이디와 비밀번호를 서버에서 확인해서 Session과 Cookie를 전송
HTTP Header, JSON Body

### 네트워크 예외 처리
status code, try catch문, 네트워크 라이브러리, axios의 공통적인 에러처리 방법

### 페이지 접속 혹은 자바스크립트의 속도가 느릴 때 확인하는 방법
