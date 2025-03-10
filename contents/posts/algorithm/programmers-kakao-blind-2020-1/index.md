---
title: "프로그래머스 2020 KAKAO BLIND RECRUITMENT - 괄호 변환"
date: 2022-01-16
update: 2022-01-16
tags:
  - Algorithm
  - programmers
series: "Algorithm"
---

### 문제 설명

카카오에 신입 개발자로 입사한 "콘"은 선배 개발자로부터 개발역량 강화를 위해 다른 개발자가 작성한 소스 코드를 분석하여 문제점을 발견하고 수정하라는 업무 과제를 받았습니다. 소스를 컴파일하여 로그를 보니 대부분 소스 코드 내 작성된 괄호가 개수는 맞지만 짝이 맞지 않은 형태로 작성되어 오류가 나는 것을 알게 되었습니다.
수정해야 할 소스 파일이 너무 많아서 고민하던 "콘"은 소스 코드에 작성된 모든 괄호를 뽑아서 올바른 순서대로 배치된 괄호 문자열을 알려주는 프로그램을 다음과 같이 개발하려고 합니다.

### 용어의 정리

`'('` 와 `')'` 로만 이루어진 문자열이 있을 경우, `'('` 의 개수와 `')'` 의 개수가 같다면 이를 균형잡힌 괄호 문자열이라고 부릅니다.
그리고 여기에 `'('`와 `')'`의 괄호의 짝도 모두 맞을 경우에는 이를 올바른 괄호 문자열이라고 부릅니다.
예를 들어, `"(()))("`와 같은 문자열은 "균형잡힌 괄호 문자열" 이지만 "올바른 괄호 문자열"은 아닙니다.
반면에 `"(())()"`와 같은 문자열은 "균형잡힌 괄호 문자열" 이면서 동시에 "올바른 괄호 문자열" 입니다.

`'('` 와 `')'` 로만 이루어진 문자열 w가 "균형잡힌 괄호 문자열" 이라면 다음과 같은 과정을 통해 "올바른 괄호 문자열"로 변환할 수 있습니다.

```
1. 입력이 빈 문자열인 경우, 빈 문자열을 반환합니다. 
2. 문자열 w를 두 "균형잡힌 괄호 문자열" u, v로 분리합니다. 단, u는 "균형잡힌 괄호 문자열"로 더 이상 분리할 수 없어야 하며, v는 빈 문자열이 될 수 있습니다. 
3. 문자열 u가 "올바른 괄호 문자열" 이라면 문자열 v에 대해 1단계부터 다시 수행합니다. 
  3-1. 수행한 결과 문자열을 u에 이어 붙인 후 반환합니다. 
4. 문자열 u가 "올바른 괄호 문자열"이 아니라면 아래 과정을 수행합니다. 
  4-1. 빈 문자열에 첫 번째 문자로 '('를 붙입니다. 
  4-2. 문자열 v에 대해 1단계부터 재귀적으로 수행한 결과 문자열을 이어 붙입니다. 
  4-3. ')'를 다시 붙입니다. 
  4-4. u의 첫 번째와 마지막 문자를 제거하고, 나머지 문자열의 괄호 방향을 뒤집어서 뒤에 붙입니다. 
  4-5. 생성된 문자열을 반환합니다.
```

__"균형잡힌 괄호 문자열"__ p가 매개변수로 주어질 때, 주어진 알고리즘을 수행해 `"올바른 괄호 문자열"`로 변환한 결과를 return 하도록 solution 함수를 완성해 주세요.

### 매개변수 설명
* p는 `'('` 와 `')'` 로만 이루어진 문자열이며 길이는 2 이상 1,000 이하인 짝수입니다.
* 문자열 p를 이루는 `'('` 와 `')'` 의 개수는 항상 같습니다.
* 만약 p가 이미 "올바른 괄호 문자열"이라면 그대로 return 하면 됩니다.

### 입출력 예

|     p      |   result   |
| :--------- | :--------- | 
| "(()())()" | "(()())()" |
|    ")("    |	   "()"   |
| "()))((()" | "()(())()" |

### 풀이
처음 문제를 보았을때 문제가 길어서 이해하는데가 오래 걸렸다. 문제의 핵심은 -> 결국 괄호의 짝은 다 맞지만 __"올바른 괄호 문자열"__ 을 리턴하는 것과 용어의 정리에서 `재귀` 쓰라고 문제에 적혀있는 대로만 작성하면 된다고 생각하여 차근차근 풀어 나갔다.

* 크게 두가지로 나누어 접근하였다.

1. "올바른 괄호인지 확인"

```java
    private boolean isParenthesis(String p) {
        if (p.charAt(0) == ')') {
            return false;
        } else {
            Stack<Character> stack = new Stack<Character>();
            for (char ch: p.toCharArray()) {
                if (ch == '(') {
                    stack.push(ch);
                } else {
                    if (stack.isEmpty()) {
                        return false;
                    }
                    stack.pop();
                }
            }
            return stack.isEmpty();
        }
    }
```
많은 알고리즘 문제에서 나오는 `괄호` 문제에서 스택을 이용하여 확인하였다.

