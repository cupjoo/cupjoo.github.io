---
layout: post
title: "Proxy와 Lazy Loading"
author: cupjoo
categories: [JPA]
image: assets/images/2020-04-03/1.png
---

## Proxy

양방향 연관관계는 기존의 단방향 연관관계에서 **반대 방향의 참조가 필요할 때** 그때 가서 참조를 추가하는 방식으로 구성된다. 그렇다면 단방향 연관관계도 일반 엔티티에서 일단 참조가 필요하기에 연관관계를 추가했지만, 참조가 필요 없는 경우가 더 많으면 어떻게 해야할까?

```java
// user 정보만 사용하는데 항상 team 정보도 가져오게 됨
User user = em.find(User.class, userId);
System.out.println(user.getUsername());
```

이런 상황을 고려해 `Lazy Loading`을 통해 DB 조회를 미루고 `가짜 (Proxy) 엔티티 객체`를 조회하는 방식이 도입됐다.

```java
// 실제로 Team 쿼리 수행 X. 내용이 비어있는 객체 생성
Team team = em.getReference(Team.class, 10L);
// 사용되는 순간에 Team 쿼리 수행
System.out.println(team.getName());
```

### 프록시 객체의 특징

![2.png]({{ site.baseurl }}/assets/images/2020-04-03/2.png)

- 실제 클래스를 **상속** 받아서 만들어져, 해당 클래스와 **겉모양이 같은 구조**를 가진다.
  - **entity1**.getClass() == **entity2**.getClass() : **true**
  - **entity**.getClass() == **proxy**.getClass() : **false**
  - **proxy** instanceof **Entity** : **true**
  - ex) User 클래스의 Proxy 클래스명 : User$HibernateProxy$hfdfsdfs
- 프록시 객체는 실제 객체의 **참조**를 보관했다가, 프록시 객체가 호출되면 실제 객체를 가져오기 위해 **영속성 컨텍스트에 초기화를 요청**한다.

### 프록시 객체의 초기화

![3.png]({{ site.baseurl }}/assets/images/2020-04-03/3.png)

- 프록시 객체의 초기화는 실제 엔티티로 바뀌는 것이 아니다. 초기화 시 **프록시 객체를 통해 실제 엔티티에 접근이 가능**해지는 것이다.
- 따라서 프록시 객체를 호출하면 프록시 객체는 실제 객체의 메소드를 호출한다.
- **PersistenceUnitUtil.isLoaded(entity)** : 프록시 객체의 초기화 여부 확인
- 준영속 상태일 때 프록시를 초기화하면 예외가 발생한다.

### 참고

영속성 컨텍스트에 찾는 엔티티가 이미 있으면 프록시 객체를 요청해도 **실제 엔티티**가 반환된다.

```java
User user1 = em.find(User.class, 10L);
User user2 = em.getReference(User.class, 10L);
System.out.println(user.getClass()); // User
System.out.println((user1 == user2)); // true. 같은 Entity
```

이 외에도 여러 케이스가 있을 수 있는데, 중요한 건 JPA는 프록시인지 여부에 관계없이 **엔티티를 항상 실제 객체처럼 사용**할 수 있게 만든다는 것이다.

## Lazy Loading

앞서 설명한 프록시 개념은 `Lazy Loading`을 구현할 때 사용된다. `지연 로딩`은 객체 호출 시 지정된 컬럼만 Proxy 객체로 조회하는 것을 말한다.

```java
import static javax.persistence.FetchType.LAZY;

@Entity
public class User {
  // ...
  @ManyToOne(fetch = LAZY)
  @JoinColumn(name = "TEAM_ID")
  private Team team;
}
```

```java
// User 쿼리 O. Team 쿼리 X
User user = em.find(User.class, 10L);
Team team = user.getTeam();
// 실제 메소드 호출 시점에 Team 쿼리
System.out.println(team.getName());
```

![4.png]({{ site.baseurl }}/assets/images/2020-04-03/4.png)

- 컬럼에 fetch 전략을 LAZY로 설정하면 지연 로딩이 적용된다. **무조건 지연 로딩을 사용**하자.
- fetch 전략을 EAGER로 설정하면 즉시 로딩이 적용된다. 하지만 절대로 즉시 로딩은 사용하지 말자.
  - 예상치 못한 문제를 일으킬 수 있음
  - JPQL은 일단 SQL문을 생성하면 바로 실행하기에 **N+1** 문제를 발생시킴
- **@ManyTo**, **@OneToOne** : 즉시 로딩이 기본값 -> LAZY로 설정하자
- **@OneToMany**, **@ManyToMany** : 지연 로딩이 기본값

> 관련 포스트 : [N+1 문제와 Fetch Join](https://cupjoo.github.io/N-1-문제와-Fetch-Join)

## 참고 자료

> [자바 ORM 표준 JPA 프로그래밍](https://www.inflearn.com/course/ORM-JPA-Basic)
