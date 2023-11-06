# Chapter 4

> JPA를 사용하는데 가장 중요한 것은 엔티티와 테이블을 잘 매핑하는 것이다.

- 객체 - 테이블 : `@Entity` ,`@Table`
- 기본키 : `@Id`
- 필드 - 컬럼 : `@Column`
- 연관관계 매핑 : `@ManyToOne` , `@JoinColumn`

### @Entity

---

JPA를 사용해서 테이블과 매핑할 클래스에는 무조건 `@Entity` 붙여야 한다.

**주의사항**

- 기본 생성자를 필수로 붙여야 한다. (public || protected)
- final 클래스, enum, interface 클래스에선 사용할 수 없다.
- 필드에 final 사용할 수 없다.

### @Table

---

엔티티와 매핑할 테이블을 지정한다.

> `name` 속성을 생략하면 엔티티 이름을 테이블 이름으로 사용한다.

- catalog : catalog 기능이 있는 DB에서 catalog를 매핑
- schema : schema 기능이 있는 DB에서 schema를 매핑

### DB 스키마 자동 생성

---

JPA는 스키마를 자동 생성하는 기능을 지원한다.

클래스의 매핑 정보를 통해 어떤 테이블이 어떤 컬럼을 사용하는지 알 수 있다.

애플리케이션 실행 시점에 DB 테이블을 자동으로 생성한다. (**ddl-auto: create**)

> schema 자동 생성으로 만들어진 DDL은 개발 환경에서만 사용하도록 하자.

**ddl-auto 속성**

```
spring:
	jpa:
		hibernate:
			dde-auto: (속성)
```

| 속성        | 설명                                                                                                     |
| ----------- | -------------------------------------------------------------------------------------------------------- |
| create      | 기존 테이블을 삭제하고 새로 생성함(DROP + CREATE)                                                        |
| create-drop | create 속성에 추가로, 애플리케이션 종료할 때 생성한 DDL 제거(DROP + CREATE + DROP)                       |
| update      | DB 테이블과 엔티티 매핑정보를 비교해서 변경사항만 수정함                                                 |
| validate    | DB 테이블과 엔티티 매핑정보를 비교해서 차이가 있으면 경고를 남기고 애플리케이션 실행을 안함 (DDL 수정 X) |
| none        | 자동 생성 기능을 사용하지 않기 위해 속성을 추가 안하거나 none 값을 주면된다.                             |

### @Enumerated

---

- `EnumType.ORDINAL` 은 enum에 정의된 순서대로 ADMIN은 0, USER는 1값이 DB에 저장된다.
  - 장점 : 저장되는 크기가 작다.
  - 단점 : 이미 저장되어 있는 enum의 순서가 바뀌거나 추가되었을 때 능동적인 대응을 할 수 없다.
- `EnumType.STRING` 은 enum 이름 그대로 ADMIN은 ‘ADMIN’, USER는 ’USER’라는 문자로 DB에 저장된다.
  - 장점 : 저장되는 크기가 크다.
  - 단점 : 순서가 바뀌거나 ENUM이 추가되어도 안전하다.

### @Temporal

---

날짜 타입을 매핑할 때 사용한다.

`@Temporal` 을 정의하면 자바의 `Date` 와 비슷한 timestamp로 정의된다. (LocalDate에서는 사용하지 않는다. )

### 기본키 매핑

---

DB 마다 기본키를 생성하는 방법이 다르므로 문제를 해결하기가 쉽지 않다. (오라클 시퀀스, MySQL의 AUTO_INCREMENT)

- 직접 할당 : 기본키를 애플리케이션에서 직접 할당한다.
- 자동 생성 : 대리키를 사용한다.
  - `IDENTITY` : 기본키를 DB에 위임한다.
  - `SEQUENCE` : DB 시퀀스를 사용해서 기본키를 할당한다.
  - `TABLE` : 키 생성 테이블을 사용한다.

**직접 할당**

---

