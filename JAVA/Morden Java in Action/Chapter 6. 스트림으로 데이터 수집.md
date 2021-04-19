# 스트림으로 데이터 수집

## 목차
- [스트림으로 데이터 수집](#스트림으로-데이터-수집)
  - [목차](#목차)
  - [스트림과 컬렉터](#스트림과-컬렉터)
  - [컬렉터란 무엇인가?](#컬렉터란-무엇인가)
    - [미리 정의된 컬렉터](#미리-정의된-컬렉터)
  - [리듀싱과 요약](#리듀싱과-요약)
    - [스트림 값에서 최댓값 최소값 검색](#스트림-값에서-최댓값-최소값-검색)
    - [요약 연산](#요약-연산)
    - [문자열 연결](#문자열-연결)
    - [범용 리듀싱 요약 연산](#범용-리듀싱-요약-연산)
  - [그룹화](#그룹화)
    - [그룹화된 요소 조작](#그룹화된-요소-조작)
    - [다수준 그룹화](#다수준-그룹화)
    - [서브그룹으로 데이터 수집](#서브그룹으로-데이터-수집)
    - [컬렉터 결과를 다른 형식에 적용하기](#컬렉터-결과를-다른-형식에-적용하기)
  - [분할](#분할)
  - [Collector 인터페이스](#collector-인터페이스)
  - [마치며](#마치며)

<br>

## 스트림과 컬렉터
우리는 이 장에서 reduce처럼 collect를 사용하여 다양한 요소 누적 방식을 인수로 받아 최종 결과를 반환하는 방법을 소개할 것이다. 다양한 요소 누적 방식은 Collector 인터페이스에 정의되어 있는데 컬렉션, 컬렉터 collect를 헷갈리지 않도록 따라와야한다.

먼저 하나 예시를 들겠다.
1. 통화별로 트랙잭션을 그룹화한 다음 해당 통화로 일어난 모든 트랙잭션 합계를 계산
2. 트랜잭션을 비싼 트랜잭션과 저렴한 트랜잭션 두 그룹으로 분류
3. 트랜잭션을 도시 등 다수준으로 그룹화하시오. 그리고 각 트랜잭션이 비싼지 저렴한지 구분하시오

다음은 자바 8 이전 코드로 작성한 코드이다.
```
Map<Currency, List<Transcation>> transactionsByCurrencies = new HashMap<>();

        for (Transaction transaction : transactions) {
            Currency currency = transaction.getCurrency();
            List<Transcation> transactionsForCurrency = transactionsByCurrencies.get(currency);
            if(transactionsForCurrency == null){
                transactionsForCurrency = new ArrayList<>();
                transactionsByCurrencies.put(currency, transactionsForCurrency);
            }
            transactionsForCurrency.add(transaction);
        }
```
트랜잭션의 리스트를 반복하여 부른 후 같은 통화를 가진 트랜잭션 리스트에 현재 탐색 중인 트랜잭션을 추가한다.
만약 현재 같은 통화가 없으면 새로운 항목을 만든다.

위의 코드는 한눈에 알아보기도 어렵고 구현하기도 어렵다. 이를 collect 메서드를 활용하여 원하는 연산을 더욱 간결하게 구현할 수 있다.

```
Map<Currency, List<Transaction>> transactionsByCurrencies =
 transactions.stream().collect(groupingBy(Transaction::getCurrency));
```
앞선 코드와 비교했을 때 한눈에 알아보기도 쉽고 코드 또한 짧아 졌다.

<br>

## 컬렉터란 무엇인가?
앞선 예제를 보면 명령형 프로그래밍에 비해 함수형 프로그래밍이 얼마나 편리한지 명확하게 보여주고 있다.
예제에서 collect 메서드로 Collector 인터페이스 구현을 전달했다. Collector 인터페이스 구현은 스트림의 요소를 어떤 식으로 도출할지 지정한다. 이번에 볼 메서드는 groupingBy 메서드로 이를 이용하여, 각 키와 각 키에 대응하는 요소 리스트를 값으로 포함하여 맵을 만들어 낼 것이다.

### 미리 정의된 컬렉터
컬렉터에서 미리 정의된 groupingBy와 같은 Collectors 클래스에서 제공하는 팩토리 메서드의 기능을 살펴볼 것이다.
Collectors에서 제공하는 메서드의 기능은 크게 세 가지로 구분할 수 있다.
- 스트림 요소를 하나의 값으로 리듀스하고 요약
- 요소 그룹화
- 요소 분할

<br>

## 리듀싱과 요약
5장에서 다룬 요리 예제를 이용하여 살펴보겠다.

첫 번째 예로 counting()이라는 팩토리 메서드가 반환하는 컬렉터로 메뉴에서 요리 수를 계산해준다.
```
long howManyDishs = menu.stream().collect(Collectors.counting());
->
long howManyDishs = menu.stream().count();
```
위의 코드처럼 불필요한 과정을 생략할 수 있다.
Collectors 클래스를 임포트를 하게되면 앞선 예제처럼 Collectors.counting()을 간단하게 counting()으로 표현할 수 있다.

<br>

### 스트림 값에서 최댓값 최소값 검색
만약 메뉴에서 칼로리가 가장 높은 요리를 찾는다고 가정하자. Collectors에 정의되어 있는 maxBy(), minBy()를 이용하여 최댓값 최소값을 계산할 수 있다.
```
Comparator<Dish> dishCaloriesComparator = Comparator.comparingInt(Dish::getCalories);

Optional<Dish> mostCalorieDish = menu.stream().collect(maxBy(dishCaloriesComparator));
```

<br>

### 요약 연산
Collectors 클래스는 스트림에 있는 객체의 숫자 필드의 합계나 평균 등을 반환하는 연산 또한 지원한다.
summingInt, averagingInt 등 합, 평균값 계산 등 여러가지 연산이 지원된다.
```
int totalCalories = menu.stream().collect(summingInt(Dish::getCalories));
```
위의 코드는 메뉴 리스트의 합계를 계산하는 코드이다. 이처럼 Collectors 또한 스트림의 리듀싱처럼 여러가지 연산을 할 수 있다.

이러한 단순 합계외에 요소 수, 최댓값 최소값, 합계와 평균을 한꺼번에 계산하여 반환해주는 summarizingInt 또한 지원해준다.
```
IntSummaryStatistics menuStatistics = menu.stream().collect(summarizingInt(Dish::getCalories));
```
위의 코드를 실행하면 IntSummaryStatistics로 모든 정보가 수집된다. menuStaticstics 객체를 출력하면 다음과 같은 정보를 확인할 수 있다.
```
IntSummaryStatistics { count = 9, sum=4300, min=120, average=477.777778, max=800}
```
앞에서는 int만 설명했지만 이와 비슷한 Double, Long에 대한 클래스도 있다.

<br>

### 문자열 연결
컬렉터는 스트림의 각 객체를 연결하여 하나의 문자열로 연결하는 joining 팩토리 메서드를 지원한다.
```
String shortMenu = menu.stream().map(Dish::getName).collect(joining(", "));
```
위의 코드는 각 객체를 , 로 연결해주는 코드이다.

<br>

### 범용 리듀싱 요약 연산
지금까지 살펴본 모든 컬렉터는 reducing 팩토리 메서드로도 정의할 수 있다. 즉, 범용 Collectors.reducing으로도 구현할 수 있다.
```
int totalCalories = menu.stream().collect(reducing(0, Dish::getCalories, (i, j) -> i + j));
```
위의 코드는 reducing 메서드로 만들어진 컬렉터로 메뉴의 모든 칼로리 합계를 계산할 수 있다.
reducing은 인수 세 개를 받는다.
1. 첫 번째 인수는 리듀싱 연산의 시작값이거나 스트림에 인수가 없을 때는 반환값이다.
2. 두 번째 인수는 요리를 칼로리 정수로 변환할 때 사용한 변환 함수다.
3. 세 번째 인수는 같은 종류의 두 항목을 하나의 값으로 더하는 BinaryOperator다. 예제에서는 두 개의 int가 사용되었다.

<br>

## 그룹화
맨 처음 보았던 통화 그룹화 예제에서 확인 했듯이 명령형 프로그래밍으로 그룹화를 구현하려면 까다롭고, 할일이 많으며, 에러도 많이 발생한다. 하지만 자바 8의 함수형을 이용하면 가독성 있는 한 줄의 코드로 그룹화를 구현할 수 있다.

이번 코드는 요리를 고기, 생선, 나머지 그룹으로 그룹화하는 코드이다.
```
Map<Dish.Type, List<Dish>> dishesByType = menu.stream().collect(groupingBy(Dish::getType));
// result
{FISH =[prawns, salmon], OTHER={french fries, rice, season fruit, pizza], MEAT=[pork, beef, chicken]}
```
이처럼 groupingBy를 사용하여 원하는 그룹으로 분류할 수 있다. 따라서 이를 분류 함수라고 부른다.

위처럼 단순 속성 접근자 대신 더 복잡한 분류 기준이 필요한 상황에서는 메서드 참조를 사용할 수 없다.
다음은 400 칼로리 이하를 `diet`로 400~ 700칼로리를 `normal`로, 700 칼로리 초과를 `fat` 요리로 분류한다고 가정한다.
이 분류 기준을 메서드 참조대신 람다 표현식으로 나타낼 수 있다.
```
public enumCaloricLevel { DIET, NORMAL, FAT}

        Map<CaloricLevel, List<Dish>> dishesByCaloricLevel = menu.stream().collect(groupingBy(dish->{
            if(dish.getCalories() <= 400) return CaloricLevel.DIET;
            else if(dish.getCalories() <= 700) return CaloricLevel.NORMAL;
            else return CaloricLevel.FAT;
        }));
```

<br>

### 그룹화된 요소 조작
앞선 예제를 통해 groupBy 메서드를 이용하여 한 가지 기준으로 그룹화하는 방법을 살펴보았다. 그렇다면 2가지 이상의 기준으로 그룹화할 수 있을까?

우선 스트림의 filter 메서드를 사용하면 될 것이라고 생각할 수 있다.
```
Map<Dish.Type, List<Dish>> caloricDishesByType = 
                menu.stream().filter(dish -> dish.getCalories() > 500)
                             .collect(groupingBy(Dish::getType));
```
위 코드로 문제를 해결할 수 있지만 단점이 존재한다.
{ OTHER=[french fries, pizza], MEAT=[pork, beef]}
문제가 무엇인지 발견했는가? FISH는 filter를 만족하는 요소가 없으므로 FISH 항목이 아예 빠져있다.
이를 해결하기 위해 Collectors 클래스는 groupingBy 팩토리 메서드의 두 번째 인수를 갖는 오버로딩을 사용한다.
```
Map<Dish.Type, List<Dish>> caloricDishesByType =
        menu.stream()
            .collect(groupingBy(Dish::getType,
                    filtering(dish -> dish.getCalories() > 500, toList())));

// result
{OTHER=[french fries, pizza], MEAT=[pork, beef], FISH=[]}
```
결과를 보면 알수 있듯이 목록이 비어있는 FISH도 항목으로 추가된다.

<br>

### 다수준 그룹화
두 인수를 받는 팩토리 메서드 Collectors.groupingBy를 이용해서 항목을 다수준으로 그룹화할 수 있다. Collectors.groupingBy는 일반적인 분류 함수와 컬렉터를 인수로 받는다. 즉, 두 가지 기준으로 항목을 그룹화할 수 있다.
```
Map<Dish.Type, Map<CaloricLevel, List<Dish>>> dishesByTypeCaloricLevel =
                menu.stream().collect(
                        groupingBy(Dish::getType,
                                groupingBy(dish -> {
                                    if(dish.getCalories() <= 400)
                                        return CaloricLevel.DIET;
                                    else if(dish.getCalories() <= 700)
                                        return CaloricLevel.NORMAL;
                                    else
                                        return CaloricLevel.FAT;
                                })
                        })
                );
```
그룹화 결과로 다음과 같은 두 수준의 맵이 만들어 진다.
```
{MEAT={DIET=[chicken], NORMAL=[beef], FAT=[pork]}, FISH={DIET=[prawns], NORMAL=[salmon]},
OTHER={DIET=[rice, sasonal fruit], NORMAL=[french fries, pizza]}}
```
첫 번째 분류 함수에서는 `fish`, `meat`, `other`를 기준으로, 두 번째는 `normal`, `diet`, `fat`을 기준으로 그룹화 된다.

<br>

### 서브그룹으로 데이터 수집
groupingBy 컬렉터에 두 번째 인수로 counting 컬렉터를 전달해서 메뉴에서 요리 수를 종류별로 계산할 수 있다.
```
Map<Dish.Type, Long> typesCount = menu.stream().collect(groupingBy(Dish::getType, counting()));
// result
{MEAT=3, FISH=2, OTHER=4}
```
분류 함수 한 개의 인수를 갖는 groupingBy(f)는 사실 groupingBy(f, toList())의 축약형이다.

<br>

### 컬렉터 결과를 다른 형식에 적용하기
마지막 그룹화 연산에서 맵의 모든 값을 Optional로 감쌀 필요가 없으므로 Optional을 삭제할 수 있다. 즉, 다음처럼 팩토리 메서드 `Collectros.collectingAndThen`으로 컬렉터가 반환한 결과를 다른 형식으로 활용할 수 있다.

```
Map<Dish.Type, Dish> mostCaloricByType =
                menu.stream()
                    .collect(groupingBy(Dish::getType,
                            collectingAndThen(
                                    maxBy(comparingInt(Dish::getCalories)), 
                            Optional::get)));
```
팩토리 메서드 `collectingAndThen`은 적용할 컬렉터와 변환 함수를 인수로 받아 다른 컬렉터를 반환한다.

<br>

## 분할
분할은 분할 함수라 불리는 프레디 케이트를 분류 함수로 사용하는 특수한 그룹화 기능이다. 분할 함수는 불리언을 반환하므로 맵의 키 형식은 Boolean이다. 결과적으로 그룹화 맵은 최대 (true or false) 두 개의 그룹으로 분류된다.
예를 들어 채식인 요리와 아닌 요리로 구분하고 싶다면 다음과 같은 코드를 사용하면된다.
```
Map<Boolean, List<Dish>> partitioneMenu = menu.stream().collect(partitioningBy(Dish::isVegetarina));
// result
{false=[pork, beef, chicken, prawns, salmon],
true=[french fries, rice, season fruit, pizza]}
```
이제 참값의 키로 맵에서 모든 채식 요리를 얻을 수 있다.
```
List<Dish> vegetarianDishes = partitionedMenu.get(true);
```

<br>

## Collector 인터페이스
이번 장에서는 Collector 인터페이스를 직접 구현해서 더 효율적으로 문제를 해결하는 컬렉터를 만드는 방법을 살펴본다.
```
public interface Collector<T, A, R> {
            Supplier<A> supplier();
            BiConsumer<A, T> accumulator();
            Function<A, R> finisher();
            BinaryOperator<A> combiner();
            Set<Characteristics> characteristics();
        }
```
위의 코드는 Collector 인터페이스의 시그니처와 다섯 개의 메서드 정의를 보여준다. 이것은 다음처럼 설명할 수 있다.
- T는 수집될 스트림 항목의 제네릭 형식이다.
- A는 누적자, 즉 수집 과정에서 중간 결과를 누적하는 객체의 형식이다.
- R은 수집 연산 결과 객체의 형식(항상 그런 것은 아니지만 대개 컬렉션 형식)이다.

이 Collector 인터페이스의 5가지 메서드를 하나씩 살펴보자. 먼저 살펴볼 네 개의 메서드는 collect 메서드에서 실행하는 함수를 반환하는 반면, 다섯 번째 메서드 characteristics는 collect 메서드가 어떤 최적화를 이용해서 리듀싱 연산을 수행할 것인지 결정하도록 돕는 힌트 특성 집합을 제공한다.

- supplier 메서드 : 새로운 결과 컨테이너 만들기
- accumulator 메서드 : 결과 컨테이너에 요소 추가하기
- finisher 메서드 : 최종 변환값을 결과 컨테이너로 적용하기
- combiner 메서드 : 두 결과 컨테이너 병합
- Characteristics 메서드 : 컬렉터의 연산을 정의하는 Characteristics 형식의 불변 집합을 반환한다.

이를 이용하여 커스텀 컬렉터를 개발할 수 있다.

<br>

## 마치며
- collect는 스트림의 요소를 요약 결과로 누적하는 다양한 방법(컬렉터라 불리는)을 인수로 갖는 최종 연산이다.
- 스트림의 요소를 하나의 값으로 리듀스하고 요약하는 컬렉터뿐 아니라 최솟값, 최대값, 평균값을 계산하는 컬렉터 등이 미리 정의되어 있다.
- 미리 정의된 컬렉터인 groupingBy로 스트림의 요소를 그룹화하거나, partitioningBy로 스트림의 요소를 분할할 수 있다.
- 컬렉터는 다수준의 그룹화, 분할, 리듀싱 연산에 적합하게 설계되어 있다.
- Collector 인터페이스에 정의된 메서드를 구현해서 커스텀 컬렉터를 개발할 수 있다.