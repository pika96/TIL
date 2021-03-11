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
    - [removeIf](#removeif)
    - [replaceAll 메서드](#replaceall-메서드)
  - [맵 처리](#맵-처리)
    - [forEach 메서드](#foreach-메서드)
    - [정렬 메서드](#정렬-메서드)
    - [getOrDefault 메서드](#getordefault-메서드)
    - [계산 패턴](#계산-패턴)
    - [삭제 패턴](#삭제-패턴)
    - [교체 패턴](#교체-패턴)
    - [합침](#합침)
  - [개선된 ConcurrentHashMap](#개선된-concurrenthashmap)
    - [리듀스와 검색](#리듀스와-검색)
    - [계수](#계수)
    - [집합뷰](#집합뷰)
    - [마치며](#마치며)
  - [출처](#출처)

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

<br>

## 리스트와 집합 처리

### removeIf

removeIf : 프레디케이트를 만족하는 요소를 제거한다.

ex) 숫자로 시작되는 참조 코드를 가진 트랜잭션을 삭제하는 코드
```java
for (Transaction transaction : transactions) {
    if (Character.isDigit(transaction.getReferenceCode().charAt(0))) {
        transactions.remove(transaction);
    }
}
```
-> __ConcurrentModificationException__ 발생!

```java
for(Iterator<Transaction> iterator = transactions.iterator();
    iterator.hasNext();) {
      Transaction transaction = iterator.next();
      if(Character.isDigit(transaction.getReferenceCode().charAt(0))) {
        trasactions.remove(transaction);
        // -> iterator.remove(); Iterator 객체를 명시적으로 사용
      }
    }
```
반복자의 상태와 컬렉션의 상태가 서로 동기화 되지 않아 발생한다.
두 개의 개별 객체가 컬렉션을 관리한다!
- Iterator 객체, next(), hasNext()를 이용해 소스를 질의한다.
- Collection 객체 자체, remove()를 호출해 요소를 삭제한다.

```java
transactions.removeIf(transaction -> Character.isDigit(transcation.getReferenceCode().charAt(0)));
```
removeIf를 사용해서 간단하게 나타낼 수 있다.
removeIf 메서드는 삭제할 요소를 가리키는 프레디 케이트를 인수로 받는다.

### replaceAll 메서드

replaceAll : 리스트에서 이용할 수 있는 기능으로 UnaryOperator 함수를 이용해 요소를 바꾼다.

UnaryOperator<T>란? : : java.util.function.Function<T,R> 에서 확장한 람다식으로서, <T>형태의 입력값을 받아 <T>형태의 출력값을 리턴한다.

<br>

ex) 첫 번째 문자를 대문자로 변경
스트림 API를 사용하는 방법
```java
referenceCodes.stream()
              .map(code -> Character.toUpperCase(code.charAt(0)) +
                      code.substring(1))
              .collect(Collectors.toList())
              .forEach(System.out::println);
```
기존의 컬렉션을 바꾸는 것이 아닌 새로운 컬렉션을 만든다.
우리가 원하는 것은 기존 컬렉션을 바꾸는 것!

<br>

ListIterator 객체 사용
```java
for(ListIterator<String> iterator = referenceCodes.listIterator();
    iterator.hasNext();) {
      String code = iterator.next();
      iterator.set(Character.toUpperCase(code.charAt(0) + code.substring(1)));
    }
```
이전에 리스트 예제처럼 컬렉션 객체를 Iterator 객체와 혼용하면 반복과 컬렉션 변경이 동시에 이루어지면서 문제가 일어날 수 있다.

```java
referenceCodes.replaceAll(code -> Character.toUpperCase(code.charAt(0)) + code.substring(1));
```
자바 8에서 지원하는 replaceAll을 이용하여 쉽게 구현할 수 있다.

<br>

## 맵 처리

### forEach 메서드

맵에 저장되어 있는 이름과 나이 출력
```java
for(Map.Entry<String, Integer> entry: ageOfFriends.entrySet()){
  String friend = entry.getKey();
  Integer age = entry.getValue();
  System.out.println(friend + " is " + age + " years old");
}
```
자바 8에서부터 Map 인터페이스는 BiConsumer를 인수로 받는 forEcah 메서드를 지원한다.
```java
ageOfFriends.forEach((friend, age) -> System.out.println(friend + " is " + age + " years old"));
```
### 정렬 메서드
- Entry.comparingByValue
- Entry.comparingByKey

```java
Map<String, String> favoriteMovies
                = Map.ofEntries(entry("Raphael", "Star Wars"),
                entry("Cristina", "Matrix"),
                entry("Olivia", "James Bond"));

favoriteMovies.entrySet()
              .stream()
              .sorted(Map.Entry.comparingByKey())
              .forEachOrdered(System.out::println);
```
Value 순서대로 정렬

### getOrDefault 메서드
요청한 키가 맵에 존재하지 않을 때 처리할 수 있는 메서드
```java
default V getOrDefault(Object key, V defaultValue)
```
첫 번째 인수로 키를, 두 번째 인수로 기본 값을 받아 맵에 키가 존재하지 않을 경우 두 번째 인수로 받은 기본 값을 반환한다.
```java
Map<String, String> favoriteMovies
                = Map.ofEntries(entry("Raphael", "Star Wars"),
                entry("Cristina", "Matrix"),
                entry("Olivia", "James Bond"));

System.out.println(favoriteMovies.getOrDefault("Olivia", "Matrix"));
// -> James Bond 출력
System.out.println(favoriteMovies.getOrDefault("Olivia", "Matrix"));
// -> Matrix 출력
```

### 계산 패턴

맵에 키가 존재하는 지 여부에 따라 어떤 동작을 실행하고 결과를 저장할 수 있다.
- computeIfAbsent : 제공된 키에 해당하는 값이 없으면(값이 없거나 널), 키를 이용해 새 값을 계산하고 맵에 추가한다.
```java
map.put("A", 5);
map.computeIfAbsent("A", key -> 10);
System.out.println(map.get("A")); // result : 5
```
- computeIfPresent : 제공된 키가 존재하면 새 값을 계산하고 맵에 추가한다.

```java
map.put("A", 5);
System.out.println(map.computeIfPresent("A", (s, number) -> number*number)); 
// result : 25
System.out.println(map.computeIfPresent("B", (s, number) -> 10));
// result : null
```
- compute : 제공된 키로 새 값을 계산하고 맵에 저장한다.
```java
map.put("A", 1);
System.out.println(map.compute("A", (s, integer) -> integer+1));
// result : 2
System.out.println(map);
// {A=2}
```

### 삭제 패턴
Map은 기존에 remove() 함수가 존재하지만, 자바 8에서는 키가 특정한 값과 연관되었을 때만 항목을 제거하는 오버로드 버전의 메서드를 제공한다.
```java
boolean remove(Object key, Object value)
```
기존의 remove(Object key)는 단순히 해당 키의 값을 삭제했지만 위 메서드는 key와 value가 둘다 일치할 경우에만 삭제된다. 같지 않을 경우 false같고 삭제됬을 경우에는 true를 반환한다.

### 교체 패턴
맵 항목을 바꾸는데 사용할 수 있는 두 개의 메서드가 맵에 추가되었다.
- replaceAll : Bifunction을 적용한 결과로 각 항목의 값을 교체한다. 이 메서드는 이전에 살펴본 List의 replaceAll과 비슷한 동작을 수행한다.

```java
Map<String, String> favoriteMovies = new HashMap<>();
// -> replaceAll을 적용하기 때문에 Map.ofEntries 사용 X

favoriteMovies.put("Raphael", "Star Wars");
favoriteMovies.put("Olivia", "james bond");
favoriteMovies.replaceAll((friends, movie) -> movie.toUpperCase());
System.out.println(favoriteMovies);
// result : {Olivai=JAMES BOND, Raphael=STAR WARS}
```
- Replace : 키가 존재하면 맵의 값을 바꾼다. 키가 특정 값으로 매핑되었을 때만 값을 교체하는 오버로드 버전도 있다.

```java
V replace(K key, V newValue)

map.put("A", 5);
System.out.println(map.replace("A", 1));
System.out.println(map);
//result : 5 {A=1}
```
```java
boolean replace(K key, V oldValue, V newValue)

map.put("A", 5);
System.out.println(map.replace("A", 1, 10));
System.out.println(map);
//result : false {A=5}
```
삭제와 동일하다 oldValue가 맞지 않으면 교체되지 않으며 false를 반환한다.

### 합침
putAll 메서드를 사용하여 두 그룹의 연락처를 포함하는 두 개의 맵 합치기
```java
Map<String, String> family = Map.ofEntries(
  entry("Teo", "Star Wars"), entry("Cristina", "James Bond"));
Map<String, String> friends = Map.ofEntries(
  entry("Raphael", "Star Wars"));
Map<String, String> everyone = new HashMap<>(family);
everyone.putAll(friends);
```
중복된 키가 없다면 위 코드는 잘 동작한다. 값을 좀 더 유연하게 합쳐야 한다면 새로운 merge 메서드를 이용할 수 있다.
merge 메서드는 다음과 같은 인자를 받는다.
- 첫 번째 인자는 key를 받는다.
- 두 번째 인자는 key가 존재하지 않을 경우 삽입될 value를 받는다.
- 세 번째 인자는 key가 존재할 경우 remapping할 BiFunction을 받는다.

즉, 첫 번째 인자인 key가 맵에서 존재하지 않으면 두 번째 인자와 함께 map에 삽입되고
key가 맵에서 존재하면 세 번째 인자인 BiFunction의 연산 결과를 value에 remapping한다.

Key가 맵에서 존재할 때 BiFunction의 연산 결과가 null이면 map에서 삭제된다.
```java
public static void example() {
    Map<String, Integer> map = new HashMap<>();
    map.put("사과", 1);

    map.merge("사과", 1, (value, putValue) -> value + 1);
    map.merge("딸기", 1, (value, putValue) -> value + 1);
    map.merge("포도", 1, (value, putValue) -> value + 1);
    map.merge("포도", 1, (value, putValue) -> map.get("수박"));

    System.out.println(map);
    // 결과 : {사과=2, 딸기=1}
}
```
## 개선된 ConcurrentHashMap
ConcurrentHashMap 클래스는 동시성 친화적이며 최신 기술을 반영한 HashMap 버전이다. ConcurrentHashMap은 내부 자료구조의 특정 부분만 잠궈 동시 추가, 갱신 작업을 허용한다. 따라서 동기화된 Hashtable 버전에 비해 읽기 쓰기 연산 성능이 월등하다. (표준 HashMap은 비동기로 동작)

### 리듀스와 검색
스트림과 비슷한 3가지 연산
- forEach : 각 (키, 값) 쌍에 주어진 액션을 실행
- reduce : 모든 (키, 값) 쌍을 제공된 리듀스 함수를 이용해 결과로 합침
- search : 널이 아닌 값을 반환할 때 까지 각 (키, 값) 쌍에 함수를 적용

네 가지 연산 형태 지원
- 키, 값으로 연산(forEach, reduce, search)
- 키로 연산(forEachKey, reduceKeys, searchKeys)
- 값으로 연산(forEachValue, reduceValues, searchValues)
- Map.Entry 객체로 연산(forEachEntry, reduceEntries, searchEntries)

### 계수
ConcurrentHashMap 클래스는 맵의 매핑 개수를 반환하는 mappingCount 메서드를 제공한다. 기존의 size 대신 int를 반환하는 mappingCount를 사용하는 것이 좋다.

### 집합뷰
keySet을 이용하여 ConcurrentHashMap을 집합 뷰로 반환할 수 있다.
맵을 바꾸면 집합도 바뀌고 반대도 마찬가지이다. 
newKeySet을 새로운 메서드를 이용하여 ConcurrentHashMap으로 유지되는 집합을 만들 수도 있다.

### 마치며
- 자바 9는 적의 원소를 포함하며 바꿀 수 없는 리스트, 집합, 맵을 쉽게 만들 수 있도록 List.of, Set.of, Map.of, MapofEntries 등의 컬렉션 팩토리를 지원한다.
- 이들 컬렉션 팩토리가 반환한 객체는 만들어진 다음 바꿀 수 없다.
- List 인터페이스는 removeIf, replaceAll, sort 세 가지 디폴트 메서드를 지원한다.
- Set 인터페이스는 removeIf 디폴트 메서드를 지원한다.
- Map 인터페이스는 자주 사용하는 패턴과 버그를 방지할 수 있도록 다양한 디폴트 메서드를 지원한다.
- ConcurrentHashMap은 Map에서 상속받은 새 디폴트 메서드를 지원함과 동시에 스레드 안정성도 제공한다.

<br>

## 출처
출처: https://sticky32.tistory.com/entry/Java8-새로운-collection-api에-대해서 [Map 처리]

출처 : http://blog.breakingthat.com/2019/04/04/java-collection-map-concurrenthashmap/ [ConcurrentHashMap]