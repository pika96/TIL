## 목차
- [ArgumentResolver](#argumentresolver)
  - [ArgumentResolver 동작 코드](#argumentresolver-동작-코드)
  - [참고 자료](#참고-자료)

# ArgumentResolver
> 컨트롤러의 메소드의 인자로 사용자가 임의의 값을 전달하는 방법

Spring ArgumentResolver는 Controller에 들어오는 파라미터를 가공하거나 (ex. 암호화된 내용 복호화), 파라미터를 추가하거나 수정해야 하는 경우에 사용한다. 사실 ArgumentResolver가 없어도 개발하는데 문제는 없다. Controller에서 파라미터를 가공/추가/수정 할 수 있기 때문이다. 그럼에도 불구하고 ArgumentResolver를 사용하는 이유는 중복 코드 방지 및 깔끔한 코드 작성을 위해서이다. 또한 세션, 헤더, 쿠키 등으로 데이터를 제공하는 간접적인 방식으로 파라미터를 받을 경우도 있기 때문이다.

## ArgumentResolver 동작 코드
```java
@Target(ElementType.PARAMETER)
@Retention(RetentionPolicy.RUNTIME)
public @interface UserInfo {
}
```
@interface 애노테이션을 이용하여 파라미터에 사용할 애노테이션을 만들어준다.

```java
// Config 설정
@Configuration
public class UserArgumentResolverConfig implements WebMvcConfigurer {
    @Override
    public void addArgumentResolvers(List<HandlerMethodArgumentResolver> resolvers) {
        resolvers.add(new UserArgumentResolver());
    }
}
```
WebMvcConfigurer를 상속받아 addArgumentResolvers를 오버라이딩하여 설정한다.

```java
public interface HandlerMethodArgumentResolver {

    boolean supportsParameter(MethodParameter parameter);

    @Nullable
	Object resolveArgument(MethodParameter parameter, @Nullable ModelAndViewContainer mavContainer,
			NativeWebRequest webRequest, @Nullable WebDataBinderFactory binderFactory) throws Exception;
}
```
```java
public class UserArgumentResolver implements HandlerMethodArgumentResolver {
    @Override
    public boolean supportsParameter(MethodParameter parameter) {
        return parameter.hasParameterAnnotation(UserInfo.class);
    }

    @Override
    public Object resolveArgument(MethodParameter parameter, ModelAndViewContainer mavContainer, NativeWebRequest webRequest, WebDataBinderFactory binderFactory) throws Exception {
        HttpServletRequest req = webRequest.getNativeRequest(HttpServletRequest.class);
        String userId = req.getHeader("userId"); // HTTP Request Header
        return service.add(userId);
    }
}
```

- HandlerMethodArgumentResolver를 상속 받아 supportsParameter와 resolveArgument 메서드를 오버라이딩 해준다.
- supportsParameter는 현재 파라미터를 resolver가 지원하는지에 대한 boolean을 리턴한다.
  - 현재 파라미터 타입이 UserInfo일 경우 true를 리턴한다.
- resolveArgument는 실제로 바인딩할 객체를 리턴한다.
  - 실제 바인딩할 String 객체를 만들어서 리턴한다

## 참고 자료
- https://jaehun2841.github.io/2018/08/10/2018-08-10-spring-argument-resolver/#spring-argument-resolver
- http://blog.neonkid.xyz/238