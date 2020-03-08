---
layout: post
title: "Segment Tree"
author: cupjoo
categories: [알고리즘]
image: assets/images/2020-02-12/1.png
hidden: true
---

[대회기출로 정리하는 알고리즘 로드맵](https://cupjoo.github.io/대회기출로-정리하는-알고리즘-로드맵)

---

## 기본 개념

다음 문제를 보자. [BOJ 2042 구간 합 구하기](https://www.acmicpc.net/problem/2042)

Fenwick Tree와 비교

```java
class SegTree {
    private long[] tree;    // root node : 1

    public SegTree(int N){
        int h = (int)Math.ceil(Math.log(N)/Math.log(2));
        tree = new long[1 << (h+1)];
    }
    public long init(long[] num, int node, int s, int e){
        if(s == e)  return tree[node] = num[s];
        int m = (s+e)/2;
        return tree[node] = init(num, node*2, s, m)
                + init(num, node*2+1, m+1, e);
    }
    // node's range: s~e & sum range: sumL~sumR
    public long sum(int node, int s, int e, int sumL, int sumR){
        if(e < sumL || sumR < s)    return 0;
        if(sumL <= s && e <= sumR)  return tree[node];
        int m = (s+e)/2;
        return sum(node*2, s, m, sumL, sumR)
                + sum(node*2+1, m+1, e, sumL, sumR);
    }
    public long update(int node, int s, int e, int idx, long diff){
        if(idx < s || idx > e)  return tree[node];
        if(s == e)  return tree[node] = diff;
        int m = (s+e)/2;
        return tree[node] = update(node*2, s, m, idx, diff)
                + update(node*2+1, m+1, e, idx, diff);
    }
}
```

## 대회 기출

## 추가 문제

## 출처

> 출처1
