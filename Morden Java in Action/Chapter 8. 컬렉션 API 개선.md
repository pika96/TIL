# 컬렉션 API 개선

<br>

- [컬렉션 API 개선](#컬렉션-api-개선)
  - [작은 컬렉션 객체](#작은-컬렉션-객체)
    - [List](#list)
    - [Set](#set)
    - [Map](#map)
    - [컬렉션 리터럴](#컬렉션-리터럴)
  - [팩토리](#팩토리)
    - [리스트 팩토리](#리스트-팩토리)
    - [집합 팩토리](#집합-팩토리)
    - [맵 팩토리](#맵-팩토리)
    - [오버로딩 vs 가변 인수](#오버로딩-vs-가변-인수)
  - [리스트와 집합 처리](#리스트와-집합-처리)

<br>

## 작은 컬렉션 객체

### List
```java
List<String> friends = new ArrayList<>();
friends.add("Raphael");
friends.add("Olivia");
friends.add("Thibaut");
->
List<String> friends = Arrays.asList("Raphael", "Olivia", "Thibaut");
```
적은 요소를 포함하는 List 생성
고정 크기의 리스트를 만들었으므로 요소를 갱신할 순 있지만 새 요소를 추가하거나 요소를 삭제 불가 -> UnsupportedOperationException 예외 발생!

<br>

### Set
```java
Set<String> friends = new HashSet<>(Arrays.asList("Raphael", "Olivia", "Thibaut"));

Set<String> friends = Stream.of("Raphael", "Olivia", "Thibaut").collect(Collectors.toSet());
```
적은 요소를 포함하는 Set 생성
내부적으로 불필요한 객체 할당을 필요로 한다.

<br>

### Map
-> 딱히 없음

<br>

### 컬렉션 리터럴
Python, Groovy 등 일부 언어는 특별한 문법을 이용해 컬렉션을 만들 수 있다!
ex) Python
```python
fruits = ["apple", "mango", "orange"] #List
numbers = (1, 2, 3) #Tuple
alphabets = {'a':'apple', 'b':'ball', 'c':'cat'} #딕셔너리(사전)
vowels = {'a', 'e', 'i' , 'o', 'u'} #Set
```
But! 자바는 너무 큰 언어 변화와 관련된 비용이 든다는 이유로 지원하지 않는다.

<br>

## 팩토리

자바 9에서부터 작은 리스트, 집합, 맵을 쉽게 만들 수 있도록 팩토리 메서드를 제공한다.
### 리스트 팩토리
```java
List<String> friends = List.of("Raphael", "Olivia", "Thibaut");
```
`List.of`로 만들어진 리스트 또한 요소를 추가하거나 set() 메서드를 호출하게되면 변경할 수 없는 리스트가 만들어졌기 때문에 UnsupportedOperationException이 발생한다.

### 집합 팩토리
```java
Set<String> friends = Set.of("Raphael", "Olivia", "Thibaut");
```
중복된 요소를 집합으로 만들려고 하면 IllegalArgumentException 이 발생한다.

### 맵 팩토리
```java
Map<String, Integer> ageOfFriends = Map.of("Raphael", 30, "Olivia", 25, "Thibaut", 26);
or
import static java.util.Map.entry;
Map<String, Integer> ageOfFriends
                = Map.ofEntries(entry("Raphael", 30),
                                entry("Olivia", 25),
                                entry("Thibaut", 26));
```

Map.of() 는 열 개 이하의 키와 값 쌍을 가진 맵
Map.ofEntries()는 열 개 이상

<br>

### 오버로딩 vs 가변 인수
왜 가변인수를 사용하지 않고 다양한 오버로딩 버전이 있을까요?
```
static <E> List<E> of(E e1);
static <E> List<E> of(E e1, E e2);
static <E> List<E> of(E e1, E e2, E e3);
...
static <E> List<E> of(E... elements)
```
내부적으로 가변 인수 버전은 추가 배열을 할당해서 리스트로 감싼다. 따라서 배열을 할당하고 초기화하며 나중에 가비지 컬렉션을 하는 비용을 지불해야 한다.
-> __비용이 커져서 미리 만들었다.__

## 리스트와 집합 처리