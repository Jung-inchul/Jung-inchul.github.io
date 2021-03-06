---
layout: reference
title: 아이템18 상속보다는 컴포지션을 사용하라
date: '2020-03-10T00:00:00.000Z'
categories: effective
summary: Effective Java 3e 아이템 18을 요약한 내용 입니다.
navigation_weight: 18
description: Effective Java 3e 아이템 18를 요약한 내용 입니다.
---

# 아이템18 상속보다는 컴포지션을 사용하라

상위 클래스와 하위 클래스를 모두 같은 프로그래머가 통제하는 `패키지` 안에서라면 상속도 안전한 방법이다. 확장할 목적으로 설계 되었고 `문서화`도 잘 된 클래스도 마찬가지로 안전하다. 하지만 일반적인 구체 클래스를 패키지 경계를 넘어, 즉 다른 패키지의 구체 클래스를 상속하는 일은 위험하다.

메서드 호출과 달리 상속은 `캡슐화`를 깨뜨린다. 다르게 말하면, 상위 클래스가 어떻게 구현 되느냐에 따라 하위 클래스의 동작에 이상이 생길 수 있다.

```text
public class InstrumentedHashSet<E> extends HashSet<E> {
    private int addCount = 0;

    @Override
    public boolean add(E e) {
        addCount++;
        return super.add(e);
    }

    @Override
    public boolean addAll(Collection<? extends E> c) {
        addCount += c.size();
        return super.addAll(c);
    }

    public int getAddCount() { 
        return addCount;
    }

    ...
}
```

이 클래스는 잘 구현된 것처럼 보이지만 제대로 작동하지 않는다. 어떻게 작동이 안되는지 살펴보도록 하겠다.

```text
InstrumentedHashSet<String> s = new InstrumentedHashSet<>();
s.addAll(List.of("틱", "탁탁", "펑"));
```

이제 `getAddCount` 메서드를 호출하면 3을 반환하리라 기대하겠지만, 실제로는 6을 반환한다.

그 원인은 `HashSet`의 `addAll` 메서드가 `add` 메서드를 사용해 구현된 데 있다. 이런 내부 구현 방식은 HahSet 문서에는 \(당연히\) 쓰여 있지 않다.

이처럼 자신의 다른 부분을 사용하는 '`자기사용(self-use)`' 여부는 해당 클래스의 내부 구현 방식에 해당하며 다음 릴리즈에도 유지될지는 알 수 없다.

> 자기 사용이란?

### 하위 클래스가 깨지기 쉬운 이유는 더 있다.

다음 릴리스에서 상위 클래스에 새로운 메서드를 추가한다면 어떨까?

다음 릴리스에서 우려한 일이 생기면 하위 클래스에서 재정의하지 못한 그 새로운 메서드를 사용해 '`허용되지 않은`'원소를 추가할 수 있게 된다. 실제로도 컬렉션 프레임워크 이전부터 존재하던 `Hashtable`과 `Vector`를 컬렉션 프레임워크에 포함시키자 이와 관련한 보안 구멍들을 수정해야 하는 사태가 벌어졌다.

> Hashtable과 Vector의 보안 사례는 어떤것이 있는가?

다행히 이상의 문제를 모두 피해 가는 묘안이 있다. 기존 클래스가 새로운 클래스의 구성 요소로 쓰인다는 뜻에서 이러한 설계를 `컴포지션`이라 한다. 새 클래스의 인스턴스 메서드들은 \(private 필드로 참조하는\) 기존 클래스의 대응하는 메서드를 호출해 그 결과를 반환한다. 이 방식을 `전달(forwarding)`이라 하며, 새 클래스의 메서드들을 전달 `메서드(forwarding method)`라 부른다.

컴포지션과 전달의 조합은 넓은 의미로 `위임(delegation)`이라고 부른다.

상속은 반드시 하위 클래스가 상위 클래스의 '진짜'하위 타입인 상황에서만 쓰여야 한다. 다르게 말하면, 클래스 B가 클래스 A와 is-a 관계일 때만 클래스 A를 상속해야 한다. = `리스코프 치환 법칙`

컴포지션을 써야 할 상황에서 상속을 사용하는 건 내부 구현을 불필요하게 노출하는 꼴이다. 그 결과 API가 내부 구현에 묶이고 그 클래스의 성능도 영원히 제한된다. 더 심각한 문제는 클라이언트가 노출된 내부에 직접 접근할 수 있다는 점이다. 가장 심각한 문제는 클라이언트에서 상위 클래스를 직접 수정하여 하위 클래스의 `불변식`을 해칠 수 있다는 사실이다.

> 불변식을 해치는 사례는?

### 컴포지션 대신 상속을 사용하기로 결정하기 전에 마지막으로 자문해야 할 질문을 소개한다.

확장하려는 클래스의 API에 아무런 결함이 없는가? 결함이 있다면, 이 결함이 여러분 클래스의 API까지 전파돼도 괜찮은가? 컴포지션으로는 이런 결함을 숨기는 새로운 API를 설계할 수 있지만, 상속은 상위 클래스의 API를 '그 결함까지도'그대로 승계한다.

## 정리

상속은 강력하지만 캡슐화를 해틴다는 문제가 있다. 상속은 상위 클래스와 하위 클래스가 순수한 is-a 관계일 때도 안심할 수만은 없는 게, 하위 클래스의 패키지가 상위 클래스와 다르고, 상위 클래스가 확장을 고려해 설계되지 않았다면 여전히 문제가 될 수 있다. 상속의 취약점을 피하려면 상속 대신 컴포지션과 전달을 사용하자. 특히 래퍼 클래스로 구현할 적당한 인터페이스가 있다면 더욱 그렇다. 래퍼 클래스는 하위 클래스보다 견고하고 강력하다.

