# 07. 스프링 데이터 JPA

#### 왜 JPA를 학습해야 하는가?
* 도메인 주도 개발이 가능합니다.
    * 애플리케이션의 코드가 SQL 데이터베이스 관련 코드에 잠식 당하는 것을 방지하고 도메인 기반의 프로그래밍으로 비즈니스 로직을 구현하는데 집중할 수 있습니다.
* 그리고 개발 생산성에 좋으며, 데이터베이스에 독립적인 프로그래밍이 가능하고, 타입 세이프한 쿼리 작성 그리고 Persistent Context가 제공하는 캐시 기능으로 성능 최적화까지 가능합니다.

## 1부: 핵심 개념 이해
### 관계형 데이터베이스와 자바
* JDBC
    * (관계형) 데이터베이스와 자바의 연결 고리
    ![3](./img/3.PNG)
    * DataSource / DriverManager
    * Connection
    * PreparedStatement
* SQL
    * DDL : 스키마/테이블/시퀀스/인덱스 생성
    * DML : select/insert/delete/update
* 무엇이 문제인가?
    * SQL을 실행하는 비용이 비싸다.
    * SQL이 데이터베이스 마다 다르다.
    * 스키마를 바꿨더니 코드가 너무 많이 바뀌네...
    * 반복적인 코드가 너무 많아.
    * 당장은 필요가 없는데 언제 쓸 줄 모르니까 미리 다 읽어와야 하나...
* 의존성 추가
```xml
<dependency>
    <groupId>org.postgresql</groupId>
    <artifactId>postgresql</artifactId>
    <version>42.2.16</version>
</dependency>
```
* PostgreSQL 설치 및 서버 실행 (docker)
```
docker run -p 5432:5432 -e POSTGRES_PASSWORD=pass -e POSTGRES_USER=jingyu -e POSTGRES_DB=springdata --name postgres_boot -d postgres
docker exec -i -t postgres_boot bash
su - postgres
psql springdata (Window10 : psql -U jingyu springdata)

데이터베이스 조회 : \list
테이블 조회 : \dt
쿼리 : SELECT * FROM account;
```
* JDBC 연결 및 query 수행
```java
String url = "jdbc:postgresql://localhost:5432/springdata";
String username = "jingyu";
String password = "pass";

try(Connection connection = DriverManager.getConnection(url, username, password)) {
    System.out.println("Connection created: " + connection);
//            String sql = "CREATE TABLE ACCOUNT (id int, username varchar(255), password varchar(255));";
    String sql = "INSERT INTO ACCOUNT VALUES(1, 'jingyu', 'pass');";
    try(PreparedStatement statement = connection.prepareStatement(sql)) {
        statement.execute();
    }
}
```
* 결과
```
springdata=# \dt
         List of relations
 Schema |  Name   | Type  | Owner
--------+---------+-------+--------
 public | account | table | jingyu
(1 row)

springdata=# select * from account;
 id | username | password
----+----------+----------
  1 | jingyu   | pass
(1 row)
```


### ORM: Object-Relation Mapping
* JDBC 사용
```java
try(Connection connection = DriverManager.getConnection(url, username, password)) {
    System.out.println("Connection created: " + connection);
    String sql = "INSERT INTO ACCOUNT VALUES(1, 'keesun', 'pass');";
    try(PreparedStatement statement = connection.prepareStatement(sql))
    {
        statement.execute();
    }
}
```
* 도메인 모델 사용
```java
Account account = new Account(“keesun”, “pass”);
accountRepository.save(account);
```
* JDBC 대신 도메인 모델 사용하려는 이유?
    * 객체 지향 프로그래밍의 장점을 활용하기 좋으니까.
    * 각종 디자인 패턴
    * 코드 재사용
    * 비즈니스 로직 구현 및 테스트 편함
    
`ORM은 애플리케이션의 클래스와 SQL 데이터베이스의 테이블 사이의 맵핑 정보를 기술한 메타데이터를 사용하여, 자바 애플리케이션의 객체를 SQL 데이터베이스의 테이블에 자동으로 (또 깨끗하게) 영속화 해주는 기술입니다.`

| 장점 | 단점 |
|---|---|
| 생산성 : 쉽고 빠르게 매핑정보만 정의하면 데이터를 넣고뺴는게 쉬워짐.<br>유지보수성 : 코드가 간결하고 테스트도 짜기 쉬워짐. <br>성능 : 단순한 작업은 더 느릴 수 있음. but, 객체와 테이블 사이에 캐시가 존재해서 불필요한 쿼리가 수행되지 않음.<br>밴더 독립성 : DB변경에 무관. | 학습비용 : ㅎㅎㅎ |

