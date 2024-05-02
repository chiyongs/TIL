# JPA 기본 정리

객체 지향 프로그래밍과 관계형 데이터베이스 간의 패러다임 불일치

- 상속
- 연관관계
- 데이터 타입
- 데이터 식별 방법

## 상속

상속관계의 객체를 DB에 저장하려면,
각각의 테이블에 따른 JOIN SQL을 작성하고, 각각의 객체를 생성하고 등등 여러 작업들이 필요하다.
→ DB에 저장할 객체에는 상속 관계를 사용하지 않게 됨

하지만, 자바 컬렉션에 저장된 객체는 조회 시 부모 타입으로 조회 후 다형성을 활용할 수 있다.

## 연관관계

테이블에 맞춰 객체를 모델링하게 되면 객체에 연관관계를 위한 외래 키 필드가 필요해진다.
하지만, 객체 지향에서는 객체 자체의 참조로 연관관계를 맺어서 객체 그래프를 순회할 수 있다.

## 엔티티 신뢰 문제

SQL로 데이터를 조회할 때 범위에 맞춰 데이터를 가져오게 되는데 객체 그래프를 순회할 때 해당 데이터가 조회되지 않아 데이터가 채워지지 않은 경우가 발생할 수 있다.
→ 용도에 따라 동일한 엔티티를 여러 번 조회해야 하는 경우가 발생

또한, 동일한 쿼리로 조회하여 만들어진 두 객체가 `==` 연산자로 비교 시 false가 나오게 된다.

객체를 객체답게 모델링할 수록 매핑 작업이 늘어남
→ 객체를 자바 컬렉션에 저장하듯이 DB에 저장하는 방법
→ JPA

## JPA

자바 진영의 ORM 기술의 표준

> ORM

Object-Relational Mapping

- 객체는 객체대로 설계하며 관계형 데이터베이스는 관계형 데이터베이스대로 설계
- ORM 프레임워크가 중간에서 매핑

이점
→ SQL 중심적인 개발에서 객체 중심으로 개발이 가능
→ 생산성 및 유지보수성 향상
→ 데이터 접근 추상화 및 벤더 독립성 등등..

### 생산성

CRUD

- Create : jpa.persist
- Read : jpa.find
- Update : entity.setXX
- Delete : jpa.remove

### 유지보수성

객체 필드 추가 시 SQL 자동 수정

### 성능 최적화

- 1차 캐시와 동일성 보장
  - 같은 트랜잭션 안에서는 같은 엔티티를 반환 → 약간의 조회 성능 향상
  - DB Isolation Level : Read Committed 여도 애플리케이션에서 Repeatable Read 보장
- 트랜잭션을 지원하는 쓰기 지연
  - 트랜잭션을 커밋할 때까지 Insert SQL을 모으고, JDBC BATCH SQL 기능을 사용해서 한 번에 SQL 전송
  - Update, Delete로 인한 Row 락 시간 최소화
  - 트랜잭션 커밋 시 Update, Delete SQL 실행하고 커밋
- 지연 로딩
  - 지연 로딩 : 객체가 실제 사용될 때 로딩
  - 즉시 로딩 : Join SQL로 한 번에 연관된 객체까지 미리 조회

## JPA 구동 방식

1. `Persistence` 클래스가 설정 정보를 조회
2. `Persistence` 클래스가 `EntityManagerFactory`를 생성
3. `EntityManagerFacotry`가 `EntityManager`를 생성

EntityManagerFactory : 1개만 생성해서 애플리케이션 전체에서 공유
EntityManager : 쓰레드 간 공유 X (사용 후 버려야 함)
JPA의 모든 데이터 변경은 트랜잭션 안에서 실행

## JPQL

JPQL : Java Persistence Query Language
-> JPA는 SQL을 추상화한 객체 지향 쿼리 언어를 제공
SQL과 문법이 유사
엔티티 객체를 대상으로 쿼리 -> 테이블이 아닌 객체를 대상으로 검색하는 객체 지향 쿼리
-> 데이터베이스에 의존적이지 않음

## 영속성 컨텍스트

JPA에서 가장 중요한 2가지

- 객체와 관계형 데이터베이스 매핑
- 영속성 컨텍스트

영속석 컨텍스트는 논리적인 개념으로 눈에 보이지 않는다.
엔티티매니저를 통해 영속성 컨텍스트에 접근 가능

엔티티매니저 : 영속성 컨텍스트 = N : 1

