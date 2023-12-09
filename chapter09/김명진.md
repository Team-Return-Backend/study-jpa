**값 타입**

JPA의 데이터 타입을 가장 크게 분류하면 `엔티티 타입`과 `값 타입`으로 나눌 수 있다. 비유하자면 엔티티 타입은 살아있는 생물이고 값 타입은 단순한 수치 정보이다.

## 기본값 타입

---

```java
@Entity
public class Member {
	@Id @GeneratedValue
	private Long id;

	private STring name;
	private int age;

}
```

Member 엔티티의 값 타입인 name, age 속성은 식별자 값도 없고 생명주기도 회원 엔티티에 의존한다. 따라서 회원 엔티티 인스턴스를 제거하면 name, age 값도 제거된다. 그리고 값 타입은 공유하면 안된다.

> 자바에서 int, double 같은 기본 타입은 절대 공유되지 않는다.

## 임베디드 타입(복합값 타입)

---

새로운 값 타입을 직접 정의해서 사용할 수 있는 JPA에서는 이걸 임베디드 타입이라고 한다.

- @Embeddable : 값 타입을 정의하는 곳에 표시
- @Embedded : 값 타입을 사용하는 곳에 표시
  임베디드 타입을 포함한 모든 값 타입은 엔티티의 생명주기에 의존한다.

**임베디드 타입과 테이블 매핑**

임베디드 타입 덕분에 객체와 테이블을 아주 세밀하게 매핑하는 것이 가능하다. 잘 설계한 ORM 애플리케이션은 **매핑한 테이블의 수보다 클래스 수가 더 많다.**

**@AttributeOverride : 속성 재정의**

임베디드 타입에 정의한 매핑 정보를 재정의하려면 엔티티에 @AttributeOverride 를 사용하면 된다.

```java
@Entity
public class Member {
  @Id @GeneratedValue
  private Long id;
  private String name;

  @Embedded
  Address homeAddress;

  @Embedded
  @AttributeOverrides({
    @AttributeOverride(name="city", column=@Column(name = "COMPANY_CITY")),
    @AttributeOverride(name="street", column=@Column(name = "COMPANY_STREET")),
    @AttributeOverride (name="zipcode", column=@Column (name = "COMPANY_ZIPCODE"))
  })
  Address companyAddress;
}
```

@AttirubuteOverride 를 사용하면 어노테이션을 너무 많이 사용해서 엔티티 코드가 더러워진다. 다행히도 한 엔티티에 같은 임베디드 타입을 중복해서 사용하는 일은 많지 않다.

## 값 타입과 불변 객체

값 타입은 복잡한 객체 세상을 조금이라도 단순화하려고 만든 개념이다.

### 값 타입 공유 참조

---

임베디드 타입 같은 값 타입을 여러 엔티티에서 공유하면 위험하다.

![images](https://github.com/4mjeo/Algorithm/assets/129156398/33051dd3-831e-431b-9391-90564da3bac9)

```java
member1.setHomeAddress(new Address("OldCity"));
Address address = member1.getHomeAddress();

address.setCity("NewCity");
member2.setHomeAddress(address);
```

회원 2의 주소만 “NewCity”로 변경되길 원했지만 회원1의 주소도 “NewCity”로 변경되어 버렸다. 이런 부작용을 막으려면 값을 복사해서 사용하면 된다.

### 값 타입 복사

---

- 값 타입의 실제 인스턴스인 값을 공유하는 것은 위험하므로 대신 인스턴스를 복사해서 사용한다.
- 자바에서 기본 타입에 값을 대입하면 알아서 값을 복사 전달하지만 객체는 참조값을 전달한다.
- 복사해서 쓰도록 해도 공유 참조 자체를 막을 수는 없기 때문에 setter 를 빼서 수정을 못하도록 막는것이 좋다.

### 불변 객체

---

**불변객체?** 한번 만들면 절대 변경할 수 없는 객체

- 객체 값을 수정 못하게 막으면 사이드 이펙트를 차단할 수 있다.
- 값 타입을 될 수 있으면 immutable object로 설계해야 한다.
  - ex) 생성자로만 값 설정, setter X
- Integer, String 은 자바가 제공하는 대표적인 불변 객체이다.

### 값 타입의 비교

---

자바가 제공하는 객체 비교는 2가지이다.

- 동일성 비교(indentity) : 인스턴스의 참조 값을 비교, `==`사용
- 동등성 비교(equivalance) : 인스턴스의 값을 비교, `equals()` 사용

값 타입을 비교할 때는 `equals()`를 사용해서 동등성 비교를 해야한다.

직접 값 타입을 정의할 경우에는 `equals()` 메소드를 재정의해줘야 한다.

- `equals()` 재정의 시 `hashCode()` 도 재정의 해줘야한다.
- 안하면 컬렉션이 정상 작동하지 않는다.

### 값 타입 컬렉션

---

값 타입을 하나 이상 저장하려면 컬렉션에 넣고 `@ElementCollection` , `@CollectionTable` 어노테이션을 사용하면 된다.

