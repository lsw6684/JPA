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
엔티티를 영구 저장하는 환경이며 눈에 보이지 않는 논리적인 개념입니다.
- `EntityManager.persist(entity);` : entity를 영속성 컨텍스트에 저장합니다.

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
- 삭제(removed) : 삭제된 상태 
<p align="center"><img src="images/생명주기.png" width="70%"></p>

### 영속성 컨텍스트의 이점
- 1차 캐시
    - `persist()` 혹은 최초 쿼리 발생 시 저장?

- 동일성(identity) 보장
    - 1차 캐시로 반복 가능한 읽기(REPEATABLE READ)등급의 트랜잭션 격리 수준을, 데이터베이스가 아닌 애플리케이션 차원에서 제공합니다.
        ```java
        Member a = em.find(Member.class, "member1");
        Member b = em.find(Member.clase, "member1");

        System.out.println(a == b); // true
        ```
- 트랜잭션을 지원하는 쓰기 지연(transactional write-behind)
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

    - `flush()` : 영속성 컨텍스트의 변경내용(쓰기 지연 SQL 저장소의 쿼리들)을 데이터베이스에 반영합니다. ***1차 캐시, 영속성 컨텍스트를 비우는 것이 아님***
        - flush() 방법 종류
            - `em.flush()` - 직접(강제) 호출
            - 트랜잭션 커밋 - 플러시 자동 호출
            - JPQL 쿼리 실행 - 플러시 자동 호출
 
- 지연 로딩(Lazy Loading)