* Q : hibernate? JPA? 뜻

### ORM: 패러다임 불일치
객체를 릴레이션에 맵핑하려니 발생하는 문제들과 해결책
* 밀도(Granularity) 문제

| 객체 | 릴레이션 |
|---|---|
| 다양한 크기의 객체를 만들 수 있음.<br>커스텀한 타입 만들기 쉬움. | 테이블<br>기본 데이터 타입 (UDT는 비추) |
 
* 서브타입(Subtype) 문제

| 객체 | 릴레이션 |
|---|---|
| 상속 구조 만들기 쉬움.<br>다형성. | 테이블 상속이라는게 없음.<br>상속 기능을 구현했다 하더라도 표준 기술이 아님.<br>다형적인 관계를 표현할 방법이 없음. |

* 식별성(Identity) 문제

| 객체 | 릴레이션 |
|---|---|
| 레퍼런스 동일성 (==)<br>인스턴스 동일성 (equals() 메소드) | 주키 (primary key) |

* 관계(Association) 문제

| 객체 | 릴레이션 |
|---|---|
| 객체 레퍼런스로 관계 표현.<br>근본적으로 ‘방향'이 존재 한다.<br>다대다 관계를 가질 수 있음 | 외래키(foreign key)로 관계 표현.<br>‘방향'이라는 의미가 없음. 그냥 Join으로 아무거나 묶을 수 있음.<br>태생적으로 다대다 관계를 못만들고, 조인테이블 또는 링크 테이블을 사용해서 두개의 1대다 관계로 풀어야 함. |

* 데이터 네비게이션(Navigation)의 문제

| 객체 | 릴레이션 |
|---|---|
| 레퍼런스를 이용해서 다른 객체로 이동 가능.<br>콜렉션을 순회할 수도 있음. | 하지만 그런 방식은 릴레이션에서 데이터를 조회하는데 있어서 매우 비효율적이다.<br>데이터베이스에 요청을 적게 할 수록 성능이 좋다. 따라서 Join을 쓴다.<br>하지만, 너무 많이 한번에 가져오려고 해도 문제다.<br>그렇다고 lazy loading을 하자니 그것도 문제다. (n+1 select) |

### JPA 프로그래밍: 프로젝트 세팅
* 데이터베이스 실행
    * PostgreSQL 도커 컨테이너 재사용
    * docker start postgres_boot
* 스프링 부트
    * 스프링 부트 v2.*
    * 스프링 프레임워크 v5.*
* 스프링 부트 스타터 JPA
    * JPA 프로그래밍에 필요한 의존성 추가
        * JPA v2.*
        * Hibernate v5.*
    * 자동 설정: HibernateJpaAutoConfiguration
        * 컨테이너가 관리하는 EntityManager (프록시) 빈 설정
        * PlatformTransactionManager 빈 설정
* JDBC 설정
    * jdbc:postgresql://localhost:5432/springdata
    * jingyu
    * pass
    
* application.properties
```properties
spring.datasource.url=jdbc:postgresql://localhost:5432/springdata
spring.datasource.username=jingyu
spring.datasource.password=pass

spring.jpa.properties.hibernate.jdbc.lob.non_contextual_creation=true
spring.jpa.hibernate.ddl-auto=create
```
* code
```java
// Entity
@Entity
@Getter
@Setter
public class Account {

    @Id @GeneratedValue
    private Long id;
    private String username;
    private String password;
}

// Runner
@Component
@Transactional
public class JpaRunner implements ApplicationRunner {

    @PersistenceContext
    EntityManager entityManager;

    @Override
    public void run(ApplicationArguments args) throws Exception {
        Account account = new Account();
        account.setUsername("jingyu");
        account.setPassword("studyhard123");

        Session session = entityManager.unwrap(Session.class);
        session.save(account);
//        entityManager.persist(account);
    }
}
```

### JPA 프로그래밍: 엔티티 맵핑
* @Entity
    * “엔티티”는 객체 세상에서 부르는 이름.
    * 보통 클래스와 같은 이름을 사용하기 때문에 값을 변경하지 않음.
    * 엔티티의 이름은 JQL에서 쓰임.
    * DB의 예약어와 같을 경우 문제가 될 수 있음.
    
* @Table
    * “릴레이션" 세상에서 부르는 이름.
    * @Entity의 이름이 기본값.
    * 테이블의 이름은 SQL에서 쓰임.
* @Id
    * 엔티티의 주키를 맵핑할 때 사용.
    * 자바의 모든 primitive 타입과 그 랩퍼 타입을 사용할 수 있음
        * Date랑 BigDecimal, BigInteger도 사용 가능.
    * 복합키를 만드는 맵핑하는 방법도 있지만 그건 논외로..
