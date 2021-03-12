# 스트림 활용
스트림 API는 필터링, 슬라이싱, 매핑, 검색, 매칭, 리듀싱 등 다양한 방식으로 데이터 처리를 할 수 있다. 이번 챕터에서는 스트림의 다양한 연산들을 살펴보고 직접 사용하며 공부할 것이다.

[TOC]

<br>

## 필터링
스트림 인터페이스는 filter 메서드를 지원한다. filter 메서드는 프레디케이트를 인수로 받아 프레디케이트와 일치하는 모든 요소를 포함하는 스트림을 반환한다.
```
List<Dish> vegetarianMenu = menu.stream().filter(Dish::isVegetarian).collect(toList());
```
위의 코드처럼 모든 채식요리를 필터링해서 채식 메뉴를 만들 수 있다.

### 고유 요소 필터링 (중복 제거)
스트림은 고유 요소로 이루어진 스트림을 반환하는 distinct 메서드도 지원한다.
```
List<Integer> numbers = Arrays.asList(1, 2, 3, 4, 5, 6);
        numbers.stream().filter(i -> i % 2 ==0).distinct().forEach(System.out::println);
```
위의 코드는 리스트의 모든 짝수를 선택하고 중복을 필터링한다.

<br>

## 스트림 슬라이싱
자바 9에서는 스트림 요소를 효과적으로 선택할 수 있도록 takeWhile, dropWhile 두 가지 새로운 메서드를 지원한다.

### TAKEWHILE 활용
만약 메뉴에서 320 칼로리 이하의 요리를 선택하고 싶다면 filter를 사용할 수 있다. 하지만 이 방법은 전체 스트림을 반복하면서 각 요소에 프레디케이트를 적용하게 된다. 만약 List가 정렬되어 있다면, filter 대신 takeWhile 연산을 이용하여 처리할 수 있다.
```
List<Dish> slicedMenu1 = specialMenu.stream().takeWhile(dish -> dish.getCalories() < 320).collect(toList());
```
위의 코드처럼 takeWhile을 사용하여 320 칼로리가 넘는 요리가 나온다면 반복 작업을 중단한다.
그렇다면 나머지 요소들은 어떻게 반환할 수 있을까?

```
List<Dish> slicedMenu2 = specialMenu.stream().dropWhile(dish -> dish.getCalories() < 320).collect(toList());
```
dropWhile을 사용하게되면 반대 작업을 할 수 있다.

### 스트림 축소
스트림은 주어진 값 이하의 크기를 갖는 새로운 스트림을 반환하는 limit(n) 메서드를 지원한다. 다음 코드는 300칼로리 이상의 세 요리를 선택해서 리스트를 반환하는 코드이다.
```
List<Dish> dishes = specialMenu.stream().filter(dish -> dish.getCalories() > 300).limit(3).collect(toList());
```

### 요소 건너뛰기
스트림은 처음 n개 요소를 제외한 스트림을 반환하는 skip(n) 메서드를 지원한다. 만약 n개 이하의 요소를 포함하는 스트림에 skip(n)을 호출하면 빈 스트림이 반환된다. 아래의 코드는 300칼로리 이상의 요리 중 처음 두 요리를 건너뛴 다음 300 칼로리가 넘는 나머지 요리를 반환한다.
```
List<Dish> dishes = menu.stream().filter(d -> d.getCalories() > 300).skip(2).collect(toList());
```

<br>

## 매핑

### 스트림의 각 요소에 함수 적용하기
스트림은 함수를 인수로 받는 map 메서드를 지원한다. 인수로 제공된 함수는 각 요소에 적용되며 함수를 적용한 결과가 새로운 요소로 매핑된다. map메서드 끼리 연결 할 수 있다.
```
List<String> dishNames = menu.stream().map(Dish::getName).collect(toList());
```
위의 코드처럼 map 메서드를 통해 요리의 이름을 추출할 수 있다.

