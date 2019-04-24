> 자바 ORM 표준 JPA 프로그래밍을 읽고 정리한 내용입니다.

# JPA 요약

목차
- [1.JPA 소개]()
- [2.JPA 시작하기]()
- [3.영속성 관리]()
- [4.엔티티 매핑]()
- [5.연관관계 매핑 기초]()
- [6. 다양한 연관관계 매핑]()
- [7. 고급 매핑]()
- [8. 프록시와 연관관계 관리]()
- [9. 값 타입]()




## 1. JPA 소개 
- SQL을 직접 다룰 때 문제점
- 객체 지향 언어와 데이터베이스간의 패러다임 불일치 문제 발생
- JPA 가 해결



```
JPA는 같은 트랜잭션이 때 같은 객체가 조회되는 것을 보장
왜? 1차 캐쉬를 생각해봐!
```




## 2. JPA 시작하기

### 1) 순수 자바 환경에서

엔티티 매니저 팩토리를 직접만들어 엔티티 매니저를 생성한다.

- Persistence가 Spring의 application.properties 메타 정보를 읽는다. 
- Persistence.createEntityManagerFactory() 매니저 팩토리를 생성한다. 
- 매니저 팩토리는 EntityManager를 여러 개 만든다. 
- EntityManger가 CRUD의 기능을 한다. 

### 2)Spring 프레임워크에서

컨테이너가 엔티티 매니저를 관리하고 제공한다. 

* `@PersistenceContext`  는 컨테이너가 관리하는 엔티티 매니저를 주입하는 어노테이션

```
Insert, update, 단건 조회는 SQL이 필요없다. 하지만, 검색 쿼리에서 여러건들을 조회할 때는 특정 조건의 쿼리문이 필요하다. 여기서 등장한게 JPQL이다.(SQL은 테이블을 다룬다 / JPQL은 엔티티를 다룬다 )
```




## 3. 영속성 관리 

영속성 컨텍스트(Persistence Context) : **엔티티를 영구히 저장하는 환경**

### 엔티티 생명주기

![](https://www.objectdb.com/files/images/manual/jpa-states.png)

- 비영속: 영속성 컨텍스트와 전혀 관계 없는 상태
- 영속: 영속성 컨텍스트에 저장 된 상태
- 준영속 : 영속성 컨텍스트에 저장 되었다가, 분리된 상태
- 삭제: 삭제된 상태



### 1) 조회 

영속 컨텍스트에서 관리한다. @Id 유니크한 키 값을 통해서 Map형태로 저장된다. 메모리 상(1차 캐쉬)에서 엔티티를 조회한다. 없으면 DB에서 조회한다. 

### 2) 저장 

`persist(memberA)` 를 호출하면 1)영속성 컨텍스에 올라가고,  2)Insert 쿼리가 쓰기 지연 저장소에 올라간다. (아직 DB에 저장은 되지 않은 상태) 

트랜잭션을 커밋하는 순간(스프링에서 @Transactional 을 단 함수가 끝났을 때 )  DB에 flush를 한다. 

### 3) 수정

영속성 상태에서 관리하는 엔티티인 경우에 변화에 감지한다. 실제 영속성 컨텍스트에서 엔티티 외에 snapshot이 떠져있다. `update(memberA)`  이런 함수가 존재하지 않는다. 

1. 트랜잭션 커밋이 일어나면 
2. 엔티티와 스냅샷을 비교하고
3. UPDATE SQL문을 쓰기 지연 저장소에 저장한다. 
4. flush()를 통해 DB에 업데이트를 한다.

```
원래는 name, age, grade 필드 중 name, age만 변경하면, 변경 필드만 update될 것 같지만 실제로는 전체 필드를 업데이트 하는 쿼리를 만든다. 해당 필드만을 원한다면 Hibernate로 확장 하면 된다.  `@DynamicUpdate` 
```

### 4) 삭제
먼저 삭제 대상 엔티티를 조회한 후, 삭제 대상 엔티티를 넘겨서 삭제 한다. 삭제 쿼리를 쓰기 지연 SQL저장소에 등록한다. 이 후, 트랜잭션 커밋해서 플러시를 호출하면 실제 데이터 베이스에 삭제 쿼리를 전달함.

## 4.엔티티 매핑
- 객체와 테이블 매핑: `@Entity`, `@Table`
- 기본 키 매핑: `@Id`
- 필드와 컬럼 매핑: `@Column`
- 연관관계 매핑: `@ManyToOne`, `@JoinColumn`

```
연관관계 매핑시 `@JoinColumn`를 사용하지 않아도, 필드의 이름의 기본키를 매핑 해주지만 기왕이면 **명시적으로 표시** 하는게 바람직함
```


