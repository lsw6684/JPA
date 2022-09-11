# JPA
Java Persistence API, 자바 진영의 ORM 기술 표준 인터페이스.

개발자 = SQL 매퍼 ㅠㅠ → 현재 하는 일

**동일 트랜잭션 내에서 조회한 엔티티가 같음을 보장하도록**, 객체를 자바 컬렉션에 저장 하듯이 DB에 저장할 수는 없을까? → JPA

JDBC → MyBatis or JdbcTemplate → JPA

---
- [ORM](#orm)

---

## ORM
Object-Relational Mapping, 객체 관계 매핑으로 객체와 관계형 데이터베이스를 독립적으로 설계하고 ORM 프레임워크가 중간에서 매핑합니다.
- 애플리케이션과 JDBC 사이에서 동작
<p align="center"><img src="images/JPA.PNG" width="50%"></p><p align="center"><img src="images/JPA2.PNG" width="50%"></p>

DAO가 PK를 보내면, JPA가 쿼리를 만들어서 JDBC API를 사용하여 DB로 쿼리를 보내고 결과를 반환 받습니다. → **패러다임의 불일치 해결**