### 엔티티 생명주기

- 비영속(new / transient)
  - 영속성 컨텍스트와 전혀 관계가 없는 새로운 상태
- 영속 (managed)
  - 영속성 컨텍스트에 관리되는 상태
- 준영속 (detached)
  - 영속성 컨텍스트에 저장되었다가 분리된 상태
- 삭제 (removed)
  - 삭제된 상태

### 영속성 컨텍스트의 이점

- 1차 캐시
- 동일성 보장
- 트랜잭션을 지원하는 쓰기 지연
- 변경 감지 (Dirty Checking)
- 지연 로딩

> 변경 감지
>
> 트랜잭션 커밋 요청이 들어오면 엔티티와 스냅샷을 비교하여 변경이 발생했다면 해당 변경에 대한 SQL을 생성하여 자동으로 반영해주는 기능

### flush

영속성 컨텍스트의 변경 내용을 데이터베이스에 반영
발생 기준

- 변경 감지
- 수정된 엔티티 쓰기 지연 SQL 저장소에 등록
- 쓰기 지연 SQL 저장소의 쿼리를 데이터베이스에 전송

영속성 컨텍스트를 플러시하는 방법

- em.flush - 직접 호출
- 트랜잭션 커밋 - flush 자동 호출
- JPQL 쿼리 실행 - flush 자동 호출

영속성 컨텍스트를 비우지 않고, 영속성 컨텍스트의 변경내용을 DB에 동기화한다.
-> 트랜잭션 작업 단위가 중요! 커밋 직전에만 동기화하면 됨

### 준영속

영속 상태의 엔티티를 영속성 컨텍스트에서 분리
-> 영속성 컨텍스트의 기능을 사용 못함

준영속 상태로 만드는 방법

- em.detach(entity)
  - 특정 엔티티만 준영속 상태로 전환
- em.clear()
  - 영속성 컨텍스트 초기화
- em.close()
  - 영속성 컨텍스트 종료

## 엔티티 매핑

- 객체와 테이블 매핑 시 : `@Entity`, `@Table`
- 필드와 컬럼 매핑 시 : `@Column`
- 기본 키 매핑 시 : `@Id`
- 연관관계 매핑 시 : `@ManyToOne`, `@JoinColumn` …

> `@Entity` 사용시 주의사항

- 기본 생성자 필수
- final 클래스, enum, interface, inner 클래스 사용 X
- 저장할 필드에 final 사용 X

### 필드와 컬럼 매핑

- 컬럼 매핑 : `@Column`
- 날짜 타입 매핑 : `@Temporal`
- Enum 타입 매핑 : `@Enumerated`
  - EnumType.ORDINAL 사용 X → Enum 순서를 DB에 저장하기 때문에, Enum 순서 변경 시 데이터가 다 틀어짐
- BLOB, CLOB 매핑 : `@Lob`
- 특정 필드를 컬럼에 매핑하지 않음 : `@Transient`

### 기본 키 매핑

- `@Id`
- `@GeneratedValue`
  - IDENTITY : 데이터베이스에 위임, MySQL
  - SEQUENCE : 데이터베이스 시퀀스 오브젝트 사용, Oracle
    - DB에서 시퀀스 기능을 지원해야만 사용 가능한 전략
    - `@SequenceGenerator` 필요
  - TABLE : 키 생성용 테이블 사용, 모든 DB에서 사용 가능
    - `@TableGenerator` 필요
  - AUTO : 방언에 따라 자동 지정, 기본 값

## 연관관계 매핑

> 객체를 테이블에 맞춰 데이터 중심으로 모델링하면 협력 관계를 만들 수 없다.

테이블은 외래 키로 조인을 사용해 연관된 테이블을 찾는다.
객체는 참조를 사용해서 연관된 객체를 찾는다.
→ 패러다임 불일치

- 단방향 연관관계
- 양방향 연관관계
  - 양방향 매핑 규칙
    - 객체의 양방향은 사실 단방향 2개로 이루어진 것
    - 객체의 두 관계 중 하나를 연관관계의 주인으로 지정
    - 연관관계의 주인만이 외래 키를 관리 (연관관계의 주인 : 외래 키가 존재하는 곳)
    - 주인이 아닌 쪽은 읽기만 가능
    - 주인은 mappedBy 속성 사용 X
    - 주인이 아니면 mappedBy 속성으로 주인 지정
  - 연관관계 편의 메소드 생성
  - 양방향 매핑 시 무한 루프 조심
    - toString, lombok, json 생성 라이브러리
  - 단방향 매핑만으로도 이미 연관관계 매핑은 완료되었지만 양방향 매핑을 통해 반대 방향으로 조회 기능이 추가된 것뿐이다.
  - 따라서, 단방향 매핑만 사용하고 필요할 때 양방향을 추가해도 됨 (테이블에 영향 주지 않음)

