>※ 인프런 김영한님 JPA 강의를 참고하여 정리한 내용입니다.

<br>

# :: ORM (Object-Relational Mapping)

Object Class - Relational DB 연결

- 개발자가 SQL문을 사용하는 것이 아니라
- 메서드를 이용해서 데이터베이스를 조작하고,
- 내부적으로만 쿼리가 동장하기 때문에
- 개발자는 비즈니스 로직에 집중 가능
- 객체는 객체대로,  관계형DB는 관계형DB대로
- **객체지향적인 코드 작성 가능 ⇒ 유지보수 용이**
<br>
---
<br>

# :: JPA (Java Persistence Api)

Java에서 제공하는 ORM 기술에 대한 API    /   Java ORM 기술 표준
<br>

---
<br>

# :: JPA CRUD

> 저장　:　jpa.persist(user)
조회　:　User user = jpa.find(userId)
수정　:　user.setName(”~~~”)
삭제　:　jpa.remove(user)
> 

<br>

---
<br>

# :: FLUSH 플러시란 ?

**영속성 컨텍스트의 변경내용을 DB에 반영해주는 작업**

- 플러시 과정
    1. 변경 감지
    2. 수정된 엔티티 쓰기 지연 SQL 저장소에 등록
    3. 쓰지 지연 SQL 저장소의 쿼리를 데이터베이스에 전송

- 영속성 컨텍스트를 플러시하는 방법
    
    em.flush()　　　 :　플러시 **직접** 호출
    
    transaction 커밋 :　플러시 **자동** 호출
    
    JPQL 쿼리 실행　:　플러시 **자동** 호출
    
<br>

### ❗ 주의 ❗

영속성 컨텍스트를 비우는 작업이 아님!! 데이터베이스에 동기화 하는 작업임

<br>

---
<br>

# :: 준영속 상태란 ?

**영속 상태였다가 영속 상태에서 빠지는 상태**

- 준영속 상태로 만드는 방법 <br>
    
    em.detach(entity)　 :　특정 Entity만 준영속 상태로 전환 <br>
    
    em.clear()　　　　　:　영속성 컨텍스트를 완전히 초기화 <br>
    
    em.close()　　　　　:　영속성 컨텍스트를 종료
    
<br>

---
<br>

# :: JPA 구동 방식

