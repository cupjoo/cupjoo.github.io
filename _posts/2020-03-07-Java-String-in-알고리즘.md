---
layout: post
title: "Java String in 알고리즘"
author: cupjoo
categories: [알고리즘]
image: assets/images/2020-03-07/1.png
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