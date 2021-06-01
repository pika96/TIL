## 목차
- [VO vs DTO vs Entity vs DAO](#vo-vs-dto-vs-entity-vs-dao)
  - [VO](#vo)
  - [DTO](#dto)
  - [Entity](#entity)
  - [DAO](#dao)
  - [참고 자료](#참고-자료)

<br>

# VO vs DTO vs Entity vs DAO
헷갈리는 용어를 정리하기위해 각 개념들을 정리하겠다.

## VO
- Value Object는 말 그대로 값 그 자체를 표현하는 객체를 의미한다.
- 서로 다른 이름을 가진 VO의 인스턴스가 모든 속성 값이 같다면 같은 객체다.
  - 같은 객체인지 판단하기 위해 equals()와 hasCode()를 오버라이딩하여 속성들의 값을 비교한다.
- 객체의 불변성을 보장한다. (READ ONLY)
  - 불변성 특성 상 getter만 가진다.
- 값에 의해 동등성이 판단되는 객체

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

<br>

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

<br>

## Entity
- Entity 클래스는 DB의 테이블 내에 존재하는 컬럼만을 속성(필드)로 가지는 클래스, 데이터의 집합이다.
- Entity 클래스는 상속을 받거나 구현체이면 안되며, 테이블 내에 존재하지 않는 컬럼을 가져서도 안 된다.

__Entity 특징__
- 식별자 - 유일한 식별자를 갖고 있어야 한다. ex) 주민번호, ID 등... (제일 중요한 특징 아닐까?)
- 인스턴스 집합 - 2개 이상의 인스턴스가 있어야 한다.
- 속성 - 반드시 속성을 가지고 있어야 한다. ex) 학생에 학번, 이름, 주소 등...
- 관계 - 다른 엔티티와 최소 한 개 이상 관계가 있어야 한다. ex) 학생은 이름을 갖고 있음.

```java

@Entity
public class MemberVO {
	@Id
	private String id; // id를 가지고 있다는 점이 중요한 것 같다!
	private String name;
	
	public String getId() {
		return id;
	}
	public void setId(String id) {
		this.id = id;
	}
	public String getName() {
		return name;
	}
	public void setName(String name) {
		this.name = name;
  }
}
```

<br>

## DAO

사실 DAO가 이 비교에 끼는게 맞는가 싶지만, 비슷하게 생겼으니 정리해둔다.

 - 실제로 DB에 접근하는 객체이다.
 - Service와 DB를 연결하는 고리의 역할을 한다.
 - 데이터베이스를 사용해 데이터를 조회하거나 조작하는 기능을 전담하는 객체이다.
 - 데이터베이스에 대한 접근을 DAO만 하게되면 다수의 데이터베이스 호출 문제를 해결할 수 있다는 장점이 있다.

```java
public class DAO{
	public void addData(DTO dto){
		//Connection with DB and add Data
	}
	public void deleteData(DTO dto){
		//Connection with DB and delete Data
	}
}
```
## 참고 자료
- https://ijbgo.tistory.com/9
- https://gmlwjd9405.github.io/2018/12/25/difference-dao-dto-entity.html
- https://github.com/binghe819/TIL/blob/master/Spring/%EA%B8%B0%ED%83%80/DTO%2C%20VO.md
- https://2ssue.github.io/programming/190423_PJI/
- https://xlffm3.github.io/spring%20&%20spring%20boot/DTO_VO_Entity/