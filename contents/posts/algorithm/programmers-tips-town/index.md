---
title: "프로그래머스 2017 팁스타운 - 짝지어 제거하기"
date: 2021-12-25
update: 2021-12-25
tags:
  - Algorithm
  - programmers
series: "Algorithm"
---
### 문제 설명

짝지어 제거하기는, 알파벳 소문자로 이루어진 문자열을 가지고 시작합니다. 먼저 문자열에서 같은 알파벳이 2개 붙어 있는 짝을 찾습니다. 그다음, 그 둘을 제거한 뒤, 앞뒤로 문자열을 이어 붙입니다. 이 과정을 반복해서 문자열을 모두 제거한다면 짝지어 제거하기가 종료됩니다. 문자열 S가 주어졌을 때, 짝지어 제거하기를 성공적으로 수행할 수 있는지 반환하는 함수를 완성해 주세요. 성공적으로 수행할 수 있으면 1을, 아닐 경우 0을 리턴해주면 됩니다.
   
예를 들어, 문자열 S = `baabaa` 라면

b aa baa → bb aa → aa →

의 순서로 문자열을 모두 제거할 수 있으므로 1을 반환합니다.

### 제한사항
* 문자열의 길이 : 1,000,000이하의 자연수
* 문자열은 모두 소문자로 이루어져 있습니다.

### 입출력 예

|   s   | result |
| :---- | :----- | 
| baabaa|    1   | 
| cdcd  |    0   | 

### 풀이

- - - 

1. 짝지어 삭제하는 문제에서는 `Stack/Queue` 구조를 많이 사용하기 때문에 이 자료구조부터 생각하였다.
2. `Stack`이 비어있으면 값을 넣고 그 다음부터 `Stack`에 넣기전에 전에 `LIFO`구조로 가장 최신으로 들어온값을 비교하여 같으면 그값을 삭제하고 `다르면` `Stack`에 push하여 진행한다.
3. 문자열의 모든 값이 다 지나고 `Stack`이 empty면 -> 모두 제거 1 else 0으로 출력.

# 코드(Java)

- - -

```java
import java.util.Stack;

class Solution {
    public int solution(String s) {
        Stack<Character> stack = new Stack<Character>();
        for (char ch: s.toCharArray()) {
            if (stack.isEmpty()) {
                stack.push(ch);
            } else {
                // 마지막에 들어온 값과 현재 들어올 값이 같음
                if (stack.peek() == ch) {
                    stack.pop();
                } else {
                    stack.push(ch);
                }
            }
        }
        return stack.isEmpty() ? 1 : 0;
    }
}
```