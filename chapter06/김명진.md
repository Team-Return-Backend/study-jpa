# Chapter 6

연관 관계에서 조심해야 할 것

**다중성**

- 다대일(@ManyToOne)
- 일대다(@OneToMany)
- 일대일(@OneToOne)
- 다대다(@ManyToMany)

**단방향, 양방향**

- 테이블은 외래키 하나로 조인을 사용해서 양방향 쿼리가 가능하다.

> 웬만하면 양방향 매핑을 추천한다.

**연관 관계의 주인**

- 연관 관계의 주인이 아니면 mappedBy 속성을 사용하고 연관 관계의 주인 필드 이름을 값으로 입력해야 한다. (master 와 slave 명확하게 생각하기)

### 다대일

---

객체 양방향 관계에서 연관 관계의 주인은 항상 **다** 쪽이다. (1:N의 N쪽)

**다대일 단방향[1:N]**

![onetomany_one](https://github.com/Team-Return-Backend/study-jpa/assets/129156398/fa4b1851-6b1b-4092-9984-8e5458036760)

**다대일 양방향[N:1, 1:N]**

![onetomany_two](https://github.com/Team-Return-Backend/study-jpa/assets/129156398/febd1878-799b-41b3-9d74-af351b314579)

일대다와 다대일 연관 관계는 항상 다 쪽에 외래키가 있다. 여기서는 다 쪽인 Member 테이블이 외래키를 가지고 있으므로, Member.team 이 연관관계의 주인이다. (JPA는 외래키를 관리할 때 연관 관계의 주인만 사용한다.)

**양방향 연관 관계는 항상 서로를 참조해야 한다.**

연관 관계의 편의 메소드를 작성하는 것이 좋다.(setTeam(), addMember() 등)

```java
@Entity
public class Member {
	@Id @GeneratedValue
	@Column(name = "MEMBER_ID")
	private Long id;

	public void setTeam(Team team) {
		this.team = team;

		if(!team.getMembers().contains(this)) {
			team.getMembers().add(this);
		}
	}
}
```

### 일대다[1:N]

---

- 일이 연관 관계의 주인이다. (=일쪽에서 외래키를 관리한다.)

> 실무 사용은 권장하지 않는다.

일대다 관계는 엔티티를 여러개 가지기 때문에 Collection, List, Set, Map 중 하나를 사용해야 한다.

**단방향**

![onetomany_four](https://github.com/Team-Return-Backend/study-jpa/assets/129156398/a603b33c-07a6-4399-8b86-8a600ce00fd8)

- DB 테이블 입장에서 보면 다 쪽에 외래키가 들어간다.
- Team에서 members 가 바뀌면, DB의 member 테이블에서 업데이트 쿼리가 나가는 상황이 발생한다. (성능상 좋지 않으나, 크게 문제되지는 않는다.)

JPA의 **@OneToMany , @JoinColumn()**을 이용해 일대다 단방향 매핑을 지정해 줄 수 있다.

- 객체와 테이블의 패러다임의 차이 때문에 객체의 반대편 테이블의 외래키를 관리하는 특이한 구조이다.

**따라서** 다대일 단방향 매핑을 사용하고, 필요시 양방향 설정을 사용하자.

### 일대일[1:1]

---

일대일 관계는 서로 하나의 관계만 가진다.

주 테이블이나 대상 테이블 중에 누가 외래키를 가질지 선택해야 한다.

- 주 테이블에 외래키
  주 테이블이 외래키를 가지고 있으므로 주 테이블만 확인해도 대상 테이블과 연관 관계가 있는지 알 수 있다.
- 대상 테이블에 외래키
  테이블 관계를 일대일에서 일대다로 변경할 때 테이블 구조를 그대로 유지할 수 있다.

**주 테이블에 외래키**

단방향

![onetoone_one](https://github.com/Team-Return-Backend/study-jpa/assets/129156398/0e81f6d5-9d69-4c3e-baa2-dd0297121f33)

```java
@Entity
public class Member {
  @Id @GeneratedValue
  @Column(name = "MEMBER_ID")
  private Long id;

  private String username;

  @0neTo0ne
  @JoinColunm (name = "LOCKER_ID")
  private Locker locker;

}
@Entity
public class Locker {
  @Id @GeneratedValue
  @Column(name = "LOCKER_ID")
  private Long id;

  private String name;

}
```

양방향

![onetoone_four](https://github.com/Team-Return-Backend/study-jpa/assets/129156398/cc57fe06-438f-4982-a86b-f225f810ccab)

```java
@Entity
public class Member {
  @Id @GeneratedValue
  @Column(name = "MEMBER_ID")
  private Long id;

  private String username;

  @OneToOne
  @JoinColumn(name = "LOCKER_ID")
  private Locker locker;

}

@Entity
public class Locker {
  @Id @GeneratedValue
  @Column(name = "LOCKER_ID")
  private Long id;

  private String name;

  @OneToOne(mappedBy = "locker")
  private Member member;

}
```

Member 테이블이 외래키를 가지고 있으므로 Member 엔티티에 있는 Member.locker가 연관 관계의 주인이다. 따라서 반대 매핑인 Locker.member는 mappedBy 를 선언해서 연관 관계의 주인이 아니라고 설정했다.

**대상 테이블에 외래키**

단방향

![onetoone_three](https://github.com/Team-Return-Backend/study-jpa/assets/129156398/6f4eea29-4022-4745-a454-1112fa048476)

양방향

```java
@Entity
public class Member {
  @Id @GeneratedValue
  @Column(name = "MEMBER_ID")
  private Long id;

  private String username;

  @OneToOne(mappedBy = "member")
  private Locker locker;
}

@Entity
public class Locker {
  @Id @GeneratedValue
  @Column(name = "LOCKER_ID")
  private Long id;

  private String name;

  @OneToOne
  @JoinColumn(name = "MEMBER_ID")
  private Member member;

}
```

### 다대다[N:N]

---

**관계형 데이터베이스에서는 정규화된 테이블 2개로 다대다 관계를 표현할 수 없다.**

따라서 다대다 관계를 일대다, 다대일 관계로 풀어내는 연결 테이블을 사용한다.

단방향

```java
@Entity
public class Member {
  @Id @Column（name = "MEMBER_ID"）
  private String id;

  private String username;

  @ManyToMany
  @JoinTable（name = "MEMBER_PRODUCT", //연결 테이블을 지정한다.
    joinColumns = @JoinColumn(name = "MEMBER_ID"),  //현재 방향인 회원과 매핑할 조인 컬럼 정보를 지정한다.
    inverseJoinColumns = @JoinColumn(name ="PRODUCT_ID"))  // 반대 방향인 상품과 매핑할 조인 컬럼 정보를 지정한다.
  private List<Product> products = new ArrayList<Product>();

}

@Entity
public class Product {
  @Id @Column(name = "PRODUCT_ID")
  private String id;

  private String name;

}
```

양방향

```java
@Entity
public class Product {

    @Id
    @Column(name = "PRODUCT_ID")
    private String id;

    @ManyToMany(mappedBy = "products")
    private List<Member> members;
}
```

- 역방향도 @ManyToMany를 사용한다. 원하는 곳에 mappedBy로 연관 관계 주인을 지정한다.

(어느 방향이든 그래프 탐색이 가능하다.)

**연결 엔티티 사용**

@ManytoMany 어노테이션을 사용하면 훨씬 간편하고 단순해진다. (실무에선 사용하기 힘들다.)

연결 엔티티를 만들고 이곳에 추가한 컬럼들을 매핑해야 한다. 그리고 엔티티 간의 관계도 테이블 관계처럼 다대다에서 일대다, 다대일 관계로 풀어야 한다.

**복합 기본키**

JPA에서 복합 기본키를 사용하려면 별도의 식별자 클래스를 만들어야 한다. 그리고 엔티티에 @IdClass를 사용해서 식별자 클래스를 지정하면 된다.

- Serializable을 구현해야 한다.
- 복합키는 별도의 식별자 클래스로 만들어야 한다.
- equals와 hashCode 메소드를 구현해야 한다.
- 기본 생성자가 있어야 한다.
- 식별자 클래스는 public 이어야 한다.
- @IdClass를 사용하는 방법 외에 @EmbeddedId를 사용할 수 있다.

**식별 관계**

식별관계? 다른 테이블과 관계를 맺으면서, 해당 테이블의 기본 키를 자신의 기본 키로 사용하는 것

**새로운 기본키 사용**

기본키 생성 전략은 DB에서 자동으로 생성해주는 대리키를 Long 값으로 사용하는 것이다.

간편하고 거의 영구적으로 쓸 수 있으며 비즈니스에 의존하지 않는다. 그리고 ORM 매핑 시에 복합키를 만들지 않아도 되므로 간단한 매핑을 완성할 수 있다.

- 식별 관계 : 받아온 식별자를 기본키 + 외래키로 사용한다.
- 비식별 관계 : 받아온 식별자는 외래키로만 사용하고 새로운 식별자를 추가한다.

> 복합키를 위한 식별자 클래스를 만들지 않아도 되므로 단순하고 편하게 ORM 매핑을 할 수 있다. 이런 이유로 **비식별 관계**를 추천한다.