![](https://images.velog.io/images/palacer/post/d481c08c-bd2e-4616-90e7-b0cd28dfa2f3/image.png)

**Jpa 정석 Code**

```java
//  "hello" = persistence-unit > name
//  **EntityManagerFactory는 Application 로딩 시점에 딱 하나만 만들어서 사용하고 공유**
EntityManagerFactory emf = Persistence.createEntityManagerFactory("hello"); 

//  Transaction 단위 - 실제 DB connection을 얻어서 SQL을 전송하고 결과를 받는
//  그런 단위의 작업을 할 때 EntityManager를 만들어서 씀
EntityManager em = emf.createEntityManager();

// Transaction 생성
EntityTransaction tx = em.getTransaction();
// Transaction 시작
tx.begin();

try {
    /*  저장
	Member member = new Member();
	member.setId(1L);
	member.setName("AA");    // 여기까지는 비영속
	em.persist(member);      // 여기서 영속 상태가 됨 (여기까지 아직 쿼리 안날림)
							// 1차 캐시에 저장됨
	*/

	/*  조회
	Member findMember = em.find(Member.class, 1L);   //  1L = PK값 (user_id)
													//  1차 캐시에서 조회
													//  SELECT 쿼리 없이도 AA Member 가져옴
	*/

	/*   수정
	Member findMember = em.find(Member.class, 1L);
	findMember.setName("BB");
	어떻게 Java 객체에서 setName으로 값만 바꿨는데 이게 DB 수정이 되는걸까?
	em.persist(findMember) 이런식으로 등록을 해줘야 하는거 아니야???
	em.update() 이런게 있어야 하지 않아??
	Jpa가 관리하는 객체는 데이터 변경 여부를 transaction commit 시점에서
	체크하고, 알아서 변경해준다. (Dirty Checking)
	**Jpa의 모든 데이터 변경은  트랜잭션 안에서 실행해야 함**
	*/

	/*   삭제
	Member findMember = em.find(Member.class, 1L);
	em.remove(findMember);
	*/

	/*  같은 객체 조회 2번 ? 
    
    // 여기서 영속성 컨텍스트(entityManager)에 올림(캐시에 등록)
	Member findMember1 = em.find(Member.class, 2L);  
    
    // 여기서는 같은 SELECT 쿼리가 안나감
	Member findMember2 = em.find(Member.class, 2L);
	// (사실 쿼리는 commit 시점에 날라가니까 쿼리도 쌓임)
	// findMember2는 캐시에서 가져옴
	//   findMember1 == findMember2    true / 당연히 같은 객체!
	*/

	tx.commit();   // 커밋 시점에 영속성 컨텍스트에 쌓인 쿼리, 변경사항 등을 DB에 반영
} catch (Exception e) {
	tx.rollback();
} finally {
	// **EntityManager는 쓰레드간 공유 X (사용하고 바로 버린다)**
	em.close();
}

emf.close();
```

<br>

---
<br>

# :: JPA Annotations

> @Entity                Jpa가 관리할 객체

@Table　　데이터베이스 Table 이름 매핑

@Id　　　 데이터베이스 PK와 매핑 (직접할당) / DB에 위임(자동생성)할 경우 @GeneratedValue 필요

@Enumerated　　EnumType 사용(java)  - **EnumType.String만 사용할 것**

@Temporal　　DATE, TIME, TIMESTAMP 사용 / JAVA 8 이상부터는 LocalDate, LocalDateTime 사용 　　　　　　　(@Temporal 사용 불필요)

@Lob　　CLOB, BLOB 사용 / String이면 자동으로 CLOB , 나머지는 BLOB 매핑

@Transient　　DB랑 관계없이 메모리에서만 쓰고 싶은 경우 (매핑x)

@Column 　　데이터베이스 Column 이름 매핑
      -　　 name                      필드와 컬럼 매핑 이름 설정
      - 　　nullable                   NULL / NOT NULL
      - 　　unique                    uniqueConstaints와 같지만, 이름이 랜덤 생성이라 잘 사용x
      - 　　length                     길이 제약조건 (String 타입에만 사용)
      - 　　columnDefinition    컬럼 정보 직접 부여 (varchar(100) default ‘EMPTY’

<br>

> ❓ **데이터베이스 스키마 자동 생성**
>- DDL을 application 실행 시점에 자동 생성
>- 데이터베이스 dialect를 활용해서 데이터베이스에 맞는 적절한 DDL 생성
>- 이렇게 생성된 DDL은 개발에서만 사용  운영 서버에서는 사용하면 안됨
> <br>
>
>**hibernate.hbm2ddl.auto 옵션**
>- create              :  기존 테이블 삭제 후 다시 생성 (DROP + CREATE)
>- create-drop     :  create와 같으나, 종료시점에는 테이블 DROP (테스트케이스에 사용)
>- update             : 변경 사항만 반영 (컬럼 추가만 가능, 삭제는 불가능)
>- validate           :  엔티티와 테이블 매핑 상태가 정상인지만 확인
>- none                : 사용하지 않음
> <br>
>
>**제약조건 추가**
>- @Column(nullable = false, unique = true, length = 10)
> <br>
>
>**운영 장비에는 절대 create, create-drop, update 사용하지 않는다.
>가급적이면 개발초기에만 사용하고 DDL은 참고 및 가공하여 직접 짠 스크립트 사용!**
>- 개발 초기                     -    create / update    사용
>- 테스트 서버                 -    update, validate   사용
>- 스테이징 및 운영서버  -    validate, none      사용

<br>

>❓ **@Enitity**
- 해당 클래스를 JPA가 관리
- JPA를 사용해서 테이블과 매핑할 클래스라면 필수
- 기본 생성자 필수 (파라미터가 없는 public / protected 생성자)
- final 클래스, inner 클래스, interface 사용 x
- DB에 저장할 필드에 final 사용 x**

<br>

> ❓ **기본키 식별 권장 전략**
- not null, 유일 값, 변하지 않는 값
- 먼 미래에도 변하지 않는 값을 사용해야 함 (만족하는 자연키를 찾는게 어려움)
- 비즈니스와 전혀 상관 없는 값 사용 권장 (랜덤이라던가.. generatedValue라던가..)
- 대체키를 사용하자
- **권장** : Long + 대체키 + 키 생성 전략 사용 (UUID)

<br>

>❓ **IDENTITY 전략 특징**
- 보통 auto_increment
- id 값을 DB에 INSERT 완료한 이후에만 알 수 있음
- JPA 입장에서 id(PK) 값을 알 수 없는데 어떻게 PK값을 이용하여 조회가 가능하냐?
이 경우에는 어쩔 수 없이 em.persist(객체) 명령으로  INSERT문을 날리고
내부적으로 PK값을 리턴받은 이후(자동으로 진행됨) getId() 이런식으로 사용

<br>

---
<br>

# :: 연관관계 매핑

- 단방향 연관관계

```java
/*   Getter, Setter 코드 생략   */

/* -----------------------Member---------------------------------------- */

@Entity
public class Member {
	
	@Id @GeneratedValue
	@Column(name="MEMBER_ID")
	private Long id;
	
	@Column(name="USERNAME")
	private String userName;
	
//  객체지향스럽지 않음
//	@Column(name="TEAM_ID")    // TEAM_ID 값으로 매핑하는건 객체지향스럽지 않음
//	private long teamId;       // TEAM 객체 자체를 매핑해야함
	
	@ManyToOne                   //   Member : Team = N : 1 (Member 기준 ManyToOne)
	@JoinColumn(name="TEAM_ID")  //   team과 Team Entity의 TEAM_ID 컬럼 매핑!
	private Team team;

/* -----------------------Team---------------------------------------- */

@Entity
public class Team {
	
	@Id @GeneratedValue
	@Column(name="TEAM_ID")
	private Long id;

	private String name;

/* -----------------------Application---------------------------------------- */

Team team = new Team();
team.setName("TeamA");
em.persist(team);         // name = TeamA 인 Team이 영속성 컨텍스트에 올라가고 PK 부여받음

Member member = new Member();
member.setUserName("member1");
member.setTeam(team);    // Team 정보에  name = TeamA인 Team이 등록되고
em.persist(member);      // 영속성 컨텍스트에 member도 올라감
			
Member findMember = em.find(Member.class, member.getId()); // 부여받은 PK값으로 Member 찾고

Team findTeam = findMember.getTeam();                      // getter로 Team 정보 가져옴
System.out.println("findTeam = " + findTeam.getName());    // findTeam = TeamA  결과 출력

Team newTeam = em.find(Team.class, 100L);
findMember.setTeam(newTeam);         // setTeam()으로 새로운 Team 객체를 설정하면
									 // 자동으로 TEAM ID FK 값을 변경한다
```
<br>

- 양방향 연관관계

```java
/* -----------------------Team---------------------------------------- */

@Entity
public class Team {
	
	@Id @GeneratedValue
	@Column(name="TEAM_ID")
	private Long id;
	private String name;
	
	@OneToMany(mappedBy="team")   // Member Entity에 team과 Mappeing 되어 있다
	private List<Member> members = new ArrayList<Member>(); // ManyToOne <--> OneToMany

/* -----------------------Application---------------------------------------- */

Member findMember = em.find(Member.class, member.getId());
			
List<Member> members = findMember.getTeam().getMembers();  // Team과 Member 사이를 왔다갔다 가능
														   // 양방향 연관관계
			
for(Member m : members) {
    System.out.println("member : " + m.getUserName());     // member : member1 결과 출력
}
```

<br>

>❓ **연관관계의 주인(Owner)과 mappedBy**
> <br>
>
>- **객체와 테이블간 연관관계를 맺는 차이를 이해해야 한다**
        - 객체는 연관관계가 2개 (서로 다른 단방향 2개 = 우리가 편하게 양방향이라고 부름)
           Member → Team  ,  Team → Member  (서로 참조값을 넣어놔야 함)
        - 테이블은 연관관계 1개
           FK값 하나로 연관관계가 끝난다.
> <br>
>
>- **양방향 매핑 규칙**
        - 객체의 두 관계 중 하나를 연관관계의 주인으로 지정
        - 연관관계의 주인만이 외래키를 관리(등록, 수정)
        - 주인이 아니면 읽기만 가능
        - 주인은 mappedBy 속성 사용 X
        - 주인이 아니면 mappedBy 속성 사용
        - 외래 키가 있는 곳을 보통 주인으로 지정 (DB에서 N쪽)
        - DB에서 보면, (1 : N)에서 외래키가 있는 곳이 항상 N
        -** 위 코드의 경우 Member.team이 연관관계의 주인      **(Member가 ‘다’)
        -** Team.members에 아무리 수정해봤자 DB에 안들어감 **(Team이 ‘일’)
> <br>
>
>- **양방향 매핑시 많이 하는 실수?**
     연관관계의 주인에 값을 입력하지 않고, 주인이 아닌쪽에 입력함
        Team 만들고, member에다가 setTeam() 해주면 **끝** (Jpa 입장에선 이게 맞음)
        **하지만**, 순수 객체 상태를 고려해서 항상 양쪽에 다 데이터 넣어주자.
        [  team.getMembers().add(member)  ]
        <U>잊지 않기 위해 **편의 메서드**를 만들자 (**주인에**) </U>
> 
>
>     public void setTeam(Team team) {
>         this.team = team;
>         team.getMembers().add(this);
>     }
>
>- **setter 보다는, 메서드 이름을 changeTeam 이런식으로 변경을 해서 사용하자 !**
> <br>
>
>- **Controller에서는 Entity를 절대 반환하지 말기 !**
      JSON으로 변환하면서 양방향 매핑된 데이터들 무한 toString()...
      Dto (데이터만 딱 들어있는)를 사용할 것

<br>

---
<br>

# :: JPQL

- JPA는 엔티티 객체를 중심으로 개발 (SQL은 테이블)
- 검색도 테이블이 아닌 엔티티 객체를 대상으로 함
- 모든 DB 데이터를 객체로 변환하는 것은 불가능하기 때문에
    
    필요한 데이터만 DB에서 가져오려면 검색 조건이 포함된 SQL이 필요하긴 함
    

ex) 나이가 18살 이상인 회원을 모두 검색하고 싶다 ?

ex) 회원 중 5~8번째 회원들만 조회하고 싶다 ?