### 스트림 평면화(flatMap)
flatMap을 이해하기 전에 예제를 하나 들어 보겠다.
리스트에서 고유 문자로 이루어진 리스트를 반환해보자. 예를들어 ["Hello", "World"] 리스트가 있다면 결과로 ["H", "e", "l", "o", "W", "r", "d"]를 포함하는 리스트가 반환되어야 한다.
앞서 배운 distinct()를 활용하여 중복된 문자를 필터링하여 해결한다는 생각을 가질 수도 있다.
```
words.stream().map(word -> word.split("")).distinct().collect(toList());
```
위 코드에서 map으로 전달한 람다는 각 단어의 String[]을 반환한다는 점이 문제다. 따라서 map 메소드가 반환한 스트림의 형식은 Stream<String[]>이다. 우리가 원하는 것은 문자열 스트림을 표현할 Stream<String> 이다. 우리는 이 문제는 flatMap 메서드를 이용하여 해결할 수 있다.

flatMap을 사용하면 다음처럼 문제를 해결할 수 있다.
```
List<String> uniqueCharacters = words.stream().map(word -> word.split("")).flatMap(Arrays::stream).distinct().collect(toList());
```
flatMap은 각 배열을 스트림이 아니라 스트림의 콘텐츠로 매핑한다. 즉, map과 달리 flatMap은 하나의 평면화된 스트림을 반환한다. 요약하자면 flatMap은 스트림의 각 값을 다른 스트림으로 만든 다음 모든 스트림을 하나의 스트림으로 연결하는 기능을 수행한다.

<br>

## 검색과 매칭
특정 속성이 데이터 집합에 있는지 여부를 검색하는 데이터 처리도 자주 사용된다. 스트림 API는 allMatch, anyMatch, noneMatch, findFirst, findAny 등 다양한 메서드를 제공한다.

### anyMatch
프레디케이트가 적어도 한 요소와 일치하는지 확인
```
if (menu.stream().anyMatch(Dish::isVegetarian)) {
            System.out.println("This menu is vegetarian");
        }
```
anyMatch는 불리언을 반환하므로 최종연산이다.

### allMatch
프레디케이트가 모든 요소와 일치하는지 검사
```
boolean isHealty = menu.stream().allMatch(dish -> dish.getCalories() < 1000);
```

### noneMatch
noneMatch는 allMatch와 반대 연산을 수행한다.
```
boolean isHealty = menu.stream().noneMatch(dish -> dish.getCalories() >= 1000);
```
anyMatch, allMatch, noneMatch 세 메서드는 스트림 쇼트 서킷기법, 즉 자바의 && || 와 같은 연산을 활용한다.

<br>

## 요소 검색
요소 검색 메서드들을 살펴보기 전에 Optional부터 알아보겠다.
Optional<T> 클래스는 값의 존재나 부재 여부를 표현하는 컨테이너 클래스이다. Optional은 값이 존재하는지 확인하고 값이 없을 때 어떻게 처리할지 강제하는 기능을 제공한다.
isPresent(), ifPresent, T get(), T orElse(T other)
```
menu.stream().filter(Dish::isVegetarian).findAny().ifPresent(dish -> System.out.println(dish.getName());
```

### findAny
findAny 메서드는 현재 스트림에서 임의의 요소를 반환한다.
```
Optional<Dish> dish = menu.stream().filter(Dish::isVegetarian).findAny();
```

### findFirst
스트림은 리스트의 첫 번째 요소를 찾을 수 있도록  findFirst 메서드를 지원한다.
```
List<Integer> someNumbers = Arrays.asList(1, 2, 3, 4, 5);
Optional<Integer> firstSquareDivisibleByThree = somNumbers.stream().map(n -> n * n).filter(n -> n % 3 == 0).findFirst(); //9
```
위의 코드는 숫자 리스트에서 3으로 나누어 떨어지는 첫 번째 제곱값을 반환하는 코드이다.

### reduce
reduce는 스트림에서 누적값을 연산하는 메서드이다. reduce를 이용하여 sum, max, min 등 다양한 연산을 구현할 수 있다.
또한 초기값의 유무에 따라 Optional<T>를 반환하는지가 결정된다.
아래는 reduce로 구현한 sum, max, min의 예제이다.
```
Optional<Integer> sum = numbers.stream().reduce(Integer::sum);
Optional<Integer> max = numbers.stream().reduce(Integer::max);
Optional<Integer> min = numbers.stream().reduce(Integer::min);
```

<br>

## 숫자형 스트림
스트림은 효율적으로 처리할 수 있도록 기본형 특화 스트림을 지원한다.
IntStream, DoubleStream, LongStream이 그 예이다.

