<div align="center">

# **JPA(Java Persistence API)**

자바 ORM 기술 표준 인터페이스
    
| [아키텍처](#아키텍처) • [JPA 사용](#JPA-사용) |

</div>

## 아키텍처

### EntityManagerFactory

싱글톤 패턴, Thread-Safe    
애플리케이션 시작 시 딱 1번 생성, 비용이 매우 비쌈    
DB 커넥션 풀 설정, 매핑 정보 로딩 등 무거운 작업을 처리      

```java
public class EntityManagerFactory {
    // 메타데이터
    private final ConcurrentHashMap<String, EntityType<?>> metamodel;
    // 커넥션 풀
    private final DataSource connectionPool;
    // JPA 설정
    private final Configuration configuration;
    ...
}
```
1. persistence.xml 파싱
 - DB 연결 정보 로딩(driver, url, user, password)
 - JPA 설정값 로딩(dialect, show_sql 등)

2. 메타모델(Metamodel) 생성
 - 내부적으로 모든 엔티티 설계도 작성(필드 정보, 연관관계, 테이블 매핑 정보 등을 모두 분석해서 저장)
  > 메타 모델이 없으면 수백 개의 엔티티가 있을때 수십 MB의 메모리를 사용하며 초기화에 수 초가 걸릴 수 있음

3. 커넥션 풀(Connection Pool) 초기화
 - HikariCP 같은 커넥션 풀 생성해 미리 10~20개 연결해 놓음(요청마다 발생하는 연결 비용 절약)

4. 쿼리 컴파일러 준비
 - JPQL -> SQL 변환기 초기화
 - 엔티티 매핑 정보 기반으로 SQL 템플릿 생성

### EntityManager

```java
public interface EntityManager {
    // CRUD 메서드
    void persist(Object entity);
    <T> T find(Class<T> entityClass, Object primaryKey);
    void remove(Object entity);
    <T> T merge(T entity);
    
    // 쿼리 생성
    Query createQuery(String jpqlString);
    
    // 영속성 컨텍스트 제어
    void flush();
    void clear();
    void detach(Object entity);
}

// Hibernate의 실제 구현체
public class SessionImpl implements EntityManager {
    private final EntityManagerFactory factory;
    private final PersistenceContext persistenceContext; // 영속성 컨텍스트(엔티티를 Key-Value로 관리)
    private final ActionQueue actionQueue; // 쓰기 지연 SQL 저장소
    private final StatefulPersistenceContext statefulPC; // 1차 캐시
    
    private Transaction transaction;
    private boolean closed = false;
}
```

```java
@Service
public class MemberService {
    @PersistenceContext // 프록시 주입(Thread-Safe)
    private EntityManager em;
    
    @Transactional
    public void save(Member member) {
        em.persist(member);
    }
}
```

### 영속성 컨텍스트

EntityManager 내부에 생성되는 엔티티 저장소  
트랜잭션이 시작될 때 생성, 종료될 때 소멸  
트랜잭션 중에는 ThreadLocal에 의해 현재 스레드가 참조해 GC(Garbage Collector)의 대상이 되지 않음  

1. 1차 캐시
 - DB를 조회 전 내부 Map을 먼저 확인 후 있으면 반환 없으면 DB 조회 후 Map 저장/반환(같은 트랜잭션에선 DB 조회 한 번)

2. 동일성
 - 같은 트랜잭션에서 조회한 객체는 주소값(참조)이 같음

3. 쓰기 지연
 - 내부에 쓰기 지연 SQL 저장소에 쿼리를 저장 후 트랜잭션 커밋(flush) 시점에 한 번에 DB로 전송

4. 변경 감지(Dirty Checking)
 - 객체 값만 바꾸면 알아서 수정됨(DB와 비교하는 것이 아니라 내부 스냅샷과 비교)
  > DB조회 순간 현재 상태를 스냅샷으로 만듬
  > 커밋 요청 시 변경된 엔티티와 스냅샷을 필드별로 비교
  > 다른 부분 발견시 SQL 쿼리 생성 후 DB에 날림

```java
public class PersistenceContext {
    // 1차 캐시
    private Map<EntityKey, Object> entitiesById;
    
    // 스냅샷 저장소
    private Map<EntityKey, Object[]> entitySnapshotsByKey;
    
    // 쓰기 지연 SQL 저장소
    private ActionQueue actionQueue;
    
    // 엔티티 상태 추적
    private Map<Object, EntityEntry> entityEntries;
}

```

## JPA 사용

`Spring 프레임워크 없이 트랜잭션 관리하는 자바 코드`

```java
// 개발자가 EntityManager를 생성하고 트랜잭션(begin, commit)까지 직접 관리

public void a() {
    
    // 트랜잭션 시작
    /*
     * 이 과정을 Spring AOP가 대신 처리
     * 팩토리에서 영속성 컨텍스트 생성/트랜잭션 관리자 가져오고 시작
     */
    EntityManager em = emf.createEntityManager();
    EntityTransaction tx = em.getTransaction();
    tx.begin();
    
    // 엔티티 생성 후 영속성 컨텍스트에 저장
    Member member = new Member();
    member.setName("A");
    member.setAge(30);
    em.persist(member);
    
    /* 내부 동작
     *  EntityKey 생성
     *   ("Member", null) ID가 아직 없음
     * 
     *  IDENTITY 전략(DB 전략대로(Auto Increment))이면 즉시 INSERT 실행
     *   INSERT INTO members (name, age) VALUES ('A', 30)
     *   > DB가 ID = 1 반환
     * 
     * 엔티티에 ID 설정
     *  member.setId(1L);
     * 
     * 1차 캐시 저장
     *  entitiesById.put(new EntityKey("Member", 1L), member);
     * 
     * 스냅샷 생성
     *  entitySnapshotsByKey.put(new EntityKey("Member", 1L), [1L, "A", 30]);
     * 
     * EntityEntry 등록
     *  entityEntries.put(member, new EntityEntry(entity: member, status: MANAGED, loadedState: [1L, "A", 30]));
     */
    
    // 엔티티 수정(변경 감지)
    member.setAge(31);
    
    /* 내부 동작
     * 객체의 age 필드만 31로 변경됨
     * 
     * 엔티티: [1L, "A", 31]
     * 스냅샷: [1L, "A", 30]
     */
    
    // 커밋(Flush)
    tx.commit(); // 내부적으로 em.flush() 호출
    
    /* flush() 내부 동작
     * 
     * 모든 엔티티 순회 - 더티 체킹
     *  for(EntityEntry entry : entityEntries.values()) {
     *     Object[] current = getValues(entry.entity);  // [1L, "A", 31]
     *     Object[] snapshot = entry.loadedState;       // [1L, "A", 30]
     *     // 확인 후 UPDATE SQL 생성 후 저장
     *     if (isDirty(current, snapshot)) {
     *         actionQueue.addUpdate(new EntityUpdateAction(entity: member, sql: "UPDATE members SET name=?, age=? WHERE id=?", values: ["A", 31, 1L]));
     *     }
     *  }
     *  // 큐에 쌓여있는 쿼리 일괄 실행
     *  actionQueue.executeActions();
     * 
     *  // 스냅샷 업데이트
     *  entitySnapshotsByKey.put(new EntityKey("Member", 1L), [1L, "A", 31]);
     */
    
    // 트랜잭션 종료
    em.close(); // 영속성 컨텍스트 소멸
}
```

`Spring 프레임워크를 사용한 코드`

```java
// @Transactional 사용으로 트랜잭션 코드 안써도 됨

@Service
public class AService {
    // Spring 컨테이너에서 관리하는 미리 만들어진 프록시 객체 주입
    @PersistenceContext
    private final EntityManager em;

    // @Transactional 어노테이션(트랜잭션 시작/종료 코드가 사라짐) - 비지니스 로직에 집중할 수 있음
    @Transactional
    public void a() {
        // 엔티티 생성 후 영속성 컨텍스트에 저장
        Member member = new Member();
        member.setName("A");
        member.setAge(30);
        em.persist(member); 

        // 엔티티 수정(변경 감지)
        member.setAge(31);
    }
}
```

`Spring data JPA를 사용한 코드`

```java
// @Transactional 사용으로 트랜잭션 코드 사라짐

// 리포지토리 인터페이스 정의
public interface ARepository extends JpaRepository<Member, Long> {
    // 기본적인 CRUD 메서드가 자동으로 제공됨
}

@Service
@RequiredArgsConstructor
public class AService {

    // EntityManager 대신 Repository를 주입받음(Spring이 만든 프록시 구현체가 주입)
    private final ARepository aRepository;

    @Transactional
    public void a() {
        // 엔티티 생성 후 영속성 컨텍스트에 저장
        Member member = new Member();
        member.setName("A");
        member.setAge(30);
        aRepository.save(member);

        // 엔티티 수정 (변경 감지)
        member.setAge(31);
    }
}
```
