---
layout: post
title: "Java Collections in 알고리즘"
author: cupjoo
categories: [알고리즘, Java]
image: assets/images/2020-02-03/1.png
---

[대회기출로 정리하는 알고리즘 시리즈 로드맵](https://cupjoo.github.io/대회기출로-정리하는-알고리즘-시리즈-로드맵)

---

이번 포스팅은 설명보다는 사용법 위주로 작성하여 Cheetsheet의 용도로 활용할 예정이다. 꼭 필요한 Collection만 정리했다.

## ArrayList

```java
class ArrayListTest {
    List<Integer> ob = new ArrayList<Integer>();
    List<Integer> nb;

    public ArrayListTest() {
        // init and add
        nb = new ArrayList<Integer>(            // O(n), same as clone()
                Arrays.asList(5, 4, 3, 2));
        ob.add(2);                              // O(1)
        ob.add(0, 2);                           // *O(n)
        ob.addAll(nb);                          // O(n)
        Collections.sort(ob);

        // check status
        int idx = ob.indexOf(2);                // *O(n), first meet else -1
        System.out.println(ob.get(idx));        // O(1)
        ob.set(idx, 9);                         // O(1)
        System.out.println(ob.contains(9));     // *O(n)

        System.out.println(ob.size());          // O(1)
        ob.clear();                             // O(n), replacement : re-initialize
    }
}
```

**특징**

- add(0,n) / indexOf / contains 가 모두 O(n) 복잡도를 갖는다.
- add(n) / get 가 O(1) 복잡도를 갖는다.

사실 특징만 보면 복잡도도 높은데 굳이 써야 할 이유를 모르겠다. 하지만 ArrayList의 장점은 2차원 ArrayList를 사용해 Graph를 인접 리스트로 표현할 때 나타나게 된다.

```java
class ArrayListTest {
    List<List<Integer>> graph;  // 2-D ArrayList

    public ArrayListTest() {
        int[][] input = { {1, 3}, {2, 5}, {3, 4}, {4, 6} };

        // adjacency list
        graph = new ArrayList<>();
        for(int i = 0; i <= input.length; i++){
            graph.add(new ArrayList<>());
        }

        for (int[] edge : input) {
            graph.add(edge[0], edge[1]);
            graph.add(edge[1], edge[0]);
        }

        System.out.print("1 connect to ");
        for (int node : graph.get(1)) {
            System.out.print(node + " ");
        }
    }
}
```

**정리**

- `Graph`를 표현할 때 사용한다.
- 배열의 sort가 필요할 때 사용한다.
- 그 외에는 무조건 다른 자료구조를 사용한다.

## Stack

스택은 FIFO를 위한 자료구조다. 설명이 필요 없을 것 같다.

```java
class StackTest {
    public StackTest(){
        int top = -1;
        Stack<Integer> st = new Stack<>();

        for(int i = 0; i < 5; i++){
            st.push(i);         // O(1)
        }
        if(!st.empty()){
            top = st.pop();     // O(1), remove and returns value
        }
        System.out.println(top);
    }
}
```

관련 문제

- [히스토그램](https://www.acmicpc.net/problem/1725)
- [탑](https://www.acmicpc.net/problem/2493)

## Queue

큐는 FILO를 위한 자료구조다. 스택과 달리 Interface로 구현되어 있으며, LinkedList와 ArrayDeque를 사용해 표현한다. 성능 면에서 더 나은 ArrayDeque를 사용하자.

```java
class QueueTest {
    public QueueTest(){
        int front = -1;
        Queue<Integer> q = new ArrayDeque<>();

        for(int i = 0; i < 5; i++){
            q.offer(i);         // O(1)
        }
        if(!q.isEmpty()){
            front = q.poll();   // O(1), remove and return value
        }
        System.out.println(front);
    }
}
```

참고로 큐는 큐 자체만의 문제보다 BFS를 구현하기 위한 도구로 쓰일 때가 더 많다.

## PriorityQueue

간단히 설명하자면 PriorityQueue는 들어온 순서에 상관 없이 우선순위가 높은 데이터가 먼저 반환되는 자료형이다. 기초 자료형에 대한 우선순위 큐는 기본적으로 그 값이 작은 데이터부터 반환한다. 하지만 사용자 클래스에 대한 우선순위 큐를 생성하는 경우 Comparable Interface를 implement하여 우선순위를 직접 정해줘야 한다.

```java
class Prisoner implements Comparable<Prisoner>{
    String name;
    int weight;

    public Prisoner(String name, int weight){
        this.name = name;
        this.weight = weight;
    }
    @Override
    public int compareTo(Prisoner o) {
        return this.weight >= o.weight ? 1 : -1;
    }
}

class PriorityQueueTest {
    public PriorityQueueTest(){
        PriorityQueue<Prisoner> pq = new PriorityQueue<>();
        pq.offer(new Prisoner("junyoung", 5));
        pq.offer(new Prisoner("cupjoo", 15));
        pq.offer(new Prisoner("Pacquiao", 10));

        if(!pq.isEmpty()){
            Prisoner p = pq.poll();
            System.out.println(p.name);
        }

        PriorityQueue<String> pqs = new PriorityQueue<>();
        pqs.offer("aaa");
        pqs.offer("bbb");
        pqs.offer("ccc");

        if(!pqs.isEmpty()){
            String s = pqs.poll();
            System.out.println(s);
        }
    }
}
```

**정리**

- 업데이트 예정

관련 문제

- 업데이트 예정

## Set

Set은 특정 값의 존재 유무 만을 판단하는 자료구조로, Java에서는 Interface로 구현되어 있고, 구현 방식에 따라 HashSet, TreeSet, LinkedHashSet으로 나뉜다. TreeSet 같은 Set은 입력순서를 보장하는 자료구조인데, 사실 알고리즘에서 필요한지는 모르겠다. 만약 정렬이 필요한 경우 List나 다른 자료구조를 써도 되는데 굳이? Set을 써야만 하는 상황을 잘 고려해보면 HashSet이 성능 면에서 적합한 것 같다. 

```java
import java.io.*;
import java.util.*;

// No duplicated. Unordered
// Only judges the existence of data
class HashSetTest {
    public HashSetTest(){
        HashSet<Integer> set = new HashSet<>();
        set.add(10);                            // O(1)
        System.out.println(set.contains(10));   // O(1)
    }
}
```

**정리**

- `값의 유무를 판단`할 때 사용한다. 그 외 용도로는 사용하지 말자.
- `HashSet`을 사용하자.

## Map

Map은 특정 key의 유무와 해당 key에 특정 value가 저장되는 자료구조이다. 마찬가지로 Map은 Interface로 구현되어 있고 구현 방식에 따라 여러 종류로 나뉘는데, 같은 이유로 알고리즘에서 Map을 사용해야만 하는 상황을 고려해본다면 다른 기능은 제쳐두고 HashMap을 사용하는 것이 적절하다.

하지만 아직까지도 Set이 있는데 굳이 Map을 써야하나? 라는 생각이 드는데 다음 문제를 풀어보면 Map을 써야하는 상황이 언제인지 대충 감이 올거다.

[BOJ 1302 베스트셀러](https://www.acmicpc.net/problem/1302)

```java
import java.io.*;
import java.util.*;

// No duplicated. Unordered
// store {key : value} data
class HashMapTest {
    HashMap<String, Integer> map = new HashMap<>();

    public HashMapTest(String[] books) {
        for(String book : books){
            if(map.containsKey(book)){              // O(1)
                map.put(book, map.get(book)+1);     // O(1), overwrite
            } else {
                map.put(book, 1);                   // O(1)
            }
        }

        int max = 0;
        String book = "";
        for(String key : map.keySet()) {
            int val = map.get(key);
            if(max < val || (max == val && book.compareTo(key) > 0)){
                book = key;
                max = val;
            }
        }
        System.out.print(book);
    }
}
```

**정리**

- `키의 유무와 그 값으로 연산`이 필요할 때 사용한다. 그 외 용도로는 사용하지 말자.
- `HashMap`을 사용하자.

## Collections

Collection 클래스의 공통 함수를 담고 있는 클래스이다.

- lower_bound / upper_bound
- sort
- reverse
- next_permutation

## Time Complexity

| Class | add (offer) | get (poll) | contains | index |
|---|:---:|:---:|:---:|:---:|
| ArrayList | O(k) | O(1) | O(n) | O(n) |
| HashSet | O(1) | O(1) | O(1) |  |
| HashMap | O(1) | O(1) | O(1) |  |
| Stack | O(1) | O(1) | O(1) |  |
| ArrayDeque | O(1) | O(1) | O(1) |  |
| PriorityQueue | O() | O() | O() | O() |
{: .custom-table }

## 출처

> 
- [Java Collections – Performance (Time Complexity)](http://infotechgems.blogspot.com/2011/11/java-collections-performance-time.html?m=1)
- http://javaqueue2010.blogspot.com/
