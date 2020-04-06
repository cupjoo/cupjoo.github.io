---
layout: post
title: "N+1 문제와 Fetch Join"
author: cupjoo
categories: [JPA]
image: assets/images/2020-04-07/1.png
---

관련 포스트 : [Proxy와 Lazy Loading](https://cupjoo.github.io/Proxy와-Lazy-Loading)

관련 포스트 : [객체지향 쿼리 언어 JPQL](https://cupjoo.github.io/객체지향-쿼리-언어-JPQL)

---

## 100명의 User에 대해 Team과 Join된 데이터를 가져오려면 어떻게 해야 할까?

여러 방법이 있겠지만 우선 순수 JPA의 find() 만으로는 해결이 불가능해 JPQL을 사용해야 한다.

### 1. Select (EAGER / LAZY)

```java
List<User> users = em.createQuery("select u from User u", User.class)
    .getResultList(); // User 쿼리

for (User user : users){
  // Team 쿼리
  System.out.prinln(user.getName()+" in "+user.getTeam().getName());
}
```

- User 100명에 대한 쿼리가 발생한다. (1 Query)
- User에 대해 루프를 돌 때마다 `지연 로딩`으로 인해 Proxy 객체인 Team에 대한 쿼리가 발생한다. (100 Query)

즉 쿼리가 101번 호출되어 N+1 문제가 발생한다. 평소 User에 대한 정보만 필요한 경우엔 상관없지만, 의도적으로 Team과 함께 조인해서 출력하는 경우가 많다면 심각한 성능 저하가 발생할 수 있다.

그렇다면 User 엔티티 내부에서 Team 엔티티에 대해 `즉시 로딩`을 설정하면 어떻게 될까?

```java
@Entity
public class User {
  @ManyToOne   // 기본값 : EAGER
  @JoinColumn(name = "TEAM_ID")
  private Team team;
}
```

- User 100명에 대한 쿼리가 발생한다. (1 Query)
- User 리스트를 조회한 직후 Team 엔티티가 Proxy 객체 상태로 비어있는 것을 확인하고, Team에 대한 각각의 쿼리가 발생한다. (100 Query)
- 루프를 돌 때는 이미 가져온 Team 엔티티를 그대로 사용한다.

즉시 로딩도 N+1 문제를 피할 수 없다.

### 2. 일반 Join

```java
List<User> users = em.createQuery("select u from User u join u.team", User.class)
    .getResultList(); // User 쿼리 후 Team에 대해 각각 쿼리

for (User user : users){
  System.out.prinln(user.getName()+" in "+user.getTeam().getName());
}
```

- User 100명에 대한 쿼리가 발생한다. (1 Query)
- User에 대해 루프를 돌 때마다 지연 로딩으로 인해 Proxy 객체인 Team에 대한 쿼리가 발생한다. (100 Query)

Join은 기본적으로 연관 객체에 대한 정보는 `지연 로딩`으로 가져오기 때문에 같은 문제가 발생한다.

### 3. Fetch Join

```java
List<User> users = em.createQuery("select u from User u join u.team", User.class)
    .getResultList(); // 한꺼번에 User 쿼리 & Team 쿼리

for (User user : users){
  System.out.prinln(user.getName()+" in "+user.getTeam().getName());
}
```

- User 100명에 대한 Team과의 조인 쿼리가 발생한다. (1 Query)

결국 Fetch Join을 사용하면 이렇게 1번의 쿼리만으로 연관된 모든 데이터를 조인해 한꺼번에 가져올 수 있다.

![2.png]({{ site.baseurl }}/assets/images/2020-04-07/2.png)

## Fetch Join과 Distinct

정확히는 N+1 문제와는 관련 없지만 Collection Fetch Join과 관련해 자주 발생하는 문제이기에 다뤄보겠다. 다음 쿼리를 실행해보자.

```java
List<Team> teams = em.createQuery("select t from Team t join fetch t.members where t.name = 'Team A'", Team.class)
    .getResultList();

for(Team team : teams){
  System.out.println("Team "+ team.getName()+" ("+team+")");
  for(User user : team.getUsers()){
    System.out.println("-> "+user.getName()+" ("+user+")");
  }
}
```

그런데 루프를 돌려 반환된 Team을 출력해보니 같은 팀이 중복되어 여러 번 호출된다. 왜 그럴까?

![4.png]({{ site.baseurl }}/assets/images/2020-04-07/4.png)

기본적으로 SQL에서 Join은 두 테이블을 연관지어 새로운 한 테이블 형태로 반환한다. 즉 어떻게 조인을 하더라도 User와 Team 테이블의 튜플은 1:1로 연관되어 1개의 튜플을 반환한다.

![5.png]({{ site.baseurl }}/assets/images/2020-04-07/5.png)

하지만 JPA에는 엔티티 Collection이라는 개념을 추가해 엔티티 내부에 1:N 관계의 리스트를 저장한다. 그러다보니 조인 결과로 SQL에서는 `같은 pk를 같는 여러 개의 팀 A를 반환`하고, JPA에서는 각각 같은 객체임을 알지만 어쩔수 없이 그대로 리스트로 받게 되는 것이다.

![6.png]({{ site.baseurl }}/assets/images/2020-04-07/6.png)

### Distinct

SQL에서도 이와 비슷한 문제를 처리하기 위해 DISTINCT라는 중복 제거 명령어가 있다. 하지만 설명했듯이 위 상황은 SQL 입장에서 보면 지극히 정상적인 반환 결과이기 때문에 중복 제거가 발생하지 않는다.

그래서 JPA에서 사용하는 DISTINCT 키워드는 2가지 기능을 제공한다.

- SQL에 DISTINCT 추가
- 어플리케이션에서 자체적으로 같은 식별자를 가진 엔티티 중복 제거

따라서 JPA에서 제공하는 Distinct 키워드를 통해 중복 엔티티 발생 문제를 해결할 수 있다.

```sql
select distinct t
from Team t join fetch t.members
where t.name = 'Team A'
```

![7.png]({{ site.baseurl }}/assets/images/2020-04-07/7.png)

![8.png]({{ site.baseurl }}/assets/images/2020-04-07/8.png)

## 참고 자료

> [자바 ORM 표준 JPA 프로그래밍](https://book.naver.com/bookdb/book_detail.nhn?bid=9252528)
