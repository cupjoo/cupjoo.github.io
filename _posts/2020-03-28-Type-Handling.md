---
layout: post
title: "Type Handling"
author: cupjoo
categories: [Java]
image: assets/images/2020-03-28/1.png
hidden: true
---

Java에 존재하는 여러 값 타입의 종류와 각 타입을 다룰 때 주의할 사항에 대해 설명하려 한다. JPA에 관련된 내용도 일부 포함된다.

---

Java는 값 타입을 크게 엔티티 타입과 값 타입으로, 다시 값 타입에서 기본값, 임베디드 ,컬렉션 타입으로 분류한다.

## 기본값 타입

- 자바 기본 타입 : int, double, ...
- 래퍼 클래스 : Integer, Long, ...
- Stirng

기본값 타입은 다른 객체에 인자로 전달 시 항상 값을 복사하기 때문에 공유되지 않는다. 따라서 복사된 값이 변경되더라도 현재 값에는 영향이 없다.

## 임베디드 타입 (복합값 타입)

```java
public class Period {
    private LocalDate startDate;
    private LocalDate endDate;
}
```

```java
public class Address {
    private String city;
    private String street;
    private String zipcode;
}
```

복합값 타입은 기본 값 타입을 모아 새롭게 정의된 객체 타입으로, JPA에서는 임베디드 타입이라 불린다. 임베디드 타입은 서로 관련있는 값들을 한 클래스로 관리해 응집도와 재사용성이 높다.

### JPA와 임베디드 타입

JPA에서 임베디드 타입을 사용 시 **@Embeddable / @Embedded** 어노테이션을 통해 다른 값 타입과 동일하게 테이블과 매핑할 수 있다. 잘 설계한 ORM 어플리케이션에서는 매핑한 테이블 수보다 클래스의 수가 더 많다.

```java
@Embeddable
public class Period {
    // ...
}
```

```java
@Entity
public class User {
    @Id @GeneartedValue
    private Long id;

    private String name;

    @Embedded
    private Period workPeriod;

    @Embedded
    private Address homeAdderess;
}
```

![2.png]({{ site.baseurl }}/assets/images/2020-03-28/2.png)

### 값 타입 공유 참조와 불변 객체

임베디드 타입은 객체 타입이기 때문에 `주소값으로 객체를 참조`한다. 따라서 객체에 인자로 임베디드 타입을 전달하면 실제 값이 아닌 참조가 전달 (`얕은 복사`) 되는 것이기 때문에, 여러 객체에서 객체 타입을 공유하면 값 변경 시 다른 객체에도 영향을 주게 된다. 따라서 같은 임베디드 타입을 여러 곳에서 사용해야 하는 경우 값을 복사해서 사용 (`깊은 복사`) 해야 한다.

하지만 값을 복사하는 방법도 결국 사용자가 실수로 참조를 대입할 수 있는 여지가 있기 때문에 안전하지 않다. 따라서 모든 객체 타입은 부작용을 원천 차단하기 위해 생성 이후 절대 값을 변경할 수 없는 **불변 객체 (Immutable Object)**로 설계해야 한다. 불변 객체를 만드는 방법은 생성자로만 값을 설정하고, setter를 만들지 않으면 된다. 물론 값의 변동이 필요한 경우 의미를 확실히 알 수 있는 이름 메소드를 정의하면 된다.

```java
public class User implements Cloneable {
    // ...
    @Override
    protected Object clone() throws CloneNotSupportedException {
        return super.clone();   // 깊은 복사
    }
}
```

```java
public class Order {
    // no other setter
    public void changeReceiver(User user){
        // this.receiver = user // 얕은 복사
        this.receiver = (User) user.clone();    // 깊은 복사
    }
}
```

### 값 타입의 비교

값 타입을 비교하는 방법에는 **==**을 사용하는 **동일성 (Identity) 비교**와 **equals()**를 사용하는 **동등성 (Equivalence) 비교**가 있다. 자바 기본 타입을 제외한 모든 값 타입은 올바른 비교를 위해 동등성 비교를 사용해야해서 임베디드 타입에는 equals 메소드를 재정의한다.

```java
@Entity
public class User {

    @Id @GeneratedValue
    private Long id;

    private String name;
    private int age;

    @Override
    public boolean equals(Object o) {
        if (this == o) return true;
        if (o == null || getClass() != o.getClass()) return false;
        User user = (User) o;
        return age == user.age &&
                id.equals(user.id) &&
                name.equals(user.name);
    }

    @Override
    public int hashCode() {
        return Objects.hash(id, name, age);
    }
}
```

IntelliJ에서 **Generate 기능**을 사용하면 실수 없이 equals()와 hashCode()를 생성할 수 있다. Map/Set을 사용하기 위해 hashCode()도 재정의해주자.

## 컬렉션 타입

List나 Set 등의 컨테이너 타입을 컬렉션 타입이라 하며, 그 중 값 타입을 저장하는 컬렉션을 **값 타입 컬렉션**이라 한다. 개념 자체는 어렵지 않은데, 실제 JPA에서는 사용을 권장하지 않는다. 대신 1:N 관계로 구현하는 것을 추천한다.

## 참고 자료

> [자바 ORM 표준 JPA 프로그래밍](https://book.naver.com/bookdb/book_detail.nhn?bid=9252528)