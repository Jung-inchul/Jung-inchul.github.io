---
description: '자바 트러블슈팅: scouter를 활용한 시스템 장애 진단 및 해결 노하우를 부록 A을 요약한 내용입니다.'
---

# 부록 A. Fatal error log 분석

시스템을 운영하다 보면 가끔 JVM이 사라질 때가 있다. 그럴땐 JVM 치명 에러 로그\(Fatal error log\) 파일을 확인하면 실마리를 잡을 수 있다. 그 파일의 기본 이름은 hs\_err\_pidXXX.log의 형식으로 생성된다.

## 치명 에러 로그 파일

치명 에러 로그에서는 다음과 같은 정보를 제공한다.

| 섹션 구분 | 제공 정 |
| :--- | :--- |
| 헤더 섹션 | 동작 오류나 치명적 에러가 발생했을 때 수행된 시그널 정보 |
|  | 버전과 설정 정보 |
| 스레드 정보 섹션 | 치명적 에러를 발생시킨 스레드에 대한 자세한 정보와 레드의 스택 추적\(stack trace\) 정보 |
|  | 실행 중인 스레드의 목록과 상태 |
| 프로세스 정보 섹션 | 힙에 대한 종합 정보 |
|  | 로드된 네이티브 라이브러리의 목록 |
|  | 자바 실행 옵션 |
|  | 환경 변수 |
| 시스템 정보 섹션 | OS와 CPU에 대한 자세한 정보 |

### 헤더 섹션

헤더 섹션은 어떤 형태인지 예제를 통해서 알아보자

| 항 | 의 |
| :--- | :--- |
| SIGILL | 시그널 이름 |
| \(0x4\) | 시그널 번호 |
| at pc=0x0000002a95c0ecd2 | 프로그램 카운터 혹은 인스트럭션 포인터 |
| pid=15898 | 프로세스 아이디 |
| tid=1077070176 | 스레드 아이디 |

모든 에러 로그가 이와 같은 형태로 제공되는 것은 아니다. 치명 에러 로그의 헤더 부분을 통해서 해당 문제에 대한 가장 큰 단서를 찾아낼 수 있다. JVM이 충돌하는 원인을 다음과 같이 분류할 수 있다.

* 네이티브 코드에서 충돌이 발생하는 경우
* 스택이 꽉 차서\(stack overflow\) 충돌이 발생하는 경우
* 핫스폿 컴파일러 스레드에서 충돌이 발생하는 경우
* 컴파일된 코드에서 충돌이 발생하는 경우
* 가상 머신의 스레드\(VMThread\)에서 충돌이 발생하는 경우

## 치명 에러가 발생하면 뭐부터 봐야 할까?

가장 먼저 어느 작업을 하다가 이러한 오류가 발생했는지 확인해 봐야 한다. 다시 말해서 헤더 섹션에 있는 내용을 가장 먼저 살펴봐야 한다. 헤더 섹션에서도 '시그널 이름'이 있는 줄과 그 바로 아래에 있는 'Problematic freame'이라고 나와 있는 부분을 먼저 봐야 한다. 왜냐하면, 이 부분에JVM이 멈춘 이유가 명확히 나와 있기 때문이다. 어떻게 보면 다른 부분은 그냥 필요한 근거 자료일 뿐이다.

만약 문제의 원인이 JVM과 관련된 부분일 경우 JVM 버전에 따라 이슈가 발생하여 업그레이드가 필요할 경우나 다운 그레이드 해야 하는 경우가 있다. 그렇다고 이러한 문제가 반드시 JVM 때문이라고 할 수는 없다. 애플리케이션의 스레드가 부족하거나, 다른 여러 가지 이유의 코딩상의 문제나 실수로 인해서 오류가 발생하기도 한다. 따라서, 문제를 발생시킨 원인이 되는 스레드가 어떤 것인지를 확실히 짚어봐야만 정확한 원인을 찾을 수 있다.

## 참고

* [https://docs.oracle.com/en/java/javase/11/troubleshoot/general-java-troubleshooting.html](https://docs.oracle.com/en/java/javase/11/troubleshoot/general-java-troubleshooting.html)