### 다양한 연관관계 매핑

연관관계 매핑 시 다중성, 단방향&양방향, 연관관계의 주인 이 3가지를 고려해야 한다.

> 다대일 N:1

- 다대일 단방향
  - 가장 많이 사용하는 연관관계
  - 반대는 일대다
- 다대일 양방향
  - 외래 키가 있는 쪽이 연관관계의 주인
  - 양쪽 객체가 서로를 참조

> 일대다 1:N

- 일대다 단방향
  - 일이 연관관계의 주인 → 테이블에서는 다 쪽에 외래 키가 존재
  - 객체와 테이블의 차이로 반대편 테이블의 외래 키는 관리하는 특이한 구조
  - `@JoinColumn`을 꼭 사용!, 그렇지 않으면 중간에 테이블을 하나 추가하는 조인 테이블 방식을 사용함
  - 따라서, 일대다 단방향 매핑보다는 다대일 양방향 매핑을 사용 권장
- 일대다 양방향
  - 공식적으로 존재하지 않는 방식
  - 다대일 양방향을 사용해야 함

> 일대일 1:1

주 테이블이나 대상 테이블 중 외래 키 선택 가능한 형태

- 주 테이블에 외래 키가 존재하는 단방향
  - 다대일 단방향 매핑과 유사
- 주 테이블에 외래 키가 존재하는 양방향
  - 다대일 양방향 매핑과 유사
  - 외래 키가 있는 곳이 연관관계의 주인, 반대편은 mappedBy 적용
- 대상 테이블에 외래 키가 존재하는 단방향
  - JPA에서 지원하지 않음
- 대상 테이블에 외래 키가 존재하는 양방향
  - 주 테이블에 외래 키가 존재하는 양방향과 같음

주 테이블에 외래 키가 존재하는 방법은 객체 지향 개발자가 선호하며 JPA 매핑이 편리하다.
주 테이블만 조회해도 대상 테이블에 데이터가 있는지 확인이 가능하지만, 값이 없으면 외래 키에 null이 허용된다.

대상 테이블에 외래 키가 존재하는 방법은 전통적인 데이터베이스 개발자가 선호한다.
주 테이블과 대상 테이블을 일대일에서 일대다 관계로 변경하여도 테이블 구조가 유지되지만, 프록시 기능의 한계로 지연 로딩으로 설정해도 항상 즉시 로딩된다.

> 다대다 N:M

관계형 데이터베이스는 정규화된 테이블 2개로 다대다 관계를 표현 불가능
→ 연결 테이블을 추가하여 일대다, 다대일 관계로 풀어야함
→ 객체는 컬렉션을 사용해 객체 2개로 다대다 관계 가능

`@ManyToMany` 를 사용하여 다대다 매핑을 할 수 있지만 실무에서는 사용하지 않음
→ 연결 테이블이 단순히 연결만 하지 않고 추가적인 정보를 가져야하는 경우가 많기 때문
→ 중간 테이블을 엔티티로 승격하여 관리

### 고급 매핑 - 상속 관계 매핑

관계형 데이터베이스는 상속 관계가 없다.
슈퍼 타입-서브 타입 관계 모델링 기법이 객체 상속과 유사

- 각각의 테이블로 변환 → 조인 전략
- 통합 테이블로 변환 → 단일 테이블 전략
- 서브타입 테이블로 변환 → 구현 클래스마다 테이블 전략

> 조인 전략

장점 : 테이블 정규화, 외래 키 참조 무결성 제약조건 활용가능, 저장공간 효율화
단점 : 조회 시 조인을 많이 사용 → 성능 저하, 조회 쿼리 복잡

> 단일 테이블 전략

장점 : 조인 X → 일반적으로 조회 성능 빠름, 조회 쿼리 단순
단점 : 자식 엔티티가 매핑한 컬럼은 모두 null 허용, 단일 테이블에 모든 것을 저장 → 테이블이 커질 수 있음

