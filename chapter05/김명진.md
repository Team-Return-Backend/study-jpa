# Chapter 5

객체 관계 매핑에서 가장 어려운 부분이 객체 연관관계와 테이블 연관관계를 매핑하는 일이다.

객체의 참조와 테이블의 외래키를 매핑할 수 있다.

**방향**

- 양방향: A → B && B → A (둘 중 한쪽만 참조)
- 단방향: A → B || B → A (둘 모두 서로 참조)

**다중성**

- 다대다(N:1)
- 일대다(1:N)
- 일대일(1:1)
- 다대일(N:M)

**연관관계 주인 -** 객체를 양방향 연관관계로 만드려면 연관관계의 주인을 정해야 한다.

> 객체 vs 테이블
> 
> - 객체는 참조(주소)로 연관관계를 맺는다.
> - 테이블은 외래키로 연관관계를 맺는다.

### 단방향 연관관계

---

- 참조를 통한 연관관계는 언제나 단방향이다.
- 객체 간에 연관관계를 양방향으로 만들고 싶다면 반대쪽에도 필드를 추가해서 참조를 보관해야 한다. (정확히는 양방향 관계가 아니라, 서로 다른 방향의 단방향 매핑 두개이다.)

1:N

```java
@Getter
@Builder
@Entity
public class Member {
    @Id
    @Column(name="MEMBER_ID")
    private String id;
    
    private String username;
    
    // 연관관계 부분
    @ManyToOne
    @JoinColumn(name="TEAM_ID")
    private Team team;
}
```

**@ManyToOne**

- N:1 관계라는 매핑 정보를 나타낸다.
- 다중성을 나타내는 어노테이션은 필수이다.

| 속성 | 기능 | 기본값 |
| --- | --- | --- |
| optional | 연관된 엔티티가 없어도 되는지 여부이다. true면 연관된 엔티티가 필요 없다.  | true |
| fetch | 글로벌 패치 전략 설정 | @ManyToOne = EAGER
@OneToMany = LAZY |
| cascade | 영속성 전이 가능 |  |
| targetEntity | 연관된 엔티티 타입 정보 설정이다. |  |

**@JoinColumn**

- 외래키를 매핑할 때 사용한다.
- name 속성에는 매핑할 외래키 이름을 지정한다.

| 속성 | 기능 | 기본값 |
| --- | --- | --- |
| name | 매핑할 외래키 이름 | 필드명을 참조하는 테이블의 기본키 컬럼명 |
| referenceColumnName | 외래키가 참조하는 대상 테이블의 컬럼명 | 참조하는 테이블의 기본키 컬럼명 |
| foreignKey(DDL) | 외래키 제약조건 정의 |  |
| unique
nullable
insertable
updatable
columnDefinition
table | @Column 의 속성과 같다. |  |

### 연관관계 사용

---

연관관계가 있는 엔티티는 두가지 방법으로 조회할 수 있다. 

- 객체 그래프 탐색
- JPQL(객체지향 쿼리)

**저장**

```java
public void testSave(){
	//팀1 저장
	Team team1 = new Team("team1", "팀1");
	em.persist(team1);

	//회원1 저장
	Member member1 = new Member("member1", "회원1");
	member1.setTeam(team1); //연관관계 설정 member1 -> team1
	em.persist(member1);
```

- 반드시 연관관계를 설정하려는 대상은 영속상태여야 한다. (Team 객체)

> 만약 team1이 persist 안된채로 member1에 연관관계 설정이 된다면 JPA는 아무런 연관관계도 짓지 않는다.
> 

```sql
INSERT INTO TEAM (TEAM_ID, NAME) VALUES ('team1','팀1');
INSERT INTO MEMBER (MEMBER_ID, TEAM_ID) VALUES ('member1', '회원1', 'team1');
```

**조회**

- **객체 그래프 탐색**
    - 자바에서 참조된 객체를 사용하는 것처럼 getter를 사용해서 참조된 엔티티를 조회하는 것
- **객체지향 쿼리 사용(JPQL)**
    
    ```java
    private static void queryLogicJoin(EntityManager em) {
    	String jpql = "select m from Member m join m.team t where" + 
    								"t.name=:teamName";
    	List<Member> resultList = em.createQuery(jpql, Member.class)
    															.setParameter("teamName", "팀1");
    															.setResultList();
    ```
    

**수정**

- 변경 감지를 통해 엔티티를 수정한다.

```java
private static void updateRelation(EntityManager em) {
	Team team2 = new Team("team2", "팀2");
	em.persist(team2);

	Member member = em.find(Member.class, "member1");
	member.setTeam(team2);
```

- update 와 같은 다른 EntityManager의 메소드 호출 없이도 영속상태의 엔티티를 변경하려면 위 코드처럼 수정해주면 된다.
- 트랜잭션이 커밋하고 플러시가 일어날 때, 해당 스냅샷과 현재 엔티티 값을 비교하여 변경 감지 기능이 일어난다.

**삭제**

- 참조 객체를 null로 만들면 된다.
- 유의사항
    - 연관된 엔티티를 삭제하려면 반드시 기존에 연관관계를 제거하고  마지막으로 참조 객체를 지워야 한다.
    - 외래키 제약조건으로 인해 DB 오류가 발생한다.
    
    > 실제 운영 환경에서는 외래키 제약조건을 거는 것이 부담스러워 안걸기도 한다.
    > 
    

### 양방향 연관관계

---

- 양방향 매핑은 참조되는 두 관계속 엔티티들이 서로를 단방향으로 참조하면 된다.
- Team 객체를 수정해야 한다.

```java
@Getter
@Builder
@Entity
public class Team {
	@Id
	@Column(name = "TEAM_ID")
	private String id;
```

- Member → Team : 다대일 관계, 다에 속하는 Member 가 연관 관계 주인으로써 @JoinColumn 을 갖는다.
- Team → Member : 일대다 관계, 일에 속하는 Team, mappedBy 속성으로 반대쪽 연관관계 주인의 매핑 필드 이름을 적어준다.
- 그래프 탐색을 사용해서 조회할 수 있다.

### 연관 관계 주인

---

**@OneToMany를 추가하여 양방향 연관관계를 맺을 때, mappedBy 속성은 왜 필요할까?**

- 엔티티를 양방향 연관관계로 설정하면 객체의 참조는 둘인데 외래키는 하나인 상황이 발생한다.
- 참조되는 두 개의 단방향 관계 중 어느쪽에서 외래키를 관리해야하는가 라는 질문이 남는데, 이 외래키를 관리하는 곳을 **연관 관계의 주인** 이라고 한다.

<aside>
💡 **연관 관계의 주인
-** 연관 관계의 주인만이 Db 연관관계와 매핑되고, 외래키를 관리할 수 있다.
- mappedBy 속성을 사용하지 않는다.
- 주인이 아닌 곳에서 mappedBy 속성을 사용해 연관 관계 주인을 지정해줘야 한다.

</aside>

**그렇다면 누가 연관 관계의 주인이 되어야 할까?**

- 일반적으로 다쪽에 연관 관계 주인을 매핑한다.
- 연관 관계의 주인 == 외래키의 관리자
- 다쪽인 @ManyToOne에는 mappedBy 속성이 없고, @OneToMany쪽에만 mappedBy 속성이 존재한다.

### 양방향 연관 관계의 주의점

---

가장 흔히 하는 실수는 연관 관계의 주인에는 값을 입력하지 않고, 주인이 아닌 곳에만 값을 입력하는 것이다. (DB에 외래키 값이 정상적으로 등록되지 않는다면?? ……)