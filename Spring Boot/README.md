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

## Hibernate

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

    Java ORM 사용 표준 인터페이스

    1. 표준 인터페이스
    1. 애노테이션 기반 매핑
    1. 표준 CRUD API
    2. 표준 객체 쿼리(JPQL)

```java

```

> 문제점

## Spring Data JPA: 편의 도구
 - Repository 인터페이스 자동 구현
 - 메서드 이름으로 쿼리 자동 생성
 - 페이징과 정렬 자동 처리
 - 커스텀 쿼리 지원 (@Query)
 - Auditing 생성시간, 수정시간 자동 관리)

---

## MyBatis: SQL 매핑 프레임워크
 - SQL 직접 실행
 - 객체 매핑
 - 동적 SQL 처리(복잡한 쿼리)