> 구현 클래스마다 테이블 전략

추천 X

> `@MappedSuperclass`

공통 매핑 정보가 필요 시 사용
직접 생성해서 사용할 일이 없으므로 추상 클래스 권장
상속관계 매핑이 아니며, 엔티티도 아니고, 테이블과 매핑되지 않는다.
→ 조회, 검색 불가

부모 클래스를 상속받는 자식 클래스에 매핑 정보만 제공한다.
테이블과 관계 없고, 단순히 엔티티가 공통으로 사용하는 매핑 정보를 모으는 역할

주로 등록일, 수정일, 등록자, 수정자 같은 전체 엔티티에서 공통으로 적용하는 정보를 모을 때 사용
참고 : `@Entity` 클래스는 다른 엔티티나 `@MappedSuperclass` 로 지정한 클래스만 상속 가능

## 프록시

EntityManager.find() : 데이터베이스를 통해 실제 엔티티 객체 조회
EntityManager.getReference() : 데이터베이스 조회를 미루는 프록시 엔티티 객체 조회

프록시 객체는 실제 클래스를 상속받아서 만들어지고 실제 클래스와 겉모양이 같음
→ 사용하는 입장에서 구분하지 않고 사용 가능

프록시 객체는 실제 객체의 참조를 가지며, 프록시 객체 호출 시 실제 객체의 메소드를 호출

- 처음 사용 시에 한 번만 초기화
- 프록시 객체를 초기화하는 것이 프록시 객체가 실제 엔티티로 바뀌는 것은 아님. 프록시 객체 그대로

→ 타입 체크 시 == 연산자로 비교하면 실패함 → instanceOf 사용

- 영속성 컨텍스트에 실제 엔티티가 이미 존재하는 상태에서 프록시 객체를 조회한다면 실제 엔티티를 반환해줌

## 지연 로딩과 즉시로딩

> 지연 로딩

`@ManyToOne(fetch = FetchType.LAZY)`

지연로딩을 통해 프록시 객체를 가지고 있다 실제 엔티티 객체가 필요한 시점에 DB 조회

> 즉시 로딩

`@ManyToOne(fetch = FetchType.EAGER)`

즉시로딩 시 연관된 엔티티를 Join을 사용해서 항상 한 번에 같이 조회
실무에서는 가급적 지연 로딩만 사용하는 것을 권장
즉시 로딩 사용시 예상치 못한 SQL 발생 & JPQL에서 N+1 문제 발생

### N+1 문제

JPA에서 연관관계 매핑된 엔티티를 조회 시 의도치 않은 n번의 추가 쿼리가 발생하는 문제
지연 로딩, 즉시 로딩과의 연관성보단 JPQL로 인해 발생하는 문제
JPQL은 연관관계 데이터를 무시하고 해당하는 엔티티만을 조회하기 때문에 연관된 엔티티 데이터가 필요한 경우 FetchType으로 지정한 시점에 조회를 별도로 또 호출하게 된다.

해결방법

- Fetch join
- BatchSize
- FetchMode.SUBSELECT
- EntityGraph

### 영속성 전이 Cascade

특정 엔티티를 영속 상태로 만들 때 연관된 엔티티도 함께 영속 상태로 만들고 싶은 경우
(연관관계 매핑과는 관련이 없음, 그저 엔티티 영속화 시 연관된 엔티티도 함께 영속화하는 편리함만을 제공할 뿐)

- ALL : 모두 적용
- PERSIST : 영속
- REMOVE : 삭제
- MERGE : 병합
- REFRESH : 갱신
- DETACH : DETACH

### 고아 객체

부모 엔티티와 연관관계가 끊어진 자식 엔티티
`orphanRemoval=true` → 고아 객체 자동 삭제

참조가 제거된 엔티티는 다른 곳에서 참조하지 않는 고아 객체로 판단하여 삭제하는 기능
→ 참조하는 곳이 한 곳이어야 하고, 특정 엔티티가 해당 엔티티를 개인으로 소유할 때 사용
→ `@OneToOne`, `@OneToMany` 만 가능

## 값 타입

### 기본 값 타입

int, String, Integer…

### 임베디드 타입

새로운 값 타입 직접 정의, 주로 기본 값 타입을 모아 만들기 때문에 복합 값 타입이라고도 함

```java
@Entity
public class Member {
  @Id
  private Long id;
  private Long name;
  @Embedded
  private Period period;
}

@Embeddable
public class Period {
  LocalDateTime start;
  LocalDateTime end;
}
```

