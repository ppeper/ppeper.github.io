---
title: "프로그래머스 2021 KAKAO BLIND RECRUITMENT - 순위 검색"
date: 2022-02-11
update: 2022-02-11
tags:
  - Algorithm
  - programmers
series: "Algorithm"
---
### 문제 설명

카카오는 하반기 경력 개발자 공개채용을 진행 중에 있으며 현재 지원서 접수와 코딩테스트가 종료되었습니다. 이번 채용에서 지원자는 지원서 작성 시 아래와 같이 4가지 항목을 반드시 선택하도록 하였습니다.

- 코딩테스트 참여 개발언어 항목에 cpp, java, python 중 하나를 선택해야 합니다.
- 지원 직군 항목에 backend와 frontend 중 하나를 선택해야 합니다.
- 지원 경력구분 항목에 junior와 senior 중 하나를 선택해야 합니다.
- 선호하는 소울푸드로 chicken과 pizza 중 하나를 선택해야 합니다.

인재영입팀에 근무하고 있는 니니즈는 코딩테스트 결과를 분석하여 채용에 참여한 개발팀들에 제공하기 위해 지원자들의 지원 조건을 선택하면 해당 조건에 맞는 지원자가 몇 명인 지 쉽게 알 수 있는 도구를 만들고 있습니다.   
예를 들어, 개발팀에서 궁금해하는 문의사항은 다음과 같은 형태가 될 수 있습니다.
`코딩테스트에 java로 참여했으며, backend 직군을 선택했고, junior 경력이면서, 소울푸드로 pizza를 선택한 사람 중 코딩테스트 점수를 50점 이상 받은 지원자는 몇 명인가?`

물론 이 외에도 각 개발팀의 상황에 따라 아래와 같이 다양한 형태의 문의가 있을 수 있습니다.

- 코딩테스트에 python으로 참여했으며, frontend 직군을 선택했고, senior 경력이면서, 소울푸드로 chicken을 선택한 사람 중 코딩테스트 점수를 100점 이상 받은 사람은 모두 몇 명인가?
- 코딩테스트에 cpp로 참여했으며, senior 경력이면서, 소울푸드로 pizza를 선택한 사람 중 코딩테스트 점수를 100점 이상 받은 사람은 모두 몇 명인가?
- backend 직군을 선택했고, senior 경력이면서 코딩테스트 점수를 200점 이상 받은 사람은 모두 몇 명인가?
- 소울푸드로 chicken을 선택한 사람 중 코딩테스트 점수를 250점 이상 받은 사람은 모두 몇 명인가?
- 코딩테스트 점수를 150점 이상 받은 사람은 모두 몇 명인가?

즉, 개발팀에서 궁금해하는 내용은 다음과 같은 형태를 갖습니다.
```
* [조건]을 만족하는 사람 중 코딩테스트 점수를 X점 이상 받은 사람은 모두 몇 명인가?
```

지원자가 지원서에 입력한 4가지의 정보와 획득한 코딩테스트 점수를 하나의 문자열로 구성한 값의 배열 info, 개발팀이 궁금해하는 문의조건이 문자열 형태로 담긴 배열 query가 매개변수로 주어질 때,
각 문의조건에 해당하는 사람들의 숫자를 순서대로 배열에 담아 return 하도록 solution 함수를 완성해 주세요.

### 제한사항

- info 배열의 크기는 1 이상 50,000 이하입니다.
- info 배열 각 원소의 값은 지원자가 지원서에 입력한 4가지 값과 코딩테스트 점수를 합친 "개발언어 직군 경력 소울푸드 점수" 형식입니다.
    - 개발언어는 cpp, java, python 중 하나입니다.
    - 직군은 backend, frontend 중 하나입니다.
    - 경력은 junior, senior 중 하나입니다.
    - 소울푸드는 chicken, pizza 중 하나입니다.
    - 점수는 코딩테스트 점수를 의미하며, 1 이상 100,000 이하인 자연수입니다.
    - 각 단어는 공백문자(스페이스 바) 하나로 구분되어 있습니다.
- query 배열의 크기는 1 이상 100,000 이하입니다.
- query의 각 문자열은 "[조건] X" 형식입니다.
    - [조건]은 "개발언어 and 직군 and 경력 and 소울푸드" 형식의 문자열입니다.
    - 언어는 cpp, java, python, - 중 하나입니다.
    - 직군은 backend, frontend, - 중 하나입니다.
    - 경력은 junior, senior, - 중 하나입니다.
    - 소울푸드는 chicken, pizza, - 중 하나입니다.
    - '-' 표시는 해당 조건을 고려하지 않겠다는 의미입니다.
    - X는 코딩테스트 점수를 의미하며 조건을 만족하는 사람 중 X점 이상 받은 사람은 - 모두 몇 명인 지를 의미합니다.
    - 각 단어는 공백문자(스페이스 바) 하나로 구분되어 있습니다.
    - 예를 들면, "cpp and - and senior and pizza 500"은 "cpp로 코딩테스트를 봤으며, 경력은 senior 이면서 소울푸드로 pizza를 선택한 지원자 중 코딩테스트 점수를 500점 이상 받은 사람은 모두 몇 명인가?"를 의미합니다.

