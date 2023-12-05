## 프록시

---

엔티티를 조회할 때 연관된 엔티티들이 항상 사용되는 것은 아니다. 연관 관계의 엔티티는 비즈니스 로직에 따라 사용될 때도 있지만 그렇지 않을 때도 있다. 

JPA 는 이런 문제를 해결하려고 엔티티가 실제 사용될때까지 DB 조회를 지연하는 방법을 제공한다.

이것을 `지연로딩` 이라고 한다. `지연로딩`을 사용하려면 실제 엔티티 객체 대상에 DB 조회를 지연할 수 있는 가짜 객체가 필요하다. 이것을 `프록시 객체`라고 한다.

### 프록시 기초

---

EntityManager.find() 를 사용하면 영속성 컨텍스트에 엔티티가 없으면 DB를 조회한다. 

```java
	Member member = em.find(Member.class, "member1");
```

실제 사용하는 시점까지 DB 조회를 미루고 싶다면 `EntityManager.getReference()` 메소드를 사용한다. 이 메소드를 호출하면 DB 접근을 위임한 `프록시 객체`를 반환한다. 

```java
	Member member = em.getReference(Member.class, "member1");
```

프록시 객체는 실제 객체에 대한 참조를 보관한다. 프록시 객체의 메소드를 호출하면 프록시 객체는 실제 객체의 메소드를 호출한다. 이것을 `프록시 객체 초기화`라고 한다.