기본 생성자가 필수로 필요함

장점

- 재사용
- 높은 응집도
- 해당 클래스로 의미 있는 메소드 생성 가능
- 생명주기를 엔티티에 의존

> 값 타입과 불변 객체

임베디드 타입 같은 직접 정의한 값 타입 : Reference type

→ 복사 시 참조 값이 복사됨
→ 공유 참조로 인해 부작용 발생할 수 있음

값 타입을 불변 객체로 설계하여 부작용 차단해야 함

### 값 타입 컬렉션

- 하나 이상의 값 타입을 저장할 때 사용
- 컬렉션 저장을 위한 별도의 테이블이 필요

```java
@Entity
public class Member {
  ...
  @ElementCollection
  @CollectionTable(name = "FAVORITE_FOOD", joinColumns = @JoinColumn(name = "MEMBER_ID"))
  private Set<String> favoriteFoods = new HashSet<>();
```

값 타입은 엔티티와 다르게 식별자 개념이 없으므로 변경 발생 시 추적이 어렵다.
상황에 따라 값 타입 컬렉션 대신 일대다 관계를 사용하는 것이 좋다.
엔티티와 값 타입을 혼동해서 엔티티를 값 타입으로 만들면 안된다.
식별자가 필요하고, 지속해서 값을 추적, 변경해야 한다면 값 타입이 아닌 엔티티!

## JPA의 다양한 쿼리 방법

- JPQL
- Criteria
- QueryDSL
- Native SQL
- JDBC API, MyBatis, SpringJdbcTemplate…

### JPQL

JPQL은 테이블이 아닌 엔티티 객체를 대상으로 검색, 특정 DB 벤더에 의존적이지 않음

→ DB의 모든 데이터는 객체로 변환해서 검색하는 것은 불가능

→ 검색 시 필요한 조건이 포함된 SQL이 필요

```java
String jpql = "select m from Member m where m.name like '%hello%'";
List<Member> result = em.createQuery(jpql, Member.class).getResultList();
```

### Criteria

문자열이 아닌 자바로 JPQL을 작성할 수 있으며 JPQL 빌더 역할

너무 복잡하고 실용성이 없음 → QueryDSL 사용 권장

### QueryDSL

문자열이 아닌 자바로 JPQL 작성 및 JPQL 빌더 역할

→ 컴파일 시점에 문법 오류를 찾을 수 있으며 동적 쿼리 작성이 편리함

```java
JPAFactoryQuery query = new JPAQueryFactory(em);
QMember m = QMember.member;

List<Member> list = query.selectFrom(m)
                          .where(m.age.gt(18))
                          .orderBy(m.name.desc())
                          .fetch();
```

### Native SQL

SQL을 직접 사용하는 방법으로 JPQL로 해결할 수 없는 특정 데이터베이스에 의존적인 기능을 위해 사용

## JPQL 사용법

`select m from Member as m where m.age > 18`

- 엔티티와 속성은 대소문자 구분해야 하고, 키워드는 대소문자를 구분하지 않아도 됨
- 테이블 이름이 아닌 엔티티 이름을 사용해야 하며, 별칭은 필수이다.

- TypedQuery : 반환 타입이 명확한 경우
  - `TypedQuery<Member> query = em.createQuery("select m from Member m", Member.class);`
- Query : 반환 타입이 명확하지 않은 경우

  - `Query query = em.createQuery("select m.username, m.age from Member m");`

- query.getResultList() : 결과가 하나 이상인 경우
  - 결과가 없으면 emptyList 반환
- query.getSingleResult() : 결과가 정확히 하나인 경우
  - 결과가 없으면 `javax.persistence.NoResultException`
  - 결과가 2개 이상이면 `javax.persistence.NonUniqueResultException`

### 프로젝션

select 절에 조회할 대상을 지정하는 것

> 여러 값을 조회하는 경우

- Query 타입으로 조회
- Object[] 타입으로 조회
- new 명령어로 조회 (단순 값을 Dto로 바로 조회하는 경우)
  - `select new jpabook.jpql.UserDto(m.username, m.age) from Member m`
  - 풀 패키지 정보가 필요하며, 순서와 타입이 일치하는 생성자 필요

### 페이징

- setFirstResult(int startPosition) : 조회 시작 위치
- setMaxResults(int maxResult) : 조회할 데이터 수

