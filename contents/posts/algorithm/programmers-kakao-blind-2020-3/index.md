---
title: "프로그래머스 2022 KAKAO RECRUITMENT - k진수에서 소수 개수 구하기"
date: 2022-01-23
update: 2022-01-23
tags:
  - Algorithm
  - programmers
series: "Algorithm"
---

### 문제 설명
양의 정수 `n`이 주어집니다. 이 숫자를 `k`진수로 바꿨을 때, 변환된 수 안에 아래 조건에 맞는 소수(Prime number)가 몇 개인지 알아보려 합니다.

* __`0P0`__ 처럼 소수 양쪽에 0이 있는 경우
* __`P0`__ 처럼 소수 오른쪽에만 0이 있고 왼쪽에는 아무것도 없는 경우
* __`0P`__ 처럼 소수 왼쪽에만 0이 있고 오른쪽에는 아무것도 없는 경우
* __`P`__ 처럼 소수 양쪽에 아무것도 없는 경우
* 단, __`P`__ 는 각 자릿수에 0을 포함하지 않는 소수입니다.
   * 예를 들어, 101은 __`P`__ 가 될 수 없습니다.

예를 들어, 437674을 3진수로 바꾸면 `211`0`2`0100`11`입니다. 여기서 찾을 수 있는 조건에 맞는 소수는 왼쪽부터 순서대로 211, 2, 11이 있으며, 총 3개입니다. (211, 2, 11을 `k`진법으로 보았을 때가 아닌, 10진법으로 보았을 때 소수여야 한다는 점에 주의합니다.) 211은 `P0` 형태에서 찾을 수 있으며, 2는 `0P0`에서, 11은 `0P`에서 찾을 수 있습니다.

정수 `n`과 `k`가 매개변수로 주어집니다. `n`을 `k`진수로 바꿨을 때, 변환된 수 안에서 찾을 수 있는 위 조건에 맞는 소수의 개수를 return 하도록 solution 함수를 완성해 주세요.

### 제한사항
* 1 ≤ `n` ≤ 1,000,000
* 3 ≤ `k` ≤ 10

### 입출력 예
<img src="https://user-images.githubusercontent.com/63226023/150669729-1c8bb414-e177-4eee-ac87-4c0b7de5f459.png">

### 제한시간 안내
* 정확성 테스트 : 10초


### 풀이
문제의 요구사항을 보면 주어진 수를 K 진법으로 교체후 이를 __'0'__ 을 기준으로 나누어 소수인지를 확인하여 그 개수를 리턴하는 문제였다.

예시에서 나온 437674를 3진수로 바꾼 예시만 보더라도 __'int'__ 형이 가질 수 있는 최대값을 넘기 때문에 __'long'__ 형으로 받아야 한다는 것을 먼저 생각하였다. 그 후 주어진 K값의 주어진 진법으로 교체와 __'0'__ 을 기준으로 split하여 소수인지 확인을 하여 구하는 값을 리턴하였다.

# 코드

```java
class K_Prime {
    public int solution(int n, int k) {
        int answer = 0;
        String[] split = convert(n,k).split("0");
        // 소수인지 확인
        for (String sNumber: split) {
            if (!sNumber.equals("") && isPrime(Long.parseLong(sNumber))) answer++;
        }

        return answer;
    }

    // k진법으로 교체
    private String convert(int n, int k) {
        StringBuilder sb = new StringBuilder();
        if (k == 10) {
            return String.valueOf(n);
        }
        while (n != 0) {
            int remainder = n % k;
            sb.insert(0, remainder);
            n /= k;
        }
        return sb.toString();
    }

    // 소수 확인
    public boolean isPrime(long num) {
        if (num == 1) return false;
        for (long i = 2; i <= Math.sqrt(num); i++) {
            if (num % i == 0) return false;
        }
        return true;
    }
}  
```

