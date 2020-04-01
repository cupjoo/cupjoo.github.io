---
layout: post
title: "Java String in 알고리즘"
author: cupjoo
categories: [알고리즘, Java]
image: assets/images/2020-02-10/1.png
hidden: true
---

[대회기출로 정리하는 알고리즘 로드맵](https://cupjoo.github.io/대회기출로-정리하는-알고리즘-로드맵)

---

## String

사실 String은 JCF (Java Collections Framework) 에 포함된 클래스가 아니지만 문자열 문제를 푸는데 필수 요소이기 때문에 내용에 추가했다.

```java
class StringTest {
    public StringTest(){
        String text = "My name is Junyoung";

        System.out.println(text.equals("My name is Junyoung"));
        System.out.println(text.indexOf("me"));  // O(n), else -1

        // replace
        text = text.replaceAll("Junyoung", "Charley");

        // substring
        text = text.substring(11) + ", " + text.substring(0, 11);
        System.out.println(text);
        // split
        String[] tokens = text.split(" ");
        for(String token : tokens){
            System.out.println(token);
        }

        // concat
        String sum = tokens[0]
                .concat(tokens[1])
                .concat(tokens[2]); // +
        sum += tokens[3];           // by StringBuilder
        System.out.println(sum);

        // StringBuilder
        StringBuilder builder = new StringBuilder();
        for(String token : tokens){
            builder.append(token);
        }
        System.out.println(builder.toString());

        // compareTo
        System.out.println("bbb".compareTo("bbb"));     // 0, equals
        System.out.println("bbb".compareTo("b"));       // 2, length
        System.out.println("bbb".compareTo("bbbbbb"));  // -3, length
        System.out.println("bbb".compareTo("bbbccc"));  // -3, length
        System.out.println("bbb".compareTo("cccccc"));  // -1, dictionary order
        System.out.println("bbb".compareTo("ccc"));     // -1, dictionary order
        System.out.println("bbb".compareTo("aaa"));     // 1, dictionary order
    }
}
```

**특징**

- 이후 추가할 예정

Java에서 문자열을 다루는 메소드에 대해 알아보기로 한다. 

  

**1\. 문자열 입력**

-   **Scanner 클래스**

// import java.util.Scanner;
Scanner scan = new Scanner(System.in);
		
String a = scan.nextLine();
String b = scan.nextLine();

자바에서 사용자 입력으로 값을 입력받으려면 Scanner 클래스를 import 해야 한다.

  

**2. ****문자열 길이 반환**

-   **length**

String a = "example";
System.out.println(a.length());

  

**3\. 문자열 비교**

-   **equals**

String a = "sample";
String b = "Sample";
String c = new String("sample");

System.out.println(a.equals(b));
System.out.println(a.equals(c));
System.out.println(a==c);

문자열 비교에는 equals 메소드를 사용하며, 대소문자를 구분한다. == 연산 사용 시 가리키는 객체가 다른 경우 문자열이 같더라도 false를 반환한다.

  

**4. ****문자열 복사**

String a = "simple is best";
String b = a;
b = b.replace("im", "am");

System.out.println(a);
System.out.println(b);

자바에서는 문자열의 수정이 불가능하다. 따라서 다른 문자열에 기존 문자열을 대입하더라도 값에 변화가 없기 때문에 안전하다.

  

**5. ****문자열 합치기**

-   **\+ 연산자**

String a = "simple";
String b = " is";
String c = " best";
a = a+b+c;
System.out.println(a);

문자열 합치기에는 concat 메소드를 사용하기도 하는데, + 연산자와는 완전히 다른 동작 방식을 갖는다.  우선 concat의 경우 문자열을 합친 후에 new String을 통해 새롭게 문자열을 선언한다. 따라서 복수의 문자열을 합칠수록 연산의 효율이 떨어지게 된다. 반면에 + 연산자의 경우, 다음과 같이 문자열을 StringBuilder 클래스로 변환한 뒤 append 메소드로 각 문자열을 더한 뒤 다시 String 형으로 변환하므로, 일정한 연산 효율을 갖는다.

(new StringBuilder(String.valueOf(a)).append(b).toString();

  

**6. ****문자열 자르기**

-   **split**

String str = "2018-06-16";
String\[\] strs = str.split("-");

문자열을 주어진 문자를 기준으로 분리한 배열을 반환한다.

  

**7. ****문자열 탐색**

-   **indexOf**

String a = "example";
System.out.println(a.indexOf("am"));

문자열 내에서 특정 문자열의 시작 인덱스를 반환한다.

  

**8\. 문자열 추출**

-   **substring**

String a = "simple is best";
String b = a.substring(4);
String c = a.substring(4,6);

System.out.println(b);
System.out.println(c);

문자열의 특정 인덱스 범위에 해당하는 문자열을 반환한다.

  

**9\. 문자열 변환**

-   **replace**

String a = "simple simpson";
String b = a.replace("sim", "ap");
System.out.println(b);

문자열에서 특정 문자열과 일치하는 부분을 모원하는 문자열로 바꾼 뒤 반환한다.

  

**10\. 공백 제거**

-   **trim**

String a = "  sample  ";
a = a.trim();

문자열에서 공백을 제거한 뒤 반환한다.

  

  

## 참고 사이트

> [concat 과 + 연산자의 차이](https://programmers.co.kr/learn/questions/571)