`@Id` 를 붙이면 된다.

`@Id` 적용 가능한 자바 타입은?

- 자바 기본형
- 자바 wrapper 형
- String
- java.util.Date
- java.sql.Date
- java.math.BigDicimal
- java.math.BigInteger

**자동 생성**

DBMS가 PK 자동 생성해준다.

자동 생성될 때는 `@Id` , `@GeneratedValue` 가 필요하다.

1 **IDENTITY 전략**

---

`IDENTITY` 전략은 AUTO_INCREMENT 를 사용한 예제처럼 **DB에 값을 저장하고 나서야 기본 키 값을 구할 수 있을 때 사용**한다.

> IDENTITY 전략은 데이터를 DB에 INSERT 한 후에 기본 키 값을 조회할 수 있다.

- em.persist() 호출 시 INSERT SQL을 즉시 DB에 전달한다.
- 식별자를 조회해서 엔티티의 식별자에 할당한다.
- 쓰기 지연이 동작하지 않는다.

2 **SEQUENCE 전략**

---

> 오라클, PostgreSQL, DB2, H2 DB에 주로 사용

유일한 값을 순서대로 생성하는 특별한 DB Object 이다.

1. em.persist() 를 호출 시 먼저 DB 시퀀스로 식별자를 조회한다.
2. 조회한 식별자를 엔티티에 할당한다.
3. 엔티티를 영속성 컨텍스트에 저장한다.
4. 트랜잭션을 커밋해서 **플러쉬**가 일어난다.
5. 엔티티를 DB에 저장한다.

> equenceGenerator.allocationSize 기본값은 50

### 3 Table 전략

---

키 생성 전용 테이블을 하나 만드는 것이다.

시퀀스 대신 테이블을 사용하는 것 빼고는 **SEQUENCE 전략과 동일**하다.

테이블을 따로 사용하기 때문에 **모든 DBMS에 적용이 가능**하다.

```bash
create table MY_SEQUENCES (
    sequence_name varchar(255) not null,
    next_val bigint,
    primary key (sequence_name)
)
```

```java
@Entity
@TableGenerator(
    name = "BOARD_SEQ_GENERATOR",
    table = "MY_SEQUENCES",
    pkColumnValue = "BOARD_SEQ", allocationSize = 1)
public class Board {

    @Id
    @GeneratedValue(strategy = GenerationType.TABLE,
                generator = "BOARD_SEQ_GENERATOR")
    private Long id;
}
```

1. `@TableGenerator` 를 사용하여 테이블 키 생성기 등록한다.
2. TABLE 전략을 사용하기 위해 GenerationType.TABLE 을 선택했다.
3. @GeneratedValue.generator에 방금 만든 테이블 키 생성기를 지정한다.
4. id 식별자 값은 BOARD_SEQ_GENERATOR 테이블 키 생성기가 할당된다.

### 4 AUTO 전략

---

GenerationType.AUTO는 선택한 DB에 따라 IDENTITY, SEQUENCE, TABLE 전략 중 하나를 **자동으로 선택**한다.

> GeneratedValue.strategy 의 기본값은 AUTO 이다.

- DB를 변경해도 코드를 수정할 필요가 없다.
- 키 생성 전략이 확정되지 않은 개발 초기 단계에 사용하기 좋다.

## 기본키 매핑 정리

---

- **직접 할당** : em.persist()를 호출하기 전 애플리케이션에서 직접 식별자 값을 지정해줘야 한다.(만약 식별자가 없다면 예외가 발생한다.)
- **SEQUENCE** : DB 시퀀스에서 식별자 값을 획득한 후 영속성 컨텍스트에 저장한다.
- **TABLE** : DB 시퀀스 생성용 테이블에서 식별자 값을 획득한 후 영속성 컨텍스트에 저장한다.
- **IDENTITY** : DB에 엔티티를 저장해서 식별자 값을 획득한 후 영속성 컨텍스트에 저장한다.