```java
/*
  일반 SQL처럼   "select member from user(테이블명) as m"   
  이런식으로 테이블에 매핑을 하지 않고,  "select m from Member(객체) as m" 객체에 매핑
*/
List<Member> result = em.createQuery("select m from Member as m", Member.class)
					    .setFirstResult(5)
					    .setMaxResults(8)
					    .getResultList();
			
for (Member member : result) {
	System.out.println("member.name = " + member.getName());
}

/*   

실제 내부적으로 전송되는 쿼리  (H2 Dialect)
 
select
	member0_.id as id1_0_,
    member0_.name as name2_0_ 
from
    Member member0_ limit ? offset ?

*/

```

<br>

---
<br>

# :: JPA의 성능 최적화 기능

- **1차 캐시와 동일성 보장**
    - 같은 트랜잭션 안에서는 같은 엔티티를 반환 - 조회 성능 향상
    
    ```java
    String userId = "drawingDream";
     User user1 = jpa.find(User.class, userId);  // 여기서의 결과를 메모리에 담고
     User user2 = jpa.find(User.class, userId);  // 그대로 반환해줌 (같은 데이터니까)
     //  SQL은 1번만 수행 (짧은 기간동안이긴 해도 약간의 캐싱 기능)
    ```
    
- **트랜잭션을 지원하는 쓰기 지연 (버퍼링 관련)**
    - 트랜잭션을 commit할때까지 INSERT SQL 모음
    - JDBC BATCH SQL 기능을 사용해서 한 번에 SQL 전송
    
    ```java
    transaction.begin();    // 트랜잭션 시작
     em.persist(userA);
     em.persist(userB);
     em.persist(userC);      // 여기까지의 INSERT문을 모두 모아서
     transaction.commit();   // 트랜잭션 커밋 - 한 번에 전송
    ```
    
