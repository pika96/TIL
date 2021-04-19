# 동적 파라미터화

## 목차
- [동적 파라미터화](#동적-파라미터화)
  - [목차](#목차)
  - [자주 바뀌는 요구사항](#자주-바뀌는-요구사항)
  - [동적 파라미터화의 장점](#동적-파라미터화의-장점)
  - [변화하는 요구사항에 대응하기 (사과)](#변화하는-요구사항에-대응하기-사과)
    - [첫 번째 시도 : 녹색 사과 필터링](#첫-번째-시도--녹색-사과-필터링)
    - [두 번째 시도 : 색을 파라미터화](#두-번째-시도--색을-파라미터화)
    - [세 번째 시도 : 가능한 모든 속성으로 필터링](#세-번째-시도--가능한-모든-속성으로-필터링)
    - [네 번째 시도 : 추상적 조건으로 필터링](#네-번째-시도--추상적-조건으로-필터링)
  - [복잡한 과정 간소화](#복잡한-과정-간소화)
    - [다섯 번째 시도 : 익명 클래스 사용](#다섯-번째-시도--익명-클래스-사용)
    - [여섯 번째 시도 : 람다 표현식 사용](#여섯-번째-시도--람다-표현식-사용)

## 자주 바뀌는 요구사항
프로그래밍을 하며 항상 바뀌는 고객의 니즈를 위해 우리는 그에 맞추어 대비해야한다.
이 책의 예를 인용하자면, 농부가 녹색 사과를 찾고 싶다는 요구사항이 다음날이면 150그램 이상인 사과를 찾고 싶다는 요구사항으로 바뀔 수 있다는 것이다. 이번 Chapter 2에서 소개하는 동작 파라미터화를 통해 이에 대한 문제를 효과적으로 대응할 수 있다.

<br>

## 동적 파라미터화의 장점
 - 리스트의 모든 요소에 대해서 '어떤 동작'을 수행할 수 있음
 - 리스트 관련 작업을 끝낸 다음에 '어떤 다른 동작'을 수행할 수 있음
 - 에러가 발생하면 '정해진 어떤 다른 동작'을 수행할 수 있음

## 변화하는 요구사항에 대응하기 (사과)
이 책에서는 독자들로 하여금 사과를 예시로 들어 코드를 한 단계씩 발전시켜 설명한다. 복잡하고 중복되는 코드에서 점점 간단해지는 코드로 바뀌는 것을 볼 수 있다.

### 첫 번째 시도 : 녹색 사과 필터링
```
public static List<Apple> filterGreenApples(List<Apple> inventory) {
        List<Apple> result = new ArrayList<>();
        for(Apple apple: inventory){
            if(GREEN.equals(apple.getColor())){
                result.add(apple);
            }
        }
        return result;
    }
```

다음 코드는 사과 리스트 중 녹색 사과를 반환하는 메소드이다. 여기서 농부는 녹색 사과 이외에도 빨간 사과 또한 필터링하고 싶어졌다. 메소드를 복사하여 GREEN을 RED로 바꾸는 방법 등 여러 방법이 있지만 색이 추가되면 적절하게 대응할 수 없다. 이런 상황에서는 반복되는 부분을 `추상화` 해준다.

### 두 번째 시도 : 색을 파라미터화

```
public static List<Apple> filterApplesByColor(List<Apple> inventory, Color color) {
        List<Apple> result = new ArrayList<>();
        for(Apple apple: inventory){
            if(apple.getColor().equals(color)){
                result.add(apple);
            }
        }
        return result;
    }
```

두 번째 코드는 보다싶이 색깔을 파라미터로 넘겨받아 여러가지 색깔에 맞추어 대응할 수 있다. 하지만 농부가 색 이외에도 무게를 기준으로 필터링하고 싶다면 무게 정보 또한 파라미터로 추가해야한다. 하지만 이렇게 되면 기존에 Color로 필터링하는 함수와 중복되는 단점이 있다.

### 세 번째 시도 : 가능한 모든 속성으로 필터링

```
public static List<Apple> filterApples(List<Apple> inventory, Color color, int weight, boolean flag) {
        List<Apple> result = new ArrayList<>();
        for(Apple apple: inventory){
            if((flag && apple.getColor().equals(color)) ||
                    (!flag && apple.getWeight() > weight))
                result.add(apple);
            }
        }
        return result;
    }
```

세 번째 시도는 모든 속성을 파라미터로 넘겨주고 flag로 구분하는 방법이다. 이 방법은 얼마 공부하지 않은 내가 봐도 최악의 방법이라고 생각이 된다. 무엇을 나타내는 것인지 알 수 없는 flag, 너무 많은 파라미터, 또한 요구사항이 추가되면 파라미터는 더 길어지게 될 것이고 이는 가독성을 떨어뜨린다.

### 네 번째 시도 : 추상적 조건으로 필터링

```
public static List<Apple> filterApples(List<Apple> inventory, ApplePredicate p) {
        List<Apple> result = new ArrayList<>();
        for(Apple apple: inventory){
            if(p.test(apple)){
                result.add(apple);
            }
        }
        return result;
    }
```

이번에는 `전략 디자인패턴(strategy design pattern)`을 사용한 `implements`를 통해 앞선 코드보다 더 나은 방법을 찾아내었다. ApplePredicate객체를 파라미터화하여 내부적으로 다양한 동작을 수행할 수 있는 것이다. 다른 요구사항이 추가되면 우리는 ApplePredicate를 implements 받는 클래스를 구현하여 요구사항을 구현할 수 있다.

<br>

## 복잡한 과정 간소화
앞서 소개한 `추상화`를 통해 문제를 해결한 방법은 성공한 것처럼 보이지만, 새로운 동작을 또 전달하려면 ApplePredicate 인터페이스를 구현하는 여러 클래스를 정의한 다음 인스턴스화를 해야한다는 단점이 있다. 우리는 이를 또 개선해야한다.

### 다섯 번째 시도 : 익명 클래스 사용

```
List<Apple> redApples = filterApples(inventory, new ApplePredicate() {
            public boolean test(Apple apple){
                return RED.equals(apple.getColor());
            }
        });
```

위의 코드는 익명 클래스로 빨간 사과를 구하는 코드이다. 익명 클래스는 유용하지만 반복되고 공간을 많이 차지한다는 점에서 아직 부족하다. 또한 책에서는 많은 프로그래머가 익숙하지 않다는 단점을 추가로 이야기 하고 있다.

### 여섯 번째 시도 : 람다 표현식 사용

```
List<Apple> result = filterApples(inventry, (Apple apple) -> RED.equals(apple.getColor()));
```

이번 모던 인 액션 자바에서 많이 다룰 자바 8, 9, 10 중 자바 8에서 등장한 람다 표현식을 사용하였다. 앞에서 우리가 지나온 코드보다 훨씬 간단해져 있다는 것을 알 수 있다.