## 5. 연관관계 매핑 기초
- 방향성: 객체에는 방향성이 존재, 테이블은 양방향
- 연관관계: 1:N,1:1 관계 생성
- 연관관계의 주인: 객체를 양방향 연관관계로 만들면 연관관계의 주인을 정해야한다. 

```
- 연관관계의 주인은 외래키가 있는 곳
- 양방향 연관관계는 연관관계 편의 메소드를 통해서 작성
```

## 6. 다양한 연관관계 매핑
## 7. 고급 매핑
- 매핑 정보만 상속 -`@MappedSuperclass` 추상클래스와 비슷, 공통되는 부분을 정의하고(테이블은 아님), 상속받아서 사용 - 등록일자, 수정일자에 사용함 

### 식별 관계, 비식별 관계
- 식별관계: 부모테이블의 기본키를 자식 테이블로 전파하면서 키가 늘어남(유연하지 못함) 하지만 별도의 인덱스를 생성할 필요가 없는 장점도 가짐
- 비식별관계: 필수적 비식별 관계(NOT NULL로 항상 관계가 있다는 것을 보장함) vs 선택적 비식별 관계(NULL을 허용, 조인할 때 외부 조인을 사용한다.)
```
정리: 될 수 있으면 비식별 관계를 사용하고 기본키는 Long 타입의 대리키를 사용하는 것
```
### 데이터베이스 테이블의 연관관계 설계 하는 방법 2가지
- 조인 컬럼 사용(외래키)
- 조인 테이블 사용(테이블 사용)


## 8. 프록시와 연관관계 관리

즉시로딩: 대부분의 JPA구현체는 즉시 로딩을 최적화 하기 위해 가능하면 **조인 쿼리를 사용한다**
지연로딩: **프록시**를 통해서 연관된 객체 탐색시, 원하는 시점에 데이터베이스를 조회할 수 있다.

### 프록시 기초
JPA에서 식별자로 엔티티 하나를 조회할 때 `EntityManager.find()`를 사용한다. 이 메서드는 영속성 컨텍스트에 엔티티가 없으면 데이터 베이스를 조회한다.
```java
Member member = em.find(Member.class, "member1");
```
만약에 엔티티를 실제 사용하는 시점까지 데이터베이스 조회를 미루고 싶으면 `EntityManger.getReference()`를 사용한다. 
```java
Member member = em.getReference(Member.class, "member1");
```
이 메소드를 호출할 때 JPA는 데이터베이스를 조회하지 않고 실제 엔티티 객체도 생성하지 않는다. 대신에 데이터베이스 접근을 위임한 프록시 객체를 반환한다. 


즉시로딩 시, JPA는 외부조인(Left Outer) 조인을 사용한다. 그 이유는 내부조인을 사용하게 되면, 외래키 null값이 존재하는 경우, 우리가 원하는 어떤 데이터도 조회할 수 없기 때문이다. 성능적인 측면에서 내부 조인이 좋기 때문에. 내부조인을 사용할려면, 외래키를 not null 조건으로 특정해야 한다
```java 
@Entity
public class Member {
  //...
  @ManyToOne(fetch = FetchType.EAGER)
  @JoinColumn(name="TEAM_ID", nullable = false)
  private Team team;
  //...
}
```

### 영속성 전이: CASCADE
- 특정 엔티티를 영속상태로 만들 때 연관된 엔티티도 함께 영속 상태로 만들고 싶으면 영속성 전이 기능을 사용한다.
- 영속성 전이는 연관관계를 매핑하는 것과는 아무 관련이 없다. 단지 연관된 엔티티도 같이 영속화하는 편리함을 제공할 뿐 


## 9. 값 타입

### 복합 값 타입 (Embeddable, Embedded)
```java 
@Entity
public class Member {
  
  @Id @GeneratedValue
  private Long id;
  private String name;
  
  @Embedded Period workPeriod;
  @Embedded Address homeAddress
}
```

```java
@Embeddable
public class Period{

  @Temporal(TemporalType.DATE) Date startDate;
  @Temporal(TemporalType.DATE) Date endDate;
  
}
```

```java
@Embeddable
public class Address{

  @Column(name="city")
  private String city;
  private String street;
  private String zipcode;
  
}
```



### 값 타입과 불변객체
객체의 공유참조는 피할수 없다. - 자바는 기본타입이면 값을 복사하고, 객체면 참조를 넘긴다.
1. 단순한 방법은 setter()메소드를 제거하는 것
2. 불변객체로 만드는 것 ( 1번의 방법을 통해서 구성함 )

|종류 |엔티티 타입  |값 타입|
-----|----------|-----|
|1   |식별자가 있다 |식별자가 없다|
|2   |생명주기가 있다|생명주기가 없다|
|3   |참조값을 공유한다.|공유하지 않는 것이 안전|





