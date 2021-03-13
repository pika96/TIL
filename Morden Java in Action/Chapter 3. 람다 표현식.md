# 람다 표현식

## 목차
- [람다 표현식](#람다-표현식)
  - [목차](#목차)
  - [람다란 무엇인가?](#람다란-무엇인가)
    - [람다 특징](#람다-특징)
  - [람다 표현식은 어디에 사용하는가?](#람다-표현식은-어디에-사용하는가)
    - [함수형 인터페이스](#함수형-인터페이스)
  - [람다 활용 : 실행 어라운드 패턴](#람다-활용--실행-어라운드-패턴)
  - [함수형 인터페이스 사용](#함수형-인터페이스-사용)
  - [메서드 참조](#메서드-참조)
    - [메서드 참조를 만드는 방법](#메서드-참조를-만드는-방법)
    - [생성자 참조](#생성자-참조)
  - [람다, 메서드 참조 활용하기](#람다-메서드-참조-활용하기)
    - [1단계 : 코드 전달](#1단계--코드-전달)
    - [2단계 : 익명 클래스 사용](#2단계--익명-클래스-사용)
    - [3단계 : 람다 표현식 사용](#3단계--람다-표현식-사용)
    - [4단계 : 메서드 참조 사용](#4단계--메서드-참조-사용)
  - [람다 표현식 정리](#람다-표현식-정리)


## 람다란 무엇인가?
람다 표현식은 메서드로 전달할 수 있는 익명 함수를 단순화한 것이라고 할 수 있다.

<br>

### 람다 특징
- 익명 - 보통의 메서드와 달리 이름이 없으므로 익명이라 표현한다. 구현해야 할 코드에 대한 걱정거리가 줄어든다.
- 함수 - 람다는 메서드처럼 특정 클래스에 종속되지 않으므로 함수라고 부른다. 하지만 메서드처럼 파라미터 리스트, 바디, 반환 형식, 가능한 예외 리스트를 포함한다.
- 전달 - 람다 표현식을 메소드 인수로 전달하거나 변수로 저장할 수 있다.
- 간결성 - 익명 클래스처럼 많은 자질구레한 코드를 구현할 필요가 없다.

람다 표현식을 사용함으로써 앞서 2장에서 본 자질구레한 코드들을 간결하게 줄일 수 있다.

```
Comparator<Apple> byWeight = new Comparator<Apple>(){
            public int compare(Apple a1, Apple a2) {
                return a1.getWeight().compareTo(a2.getWeight());
            }
        };
```
```
Comparator<Apple> byWeight = (Apple a1, Apple a2) -> a1.getWeight().compareTo(a2.getWeight());
```
위의 두 코드를 비교하면 확실히 간결해 진것을 알 수 있다. 우리는 이런 비용과 가독성 등 여러가지 이득을 위해 람다를 꼭 공부해야한다.

람다 표현식은 아래 코드처럼 화살표를 기준으로 파라미터, 화살표, 바디로 이루어진다.
```
(Apple a1, Apple a2) -> a1.getWeight().compareTo(a2.getWeight());
```

<br>

## 람다 표현식은 어디에 사용하는가?
```
List<Apple> greenApples = filter(inventory, (Apple a) -> GREEN.equals(a.getColor()));
```
람다 표현식을 배웠으니 우리는 이를 써먹어야한다. 람다 표현식은 어디에, 어떻게 사용할 수 있는가? 앞서 공부한 2장에서 구현했던 필터 메서드에도 람다를 활용할 수 있었다. 또한 이제 소개할 함수형 인터페이스라는 문맥에서 람다 표현식을 사용할 수 있다.  위의 예제에서는 함수형 인터페이스 Predicate<T>를 기대하는 filter 메서드의 두 번째 인수로 람다 표현식을 전달한 것을 확인할 수 있다. 그렇다면 함수형 인터페이스는 무엇일까?

<br>

### 함수형 인터페이스
```
        public interface Predicate<T> {
            boolean test (T t);
        }
```
2장에서 만든 Predicate<T> 인터페이스를 통해 사과의 색깔, 무게를 필터링할 수 있었다. 바로 이 Predicate<T>가 함수형 인터페이스이다. 간단히 말해 함수형 인터페이스는 정확히 하나의 추상 메서드를 지정하는 인터페이스다.
함수형 인터페이스를 알아보았으니 이것을 가지고 무엇을 할 수 있을까? 앞서 언급한 람다 표현식으로 함수형 인터페이스의 추상 메서드 구현을 직접 전달할 수 있으므로 전체 표현식을 함수형 인터페이스의 인스턴스로 취급할 수 있다. 이와 비슷하게 조금 덜 깔끔하지만 익명 내부 클래스로도 같은 기능을 구현할 수 있다.

<br>

## 람다 활용 : 실행 어라운드 패턴
자원 처리에 사용하는 순환 패턴은 자원을 열고, 처리한 다음, 자원을 닫는 순서로 이루어진다. 설정과 정리 과정은 대부분 비슷하다. 즉, 실제 자원을 처리하는 코드를 설정과 정리 두 과정이 둘러싸는 형태를 갖는다. 이러한 것을 실행 어라운드 패턴이라고 부른다.

<br>

## 함수형 인터페이스 사용
자바 8 라이브러리 설계자들은 `java.util.function` 패키지로 여러 가지 새로운 함수형 인터페이스를 제공한다.
|함수형 인터페이스|함수 디스크립터|기본형 특화|
|---|---|---|
|Predicate<T>|T->boolean|IntPredicate, LongPredicate, DoublePredicate|
|Consumer<T>|T->void|IntConsumer, LongConsumer, DoubleConsumer|
|function<T, R>|T->R|IntFunction<R>, IntToDoubleFunction, IntToLongFunction...|

표에 나타난 3가지 이외에도 자바 8 이후에서는 Supplier<T>, UnaryOperator<T> 등 여러 가지 함수형 인터페이스를 제공한다.

<br>

## 메서드 참조
메서드 참조를 이용하면 기존의 메서드 정의를 재활용해서 람다처럼 전달할 수 있다. 이번 우테코에서 미션들을 진행하면서 `LottoGenerate::Lotto` 와 같은 메서드들을 접하였는데 책에서 접하니 더 이해하기 쉽고 친숙했다.
그렇다면 이 메서드 참조는 왜 중요할까? 메서드 참조는 특정 메서드만을 호출하는 람다의 축약형이라고 생각할 수 있다. 이때 명시적으로 메서드명을 참조함으로써 가독성을 높일 수 있다. 메서드 참조는 앞서 언급했던 것처럼 메서드명 앞에 구분자(::)를 뭍이는 방식으로 메서드 참조를 활용할 수 있다. 예를 하나 더 들자면 `Apple::getWeight`는 Apple 클래스에 정의된 getWeight의 메서드 참조다. 이것은 람다 표현식 `(Apple a) -> a.getWeight()`를 축약한 것이다.

### 메서드 참조를 만드는 방법
메서드 참조는 세 가지 유형으로 구분할 수 있다.
1. 정적 메서드 참조 - Integer의 parseInt 메서드는 `Integer::parseInt`로 표현할 수 있다.
2. 다양한 형식의 인스턴스 메서드 참조 - String의 length 메서드는 `String::length`로 표현할 수 있다.
3. 기존 객체의 인스턴스 메서드 참조 - Transcation 객체를 할당받은 expensiveTranscation 지역 변수가 있고, Transcation 객체에는 getValue 메서드가 있다면, 이를 `expensiveTransaction::getValue`라고 표현할 수 있다.

책에서는 3가지로 구분하였지만 쓰는 방식은 비슷한 것 같다.
기존의 메서드 구현을 재활용해서 메서드 참조를 만드는 방법을 살펴 보았는데, 클래스의 생성자를 이용하는 방법도 있다.

### 생성자 참조
ClassName::new처럼 클래스명과 new 키워드를 이용해서 기존 생성자의 참조를 만들 수 있다.
```
Supplier<Apple> c1 = () -> new Apple();
Apple a1 = c1.get();
```
```
Supplier<Apple> c1 = Apple::new;
Apple a1 = c1.get();
```
위의 코드처럼 람다 표현식을 생성자 참조로 바꿀 수 있다.

<br>

## 람다, 메서드 참조 활용하기
앞서 배운 람다 표현식과 메서드 참조를 활용하여 사과 리스트 정렬 문제를 간단한 코드로 만들 수 있다.

### 1단계 : 코드 전달
```
        public class AppleComparator implements Comparator<Apple> {
            public int compare(Apple a1, Apple a2) {
                return a1.getWeight.compareTo(a2.getWeight());
            }
        }
        inventory.sort(new AppleComparator());
```
sort 메서드는 다음과 같다 . 
`void sort(Comparator<? super E> c)`
이 메서드는 Comparator 객체를 인수로 받아 두 대상을 비교한다. 이 sort를 활용하여 1단계 코드를 완성 시킨 것을 볼 수 있다.

### 2단계 : 익명 클래스 사용
한 번만 사용할 Comparator를 1단계 처럼 구현하는 것 보다는 익명 클래스를 이용하는 것이 좋다.
```
        inventroy.sort(new Comparator<Apple>()){
            public int compare(Apple a1, Apple a2){
                return a1.getWeight().compareTo(a2.getWeight());
            }
        }
```

### 3단계 : 람다 표현식 사용
2단계 코드는 아직 코드의 부피가 커다란 편이다. 앞서 배운 람다를 활용하여 코드를 더욱 간단히 리펙토링 할 수 있다.
```
inventory.sort(comparing(apple -> apple.getWeight()));
```

### 4단계 : 메서드 참조 사용
마지막으로 메서드 참조를 활용하면 더 깔끔하게 전달할 수 있다.
```
inventory.sort(comparing(Apple::getWeight));
```
코드의 길이도 짧아지고 코드의 의미 또한 명확해진 것을 볼 수 있다. 코드 자체로 'Apple을 weight별로 비교해서 inventory에 sort하라'라는 의미를 전달할 수 있다. 1단계와 비교해보면 엄청난 변화가 이루어 진 것을 확인할 수 있다.

<br>

## 람다 표현식 정리
- 람다 표현식은 익명 함수의 일종이다. 이름은 없지만, 파라미터 리스트, 바디, 반환 형식을 가지며 예외를 던질 수 있다
- 람다 표현식으로 간결한 코드를 구현할 수 있다.
- 함수형 인터페이스는 하나의 추상 메서드만을 정의하는 인터페이스다.
- 함수형 인터페이스를 기대하는 곳에서만 람다 표현식을 사용할 수 있다.
- 람다 표현식을 이용해서 함수형 인터페이스의 추상 메서드를 즉석으로 제공할 수 있으며 람다 표현식 전체가 함수형 인터페이스의 인스턴스로 취급된다.
- java.util.function 패키지는 Predicate<T>, Function<T, R>, Supplier<T>, Consumer<T>, BinaryOperator<T> 등을 포함해서 자주 사용하는 다양한 함수형 인터페이스를 제공한다.
- 자바 8은 Predicate<T>와 Function<T, R> 같은 제네릭 함수형 인터페이스와 관련한 박싱 동작을 피할 수 있는 IntPredicate, IntToLongFunction 등과 같은 기본형 특화 인터페이스도 제공한다.
- 실행 어라운드 패턴을 람다와 활용하면 유연성과 재사용성을 추가로 얻을 수 있다.
- 람다 표현식의 기대 형식을 대상 형식이라고한다.
- 메서드 참조를 이용하면 기존의 메서드 구현을 재사용하고 직접 전달할 수 있다.
- Comparator, Predicate, Function 같은 함수형 인터페이스는 람다 표현식을 조합할 수 있는 다양한 디폴트 메서드를 제공한다.