JPA에서는 위 2 API로 페이징을 추상화하여 데이터베이스별로 달라질 수 있는 페이징 쿼리를 개발자가 신경쓰지 않아도 됨

### 조인

- 내부조인 : `SELECT m FROM Member m INNER JOIN m.team t`
- 외부조인 : `SELECT m FROM Member m LEFT OUTER JOIN m.team t`
- 세터조인 : `SELECT m FROM Member m, Team t WHERE m.username = t.name`

### 서브 쿼리

> 서브 쿼리 지원 함수

- [NOT] EXISTS (subquery) : 서브 쿼리 결과가 존재한다면 참
- ALL | ANY | SOME (subquery)
- [NOT] IN (subquery) : 서브 쿼리 결과 중 하나라도 같은 것이 있으면 참

JPA에서는 WHERE, HAVING 절에서만 서브 쿼리를 사용할 수 있음

→ FROM 절의 서브 쿼리는 현재 JPQL에서 불가능하므로 조인으로 풀 수 있으면 조인으로 해결

### 사용자 정의 함수 호출

하이버네이트는 사용전 방언에 추가해야 함
→ 사용하는 DB 방언을 상속받고, 사용자 정의 함수를 등록한다
`select function('group_concat', i.name) from Item i`

### 경로 표현식

`.`을 통해 객체 그래프 탐색하는 것
단순히 값을 저장하는 필드인 상태 필드는 경로 탐색의 끝이어서 더 이상 탐색이 불가능
연관관계를 위한 필드 중 단일 값 연관 경로는 묵시적 내부 조인이 발생하여 탐색이 가능
컬렉션 값 연관 경로는 묵시적 내부 조인이 발생하지만, 더 이상 탐색이 불가능하다
→ FROM 절에서 명시적 조인을 통해 별칭을 얻으면 별칭을 통해 탐색이 가능

> 묵시적 조인, 명시적 조인

묵시적 조인 : `select m.team from Member m`
명시적 조인 : `select m from Member m join m.team t`

묵시적 조인은 항상 내부 조인이며, 컬렉션은 마지막 경로 탐색 지점이므로, 명시적 조인을 통해 별칭을 얻어야 더 탐색이 가능하다.

→ 실무에서는 가급적 명시적 조인을 사용하는 것이 좋음 (묵시적 조인은 조인이 일어나는 상황을 한눈에 파악하기 어려움)

### 페치 조인 fetch join

SQL이 가지고 있는 join이 아니고, JPQL에서 성능 최적화를 위해 제공하는 기능이다.
연관된 엔티티나 컬렉션을 1번의 SQL로 함께 조회하기 위한 기능

→ `join fetch`

`select m from Member m join fetch m.team`
`select t from Team t join fetch t.members where t.name = '~~'`

페치 조인 사용 시 한 번에 연관관계를 같이 조회하기 때문에 지연 로딩 속성이 적용되지 않음

> DISTINCT

JPQL의 DISTINCT

- SQL에 DISTINCT 키워드를 추가
- 애플리케이션에서 엔티티의 중복을 제거

JPQL은 결과를 반환할 때 연관관계를 고려하지 않고, 단순히 SELECT 절에 지정한 엔티티만 조회하기 때문에 일반 조인은 연관된 엔티티를 함께 조회하지 않음

> 페치 조인의 한계

- 페치 조인 대상에는 별칭을 줄 수 없음
- 둘 이상의 컬렉션은 페치 조인 할 수 없음
- 컬렉션을 페치 조인하면 페이징 API를 사용할 수 없음

### 다형성 쿼리

- TYPE
  - 조회 대상을 특정 자식으로 한정
  - `select i from Item i where type(i) in (Book, Movie)`
- TREAT
  - 자바의 타입 캐스팅과 유사, 상속 구조에서 부모 타입을 특정 자식 타입으로 다룰 때 사용
  - `select i from Item i where treat(i as Book).author = 'kim'`

### 벌크 연산

JPA 더티 체킹으로 엔티티의 변경에 대한 SQL을 실행하게 되면 너무 많은 SQL을 실행해야 하는 경우가 존재
→ 한 번의 쿼리로 여러 테이블의 로우를 변경해야 함
영속성 컨텍스트를 무시하고 데이터베이스에 직접 쿼리하기 때문에 벌크 연산을 먼저 수행한후 영속성 컨텍스트를 초기화해줘야함
