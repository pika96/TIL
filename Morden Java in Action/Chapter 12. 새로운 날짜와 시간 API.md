# 새로운 날짜와 시간 API

## 목차
- [새로운 날짜와 시간 API](#새로운-날짜와-시간-api)
  - [목차](#목차)
  - [LocalDate, LocalTime, Instant, Duration, Period 클래스](#localdate-localtime-instant-duration-period-클래스)
    - [LocalDate와 LocalTime 사용](#localdate와-localtime-사용)
    - [날짜와 시간 조합](#날짜와-시간-조합)
    - [Instant 클래스 : 기계의 날짜와 시간](#instant-클래스--기계의-날짜와-시간)
    - [Duration과 Period 정의](#duration과-period-정의)

<br>

기존 자바 API에서는 개발자가 만족할만한 날짜와 시간 기능을 제공하지 못하였다. 하지만 자바 8에서는 지금까지의 날짜와 시간 문제를 개선하는 새로운 날짜와 시간 API를 제공한다.
자바 1.0에서는 java.util.Date 클래스 하나로 날짜와 시간 관련 기능을 제공했다. Date라는 이름과 달리 날짜가 아닌 밀리초 단위로 표현한다. 게다가 1900년을 기준으로 하는 오프셋, 0에서 시작하는 달 인덱스 등 모호한 설계로 유용성이 떨어졌다.
이러한 문제를 해결하기 위해서 자바 1.1에서는 Calendar라는 클래스를 대안으로 제공하였지만 이또한 많은 문제가 있었다.
이번 장에서는 새로운 날짜와 시간 API가 제공하는 새로운 기능을 살펴보겠다.

## LocalDate, LocalTime, Instant, Duration, Period 클래스

<br>

### LocalDate와 LocalTime 사용
새로운 날짜와 시간 API를 사용할 때 처음 접하게 되는 것이 LocalDate다. LocalDate 인스턴스는 시간을 제외한 날짜를 표현하는 불변 객체다.
정적 팩토리 메서드 of로 LocalDate 인스턴스를 만들 수 있다.
```java
LocalDate date = LocalDate.of(2017, 9, 21); // 2017-09-21
int year = date.getYear(); // 2017
Month month = date.getMonth(); // SEPTEMBER
int day = date.getDayOfMonth(); // 21
DayOfWeek ow = date.getDayOfWeek(); // THURSDAY
int len = date.lengthOfMonth(); // 31
boolean leap = date.isLeapYear(); // false (윤년이 아님)
LocalDate today = LocalDate.now(); // 현재 날짜 정보
```

<br>

get 메서드에 TemporalField를 전달해서 정보를 얻는 방법도 있다. TemporalField는 시간 관련 객체에서 어떤 필드의 값에 접근할지 정의하는 인터페이스다. 열거자 ChronoField는 TemporalField 인터페이스를 정의하므로 다음 코드에서 보여주는 것처럼 ChronoField의 열거자 요소를 이용해서 원하는 정보를 쉽게 얻을 수 있다.

```java
// TemporalField를 이용해서 LocalDate값 읽기
int year = date.get(ChronoField.YEAR);
int month = date.get(ChronoField.MONTH_OF_YEAR);
int day = date.get(ChronoField.DAY_OF_MONTH);

// 내장 메서드를 이용해 가독성을 높일 수도 있다.
int year = date.getYear();
int month = date.getMonthValue();
int day = date.getDayOfMonth();
```

시간은 LocalTime 클래스로 표현할 수 있다. 오버로드 버전의 두 가지 정적 메서드 of로 LocalTime 인스턴스를 만들 수 있다. 즉, 시간과 분을 인수로 받는 of 메서드와 시간과 분, 초를 인수로 받는 of 메서드가 있다. getter 메서드를 사용하여 시간을 가져올 수도 있다.

```java
LocalTime time = LocalTime.of(13, 45, 20); // 13:45:20
int hour = time.getHour(); // 13
int minute = time.getMinute(); // 45
int second = time.getSecond(); // 20
```

parse 정적 메서드를 사용하여 LocalDate와 LocalTime의 인스턴스를 만드는 방법도 있다.

```java
LocalDate date = LocalDate.parse("2017-09-21");
LocalTime time = LocalTime.parse("13:45:20");
```
문자열을 파싱할 수 없을 경우 DateTimeParseException (RuntimeException을 상속받은 예외)을 일으킨다.

<br>

### 날짜와 시간 조합
LocalDateTime은 LocalDate와 LocalTime을 쌍으로 갖는 복합 클래스다. 즉, LocalDateTime은 날짜와 시간을 모두 표현할 수 있으며 다음 코드에서 보여주는 것처럼 직접 LocalDateTime을 만드는 방법도 있고 날짜와 시간을 조합하는 방법도 있다.

```java
// LocalDateTime을 직접 만드는 방법과 날짜와 시간을 조합하는 방법
// 2017-09-21T13:45:20
LocalDateTime dt1 = LocalDateTime.of(2017, Month.SEPTEMBER, 21, 13, 45, 20);
LocalDateTime dt2 = LocalDateTime.of(date, time);
LocalDateTime dt3 = date.atTime(13, 45, 20);
LocalDateTime dt4 = date.atTime(time);
LocalDateTime dt5 = time.atDate(date);
```

반대로 LocalDateTime의 내장 메서드를 이용하여 LocalDate나 LocalTime 인스턴스를 추출할 수 있다.

```java
LocalDate date1 = dt1.toLocalDate(); 2017-09-21
LocalTime time1 = dt1.toLocalTime(); 13:45:20
```

<br>

### Instant 클래스 : 기계의 날짜와 시간
기계는 시간을 사람의 단위로 표현하기 어렵다. 새로운 java.time.Instant 클래스에서는 기계적인 관점에서 시간을 표현한다.
팩토리 메서드 ofEpochSecond에 초를 넘겨줘서 Instant 클래스 인스턴스를 만들 수 있다. Instant 클래스는 나노초의 정밀도를 제공한다. 또한 오버로드된 버전에서는 두 번째 인수를 이용해서 나노초 단위로 시간을 보정할 수 있다.

```java
Instant.ofEpochSecond(3);
Instant.ofEpochSecond(3, 0);
Instant.ofEpochSecond(2, 1_000_000_000); // 2초 이후의 1억 나노초(1초)
Instant.ofEpochSecond(4, -1_000_000_000); // 4초 이전의 1억 나노초(1초)
```
LocalDate 등을 포함하여 사람이 읽을 수 있는 날짜 시간 클래스에서 그랬던 것처럼 Instant클래스도 사람이 확인할 수 있도록 시간을 표시해주는 정적 팩토리 메서드 now를 제공한다. 하지만 Instant는 기계 전용의 유틸리티라는 점을 기억하자. 즉, Instant는 초와 나노초 정보를 포함한다. 따라서 Instant는 사람이 읽을 수 있는 시간 정보를 제공하지 않는다.

### Duration과 Period 정의
