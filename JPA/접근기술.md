<div align="center">

# **Java Spring Boot 데이터베이스 접근 기술**

| [JDBC](#jdbcjava-database-connectivity) • [Spring JDBC Template](#spring-jdbc-template) • [ORM](#ormobject-relational-mapping) • [Hibernate](#하이버네이트hibernate) • [JPA](#jpa-java-persistence-api) • [Spring Data JPA](#spring-data-jpa) • [MyBatis](#mybatis) • [하이브리드](#spring-data-jpa-vs-mybatis) |

</div>

## JDBC(Java Database Connectivity)

    Java에서 데이터베이스에 연결하기 위한 가장 기본적인 API

```java
// 1. 드라이버 로드
Class.forName("com.mysql.cj.jdbc.Driver");

// 2. 연결 생성
Connection conn = DriverManager.getConnection(
    "jdbc:mysql://localhost:3306/testdb", "user", "password");

// 3. SQL 작성/실행
PreparedStatement pstmt = conn.prepareStatement("SELECT * FROM users WHERE id = ?");
pstmt.setLong(1, userId);
ResultSet rs = pstmt.executeQuery();

// 4. 결과 처리
while(rs.next()) {
    String name = rs.getString("name");
    String email = rs.getString("email");
}

// 5. 리소스 정리
rs.close();
pstmt.close();
conn.close();
```

> 연결/해제/예외처리와 같은 반복적인 코드가 많음 

## Spring JDBC Template

    JDBC의 반복적인 리소스 관리를 자동화하여 비즈니스 로직에 집중할 수 있게 해주는 프레임워크

```java
@Repository
public class UserRepository {
    // 1. 커넥션 풀에서 자동으로 연결 관리(DataSource)
    @Autowired
    private JdbcTemplate jdbcTemplate;
    
    public User findById(Long userId) {
        // 2. SQL 작성(드라이버 로드, 연결 생성 자동화)
        String sql = "SELECT * FROM users WHERE id = ?";
        
        // 3. SQL 실행 + 결과 처리 + 리소스 정리(모두 자동)
        return jdbcTemplate.queryForObject(sql, new Object[]{userId}, (rs, rowNum) -> {
            // 4. 간단한 매핑 로직만 작성
            User user = new User();
            user.setName(rs.getString("name"));
            user.setEmail(rs.getString("email"));
            return user;
        });
        // 5. 리소스 자동 정리
    }
}
```

> 여전히 SQL을 직접 작성해야 하고 객체-테이블 간 수동 매핑이 필요함

## ORM(Object-Relational Mapping)

    객체 지향 프로그래밍 언어의 객체와 관계형 데이터베이스의 테이블을 자동으로 매핑

    1. 객체와 관계형 데이터베이스 차이 해결
    2. CRUD SQL 자동 생성  
    3. 연관관계 자동 처리
    4. 변경 감지 자동화
    5. DB별 SQL 방언 차이 해결

```
// Spring JDBC Template으로 해결 안 되는 문제
// 1. 복잡한 연관관계 조회(수동 JOIN + 복잡한 매핑)
String sql = "SELECT u.*, o.* FROM users u LEFT JOIN orders o ON u.id = o.user_id WHERE u.id = ?";

// 2. 연관 데이터 접근 불가능
User user = userRepository.findById(1L);
user.getOrders();
```

> Java에서는 표준이 없어서 각 DB마다 다른 API를 사용해야만 함

## 하이버네이트(Hibernate)

    Java ORM 프레임워크
    
`POJO + XML 매핑`

```java
public class User {
    private Long id;
    private String name;
    private String email;
    // getter/setter...
}
```
```xml
<hibernate-mapping>
    <class name="com.example.User" table="users">
        <id name="id" column="id">
            <generator class="identity"/>
        </id>
        <property name="name" column="name"/>
        <property name="email" column="email"/>
    </class>
</hibernate-mapping>
```

```java
public class UserDAO {
    public User findById(Long userId) {
        Session session = sessionFactory.getCurrentSession();
        // XML 매핑 기반으로 SQL 자동 생성
        return (User) session.get(User.class, userId);
    }
    
    // 객체 지향 쿼리(HQL): 테이블이 아닌 클래스명 사용
    public List<User> findByName(String name) {
        String hql = "FROM User WHERE name = :name";
        return session.createQuery(hql).setParameter("name", name).list();
    }
}
```

> 각 프로젝트 마다 Hibernate에 종속된 코드 생김        
> 다른 DB ORM으로 바꾸려면 모든 코드 수정해야함

## JPA(Java Persistence API)

    Java ORM 표준 API

    1. 표준 인터페이스 제공
    2. 어노테이션 기반 매핑  
    3. 표준 CRUD API(EntityManager)
    4. 표준 객체 쿼리(JPQL)

```java
// 어노테이션 기반 매핑
@Entity
@Table(name = "users")
public class User {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    
    @Column(nullable = false)
    private String name;
    private String email;
}

@Repository  
public class UserRepository {
    // 1. 표준 인터페이스 사용
    @PersistenceContext
    private EntityManager entityManager;
    
    public User findById(Long userId) {
        // 2. 표준 API
        return entityManager.find(User.class, userId);
    }
    
    // 3. JPQL
    public List<User> findByName(String name) {
        String jpql = "SELECT u FROM User u WHERE u.name = :name";
        return entityManager.createQuery(jpql, User.class)
            .setParameter("name", name).getResultList();
    }
```

> Repository 클래스마다 반복적인 CRUD 메서드를 직접 작성해야 함

## [Spring Data JPA](https://github.com/yi5oyu/Study/tree/main/JPA)

    JPA Repository 반복 코드 문제를 해결하기위한 프레임워크

    1. 인터페이스만 선언하면 CRUD 구현체 자동 생성
    2. 메서드 이름으로 쿼리 자동 생성
    3. 페이징과 정렬 자동 지원
    4. @Query로 커스텀 쿼리

```java
// 인터페이스만 선언
public interface UserRepository extends JpaRepository<User, Long> {
    // 1. 기본 CRUD 자동 제공: save(), findById(), findAll(), delete()
    
    // 2. 메서드 이름으로 쿼리 자동 생성
    List<User> findByName(String name); 
    List<User> findByNameContaining(String name); 
    List<User> findByAgeGreaterThan(Integer age);
    
    // 3. 페이징 자동 지원
    Page<User> findByNameContaining(String name, Pageable pageable);
    
    // 4. 커스텀 쿼리
    @Query("SELECT u FROM User u WHERE u.email LIKE %:domain%")
    List<User> findByEmailDomain(@Param("domain") String domain);
}
```

## [MyBatis](https://github.com/yi5oyu/Study/tree/main/MyBatis)

    JPA의 복잡한 쿼리와 성능 한계를 해결하는 SQL 중심 프레임워크

    1. SQL 직접 제어로 복잡한 쿼리 해결
    2. 성능 최적화(필요한 컬럼만 조회, 쿼리 튜닝)
    3. 동적 SQL 조건부 쿼리 처리
    4. 기존 SQL 레거시 코드 활용 가능

`SQL을 직접 작성해 데이터베이스의 모든 기능을 활용하고 성능 제어`

```java
@Mapper
public interface UserMapper {
    // 기본 CRUD
    @Insert("INSERT INTO users (name, email, age) VALUES (#{name}, #{email}, #{age})")
    @Options(useGeneratedKeys = true, keyProperty = "id")
    void save(User user);
    
    @Select("SELECT * FROM users WHERE id = #{id}")
    User findById(Long id);
    
    @Select("SELECT * FROM users ORDER BY created_at DESC")
    List<User> findAll();
    
    @Delete("DELETE FROM users WHERE id = #{id}")
    void deleteById(Long id);
    
    // 복잡한 쿼리
    @Select("""
        SELECT ~
        FROM ~
        LEFT JOIN ~
        WHERE ~
        GROUP BY ~
        ORDER BY ~
        """)
    List<User> getUserStats();
}
```

## Spring Data JPA vs MyBatis

| 시나리오 | Spring Data JPA | MyBatis | 추천 |
|----------|-----------------|---------|------|
| **단순 CRUD** | 자동 제공 | 직접 작성 필요 | **Spring Data JPA** |
| **복잡한 검색** | Java 코드로 복잡 | Provider/XML로 처리 | **MyBatis** |
| **집계/리포팅** | JPQL 제한적 | SQL 자유자재 | **MyBatis** |
| **성능 튜닝** | 간접적, 제한적 | SQL 직접 최적화 | **MyBatis** |
| **빠른 개발** | 자동 생성으로 빠름 | SQL 작성 시간 | **Spring Data JPA** |
| **기존 SQL 활용** | 변환 작업 필요 | 그대로 사용 | **MyBatis** |

> 하이브리드      
> 단순 CRUD는 Spring Data JPA 활용       
> 복잡한 쿼리, 성능 최적화가 필요한 경우 MyBatis 사용

<!-- 
## QueryDSL
## Spring Data JDBC
-->
