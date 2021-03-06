---
layout: reference
title: 아이템8 finalizer와 cleaner 사용을 피하라
date: 2020-03-10T00:00:00.000Z
categories: effective
summary: Effective Java 3e 아이템 8을 요약한 내용 입니다.
navigation_weight: 8
---

# 2020-03-20-effective-8item

> Effective Java 3e 아이템 8를 요약한 내용 입니다.

자바는 `두 가지 객체 소멸자`를 제공한다. 그중 finalizer는 예측할 수 없고, 상황에 따라 위험할 수 있어 일반적으로 불필요하다. finalizer는 나름의 쓰임새가 몇 가지 있긴 하지만 기본적으로 '`쓰지 말아야`' 한다. 그래서 자바 9에서는 finalizer를 `사용 자제(deprecated) API`로 지정하고 `cleaner`를 그 대안으로 소개했다. `cleaner`는 `finalizer`보다는 덜 위험하지만, 여전히 예측할 수 없고, 느리고, 일반적으로 불필요하다.

자바에서는 `try-with-resources`와 `try-finally`를 사용해 해결한다.

finalizer와 cleaner는 즉시 수행된다는 보장이 없다. 얼마나 신속히 수행할지는 전적으로 `가비지 컬렉터 알고리즘`에 달렸으며, 이는 가비지 컬렉터 구현마다 천차만별이다. 그래서 클래스에 finalizer를 달아두면 `그 인스턴스의 자원 회수가 제멋대로 지연`될 수 있다.

한편, cleaner는 자신을 수행할 `스레드를 제어`할 수 있다는 면에서 조금 낫다. 하지만 여전히 백그라운드에서 수행되며 가비지 컬렉터의 통제하에 있으니 즉각 수행되리라는 보장은 없다.

> 따라서 프로그램 생애주기와 상관없는 상태를 영구적으로 수정하는 작업에서는 절대 finalizer나 cleaner에 의존해서는 안 된다.

finalizer의 부작용은 여기서 끝이 아니다. `finalizer 동작 중 발생한 예외는 무시되며, 처리할 작업이 남았더라도 그 순간 종료된다.` 잡지 못한 예외 때문에 해당 객체는 자칫 마무리가 덜 된 상태로 남을 수 있다. 보통의 경우엔 잡지 못한 예외가 스레드를 중단시키고 스택 추적 내역을 출력하겠지만, 같은 일이 finalizer에서 일어난다면 경고조차 출력하지 않는다.

AutoCloseable 객체를 생성하고 가비지 컬렉터가 수거하기까지 12ns가 걸린 반면 finalizer를 사용하면 550ns가 걸렸다. 다시 말해 finalizer를 사용한 객체를 생성하고 파괴하니 50배나 느렸다. `finalizer가 가비지 컬렉터의 효율을 떨어뜨리기 때문이다.`

finalizer가 효율을 어떻게 떨어뜨리는지 구체적인 사례를 알 수 있을까?

finalizer를 사용한 클래스는 finalizer 공격에 노출되어 심각한 보안 문제를 일으킬 수도 있다. `생성자나 직렬화 과정에서 예외가 발생하면 이 생성되다 만 객체에서 악의적인 하위 클래스의 finalizer가 수행될 수 있게 된다.` final이 아닌 클래스를 finalizer 공격으로부터 방어하려면 아무 일도 하지 않는 finalize 메서드를 만들고 final로 선언하자

finalizer의 구체적인 보안 이슈는?

`그저 AutoCloseable을 구현해주고 클라이언트에서 인스턴스를 다 쓰고 나면 close 메서드를 호출하면 된다.` close 메서드에서 이 객체는 더 이상 유효하지 않음을 필드에 기록하고 다른 메서드는 이 필드를 검사해서 객체가 닫힌 후에 불렸다면 `IlligalStateException`을 던지는 것이다.

> 그렇다면 cleaner와 finalizer는 무슨 용도로 사용할 수 있을까?

하나는 자원의 소유자가 close 메서드를 호출하지 않는 것에 대비한 안전망 역할이다. cleaner나 finalizer가 즉시 \(혹은 끝까지\) 호출되리라는 보장은 없지만, 클라이언트가 하지 않은 자원 회수를 늦게라도 해주는 것이 아예 안 하는 것보다는 나으니 말이다?

cleaner와 finalizer를 적절히 활용하는 두 번째 예는 네이티브 피어와 연결된 객체에서다.

네이티브 피어란 일반 자바 객체가 네이티브 메서드를 통해 기능을 위임한 네이티브 객체를 말한다. 네이티브 피어는 자바 객체가 아니니 가비지 컬렉터는 그 존재를 알지 못한다.

성능 저하를 감당할 수 없거나 네이티브 피어가 사용하는 자원을 즉시 회수해야 한다면 앞서 설명한 close 메서드를 사용해야 한다.

만약 안전망으로 사용하고 싶지 않다면 아래와 같이 코드를 작성 하면 될것이다.

```java
    public class Adult {
        public static void main(String[] args) {
            try (Room myRoom = new Room(7)) {
                System.out.println("안녕~");
            }
        }
    }
```

## 정리

cleaner\(자바 8까지는 finalizer\)는 안전망 역할이니 중요하지 않은 네이티브 자원 회수용으로만 사용하자. 물론 이런 경우라도 불확실성과 성능 저하에 주의해야 한다.