### 입출력 예
<img src="https://user-images.githubusercontent.com/63226023/153607140-6278337f-e49e-44d4-97d5-ee0f002150ba.png">


### 풀이
문제의 요구사항을 보고 `어떠한 query가 맞나` -> `점수 비교` 라고 간단하게 생각을 해보았을 때 key-value 형태로 나타내어 일단 `HashMap`을 사용해야 하겠다고 생각하였다.

info와 query에서 `-`는 해당 조건을 고려하지 않는다고 하였으므로 info에서 query로 나올 수있는 `모든 조합`을 `-`를 포함하여 만들어 해당하는 query는 다 같은 info의 점수로 저장한다.   
점수를 여러개 저장하기 위하려면 (String, List) 형태를 사용하였다.
```java
 // 모든 나올수 있는 조합을 뽑아서 List에 넣는다
private void DFS(String str, int depth, String[] info) {
    if (depth == 4) {
        // 해당 조합의 사용자 정보가 저장 안되어있다면
        if (!allInfoValue.containsKey(str)) {
            ArrayList<Integer> list = new ArrayList<>();
            // 점수 저장
            list.add(Integer.parseInt(info[4]));
            allInfoValue.put(str, list);
        } else {
            // 해당 조합의 사용자 전보가 저장되어 있으면 -> list를 가져와서 해당 점수만 추가
            allInfoValue.get(str).add(Integer.parseInt(info[4]));
        }
        return;
    }
    DFS(str + "-", depth + 1, info);
    DFS(str + info[depth], depth + 1, info);
}
```
점수의 비교를 위하여 `오름차순`으로 점수를 저장을하여 `이진탐색`을 사용하여 시간복잡도를 더 줄이는 방법을 사용한다.
```java
// 해당 key값에 대한 점수를 오름차순으로 모두 정렬한다.
ArrayList<String> keys = new ArrayList<String>(allInfoValue.keySet());
for (String str: keys) {
    Collections.sort(allInfoValue.get(str));
}

// 이진탐색을 사용하여 answer을 구해준다.
    private int searchValue(String query, int score) {
        // 해당하는 질의에 만족하지 않으면
        if (!allInfoValue.containsKey(query)) {
            return 0;
        } else {
            ArrayList<Integer> value = allInfoValue.get(query);
            int start = 0, end = value.size() - 1;
            // 이진탐색
            while (start <= end) {
                int mid = (start + end) / 2;
                if (value.get(mid) < score) {
                    start = mid + 1;
                } else {
                    end = mid - 1;
                }
            }
            return value.size() - start;
        }
    }
```
# 코드

```java
import java.util.*;

class Solution {
    HashMap<String, ArrayList<Integer>> allInfoValue = new HashMap<>();
    public int[] solution(String[] info, String[] query) {
        int[] answer = new int[query.length];
        // 모든 경우의 key값을 구한다.
        for (String str : info) {
            DFS("", 0, str.split(" "));
        }
        // 해당 key값에 대한 점수를 오름차순으로 모두 정렬한다.
        ArrayList<String> keys = new ArrayList<String>(allInfoValue.keySet());
        for (String str: keys) {
            Collections.sort(allInfoValue.get(str));
        }

        // 원하는 값 구하기
        for (int i = 0; i < query.length; i++) {
            // java and backend and junior and pizza 100 -> javabackendjuniorpizza 100
            String replace = query[i].replaceAll(" and ", "");
            String[] str = replace.split(" ");
            answer[i] = searchValue(str[0], Integer.parseInt(str[1]));
        }
        return answer;
    }

    // 모든 나올수 있는 조합을 뽑아서 List에 넣는다
    private void DFS(String str, int depth, String[] info) {
        if (depth == 4) {
            // 해당 조합의 사용자 정보가 저장 안되어있다면
            if (!allInfoValue.containsKey(str)) {
                ArrayList<Integer> list = new ArrayList<>();
                // 점수 저장
                list.add(Integer.parseInt(info[4]));
                allInfoValue.put(str, list);
            } else {
                // 해당 조합의 사용자 전보가 저장되어 있으면 -> list를 가져와서 해당 점수만 추가
                allInfoValue.get(str).add(Integer.parseInt(info[4]));
            }
            return;
        }
        DFS(str + "-", depth + 1, info);
        DFS(str + info[depth], depth + 1, info);
    }

    // 이진탐색을 사용하여 answer을 구해준다.
    private int searchValue(String query, int score) {
        // 해당하는 질의에 만족하지 않으면
        if (!allInfoValue.containsKey(query)) {
            return 0;
        } else {
            ArrayList<Integer> value = allInfoValue.get(query);
            int start = 0, end = value.size() - 1;
            // 이진탐색
            while (start <= end) {
                int mid = (start + end) / 2;
                if (value.get(mid) < score) {
                    start = mid + 1;
                } else {
                    end = mid - 1;
                }
            }
            return value.size() - start;
        }
    }
}
```