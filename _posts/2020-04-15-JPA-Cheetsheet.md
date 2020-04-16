---
layout: post
title: "JPA Cheetsheet"
author: cupjoo
categories: [Spring, JPA]
image: assets/images/2020-04-15/1.png
hidden: true
---

Spring Boot와 Spring Data JPA, QueryDSL 환경에서 프로젝트를 구축할 때 어떤 구조로 구축하는지, 어떻게 최적화하는지를 참고하기 위해 Cheetsheet를 만들었다.

---

## 1. Spring & IntelliJ 초기 세팅

### 1) Prerequisites

- [Java 8](https://www.oracle.com/technetwork/java/javase/downloads/jdk8-downloads-2133151.html)
- Gradle 4
- Spring Boot 2.2.6
- jUnit 5 (jupyter : Spring Boot 2.2 이상)
- [H2 Database 1.4.200](https://www.h2database.com/html/main.html)

### 2) Dependencies

- Spring Boot DevTools
- Lombok
- Spring Web
- Spring Data JPA
- p6spy 1.6.1 (SQL 파라미터 로깅. 외부 라이브러리)

```gradle
plugins {
    id 'org.springframework.boot' version '2.2.6.RELEASE'
    id 'io.spring.dependency-management' version '1.0.9.RELEASE'
    id 'java'
}

group = 'study'
version = '0.0.1-SNAPSHOT'
sourceCompatibility = '1.8'

configurations {
    compileOnly {
        extendsFrom annotationProcessor
    }
}

repositories {
    mavenCentral()
}

dependencies {
    implementation 'org.springframework.boot:spring-boot-starter-data-jpa'
    implementation 'org.springframework.boot:spring-boot-starter-web'
    implementation 'org.springframework.boot:spring-boot-devtools'

    compileOnly 'org.projectlombok:lombok'
    runtimeOnly 'com.h2database:h2'
    annotationProcessor 'org.projectlombok:lombok'
    testImplementation('org.springframework.boot:spring-boot-starter-test') {
        exclude group: 'org.junit.vintage', module: 'junit-vintage-engine'
    }
    compile 'com.github.gavlyukovskiy:p6spy-spring-boot-starter:1.6.1'
}

test {
    useJUnitPlatform()
}
```

### 3) IntelliJ 초기 세팅

- Project Build 변경

![2.png]({{ site.baseurl }}/assets/images/2020-04-15/2.png)

- Table 매핑 오류 annotation 해제

![3.png]({{ site.baseurl }}/assets/images/2020-04-15/3.png)

- 콘솔 한글 깨짐 설정

![4.png]({{ site.baseurl }}/assets/images/2020-04-15/4.png)

- Lombok Plugin 설치 & Annotation 활성화

![5.png]({{ site.baseurl }}/assets/images/2020-04-15/5.png)

- H2 Database 파일 생성 (**jdbc:h2:~/프로젝트명**)
- 테스트용 application.yml 생성 (**test/resources/application.yml**)

```yml
logging.level:
  org.hibernate.SQL: debug  # Log
```

## 2. Spring Boot 프로젝트 구성

```yml
spring:
  #datasource:
    #url: jdbc:h2:tcp://localhost/~/jpashop
    #username: sa
    #password:
    #driver-class-name: org.h2.Driver

  jpa:
    #hibernate:
      #ddl-auto: create  # DDL
      format_sql: true
    properties:
      hibernate:
        default_batch_fetch_size: 1000

logging.level:
  org.hibernate.SQL: debug  # Log
```

## 3. Entity

### 1) 기본 엔티티 매핑

[도메인 모델 패턴](http://martinfowler.com/eaaCatalog/domainModel.html)에 기반해 엔티티를 작성한다. (vs [트랜잭션 스크립트 패턴](http://martinfowler.com/eaaCatalog/transactionScript.html))

```java
// Student
import lombok.*;
import javax.persistence.*;
import javax.validation.constraints.NotNull;

@Getter
@NoArgsConstructor(access = AccessLevel.PROTECTED)
@Entity
@Table(uniqueConstraints = {@UniqueConstraint(
        columnNames={"studentNumber"})})
public class Student extends BaseUser {

    @Id
    @GeneratedValue
    @Column(name = "student_id")
    private Long id;

    @Column(updatable = false, length = 50)
    private String studentNumber;

    private String name;
    private int age;

    @Enumerated(EnumType.STRING)
    private Sex sex;

    @Embedded
    private Address address;

    @Builder
    public Student(String studentNumber, String name, Sex sex, int age, Address address){
        this.studentNumber = studentNumber;
        this.name = name;
        this.sex = sex;
        this.age = age;
        this.address = address;
    }

    public void changeMajor(Department major){
        this.major = major;
        major.getStudents().add(this);
    }
}
```

@@ mappedsuperclass -> JPA Auditing으로 해결 가능하나?

```java
// BaseUser
@MappedSuperclass
public abstract class BaseUser {

    private LocalDate registerDate;
}
```

```java
// Address
@Getter
@NoArgsConstructor(access = AccessLevel.PROTECTED)
@Embeddable
public class Address {

    private String city;
    private String street;

    @Builder
    public Address(String city, String street){
        this.city = city;
        this.street = street;
    }
}
```

### 2) 연관관계 매핑

연관관계 매핑 규칙

- **@~~ToOne** : Lazy Loading 설정하기
- `단방향 매핑`을 기본으로 하고, 필요한 경우에 `양방향 매핑`을 추가한다.
  - 연관관계 주인에서 `양방향으로` 외래키를 관리해야 한다.
  - 연관관계와 별개로 영속성 전이는 필요한 경우에만 추가한다. (cascade)

```java
// Student
@ToString(exclude = "major")
public class Student extends BaseUser {
    // 단방향 매핑
    @ManyToOne(fetch = LAZY)
    @JoinColumn(name = "department_id")
    private Department major;

    // 연관관계 양방향 관리
    public void changeMajor(Department major){
        this.major = major;
        major.getStudents().add(this);
    }
}
```

```java
// Department
@Entity
@NoArgsConstructor
@Getter
public class Department {

    @Id @GeneratedValue
    @Column(name = "department_id")
    private Long id;

    private String name;

    // 양방향 매핑
    @OneToMany(mappedBy = "major")
    List<Student> students = new ArrayList<>();

    @Builder
    public Department(String name) {
        this.name = name;
    }
}
```

## 4. Repository

### 1) 기본 Repository

```java
// StudentRepository
import org.springframework.data.jpa.repository.JpaRepository;

public interface StudentRepository extends JpaRepository<Student, Long> {

    Optional<Student> findByStudentNumber(String studentNumber);
}
```

```java
// StudentRepositoryTest
import org.junit.jupiter.api.Test;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.transaction.annotation.Transactional;

import static org.junit.jupiter.api.Assertions.*;

@SpringBootTest
//@RunWith(SpringRunner.class)  // no more needs
@Transactional
class StudentRepositoryTest {

    @Autowired StudentRepository studentRepository;
    @Autowired DepartmentRepository departmentRepository;

    @Test
    public void studentCrudTest(){
        // given
        Department department = Department.builder().name("Software").build();
        departmentRepository.save(department);
        Address address = Address.builder().city("Seoul").street("Dobongro").build();
        Student student =  Student.builder()
                .studentNumber("20150000").name("Junyoung").age(26)
                .sex(Sex.Male).address(address).build();
        student.changeMajor(department);
        Long id = studentRepository.save(student).getId();

        // when
        Optional<Student> findStudent = studentRepository.findById(id);

        // then
        assertEquals(findStudent.get().getName(), student.getName());
    }
}
```

### 2) QueryRepository

@@

```java
// StudentQueryRepository
public interface StudentQueryRepository extends JpaRepository<Student, Long> {
}
```

## 5. Service

### 1) Service

Controller와 Repository만 세분화하고, Service 계층은 1개만 둔다.

```java
// StudentService
@Service
@Transactional(readOnly = true)
@RequiredArgsConstructor
public class StudentService {

    private final StudentRepository studentRepository;
    private final StudentQueryRepository studentQueryRepository;

    @Transactional
    public Long join(Student student){
        validateDuplicateStudent(student);
        return studentRepository.save(student).getId();
    }

    // @@ 페이지네이션 필요
    public List<Student> findStudents(){
        return studentRepository.findAll();
    }

    public Student findOne(Long studentId){
        return studentRepository.findById(studentId).orElse(null);
    }

    private void validateDuplicateStudent(Student student){
        boolean objectExists = studentRepository
                .findByStudentNumber(student.getStudentNumber())
                .isPresent();
        if(objectExists){
            throw new IllegalStateException("이미 존재하는 학생입니다.");
        }
    }
}
```

```java
// StudentServiceTest
import org.junit.jupiter.api.Test;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.transaction.annotation.Transactional;

import static org.junit.jupiter.api.Assertions.*;

@SpringBootTest
@Transactional
class StudentServiceTest {

    @Autowired StudentService studentService;
    @Autowired DepartmentRepository departmentRepository;

    @Test
    public void duplicatedJoin() {
        // given
        Department department = createDepartment();
        departmentRepository.save(department);
        Student student1 = createStudent(department, 10);
        Student student2 = createStudent(department, 20);
        studentService.join(student1);

        // then
        assertThrows(IllegalStateException.class, ()-> {
            studentService.join(student2);
        });
    }

    private Department createDepartment(){
        return Department.builder().name("Software").build();
    }
    private Student createStudent(Department department, int age){
        Address address = Address.builder().city("Seoul").street("Dobongro").build();
        Student student =  Student.builder()
                .studentNumber("20150000").name("Junyoung").age(age)
                .sex(Sex.Male).address(address).build();
        student.changeMajor(department);
        return student;
    }
}
```

## 6. Controller

### 1) 기본 API Controller

@@ Request Builder 먹히는거 맞아?

[Dtos Gist](https://gist.github.com/cupjoo/06d7bf33c0c3e1d1b0e006e40320ef5a)

```java
// StudentApiController
import lombok.RequiredArgsConstructor;
import org.springframework.web.bind.annotation.*;

import javax.validation.Valid;
import java.util.List;
import java.util.stream.Collectors;

@RequiredArgsConstructor
@RestController
public class StudentApiController {

    private final StudentService studentService;

    @PostMapping("/api/students")
    public CreateStudentResponse saveStudent(
            @RequestBody @Valid CreateStudentRequest request){

        Long studentId = studentService.join(request.toEntity());
        return new CreateStudentResponse(studentId);
    }

    @PutMapping("/api/students/{id}")
    public UpdateStudentResponse updateStudent(
            @PathVariable("id") Long id,
            @RequestBody @Valid UpdateStudentRequest request){

        studentService.updateMajor(id, request.getMajor());
        return UpdateStudentResponse.builder()
                .id(studentService.findOne(id).getId()).build();
    }

    @GetMapping("/api/students")
    public StudentListDto students(){
        List<Student> findStudents = studentService.findStudents();
        List<StudentDto> collect = findStudents.stream()
                .map(s -> StudentDto.builder()
                        .id(s.getId())
                        .name(s.getName())
                        .major(s.getMajor()).build())
                .collect(Collectors.toList());
        return new StudentListDto(collect);
    }

    @GetMapping("/api/student/{id}")
    public StudentDto detail(@PathVariable("id") Long id){
        Student findStudent = studentService.findOne(id);
        return StudentDto.builder()
                .id(findStudent.getId())
                .name(findStudent.getName())
                .major(findStudent.getMajor())
                .sex(findStudent.getSex()).build();
    }
}
```

### 2) Query API Controller

@@
