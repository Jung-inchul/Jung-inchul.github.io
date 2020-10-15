---
layout: reference
title: 아이템7 다 쓴 객체 참조를 해제하라
date: '2020-03-10T00:00:00.000Z'
categories: effective
summary: Effective Java 3e 아이템 7을 요약한 내용 입니다.
navigation_weight: 7
description: Effective Java 3e 아이템 7를 요약한 내용 입니다.
---

# 아이템7 다 쓴 객체 참조를 해제하라

자바처럼 가비지 컬렉터를 갖춘 언어로 넘어오면 프로그래머의 삶이 훨씬 평안해진다.

그래서 자칫 `메모리 관리에 더 이상 신경 쓰지 않아도 된다고 오해할 수 있는데 절대 사실이 아니다`

```java
    public Class Stack {

        public Object pop() {
            if (size == 0)
                throw new EmptyStackException();
            return elements[--size];
        }
        ...

    }
```

위의 코드는 기본적으로 많이 사용하는 스택 클래스이다.

그런데 이 Stack 클래스를 오래 실행하다 보면 점차 `가비지 컬렉션 활동`과 `메모리 사용량`이 늘어나 결국 성능이 저하될 것이다.

이 코드에서는 스택이 커졌다가 줄어들었을 때 스택에서 꺼내진 객체들을 가비지 컬렉터가 회수하지 않는다. 여기서 다 쓴 참조란 문자 그대로 앞으로 다시 쓰지 않을 참조를 뜻한다. 앞의 코드에서는 elements 배열의 '활성 영역'밖의 참조들이 모두 여기에 해당한다.

객체 참조 하나를 살려두면 가비지 컬렉터는 그 객체뿐 아니라 `그 객체가 참조하는 모든 객체(그리고 또 그 객체들이 참조하는 모든 객체)를 회수해가지 못한다.`

JCF 에서 관리되는 리스트의 필드는 수동으로 메모리를 회수해야 할까?

해법은 간단하다. 해당 참조를 다 썼을 때 null 처리\(참조 해제\)하면 된다.

```java
    public Object pop() {
        if (size == 0)
                throw new EmptyStackException();
        Object result = elements[--size];
        elements[size] = null; // 다 쓴 참조 해제
        return result;    
    }
```

다 쓴 참조를 null 처리하면 다른 이점도 따라온다. 만약 null 처리한 참조를 실수로 사용하려 하면 프로그램은 즉시 `NullPointerException`을 던지며 종료 된다.

하지만 모든 데이터를 사용한 이후에 null 처리할 필요는 없다. 이는 바람직하지도 않고 코드를 지저분하게 만들 뿐이다.

객체 참조를 null 처리하는 일은 `예외적인 경우`여야 한다. 다 쓴 참조를 해제하는 가장 좋은 방법은 `그 참조를 담은 변수를 유효 범위(scope)밖으로 밀어내는 것이다.`

그렇다면 null 처리는 언제 해야 할까? Stack 클래스는 왜 메모리 누수에 취약한 걸까?

`바로 스택이 자기 메모리를 직접 관리하기 때문이다.` 이 스택은 \(객체 자체가 아니라 객체 참조를 담는\) elements 배열로 저장소 풀을 만들어 원소들을 관리한다.

> 비활성 영역의 객체가 더 이상 쓸모없다는 건 프로그래머만 아는 사실이다. 그러므로 프로그래머는 비활성 영역이 되는 순간 null 처리해서 해당 객체를 더는 쓰지 않을 것임을 가비지 컬렉터에 알려야 한다.

일반적으로 자기 메모리를 직접 관리하는 클래스라면 프로그래머는 항싱 `메모리 누수`에 주의해야 한다.

캐시 역시 메모리 누수를 일으키는 주범이다. 운 좋게 캐시 외부에서 키\(Key\)를 참조하는 동안만\(값이 아니다\) 엔트리가 살아 있는 캐시가 필요한 상황이라면 `WeakHashMap`을 사용해 캐시를 만들자

WeakHashMap는 언제 사용할 수 있을까?

LinkedHashMap은 removeEldestEntry 메서드를 써서 후자의 방식으로 처리한다.

메모리 누수의 세 번째 주범은 바로 `리스너 혹은 콜백`이라 부르는 것이다. 클라이언트가 콜백을 등록만 하고 명확히 해지하지 않는다면 뭔가 조치해주지 않는 한 콜백은 계속 쌓여갈 것이다. 이 또한 `WeakHashMap`에 키로 저장하면 해결 가능 하다.

콜백을 등록만 하고 명확히 해지 하지 않는 케이스는 어떤 케이스 일까?

## 정리

메모리 누수는 겉으로 잘 드러나지 않아 시스템에 수년간 잠복하는 사례도 있다. 이런 누수는 철저한 `코드 리뷰`나 `힙 프로파일러` 같은 디버깅 도구를 동원해야만 발견되기도 한다. 그래서 이런 종류의 문제는 예방법을 익혀두는 것이 매우 중요하다.