앞선 예제인 ``menu.stream().map(Dish::getCalories).reduce(0, Integer::sum)``에서 사실 박싱 비용이 숨어있다. 내부적으로 합계를 계산하기전 Integer를 기본형으로 언박싱해야한다. 이런점을 효율적으로 하기 위해 기본형 특화 스트림을 지원한다.

<br>

## 스트림 만들기

### 값으로 스트림 만들기
임의의 수를 인수로 받는 정적 메서드 Stream.of를 이용해서 스트림을 만들 수 있다.
```
Stream<String> stream = Stream.of("Modern", "Java", "In", "Action");
```
또한 다음처럼 empty 메서드를 이용하여 스트림을 비울 수 있다.
```
Stream<String> emptyStream = Stream.empty();
```

### null이 될 수 있는 객체로 스트림 만들기
자바 9에서는 null이 될 수 있는 개체를 스트림으로 만들 수 있는 새로운 메소드가 추가되었다.
```
Stream<String> homeValueStream = Stream.ofNullable(System.getProperty("home"));
```

### 배열로 스트림 만들기
배열을 인수로 받는 정적 메서드 `Arrays.stream`을 이용해서 스트림을 만들 수 있다.
```
int[] numbers = {2, 3, 5, 7, 11, 13};
int sum = Arrays.stream(numbers).sum();
```

### 파일로 스트림 만들기
파일을 처리하는 등의 I/O 연산에 사용하는 자바의 API도 스트림 API를 활용할 수 있도록 업데이트 되었다.

### 함수로 무한 스트림 만들기
스트림 API는 함수에서 스트림을 만들 수 있는 두 정적 메서드 `Stream.iterate`와 `Stream.generate`를 제공한다. 이 메서드들을 이용해서 크기가 고정되지 않은 스트림을 만들 수 있다. 하지만 보통 무한한 값을 출력하지 않도록 limit(n)함수를 함께 연결해서 사용한다.
iterate 메서드 예제
```
Stream.iterate(0, n -> n+ 2).limit(10).forEach(System.out::println);
```
generate 메서드 예제
```
Stream.generate(Math::random).limit(5).forEach(System.out::println);
```
iterate와 generate의 차이점은 iterate는 생산된 값을 연속적으로 계산할 수 있고 generate는 새로운 값을 생성한다는 차이점이 있다.

<br>

## 마치며

- 스트림 API를 이용하면 복잡한 데이터 처리 질의를 표현할 수 있다.
- filter, distinct, takeWhile, dropWhile, skip, limit 메서드로 스트림을 필터링하거나 자를 수 있다.
- 소스가 정렬되어 있다는 사실을 알고 있을 때 takeWhile과 dropWhile 메소드를 효과적으로 사용할 수 있다.
- map, flatMap 메서드로 스트림의 요소를 추출하거나 변환할 수 있다.
- findFirst, findAny 메서드로 스트림의 요소를 검색할 수 있다. allMatch, noneMatch, anyMatch 메서드를 이용해서 주어진 프레디케이트와 일치하는 요소를 스트림에서 검색할 수 있다.
- 이들 메서드는 쇼트서킷, 즉 결과를 찾는 즉시 반환하며, 전체 스트림을 처리하지 않는다.
- reduce 메서드로 스트림의 모든 요소를 반복 조합하며 값을 도출할 수 있다. 예를 들어 reduce로 최댓값이나 모든 요소의 합계를 계산할 수 있다.
- filter, map등은 상태를 저장하지 않는 상태 없는 연산이다. reduce같은 연산은 값을 계산하는데 필요한 상태를 저장한다. sorted, distinct 등의 메서드는 새로운 스트림을 반환하기에 앞서 스트림의 모든 요소를 버퍼에 저장해야한다. 이런 메서드를 상태 있는 연산이라고 부른다.
- IntStream, DoubleStream, LongStream은 기본형 특화 스트림이다. 이들 연산은 각각의 기본형에 맞게 특화되어 있다.
- 컬렉션뿐 아니라 값, 배열, 파일, iterate와 generate 같은 메서드로도 스트림을 만들 수 있다.
- 무한한 개수의 요소를 가진 스트림을 무한 스트림이라 한다.