* @GeneratedValue
    * 주키의 생성 방법을 맵핑하는 애노테이션
    * 생성 전략과 생성기를 설정할 수 있다.
        * 기본 전략은 AUTO: 사용하는 DB에 따라 적절한 전략 선택
        * TABLE, SEQUENCE, IDENTITY 중 하나.
* @Column
    * unique
    * nullable
    * length
    * columnDefinition
    * ...
* @Temporal
    * 현재 JPA 2.1까지는 Date와 Calendar만 지원.
* @Transient
    * 컬럼으로 맵핑하고 싶지 않은 멤버 변수에 사용.
* application.properties
```
spring.jpa.show-sql=true
spring.jpa.properties.hibernate.format_sql=true
```

### JPA 프로그래밍: Value 타입 맵핑
* 엔티티 타입과 Value 타입 구분
    * 식별자가 있어야 하는가.
    * 독립적으로 존재해야 하는가.
* Value 타입 종류
    * 기본 타입 (String, Date, Boolean, ...)
    * Composite Value 타입
    * Collection Value 타입
        * 기본 타입의 콜렉션
        * 컴포짓 타입의 콜렉션
* Composite Value 타입 맵핑
    * @Embeddable
    * @Embedded
    * @AttributeOverrides
    * @AttributeOverride
```java
// Address
@Embeddable
public class Address {
 private String street;
 private String city;
 private String state;
 private String zipCode;
}

// Account
@Embedded
@AttributeOverrides({
@AttributeOverride(name = "street", column = @Column(name = "home_street"))})
private Address address;
```

### JPA 프로그래밍: 1대다 맵핑
* 관계에는 항상 두 엔티티가 존재 합니다.
    * 둘 중 하나는 그 관계의 주인(owning)이고
    * 다른 쪽은 종속된(non-owning) 쪽입니다.
    * 해당 관계의 반대쪽 레퍼런스를 가지고 있는 쪽이 주인.
* 단방향에서의 관계의 주인은 명확합니다.
    * 관계를 정의한 쪽이 그 관계의 주인입니다.
* 단방향 @ManyToOne
    * 기본값은 FK 생성
* 단방향 @OneToMany
    * 기본값은 조인 테이블 생성
* 양방향
    * FK 가지고 있는 쪽이 오너 따라서 기본값은 @ManyToOne 가지고 있는 쪽이 주인.
    * 주인이 아닌쪽(@OneToMany쪽)에서 mappedBy 사용해서 관계를 맺고 있는 필드를 설정해야 합니다.
* 양방향
    * @ManyToOne (이쪽이 주인)
    * @OneToMany(mappedBy)
    * 주인한테 관계를 설정해야 DB에 반영이 됩니다.

### JPA 프로그래밍: Cascade
엔티티의 상태 변화를 전파 시키는 옵션.
* 잠깐? 엔티티의 상태가 뭐지?
    * Transient: JPA가 모르는 상태
    * Persistent: JPA가 관리중인 상태 (1차 캐시, Dirty Checking, Write Behind, ...)
    * Detached: JPA가 더이상 관리하지 않는 상태.
    * Removed: JPA가 관리하긴 하지만 삭제하기로 한 상태.
    
![10](./img/10.PNG)

```java
@Entity
public class Commnet {

    @Id @GeneratedValue
    private Long id;

    private String comment;

    @ManyToOne
    private Post post;
}

@Entity
public class Post {
    @Id @GeneratedValue
    private Long id;

    private String title;

    @OneToMany(mappedBy = "post", cascade = CascadeType.ALL)
    private Set<Commnet> comments = new HashSet<>();

    public void addComment(Commnet commnet) {
        this.getComments().add(commnet);
        commnet.setPost(this);
    }
}

//Runner
@Component
@Transactional
public class JpaRunner implements ApplicationRunner {

    @PersistenceContext
    EntityManager entityManager;

    @Override
    public void run(ApplicationArguments args) throws Exception {

        Post post = new Post();
        post.setTitle("Spring");

        Commnet commnet = new Commnet();
        commnet.setComment("studyhard");
        post.addComment(commnet);

        Commnet commet1 = new Commnet();
        commet1.setComment("123");
        post.addComment(commet1);

        Session session = entityManager.unwrap(Session.class);
        session.save(post);
    }
}
```
### JPA 프로그래밍: Fetch
* 연관 관계의 엔티티를 어떻게 가져올 것이냐... 지금(Eager)? 나중에(Lazy)?
    * @OneToMany의 기본값은 Lazy
    * @ManyToOne의 기본값은 Eager