- **지연 로딩**
    - 지연 로딩 : 객체가 실제 사용될 때 로딩
    - 즉시 로딩 : JOIN문으로 한 번에 연관된 객체까지 미리 조회
    
    ```java
    /* 지연 로딩 */
    User user = userDAO.find(userId);  // User 객체 가져오고
    Team team = user.getTeam();        // 여기서 Team 정보를 가져오는게 아니라,
    String teamName = team.getName();  // Team 정보는 **실제 사용될 때** SQL 수행
    ```
    
    ```java
    /* 즉시 로딩 */
    //  Option으로 "User객체를 가져올 때 항상 Team 정보까지 가져와라" 설정하면
    User user = userDAO.find(userId);  // 여기서 JOIN문을 포함한 SQL을 1번 수행
    Team team = user.getTeam();
    String teamName = team.getName();  // 추가적인 SQL 필요하지 않음
    ```
    
<br>

---
<br>

## 비교하기

- 계층 분할

```java
/*   일반적인 SQL 사용하는 경우   */
String userId = "100";
Member user1 = userDao.getUser(userId);
Member user2 = userDao.getUser(userId);

user1 == user2  // false
/*
	MemberDAO 클래스 내부 getMember(String memberId) 메서드가
	그때그때 SELECT 쿼리를 날려서 Member 객체를 리턴하기 때문
*/

----------------------------------------------------------------

/*   Java Collection에서 조회하는 경우   */
String memberId = "100";
Member member1 = list.get(memberId);
Member member2 = list.get(memberId);

member1 == member2  // true : 참조값이 같기 때문에
```
<br>

---
<br>

# :: Jpa Version (hibernate Version) 선택

1. [spring.io](http://spring.io) 접속 
2. projects 탭 - Spring Boot
3. learn  →  사용하는 Spring Boot 버전 [ Reference Doc. ]  →  single HTML page로 열기
    
    
    ![](https://images.velog.io/images/palacer/post/25accd77-23b2-43a2-93cb-f02deec07c70/image.png)
    

4.  CTRL + F 로 org.hibernate 검색  →  [ Appendix F.1. Managed Dependency Coordinates ] 에서
    
    권장 hibernate Version 확인
    
    ![](https://images.velog.io/images/palacer/post/3ed651cb-30ec-4960-a413-bf787df8a3a1/image.png)
