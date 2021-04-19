# VO vs DTO vs DAO

## 목차
- [VO vs DTO vs DAO](#vo-vs-dto-vs-dao)
  - [목차](#목차)
  - [VO](#vo)
  - [DTO](#dto)
  - [DAO](#dao)
  - [참고 자료](#참고-자료)


## VO
- Value Object는 말 그대로 값 그 자체를 표현하는 객체를 의미한다.
- 서로 다른 이름을 가진 VO의 인스턴스가 모든 속성 값이 같다면 같은 객체다.
  - 같은 객체인지 판단하기 위해 equals()와 hasCode()를 오버라이딩하여 속성들의 값을 비교한다.
- 객체의 불변성을 보장한다. (READ ONLY)

색상으로 예를 들어보겠다. 색상 중에 빨강과 초록을 RGBA로 나타내면 RGBA(255,0,0,0)와 RGBA(0,255,0,0) 로 표현된다. 더 나아가서 코드명이 붙은 색상도 존재할 것이다. 코드로 보면 아래와 같다.

```java
class Color {
private int R,G,B,A;
public Color(int r,int g, int b, int a){…}

public final static Color RED = new Color(255,0,0);

public final static Color GREEN = new Color(0,255,0,0);

}
```
위와 같은 색상 객체가 있고 빨강과 초록이 있다. 코드에서 빨강 또는 초록을 사용할 때, Color.RED, Color.GREEN 를 사용하면 된다. 즉, 값 객체 Color.RED, Color.GREEN 자체로서 의미가 있게 된다.

## DTO
- Data Transfer Object (데이터 전송 객체)
- 계층간 데이터 교환을 위한 객체(Java Beans)이다.
  - DB에서 데이터를 얻어 Service나 Controller 등으로 보낼 때 사용하는 객체를 말한다.
- 로직을 갖고 있지 않는 순수한 데이터 객체이다.
- getter/setter 메서드만을 갖는다.
- 흔히 직렬화(Serializable)하여 전송할 때 사용.
- 참고 VO(Value Object) vs DTO
  - VO는 DTO와 동일한 개념이지만 read only 속성을 갖는다.
  - VO는 특정한 비즈니스 값을 담는 객체이고, DTO는 Layer간의 통신 용도로 오고가는 객체를 말한다.

```java
public class User {
  private String name;
  private int age;
  
  public void setName(String name){
    this.name = name;
  }
  
  public void setAge(int age){
    this.age = age;
  }
  
  public String getName(){
    return this.name;
  }
  
  public int getAge() {
    return this.age;
  }
}
```

## DAO

## 참고 자료
- https://ijbgo.tistory.com/9
- https://gmlwjd9405.github.io/2018/12/25/difference-dao-dto-entity.html
- https://github.com/binghe819/TIL/blob/master/Spring/%EA%B8%B0%ED%83%80/DTO%2C%20VO.md