# JPA
Java Persistence API, 자바 진영의 ORM 기술 표준 인터페이스.

개발자 = SQL 매퍼 ㅠㅠ → 현재 하는 일

**동일 트랜잭션 내에서 조회한 엔티티가 같음을 보장하도록**, 객체를 자바 컬렉션에 저장 하듯이 DB에 저장할 수는 없을까? → JPA

JDBC → MyBatis or JdbcTemplate → JPA

---
- [ORM](#orm)
- [JPA의 동작](#jpa의-동작)
- [JPQL](#jpql)
- [영속성 관리](#영속성-관리)
- [엔티티 매핑](#엔티티-매핑)
---

## ORM
Object-Relational Mapping, 객체 관계 매핑으로 객체와 관계형 데이터베이스를 독립적으로 설계하고 ORM 프레임워크가 중간에서 매핑합니다.

---

## JPA의 동작

### 애플리케이션과 JDBC 사이에서 동작
<p align="center"><img src="images/jpaA.png" width="50%"></p><p align="center"><img src="images/jpaB.png" width="50%"></p>

DAO가 PK를 보내면, JPA가 쿼리를 만들어서 JDBC API를 사용하여 DB로 쿼리를 보내고 결과를 반환 받습니다. → **패러다임의 불일치 해결**

### JPA의 방언(dialect)
<p align="center"><img src="images/dialect.png" width="50%"></p>

### JPA의 구동 방식
<p align="center"><img src="images/jpa구동.png" width="50%"></p>

- `EntityManagerFactory`
최초 하나만 생성하여 애플리케이션 전체에서 공유합니다.
- `EntitiManager`
Thread 간에 공유하면 안 됩니다. Database Connection처럼 사용하고 버려야 합니다.
- JPA의 모든 데이터 변경은 트랜잭션 안에서 실행됩니다.

---

## JPQL
JPA를 사용하여 엔티티 객체를 중심으로 개발하는 쿼리로 SQL을 추상화한 객체 지향 쿼리 언어입니다.

### JPQL 특징
- SQL과 문법 유사하며 객체지향 SQL이라 할 수 있음.
- SQL은 데이터베이스 테이블을 대상으로 쿼리, JPQL은 엔티티 객체를 대상으로 쿼리로 **SQL에 의존적이지 않음.**
- 검색을 할 때도 **테이블이 아닌, 엔티티 객체를 대상으로 검색**
- 모든 DB 데이터를 객체로 변환해서 검색하는 것은 불가능
- 애플리케이션이 필요한 데이터만 DB에서 불러오려면, 결국 검색 조건이 포함된 SQL이 필요
- 검색 쿼리가 자유롭지 못 한 문제

---

## 영속성 관리
<p align="center"><img src="images/영속성.png" width="70%"></p>

### 영속성 컨텍스트
엔티티를 영구 저장하는 환경이며 눈에 보이지 않는 논리적인 개념입니다. 엔티티 매니저를 통해 영속성 컨텍스트에 접근합니다.
- `EntityManager.persist(entity);` : entity를 영속성 컨텍스트에 저장합니다.
- J2SE 환경 - EntityManager : PersistenceContext = 1:1
- J2EE, 스프링 프레임워크 같은 컨테이너 환경 - EntityManager : PersistenceContext = N:1
### 엔티티의 생명주기
- 비영속(new/transient) : 영속성 컨텍스트와 전혀 관계 없는 새로운 상태
    ```java
    // 객체 생성(비영속)
    Member member = new Member();
    member.setId("member1");
    member.setUsername("회원1");
    ```
- 영속(managed) : 영속성 컨텍스트에 관리되는 상태
    ```java
    // 객체 생성(비영속)
    Member member = new Member();
    member.setId("member1");
    member.setUsername("회원1");
    
    EntityManager em = emf.createEntityManager();
    em.getTransacntion().begin();

    // 객체를 저장한 상태(영속)
    em.persist(member);

    ```
- 준영속(detached) : 영속성 컨텍스트에 저장되었다가 분리된 상태
    ```java
    // 영속
    Member member = em.find(Member.class, 150L);
    member.setName("AAAA"); // Dirty Checking

    // 더 이상 영속성 컨텍스트로 관리하기 싫으면
    em.detach();    // 이 것이 준영속
    ```
    - `entityManager.detach()` : 특정 엔티티만 준영속 상태로 전환
    - `entityManager.clear()` : 영속성 컨텍스트를 완전히 초기화
    - `entityManager.close()` : 영속성 컨텍스트를 종료
- 삭제(removed) : 삭제된 상태 
<p align="center"><img src="images/생명주기.png" width="70%"></p>

### 영속성 컨텍스트의 이점
- 1차 캐시
    - 동일 트랜잭션 안에서 다음 조회 시 DB 쿼리를 날리지 않도록 합니다.
        - 하지만, 클라이언트 10개가 동시에 요청을 날리면 10개의 별도 1차 캐시를 가집니다. 즉, 성능상의 이점은 크지는 않으며 해당 메커니즘을 통해 얻을 수 있는 이점들이 있습니다.
    - `persist()` 혹은 최초 쿼리 발생 시 저장.

- 동일성(identity) 보장
    - 1차 캐시로 반복 가능한 읽기(REPEATABLE READ)등급의 트랜잭션 격리 수준을, 데이터베이스가 아닌 애플리케이션 차원에서 제공합니다.
        ```java
        Member a = em.find(Member.class, "member1");
        Member b = em.find(Member.clase, "member1");

        System.out.println(a == b); // true
        ```
- 트랜잭션을 지원하는 쓰기 지연(transactional write-behind)
    - 버퍼링 : write를 모아서 한 번에 시행
        ```java
        EntityManager em = emf.createEntityManager();
        EntityTransaction transaction = em.getTransaction();
        // 엔티티 매니저는 데이터 변경 시 트랜잭션을 시작해야 합니다.
        transaction.begin();

        em.persist(memberA);
        em.persist(memberB);
        // 여기까진 SQL을 보내지 않습니다. 

        transaction.commit();   // 트랜잭션 커밋, 커밋하는 순간 DB에 INSERT SQL 전송
        // hibernate 설정으로 size조절 가능 >> 버퍼링
        ```
    - 쓰기 지연 SQL 저장소가 존재하여, `persist()` 사용 시 쿼리가 삽입됩니다.
    - `transaction.commit()` 시 쓰기 지연 SQL에 쌓인 쿼리들이 전송됩니다. 이 것을 `flush()`라고 합니다.
- 변경 감지(Dirty Checking)
    - JPA가 변경을 자동으로 감지하여 알려줍니다.
        ```java
        EntityManager em = emf.createEntityManager();
        EntityTransaction transaction = em.getTransaction();
        transaction.begin();

        // 영속 엔티티 조회
        Member memberA = em.find(Member.class, "memberA");

        // 영속 엔티티 데이터 수정
        memberA.setUsername("hi");
        memberA.setAge(10);

        transaction.commit();
        ```
    - `commit()` 시 `flush()`가 발생하는데, 1차 캐시의 Entity와 스냅샷을 비교하여 쓰기 지연 SQL저장소에 쿼리를 만들어 두고 한 번에 전송합니다.
    - `commit()`직전에만 동기화 하면 됩니다.
    <p align="center"><img src="images/변경감지.png" width="70%"></p>

    - `flush()` : 영속성 컨텍스트의 변경내용(쓰기 지연 SQL 저장소의 쿼리들)을 데이터베이스에 동기화합니다. ***1차 캐시, 영속성 컨텍스트를 비우는 것이 아님***
    - 보통 `FlushModeType.AUTO`로 자동으로 사용합니다.
        - flush() 방법 종류
            - `em.flush()` - 직접(강제) 호출
            - 트랜잭션 커밋 - 플러시 자동 호출
            - JPQL 쿼리 실행 - 플러시 자동 호출
 
- 지연 로딩(Lazy Loading)

---

## 엔티티 매핑
### 객체와 테이블 매핑 `@Table`
- `@Entity` : JPA가 관리하며, JPA를 사용하여 테이블과 매핑할 클래스에 필수로 붙습니다.
    - 기본 생성자 필수(파라미터가 없는 public 또는 protected 생성자)
    - final 클래스, enum, interface, inner 클래스 사용 불가
    - 저장할 필드에 final 사용 불가
    - 속성 : `name`
        - JPA에서 사용할 엔티티 이름 지정
        - 기본값 : 클래스 이름을 그대로 사용
        - 같은 클래스 이름이 없으면 가급적 기본값을 사용합니다.
- 필드와 컬럼 매핑 : `@Column`
- 기본키 매핑 : `@Id`
- 연관관계 매핑 : `@ManyToOne`, `@JoinColumn`


### 데이터 스키마 자동 생성
- DDL(`CREATE`,`ALTER`,`DROP`,`TRUNCATE`)을 애플리케이션 실행 시점에 자동 생성
- 테이블 중심 → 객체 중심
- DB 방언을 활용하여 DB에 맞는 적절한 DDL 생성 ***개발에서만 사용, 운영에서는 X → 적절히 다듬하여 사용***
- `hibernate.hbm2ddl.auto`
    |옵션|설명|
    |:---|:---|
    |`create`|기존테이블 삭제 후 다시 생성 (DROP + CREATE)|
    |`create-drop`|CREATE와 같으나, **종료 시점에 테이블 DROP**|
    |`update`|변경분만 반영 ***운영 DB 사용 자제, db lock 위험***|
    |`validate`|엔티티와 테이블이 정상 매핑되었는지만 확인|
    |`none`|사용하지 않음|
    - **운영 DB에는 절대 `create`, `create-drop`, `update`를 사용하면 안 됩니다.**
    - 개발 초기 단계는 `create`, `update` 사용
    - 테스트 서버는 `update`, `validate` 사용
    - 스테이징과 운영 서버는 `validate`, `none` 사용
- DDL 생성 기능 : DDL 자동 생성 시에만 사용 되고, JPA의 실행 로직에는 영향을 주지 않습니다.
    - 제약 조건 : 회원 이름 필수, 10자 초과 불가능
        - `@Column(nullable = false, length = 10)`
        - `@Column(unique = true, length = 10)`
    - 유니크 제약조건 추가
        - `@Table(uniqueConstraints = {UniqueConstraint(name = "NAME_AGE_UNIQUE", columnNames={"NAME", "AGE"})}`