```java
@Entity
public class Member {
  @Id @GeneratedValue
  private Long id;

  @Embedded
  private Address homeAddress;

  @ElementCollection
  @CollectionTable(
    name = "FAVORITE_FOODS",
    joinColumns = @JoinColumn(name = "MEMBER_ID"))
  @Column(name="FOOD_NAME")
  private Set<String> favor丄teFoods = new HashSet<String>();

  @ElementCollection
  @CollectionTable(
    name = "ADDRESS”,
    joinColumns = @JoinColumn(name = "MEMBER_ID"))
  private List<Address> addressHistory = new ArrayList<Address>();
  //...
}

@Embeddable
public class Address {
  @Column
  private String city;
  private String street;
  private String zipcode
  //...
}
```

- favoriteFoods 처럼 값으로 사용되는 컬럼 하나만 `@Column`을 사용해서 컬럼명을 지정할 수 있다.
- @CollectionTable 을 생략하면 기본값으로 매핑된다.
  - default : {엔티티 이름}\_{컬렉션 속성 이름}
  - ex) Member 엔티티의 addressHistory는 Member_addressHistory 테이블과 매핑된다.

### 값 타입 컬렉션 사용

---

```java
Member member = new Member();

//임베디드 값 타입
member.setHomeAddress(new Address("통영", "몽돌해수욕장", "660-123"));

//기본값 타입컬렉션
member.getFavoriteFoods().add("짬뽕");
member.getFavoriteFoods().add("짜장");
member.getFavoriteFoods().add("탕수육");

//임베디드 값 타입 컬렉션
member.getAddressHistory().add(new Address("서울”, "강남"，"123-123"));
member.getAddressHistory().add(new Address("서울", "강북", "000-000"》);
em.persist(member);
```

마지막에 member 엔티티만 영속화했다. 따라서 em.persist(member) 한번 호출로 6번의 INSERT SQL 을 실행한다.

- member : INSERT 쿼리 1번
- member.homeAddress : 임베디드 값 타입이라 member 저장 쿼리에 포함
- member.favoriteFoods : INSERT 쿼리 3번
- member.addressHistory : INSERT 쿼리 2번

> 값 타입 컬렉션은 영속성 전이 + 고아 객체 제거 기능을 필수로 가진다고 볼 수 있다.

**값 타입 수정**

```java
Member member = em.find(Member.class, 1L);

// 임베디드 값 타입 수정
member.setHomeAddress(new Address("new a", "NEW A", "1-1"));

// 기본값 타입 컬렉션 수정
Set<String> favoriteFoods = member.getFoavriteFoods();
member.getFavoriteFoods().remove("Q");
member.getFavoriteFoods().add("W");

// 임베디드 값 타입 컬렉션 수정
List<Address> addressHistory = member.getAddressHistory();
member.getAddressHistory().remove(new Address("b", "B", "2"));
member.getAddressHistory().add(new Address("new b", "new B", "2-2"));
```

1. 임베디드 값 타입 수정 : homeAddress 임베디드 값 타입은 Member 테이블과 매핑했으므로 Member 테이블만 UPDATE 한다.

2. 기본값 타입 컬렉션 수정 : 자바의 String 타입은 수정 불가이므로 제거하고 새로 추가해야 한다.

3. 임베디드 값 타입 컬렉션 수정 : 값 타입은 불변해야 한다. 따라서 기존 주소를 삭제하고 새로운 주소를 등록했다. 참고로 값 타입은 equals, hashCode 를 필수로 등록해야 한다.

### 값 타입 컬렉션의 제약 사항

---

- 특정 엔티티 하나에 소속된 값 타입은 변경되어도 자신이 속한 엔티티를 DB에서 찾아 값을 바꾸면 된다.
  - 값 타입은 식별자라는 개념이 없고 단순한 값들의 모음이므로 값을 변경해버리면 DB에 저장된 원본 데이터를 찾기는 힘들다.
    - JPA 구현체들은 값 타입 컬렉션에 변경 사항이 발생하면 컬렉션이 매핑된 테이블에 연관된 모든 데이터를 삭제 후 현재 들어있는 값을 새로 DB에 저장한다.
- 값 타입 컬렉션을 매핑하는 테이블은 모두 컬럼을 묶어서 PK로 구성해야 한다.
  - PK 제약조건으로 nullable 하게 사용 못하고 값은 값 중복 저장도 안된다.

위 문제를 해결하려면 값 타입 컬렉션을 쓰는 대신 새 엔티티를 만들어서 1:N 관계로 사용하면 된다.

- 영속성 전이 + 고아 객체 제거 기능을 적용하면 값 타입 컬렉션처럼 사용할 수 있다.

### 정리

---

**엔티티 타입의 특징**

- 식별자가 있다.
- 생명주기가 있다.
- 공유할 수 있다.

**값 타입의 특징**

- 식별자가 없다.
- 생명주기를 엔티티에 의존한다.
- 공유하지 않는 것이 안전하다.
