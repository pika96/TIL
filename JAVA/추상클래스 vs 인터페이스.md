# 목차
- [목차](#목차)
  - [추상 클래스](#추상-클래스)

## 추상 클래스

추상 클래스는 상태와 구현 메서드를 가지기 때문에 강한 결합이 되어 있다.
그렇기 때문에 의존 관계를 쉽게 바꿀 수가 없다!
ex)
만약 추상 클래스 Animal이 있다고 치면
Animal animal = new Dog(); 를
Animal animal = new Cat(); 으로 바꾸기 어렵다! 왜냐? Dog와 Cat의 상태가 다르기 때문!

하지만 인터페이스는 구현체이기 때문에 바꾸기가 쉽다!! -> 왜냐? 모두 abstract이기 때문에 쉽게 바꿀 수 있음!

컴파임 레벨, 런타임 레벨