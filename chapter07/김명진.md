# Chapter 7

### 상속 관계 매핑

---

RDBMS 에는 상속의 개념이 없다. 대신 슈퍼타입, 서브타입 관계라는 모델링 기법으로 대신할 수 있다. 실제 물리 모델인 테이블로 이를 구현하기 위한 3가지 방법이 있다.

- 각각의 테이블로 변환 - 조인 전략
- 통합 테이블로 변환 - 단일 테이블 전략
- 서브타입 테이블로 변환 - 구현 클래스마다 테이블 전략

### **조인 전략**

조인 전략은 엔티티 각각을 모두 테이블로 만들고 자식 테이블이 부모 테이블의 기본키를 받아서 기본키 + 외래키로 사용하는 전략이다. 조회할 때 조인을 자주 사용한다.

> 주의사항

- 객체는 타입으로 구분할 수 있지만 테이블은 타입의 개념이 없다.
- 타입을 구분하는 컬럼을 추가해야 한다. 여기서는 DTYPE 컬럼을 구분 컬럼으로 사용한다.
  >

```java
@Entity
@Inheritance(strategy = InheritanceType.JOINED) // 상속 매핑은 부모 클래스에 선언해야 한다.
@DiscriminatorColumn(name = "DTYPE") // 부모 클래스에 구분 컬럼을 지정한다.
public abstract class Item {

	@Id @GeneratedValue
	@Column(name = "ITEM_ID")
	private Long id;

	private String name; //이름
	private int price; //가격
	...
}

@Entity
@DiscriminatorValue("A") // 엔티티를 저장할 때 구분 컬럼에 입력할 값을 지정한다.
public class Album extends Item {
  private String artist;
	...
}

@Entity
@DiscriminatorValue("M")
@PrimaryKeyJoinColumn(name = "MOVIE_ID") // 자식 테이블의 기본 키 컬럼명을 변경 (기본 값은 부모 테이블의 ID 컬럼명)
public class Movie extends Item {
  private String director; //감독
  private String actor; //배우
	...
}
```

장점

- 테이블이 `정규화` 된다.
- `외래 키 참조 무결성` 제약 조건을 활용할 수 있다.
- 저장 공간을 효율적으로 사용한다.

단점

- 조회할 때 조인이 많이 사용되므로 성능이 저하될 수 있다.
- 조회 쿼리가 복잡하다.
- 데이터를 등록할 `INSERT` SQL을 두 번 실행한다

특징

- JPA 표준 명세는 구분 컬럼을 사용하도록 하지만 Hibernate 를 포함한 몇몇 구현체는 구분 컬럼 없이도 동작한다.(@DiscriminationColumn)

### 단일 테이블 전략

---

테이블을 하나만 사용한다. 그리고 구분 컬럼(DTYPE)으로 어떤 자식 데이터가 저장되었는지 구분한다. 조회할 때 조인을 사용하지 않으므로 일반적으로 가장 빠르다.

> 자식 엔티티가 매핑한 컬럼은 모두 null 을 허용해야 한다.

```java
@Entity
@Inheritance(strategy = InheritanceType.SINGLE_TABLE)
@DiscriminatorColumn(name = "DTYPE")
public abstract class Item {

	@Id @GeneratedValue
	@Column(name = "ITEM_ID")
	private Long id;

	private String name; //이름
	private int price; //가격
	...
}

@Entity
@DiscriminatorValue("A")
public class Album extends Item {
	...
}

@Entity
@DiscriminatorValue("M")
public class Movie extends Item {
	...
}
```

장점

- 조인이 필요 없으므로 일반적으로 조회 성능이 빠르다.
- 조회 쿼리가 단순하다.

단점

- 자식 엔티티가 매핑한 컬럼은 모두 null 을 허용해야 한다.
- 단일 테이블에 모든 것을 저장하므로 테이블이 커질 수 있다. (성능이 더 안좋아질 수 있다.)

특징

- 구분 컬럼을 꼭 사용행야 한다. (지정하지 않으면 기본으로 엔티티 이름을 사용한다.)

### 구분 클래스마다 테이블 전략

---

자식 엔티티마다 테이블을 만든다. 그리고 자식 테이블 각각에 필요한 컬럼이 모두 있다.

> 일반적으로 추천하지 않는다.

```java
@Entity
@Inheritance(strategy = InheritanceType.TABLE_PER_CLASS)
public abstract class Item {

	@Id @GeneratedValue
	@Column(name = "ITEM_ID")
	private Long id;

	private String name; //이름
	private int price; //가격
	...
}
```

장점

- 서브 타입을 구분해서 처리할 때 효과적이다.
- `not null` 제약 조건을 사용할 수 있다.

단점

- 여러 자식 테이블을 함께 조회할 때 성능이 느리다.(SQL에 UNION을 사용해야 한다.)
- 자식 테이블을 통합해서 쿼리하기 어렵다.

특징

- 구분 컬럼을 사용하지 않는다.

## @MappedSuperclass

