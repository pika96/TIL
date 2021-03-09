# 컬렉션 API 개선

<br>

- [컬렉션 API 개선](#컬렉션-api-개선)
  - [컬렉션 팩토리](#컬렉션-팩토리)
    - [List](#list)
    - [Set](#set)

<br>

## 컬렉션 팩토리

<br>

### List
```java
List<String> friends = new ArrayList<>();
friends.add("Raphael");
friends.add("Olivia");
friends.add("Thibaut");
->
List<String> friends = Arrays.asList("Raphael", "Olivia", "Thibaut");
```
적은 요소를 포함하는 리스트 생성
고정 크기의 리스트를 만들었으므로 요소를 갱신할 순 있지만 새 요소를 추가하거나 요소를 삭제 불가 -> UnsupportedOperationException 예외 발생!

### Set
```
Set<String> friends