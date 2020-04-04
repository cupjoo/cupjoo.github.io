---
layout: post
title: "Hello, Spring!"
author: cupjoo
categories: [Spring]
image: assets/images/2020-03-20/1.png
featured: true
hidden: true
---

이 포스트에서는 Spring 입문자를 대상으로 Spring 기초 개념과 용어, Spring에 속하지는 않지만 자주 사용되는 라이브러리 등에 대해 설명한다. 여러 주제를 병렬적으로 정리하다 보니 다소 두서가 없을 수 있으니 필요한 내용만 선택해 보자. 예시 프로젝트 기본 세팅은 다음 환경으로 구성된다.

- IDE : IntelliJ
- [Java 8](https://www.oracle.com/java/technologies/javase-jdk8-downloads.html)
- Gradle 4.8
- Spring Boot 2.1.x

---

## 1. Spring Boot

### `Framework = Design Pattern + Library (Dependency)`

프레임워크의 정의는 **자주 사용되는 라이브러리 (의존성)** 들이 모여 특정 **디자인 패턴**으로 구성된 것을 말한다. 자바에서는 웹 개발 시 `Spring MVC` 프레임워크를 사용하는데, 현재 가장 많이 사용하는 버전은 3과 4다.

하지만 Spring MVC는 프로젝트 설정과 웹 서버 연동 등이 복잡해 진입 장벽이 높았고, 이런 수요를 반영해 비즈니스 로직 자체에만 집중할 수 있도록 좀 더 가벼운 Spring 프레임워크인 `Spring Boot`가 등장했다.

![2.png]({{ site.baseurl }}/assets/images/2020-03-20/2.png)

[프로젝트 코드](https://gist.github.com/cupjoo/4d17f6d7547d09e7af8e1dd392259579)

스프링 부트의 기본 구조는 크게 Web / Service / Repository Layer로 나뉜다.

- **Web Layer**
  - controllers / exception handelers / filters / view templates, ...
  - 외부 요청과 응답에 대한 영역
- **Service Layer**
  - application services / infrastructure services
  - 트랜잭션, 도메인 기능 간 순서를 보장하는 서비스 영역
- **Repository Layer**
  - repository interfaces and their implementations
  - DB 접근 영역 (DAO)
- **DTOs**
  - 계층 간 데이터 교환을 위한 객체
- **Domain Model**
  - domain services / entities / value objects
  - 비지니스 처리를 담당하는 곳

스프링 부트는 메인 클래스에 있는 **@SpringBootApplication** 어노테이션의 위치를 시작으로 모든 설정값을 읽기 때문에 해당 어노테이션은 항상 프로젝트 최상단에 위치해야 한다.

Spring Boot는 어플리케이션 실행 시 내장 **WAS (Wep Application Server)** 를 실행한다. 따라서 Tomcat과 같은 외부 WAS 없이도 생성된 Jar 파일만 실행하면 항상 같은 환경의 어플리케이션을 배포할 수 있다.

> 관련 포스트 : [Spring의 핵심 기술](https://cupjoo.github.io/Spring의-핵심-기술)

## 2. Gradle

`빌드 도구`는 프레임워크의 기본 구조를 자동으로 생성해주고, 필요한 의존성을 호출 및 관리해주는 역할을 한다. 자바에서 사용하는 대표적인 빌드 도구에는 `Maven`과 `Gradle`이 있다. 현재는 Gradle이 좀 더 우수한 성능을 보이면서, 신규 프로젝트의 경우 그 수요가 더 높아지고 있는 상황이다.

- 프로젝트 빌드 시 Gradle이 Maven보다 평균적으로 100배 정도 더 빠른 속도로 빌드한다.
- 의존성 관리 시 Maven은 **XML**을 사용하는 반면, Gradle은 **Groovy**를 사용해서 코드가 더 짧고, 가독성이 높다.

> 출처 : [Gradle vs Maven Comparison](https://gradle.org/maven-vs-gradle/) / [Maven vs Gradle](https://bkim.tistory.com/13)

Spring Initializer에서 Gradle로 프로젝트 빌드 시 Gradle 설정 파일(**build.gradle**)이 생성된다.

![4.png]({{ site.baseurl }}/assets/images/2020-03-20/4.png)

- **ext** : 전역 변수를 선언함
- **repositories** : 의존성을 다운받을 원격 저장소를 지정함
- **apply plugin** : 선언한 플러그인 의존성을 적용할 지 결정함
- **dependencies** : 의존성 선언 파트. 의존성 버전을 생략해야, 선언한 plugin 버전에 맞는 의존성 버전을 import하게 됨

repositories는 대부분 **mavenCentral**을 많이 사용하지만, 몇 가지 문제점으로 인해 최근에는 **jCenter**에 대한 수요가 늘고 있다.

> 참고 사이트 : [Difference: mavenCentral() vs jCenter()](https://stackoverflow.com/questions/50726435/difference-among-mavencentral-jcenter-and-mavenlocal/50726436))

### 의존성 추가하기

![5.png]({{ site.baseurl }}/assets/images/2020-03-20/5.png)

**Alt + Insert** 키를 누르면 Generate 옵션이 활성화된다. Maven Dependency 기능을 누르면 의존성 검색창이 나오게 된다. 필요한 의존성을 추가하면 된다.

![6.png]({{ site.baseurl }}/assets/images/2020-03-20/6.png)

참고로 IntelliJ에는 **Enable Auto-Import** 설정이 있는데, 활성화하면 의존성 파일 변경 시 자동으로 의존성을 갱신한다. 유용한 기능이니 활성화하자.

## 3. Lombok

```java
@Builder
@NoArgsConstructor
@ToString(exclude = {"owner", "price"})
@Getter
public class Item {

    @NotNull
    private Long sn;

    private Owner owner;
    private String name;
    private int price;

    public void changePrice(int price){
        this.price = price;
    }
}
```

```java
Item item = Item.builder()
        .sn(101L)
        .name("Banana")
        .price(1000)
        .build();
item.changePrice(2000);
System.out.println(item.toString());
System.out.println("name: "+ item.getName());
```

롬복은 어노테이션을 사용해 객체와 관련된 메소드들을 자동 생성해주는 자바 라이브러리이다. 사용하려면 몇 가지 설정이 필요하다.

### 1) Lombok 의존성 추가

```java
// build.gradle

dependencies {
    // ...
    compileOnly 'org.projectlombok:lombok'
    annotationProcessor 'org.projectlombok:lombok'
}
```

### 2) IntelliJ Lombok 플러그인 설치

- Action 검색창 (**Ctrl + Shift + A**)에서 Plugins 메뉴 접속
- lombok 플러그인 설치
- 재시작 시 플러그인이 적용됨
- 프로젝트 우측 하단 **Lombok Annotation Processing** 설정 활성화

![7.png]({{ site.baseurl }}/assets/images/2020-03-20/7.png)

### 자주 사용하는 롬복 어노테이션

- **@Getter / @Setter**
- **@Builder** : 빌더 패턴 메소드 생성
- **@NoArgsConstructor** : 파라미터가 없는 기본 생성자 생성
- **@RequiredArgsConstructor** : **final**이나 **@NotNull** 필드 값만 파라미터로 받는 생성자 생성
- **@NotNull** : Null 불가 명시
- **@ToString** : toString() 메소드 생성

### 주의 1

절대 Setter를 무분별하게 사용하면 안된다. 값 변동이 필요한 경우에만 목적과 의미가 분명한 이름의 메소드를 따로 생성하자.

```java
public void changePrice(int price){
  if(price < 0){
    System.out.println("올바른 범위의 가격이 아닙니다.");
    return;
  }
  this.price = price;
}
```

Builder 또한 마찬가지로, 필요한 값만 초기화하고 나머지 값은 DB로부터 생성된 값을 받아오던가 setter를 사용하자.

```java
// ... @Builder 선언 X
public class Posts {
  @Id
  @GeneratedValue(strategy = GenerationType.IDENTITY)
  private Long id;

  private String title;
  private String content;
  private String author;

  @Builder  // 필요한 값만 받아오도록 재정의
  public Posts(String title, String content, String author) {
      this.title = title;
      this.content = content;
      this.author = author;
  }
}
```

### 주의 2

JPA에서 양방향 연관관계로 매핑된 객체에서 **@ToString**을 사용할 경우, 무한 순환 참조가 발생한다. 따라서 **exclude** 속성으로 해당 필드를 제외해야 한다.

```java
@ToString(exclude ="owner")
public class Member {
    // ...
    private Owner owner;
}
```

## 4. JPA

초기 Spring에서는 데이터 처리에 순수 JDBC를 사용해 쿼리문을 직접 작성해야 했다. 이후 중복되는 코드를 줄이고자 iBatis, MyBatis, JdbcTemplate과 같은 `SQL Mapper`가 등장하지만, 여전히 문제점들이 존재했다.

- Connection, ORM 등은 자동화됐지만 여전히 반복되는 SQL 코드를 작성하는 SQL 의존적인 개발
- 패러다임의 불일치 (RDB vs 객체지향)
  - 객체지향 : `추상화, 캡슐화, 정보은닉, 상속, 다형성` 등 다양한 기법으로 데이터를 관리함
  - RDB : `테이블` 기반의 데이터 관리
- 진정한 의미의 계층 분할이 어려워짐
  - ex) DAO가 동일한 객체를 두 번 불러오더라도 같은 객체인지 보장되지가 않음
- 객체답게 모델링할수록 매핑 작업이 늘어남

이를 극복하고자 Java 진영에서는 **자바 표준 ORM 인터페이스**인 `JPA (Java Persistence API)`를 만들었다. JPA는 Java (JDBC)와 DB (SQL) 사이에서 패러다임을 일치시켜주는 기술로, 객체를 쿼리가 아닌 엔티티로 관리해 특정 DB에 종속되지 않고 **SQL에 종속적인 개발**로부터 벗어나게 한다.

![8.png]({{ site.baseurl }}/assets/images/2020-03-20/8.png)

> 참고 사이트 : [JPA, Hibernate, 그리고 Spring Data JPA의 차이점](https://suhwan.dev/2019/02/24/jpa-vs-hibernate-vs-spring-data-jpa/)

**인터페이스 (기술 명세)**인 JPA는 `Hibernate`와 같은 구현체 형태로 기능을 수행한다. 물론 실무에서는 Hibernate보다 더 간편한 `Spring Data JPA` 모듈을 사용하지만, JPA의 기본 원리에 대한 이해는 베이스로 깔려 있어야 한다.

> 관련 포스트 : [영속성 컨텍스트](https://cupjoo.github.io/영속성-컨텍스트)

Spring 팀에서는 Hiberante가 있음에도 몇 가지 이유로 Spring Data JPA를 사용할 것을 권장하고 있다.

- Hibernate를 대체하는 새로운 구현체 등장 시 교체가 쉽다.
- RDB 외에 다른 저장소로 교체가 쉽다.
- Spring Data의 하위 프로젝트는 모두 같은 인터페이스를 갖는다.
- ex) Spring Data JPA, Spring Data MongoDB, Spring Data Redis
- 따라서 저장소를 변경해도 기존 메소드들을 그대로 사용 가능하다.

## 5. H2 Database

H2는 별도의 설치 없이 사용하는 인메모리 RDB로, 어플리케이션 실행 중에만 유지되다가 종료 시 초기화된다. 주로 테스트 용도로 많이 쓰인다.

```java
// build.gradle

dependencies {
    // ...
    runtimeOnly 'com.h2database:h2'
}
```

```yml
# resources/application.yml

server:
  port: 8099  # Spring boot Server port

spring:
  h2:
    console:
      enabled: true

  datasource:   # 생략 가능
    driver-class-name: org.h2.Driver
    url: jdbc:h2:mem:testdb # 기본값
    username: sa    # 기본값 : sa
    password: 1234  # 기본값 : (공백)
```

의존성 추가 후 어플리케이션 실행 시 [https://localhost:8080/h2-console](https://localhost:8080/h2-console)에 접속하면 다른 RDB처럼 사용이 가능하다.

![13.png]({{ site.baseurl }}/assets/images/2020-03-20/13.png)