2. 용어의 정리에서 주어진 1 ~ 4까지 작성

```java
 // 균형잡힌 괄호 문자열 split
    private String split(String p) {
        if (p.equals("")) {
            return p;
        }
        int start = 0, end = 0, index = 0;
        // 문자열 u, v로 나누기 index
        for (int i = 0; i < p.length(); i++) {
            if (p.charAt(i) == '(') {
                start++;
            } else {
                end++;
            }
            if (start == end) {
                index = i;
                break;
            }
        }
        StringBuilder u = new StringBuilder(p.substring(0, index + 1));
        StringBuilder v = new StringBuilder(p.substring(index + 1));
```
문제에서 나온 문자열 u, v로 나누기위해 변수 두개를 설정하여 열린괄호와 닫힌괄호의 개수가 같은 -> "균형잡힌 괄호 문자열"로 나누어서 두 변수 설정하였다.

```java
 // 3 : 올바른 괄호 확인 -> 올바르면 올바른 문자열(u)에 v를 1단계부터 다시하여 붙임
        if (isParenthesis(u.toString())) {
            u.append(split(v.toString()));
        } else { // 4 : 올바른 괄호 x -> 빈 문자열에 '(' 에 문자열 v에 대해 1단계부터 다시하여 붙이고 ')'를 다시 붙임
            // 4-1, 4-2, 4-3
            StringBuilder sb = new StringBuilder("(" + split(v.toString()) + ")");
            // 4-4 첫번째와 마지막 분자 제거후 나머지 문자열 괄호 방향 바꿈
            u.deleteCharAt(u.length() - 1);
            u.deleteCharAt(0);
            u = new StringBuilder(reverse(u.toString()));
            return sb.append(u.toString()).toString();
        }
        return u.toString();
    }
```
3 ~ 4번에서 주어진 문제대로 `"올바른 문자열"`과 아닌것을 구분하여 작성하였다. 괄호 방향의 바꿈은 `reverse`함수를 작성하였다.

```java
    private String reverse(String p) {
        StringBuilder sb = new StringBuilder();
        for (String s : p.split("")) {
            if (s.equals(")")) {
                sb.append("(");
            } else if (s.equals("(")){
                sb.append(")");
            }
        }
        return sb.toString();
    }
```

__전체 코드__
```java
import java.util.Stack;

class Parenthesis {
    public String solution(String p) {
        StringBuilder sb = new StringBuilder();
        // p가 빈문자열이면 빈문자열 , 올바은 괄호면 올바른 괄호 리턴
        if (p.equals("") || isParenthesis(p)) {
            return p;
        } else {
            sb.append(split(p));
        }
        return sb.toString();
    }

    // 균형잡힌 괄호 문자열 split
    private String split(String p) {
        if (p.equals("")) {
            return p;
        }
        int start = 0, end = 0, index = 0;
        // 문자열 u, v로 나누기 index
        for (int i = 0; i < p.length(); i++) {
            if (p.charAt(i) == '(') {
                start++;
            } else {
                end++;
            }
            if (start == end) {
                index = i;
                break;
            }
        }
        StringBuilder u = new StringBuilder(p.substring(0, index + 1));
        StringBuilder v = new StringBuilder(p.substring(index + 1));

        // 3 : 올바른 괄호 확인 -> 올바르면 올바른 문자열(u)에 v를 1단계부터 다시하여 붙임
        if (isParenthesis(u.toString())) {
            u.append(split(v.toString()));
        } else { // 4 : 올바른 괄호 x -> 빈 문자열에 '(' 에 문자열 v에 대해 1단계부터 다시하여 붙이고 ')'를 다시 붙임
            // 4-1, 4-2, 4-3
            StringBuilder sb = new StringBuilder("(" + split(v.toString()) + ")");
            // 4-4 첫번째와 마지막 분자 제거후 나머지 문자열 괄호 방향 바꿈
            u.deleteCharAt(u.length() - 1);
            u.deleteCharAt(0);
            u = new StringBuilder(reverse(u.toString()));
            return sb.append(u.toString()).toString();
        }
        return u.toString();
    }

    private boolean isParenthesis(String p) {
        if (p.charAt(0) == ')') {
            return false;
        } else {
            Stack<Character> stack = new Stack<Character>();
            for (char ch: p.toCharArray()) {
                if (ch == '(') {
                    stack.push(ch);
                } else {
                    if (stack.isEmpty()) {
                        return false;
                    }
                    stack.pop();
                }
            }
            return stack.isEmpty();
        }
    }

    private String reverse(String p) {
        StringBuilder sb = new StringBuilder();
        for (String s : p.split("")) {
            if (s.equals(")")) {
                sb.append("(");
            } else if (s.equals("(")){
                sb.append(")");
            }
        }
        return sb.toString();
    }
}

```
