---
title: "자바에서의 Comparable, Comparator 정렬"
date: 2023-02-06
update: 2023-02-06
tags:
  - Java
  - Sort
series: "Java"
---
다시 한번 마음을 잡고 기본을 쌓는 중에 객체 정렬에 대해서 다시 정리해 보았다. 최근 다시 공부를 하면서 내가 몰랐던 내용이나 애매했던 부분들을 잡아가면서 이번에는 객체 지향 언어에서의 클래스와 같은 객체 정렬에 대한 내용을 담아보려고 한다.
# 자바 Primitive 정렬
- 기본적으로 `Arrays.sort()` 를 통하여 정렬이 가능하다 (기본은 오름차순이다.)

```java
int[] arr = {1, 28, 16, 25, 99, 44};

Arrays.sort(arr);

System.out.println(Arrays.toString(arr)); 
// [1, 17, 25, 26, 44, 99, 303]
```
## Arrays.sort()
- 배열의 기본형 타입은 모두 정렬이 가능하다.

<img src="https://user-images.githubusercontent.com/63226023/216955538-498a4dd1-c748-452d-86ad-26293a2935f6.png">

- List 컬렉션의 경우 `Collections.sort()` 를 사용하여 정렬이 가능하다.

## 내림차순 정렬
내림차순으로 정렬하려면 `sort()`의 인자에 추가로 `Collections.reverseOrder()` 를 전달하면 된다.

```java
Integer[] arr = {1, 28, 16, 25, 99, 44};

Arrays.sort(arr, Collections.reverseOrder());

System.out.println(Arrays.toString(arr));
// [303, 99, 44, 26, 25, 17, 1]
```
> -> Collections.reversOrder()??
>
> 위에서의 인자로 전달된 친구는 사실 Comparator 객체이다. Comparator는 직접 구현해야 하지만, 내림차순은 자주 사용되어 Collections에서 기본적으로 제공해 주고 있는 것이다.

- - -

# Comparator vs Comparable
- 객체를 비교할 수 있도록 만든다.
- 기본형 타입의 경우 부등호로 비교가 가능하지만 객체의 경우는 불가능하다.

## Comparator - java.lang
- compare(T o1, T o2)를 구현해야 한다.
- 두 매개변수 객체를 비교

## Comparator - java.util
- compareTo(T o) 를 구현해야 한다.
- 자기 자신과 매개변수 객체를 비교

> <h3>❗주의 할 점 </h3>
> 객체에서 나이와 같은 정수형을 비교한다고 하였을때 예시와 같이 int형의 범위에 따른 underflow, overflow 를 조심해야한다!
> 따라서 Primitive 값에 대한 비교를 하게 되었을때는 <,>,==의 대소비교를 통하는 것이 안전하다.

## 차이점 비교
Comparator와 Comparable의 차이는 __`자기 자신`__ 과의 비교가 가능한가? 에서 있다. 이를 다시 생각해 보면 Comparable은 class 객체와 같이 정렬 기준을 정할 때 사용한다.

```java
class Student implements Comparable<Student> {
    String name;
    int age;

    public Student(String name, int age) {
        this.name = name;
        this.age = age;
    }

    @Override
    public int compareTo(Student o) {
        // 나이가 같을때
        if (this.age == o.age) {
            // 이름 순으로
            return this.name.compareTo(o.name);
        }
        // 나이가 어린순으로
        return this.age - o.age;
    }

    @Override
    public String toString() {
        return "Student{" +
                "name='" + name + '\'' +
                ", age=" + age +
                '}';
    }
}
// 정렬 기준이 정해짐
public class Main {

    public static void main(String[] args) {
        Student student1 = new Student("가", 27);
        Student student2 = new Student("나", 26);
        Student student3 = new Student("라", 28);
        Student student4 = new Student("다", 28);
        Student[] students = {student1, student2,student3, student4};
        // 정렬 X
        System.out.println("정렬 X");
        for (Student s: students) {
            System.out.println(s);
        }
        System.out.println("-------------------");
        System.out.println("정렬 -> 나이가 어린순, 같으면 이름이 빠른순");
        Arrays.sort(students);
        for (Student s: students) {
            System.out.println(s);
        }
        System.out.println("-------------------");
    }
}
정렬 X
Student{name='가', age=27}
Student{name='나', age=26}
Student{name='라', age=28}
Student{name='다', age=28}
-------------------
정렬 -> 나이가 어린순, 같으면 이름이 빠른순
Student{name='나', age=26}
Student{name='가', age=27}
Student{name='다', age=28}
Student{name='라', age=28}
-------------------
```

- Comparable을 상속받아 사용하면 해당하는 정렬 기준만 사용할 수 있지만 __기본 정렬 기준과 다르게 정렬__ 하고 싶을 때는 Comparator를 사용할 수 있다.

```java
class Student {
    String name;
    int age;

    public Student(String name, int age) {
        this.name = name;
        this.age = age;
    }

    @Override
    public String toString() {
        return "Student{" +
                "name='" + name + '\'' +
                ", age=" + age +
                '}';
    }
}
public class Main {

    public static void main(String[] args) {
        Student student1 = new Student("가", 27);
        Student student2 = new Student("나", 26);
        Student student3 = new Student("라", 28);
        Student student4 = new Student("다", 28);
        Student[] students = {student1, student2,student3, student4};
        // 정렬 X
        System.out.println("정렬 X");
        for (Student s: students) {
            System.out.println(s);
        }
        System.out.println("-------------------");
        System.out.println("정렬 -> 나이가 어린순");
        Arrays.sort(students, new Comparator<Student>() {
            @Override
            public int compare(Student o1, Student o2) {
                return o1.age - o2.age;
            }
        });
        for (Student s: students) {
            System.out.println(s);
        }
        System.out.println("-------------------");
        System.out.println("정렬 -> 이름순");
        Arrays.sort(students, (o1, o2) -> o1.name.compareTo(o2.name));
        System.out.println("-------------------");
        for (Student s: students) {
            System.out.println(s);
        }

    }
}
정렬 X
Student{name='가', age=27}
Student{name='나', age=26}
Student{name='라', age=28}
Student{name='다', age=28}
-------------------
정렬 -> 나이가 어린순
Student{name='나', age=26}
Student{name='가', age=27}
Student{name='라', age=28}
Student{name='다', age=28}
-------------------
정렬 -> 이름순
-------------------
Student{name='가', age=27}
Student{name='나', age=26}
Student{name='다', age=28}
Student{name='라', age=28}
```

## 람다식 표현

```java
// 람다식 사용 X
Arrays.sort(students, new Comparator<Student>() {
      @Override
      public int compare(Student o1, Student o2) {
          return o1.age - o2.age;
      }
});
// Comparator 객체 생성
Comparator<Student> comparator = new Comparator<Student>() {
	      @Override
      public int compare(Student o1, Student o2) {
          return o1.age - o2.age;
      }
}
Arrays.sort(students, comparator);
// 람다식 사용
Arrays.sort(students, (o1, o2) -> o1.age - o2.age);
// Comparator 제공 메소드
Arrays.sort(students, Comparator.comparingInt(o -> o.age));
```