![image](https://github.com/4mjeo/TIL/assets/129156398/afe3be67-c0aa-43b9-b7b9-97ad64e4be8c)


### 특징

---

- 처음 사용할 때 한번만 초기화 된다.
- 초기화를 한다고 해서 **실제 엔티티로 바뀌는 것은 아니다.** 프록시 객체가 초기화되면 프록시 객체를 통해 실제 객체로 접근할 수 있다.
- 프록시 객체는 원본 엔티티를 상속받은 객체이므로 타입 체크 시에 주의해야 한다.
- 영속성 컨텍스트에 찾는 엔티티가 이미 있다면 DB 조회 필요가 없으므로, em.getReference() 를 호출해도 프록시가 아닌 실제 엔티티를 반환한다.
- 초기화는 영속성 컨텍스트의 도움을 받아야 한다. 영속성 컨텍스트의 도움을 받을 수 없는 준영속 상태의 프록시를 초기화한다면 문제가 발생한다.

### 프록시와 식별자

---

```java
Team team = em.getReference(Team.class, "team1"); //식별자 보관
team.getId(); //초기화 되지 않는다.
```

프록시 객체는 식별자 값을 가지고 있으므로 식별자 값을 조회하는 team.getId()를 호출해도 프록시를 초기화하지 않는다.

`PersistenceUnitUtil.isLoaded(Object entiity)` 메소드를 사용하면 프록시 인스턴스의 초기화 여부를 확인할 수 있다. 

```java
boolean isLoad = em.getEntityManagerFactory().getPersistenceUnitUtil().isLoaded(entity);
//또는 boolean isLoad = emf.getPersistenceUnitUtil().isLoaded(entity);

System.out.println("isLoad = " + isLoad); // 초기화 여부 확인
```

> 하이버 네이트의 initialize() 메소드를 사용해서 프록시를 강제 초기화 시킬수도 있다.
> 

### 즉시 로딩과 지연 로딩

---

회원 엔티티를 조회할 때 연관된 팀 엔티티도 함께 DB에서 조회하는 것이 좋을까..?

아니면 회원 엔티티만 조회해두고 팀 엔티티는 실제 사용하는 시점에 DB에서 조회하는 것이 좋을까? **정답은 없음!!**

- 즉시로딩 : 엔티티를 조회할 때 연관된 엔티티도 함께 조회한다.
    - @ManyToOne(fetch = FetchType.EAGER)
- 지연로딩 : 연관된 엔티티를 실제 사용할 때 조회한다.
    - @ManyToOne(fetch = FetchType.LAZY)

### 즉시 로딩

---

즉시 로딩을 최적화하기 위해서는 조인 쿼리를 사용한다. 여기서는 회원과 팀을 조인해서 쿼리 한번으로 두 엔티티를 모두 조회한다. 

**NULL 제약조건과 JPA 조인 전략**

외부 조인보다 내부 조인이 성능과 최적화에서 더 유리하다. 

**내부 조인을 사용하려면 어떻게 해야할까?**

외래키에 NOT NULL 제약 조건을 설정하면 값이 있는 것을 보장한다. NOT NULL 을 표현하는 방법은 두 가지가 있다. 

- @JoinColumn(name = “TEAM_ID”, nullable = false)
- @ManyToOne(fetch = FetchType.EAGER, optional = false)

→ JPA는 **선택적 관계면 외부조인을 사용**하고, **필수 관계면 내부 조인**을 사용한다.

### 지연 로딩

---

조회 대상이 영속성 컨텍스트에 이미 있으면 프록시 객체를 사용할 이유가 없다. 따라서 프록시가 아닌 실제 객체를 사용한다. 

하이버네이트는 엔티티를 영속 상태로 만들 때 엔티티에 컬렉션이 있으면 컬렉션을 추적하고 관리할 목적으로 원본 컬렉션을 하이버네이트가 제공하는 내장 컬렉션으로 변경하는데 이것을 `컬렉션 래퍼`라고 한다. 

```java
@Entity
public class Member {
	@Id
	private String id;

	@OneToMany(mappedBy = "member", fetch = FetchType.LAZY)
	private List<Order> orders;
}
```

### JPA 기본 패치 전략

---

- @ManyToOne, @OneToOne : 즉시 로딩(FetchType.EAGER)
- @OneToMany, @ManyToMany : 지연 로딩(FetchType.LAZY)

JPA의 기본 패치 전략은 연관된 엔티티가 하나면, `즉시 로딩` 을 컬렉션이면, `지연 로딩`을 사용한다.

### FetchType.EAGER 사용시 주의점

---

- **컬렉션을 하나 이상 즉시 로딩하는 것은 권장하지 않는다.**
- **컬렉션 즉시 로딩은 항상 외부 조인을 사용한다.**

FetchType.EAGER 설정과 조인 전략

- @ManyToOne, @OneToOne
    - (optional = false) : 내부 조인
    - (optional = true) : 외부 조인
- @OneToMany, @ManyToMany
    - (optional = false) : 외부 조인
    - (optional = true) : 내부 조인
    

## 영속성 전이: CASCADE

---

특정 엔티티를 영속 상태로 만들 때 연관된 엔티티도 함께 영속 상태로 만들고 싶으면 영속성 전이 기능을 사용하면 된다. JPA는 CASCADE 옵션으로 영속성 전이를 제공한다.

```java
private static void saveNoCascade(EntityManager em) {
    // 부모 저장
    Parent parent = new Parent();
    em.persist(parent) ;

    // 1번 자식 저장
    Child child1 = new Child();
    child1.setParent(parent); //자식 -> 부모 연관관계 설정
    parent.getChildren().add(childl) ; //부모 -> 자식
    em.persist(childl);

    // 2번 자식 저장
    Child child2 = new Child();
    child2.setParent(parent); //자식 -> 부모 연관관계 설정
    parent.getChildren().add(child2); //부모 -> 자식
    em.persist(child2);
}
```

JPA에서 엔티티를 저장할 때 연관된 모든 엔티티는 영속 상태여야 한다. 따라서 부모, 자식 엔티티 각각 영속 상태로 만든다.

### 영속성 전이: 저장

---

```java
@Entity
public class Parent {
	...

	@OneToMany(mappedBy = "parent", cascade = CascadeType.PERSIST)
	private List<Child> children = new ArrayList<Child>();
}
```

```java
private static void saveWithCascade(EntityManager em) {
    Child child1 = new Child();
    Child child2 = new Child();

    Parent parent = new Parent();
    childl.setParent(parent) ; //연관관계 추가
    child2.setParent(parent) ; //연관관계 추가
    parent.getChildren().add(child1);
    parent.getChildren().add(child2);
    
    //부모저장, 연관된자식들저장
    em.persist(parent);
}
```

영속성 전이는 연관 관계를 매핑하는 것과는 아무 관련이 없다. 단지 엔티티를 영속화할 때 연관된 엔티티도 같이 영속화하는 편리함을 제공할 뿐이다. 

### CASCADE 종류

---

```java
public enum CascadeType {
    ALL, //모두 적용
    PERSIST, //영속
    MERGE, //병합
    REMOVE, //삭제
    REFRESH, //REFRESH
    DETACH //DETACH
}
```

참고로 CascadeType.PERSIST, CascadeType.REMOVE, em.persist(), em.remove() 를 실행할 때 바로 전이가 발생하지 않고 **`플러시`**를 호출할 때 전이가 발생한다.

### 영속성 전이 + 고아 객체, 생명주기

---

`CascadeType.ALL` + `orphanRemoval = true` 를 동시에 사용하면 어떨가?

일반적으로 엔티티는 EntityManager.persist() 를 통해 영속화되고 EntityManager.remove() 를 통해 제거된다. 이것은 엔티티 스스로 생명주기를 관리한다는 뜻이다. 그런데 두 옵션을 모두 활성화하면 부모 엔티티를 통해 자식의 생명주기를 관리할 수 있다. 

> 영속성 전이는 DDD의 Aggregate Root 개념을 구현할 때 사용하면 편리하다.
> 

## 정리

---

- JPA 구현체들은 객체 그래프를 마음껏 탐색하게 지원하는데 이때 프록시 기술을 사용한다.
- 객체를 조회할 때 연관된 객체를 즉시 로딩하는 방법을 `즉시 로딩`이라고 하고, 연관된 객체를 지연해서 로딩하는 방법을 `지연 로딩`이라고 한다.
- 객체를 저장하거나 삭제할 때 연관된 객체도 함께 저장하거나 삭제할 수 있는데 이것을 `영속성 전이`라고 한다.
- 부모 엔티티와 연관 관계가 끊어진 자식 엔티티를 자동으로 삭제하려면 `고아 객체 제거 기능`을 사용해야 한다.