---

상속 관계는 부모, 자식 클래스 모두 DB 테이블에 매핑 시켰는데 이것은 부모 클래스는 매핑하지 않고 상속 받은 자식 클래스에게 매핑 정보만 제공되고 싶을 때 사용한다.

> @Entity 와는 다르게 실제 테이블에 매핑되지는 않는다.

특징

- 테이블과 매핑되지 않고 자식 클래스에 엔티티의 매핑 정보를 상속하기 위해 사용한다.
- @MappedSuperclass로 지정한 클래스는 엔티티가 아니므로, em.find()나 JPQL을 사용할 수 있다.
- (직접 사용할 일이 없으므로 `추상 클래스`로 만들자)

@MappedSuperclass를 사용하면 등록일자, 수정일자, 등록자 같은 여러 엔티티에서 공통으로 사용하는 속성을 효과적으로 관리할 수 있다.

## 복합키와 식별 관계 매핑

---

**식별관계**

부모 테이블의 기본키를 내려받아서 자식 테이블의 기본키 + 외래키 로 사용하는 관계이다.

![7-6](https://github.com/4mjeo/Algorithm/assets/129156398/4e42a83d-5195-485a-b58e-8772bb629142)

**비식별 관계**

부모 테이블의 기본키를 받아서 자식 테이블의 외래키로만 사용하는 관계이다.

![7-7](https://github.com/4mjeo/Algorithm/assets/129156398/d7c58c43-6526-4ecd-8e1a-6b75d61fe47f)

- 필수적 비식별 관계 : 외래키에 null을 허용하지 않는다.
- 선택적 비식별 관계 : 외래키에 null을 허용한다. (대부분 비식별 관계로 유지한다.)

### 복합키

---

JPA는 복합키를 지원하기 때문에 @IdClass 와 @EmbeddedId 2가지 방법을 제공하는데 @IdClass는 관계형 데이터베이스에 가까운 방법이고 @EmbeddedId는 객체지향에 가까운 방법이다.

**@IdClass**

복합 키 테이블은 비식별 관계고 PARENT 는 복합 기본키를 사용한다.(객체의 상속과는 무관하다.)

![7-8](https://github.com/4mjeo/Algorithm/assets/129156398/128a1736-be07-4363-8772-210ab5c3b869)

> **@IdClass는 다음과 같은 조건을 만족해야 한다.**

- 식별자 클래스의 속성명과 엔티티에서 사용하는 식별자의 속성명이 같아야 한다.
- Serializable 인터페이스를 구현해야 한다.
- equals, hashCode를 구현해야 한다.
- 기본 생성자가 있어야 한다.
- 식별자 클래슨ㄴ public 이어야 한다.
  >

**@EmbeddedId**

좀 더 객체 지향적인 방법이다.

@EmbeddId 를 적용한 식별자 클래스는 식별자 클래스에 기본키를 직접 매핑한다.

> **@EmbeddId는 다음과 같은 조건을 만족해야 한다.**

- @Embeddable 어노테이션을 붙여야 한다.
- Serializable 인터페이스를 구현해야 한다.
- equals, hashCode 를 구현해야 한다.
- 기본 생성자가 있어야 한다.
- 식별자 클레스는 public 이어야 한다.
  >

### 복합 키와 equals(), hashCode()

---

복합키는 equals()와 hashCode()를 필수적으로 구현해야 한다.

```java
ParentId id1 = new parentId() ;
id1.setld1("myId1”);
id1.setld2("myId2”);

ParentId id2 = new parentId();
id2.setId1("myId1");
id2.setId2("myId2");

id1.equals(id2) -> ???
```

영속성 컨텍스트는 엔티티의 식별자로 키를 사용해서 엔티티를 관리한다. 식별자 객체의 동등성이 지켜지지 않으면 , 다른 엔티티가 조회되거나 엔티티를 찾을 수 없는 등 영속성 컨텍스트가 엔티티를 관리하는데 심각한 문제가 발생한다.

**@IdClass vs @EmbeddedId**

@EmbeddId 가 좀 더 객체 지향적이고 중복도 없어서 좋아 보이긴 하지만 특정 상황에 JPQL 이 조금 더 길어질 수 있다.

```java
em.createQuery("select p.id.id1, p.id.id2 from Parent p"); //@Embeddedld
em.createQuery("select p.id1, p.id2 from Parent p"); //@IdClass
```

### 복합키 : 식별 관계 매핑

---

식별 관계에서 자식 테이블은 부모 테이블의 기본키를 포함해서 복합키를 구성해야 하므로 @IdClass나 @EmbeddId를 사용해서 식별자를 매핑해야 한다.

![7-9](https://github.com/4mjeo/Algorithm/assets/129156398/51f8af6b-8ac8-42ff-8d7a-0ad0906ed673)

**@IdClass 와 식별 관계**

필요한 키만큼 JoinColumn 으로 자식에서 필요하다면 불러오면 된다.

**@EmbeddId 와 식별 관계**

- @MapsId를 사용한다.
- Id 클래스의 변수값을 @MapsId(”child”) 식으로 매핑한다.

```java
//비식별 관계 구현
//식별 관계 복합 키를 사용한 코드에 비하면 코드가 단순하다.
@Entity
public class Parent {
    @Id
    @GeneratedValue
    @Column(name = "PARENT_ID")
    private Long id;
    private String name;
    ...
}

@Entity
public class Child {
    @Id
    @GeneratedValue
    @Column(name = "CHILD_ID")
    private Long id;
    private String name;

    @ManyToOne
    @JoinColumn(name = "PARENT_ID")
    private Parent parent;
}
```

### 일대일 식별 관계

---

일대일 식별 관계는 자식 테이블의 기본 키 값으로 부모 테이블의 기본키 값만 사용한다.

### 식별, 비식별 관계의 장단점

---

**DB 설계 관점**

- 식별 관계는 부모 테이블의 기본키를 자식 테이블로 전파하면서 자식 테이블의 기본키 컬럼이 점점 늘어난다. 결국 조인할 때 SQL 이 복잡하고 기본키 인덱스가 불필요하게 커질 수 있다.
- 비즈니스 요구사항은 시간이 지남에 따라 언젠가는 변한다. 식별 관계의 자연키 컬럼들이 자식에 손자까지 전파되면 변경하기 힘들다.
- 식별 관계는 테이블 구조가 유연하지 못하다.

**객체 관계 매핑 관점**

- JPA 에서 복합키는 별도의 복합키 클래스를 만들어 사용해야 한다.
- JPA는 @GenerateValue 처럼 대리키를 생성하기 위한 편리한 방법을 제공한다. 그러나 식별 관계에서는 사용하기 힘들다.

**식별 관계가 가지는 장점도 있다.**

- 기본키 인덱스를 활용하기 좋다. (상위 테이블에서 정의해놓은 인덱스를 그대로 사용할 수 있다.)

### 조인 테이블

---

DB테이블의 연관 관계 설계 방법

- 조인 컬럼 사용(외래키)
  - 테이블 간의 관계는 주로 조인 컬럼이라 부르는 외래키 컬럼을 사용하여 관리한다.
    ![7-11](https://github.com/4mjeo/Algorithm/assets/129156398/7d342990-6701-4f2d-a731-91fcd5e0cdb3)
- 조인 테이블 사용(테이블 사용)
  - 연관 관계를 관리하는 조인 테이블을 추가하고 조인하려는 두 테이블의 외래키를 가지고 연관 관계를 관리한다.
  ![7-12](https://github.com/4mjeo/Algorithm/assets/129156398/6584d6b6-da1f-4be1-8af4-ccfd4963d78d)

### 일대일 조인 테이블

---

일대일 관계를 만드려면 조인 테이블의 외래키 컬럼 각각에 총 2개의 유니크 제약 조건을 걸어야 한다. (PARENT_ID 는 기본키이므로 유니크 제약조건이 걸려있다.)

```java
//부모
@Entity
public class Parent {
  @Id @GeneratedValue
  @Column(name = "PARENT_ID")
  private Long id;

  private String name;

  @OneToOne
  @JoinTable(name = "PARENT_CHILD",
    joinColumns = @JoinColumn(name = "PARENT_ID"),
    inverseJoinColumns = @JoinColumn(name = "CHILD_ID")
  )
  private Child child;
  ...
}

//자식
@Entity
public class ChiId {
  @Id @GeneratedValue
  @Column(name = "CHILD_ID")
  private Long id;

  private String name;
  ...
}
```

### 일대다 조인 테이블

---

일대다 관계를 만드려면 조인 테이블의 컬럼 중 다와 관련된 컬럼인 CHILD_ID에 유니크 제약 조건을 걸어야 한다.

### 다대일 조인 테이블

---

일대다 방향에서 방향만 반대이다.

### 다대다 조인 테이블

---

다대다 관계는 조인 테이블의 두 컬럼을 합해서 하나의 복합 유니크 제약 조건을 걸어야 한다.

### 엔티티 하나의 여러 테이블 매핑

---

@SecondaryTable 을 사용해서 @Table 로 먼저 매핑해주고 난다음 추가로 매핑할 수 있다.

**@SecondaryTable 속성**

- @SecondaryTable.name : 매핑할 다음 테이블로 이동
- @SecondaryTable.pkJoinColumns : 매핑할 다른 테이블의 기본키 컬럼 속성

여러 테이블에 접목하려면 @Column(table = "두번째 테이블")로 이어주면 된다.

더 많은 테이블을 매핑하려면 @SecondaryTables 사용

> @SecondaryTable 을 사용해서 두 테이블을 하나에 엔티티에 매핑하는 방법보다는 테이블당 엔티티를 각각 만들어서 일대일 매핑하는 것을 권장한다. 이 방법은 **항상 두 테이블을 조회하므로 최적화하기 어렵다.**
