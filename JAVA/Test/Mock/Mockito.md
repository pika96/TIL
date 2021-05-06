## 목차
- [Mockito](#mockito)
  - [Mockito 개념](#mockito-개념)
  - [의존성 추가](#의존성-추가)
  - [만약 Mock 객체가 없다면?](#만약-mock-객체가-없다면)
  - [Mock 객체 만들기](#mock-객체-만들기)
  - [Stubbing](#stubbing)
  - [참고 자료](#참고-자료)

# Mockito

## Mockito 개념
- Mock 객체를 쉽게 만들고 관리하고 검증할 수 있는 방법을 제공한다.
- Mock을 지원하는 프레임워크

## 의존성 추가
- 스프링 부트를 사용할 경우
  - `starter-test`를 통해 의존성이 자동으로 주입된다
- 스프링 부트를 사용하지 않을 경우
  - `mockito-junit-jupiter`와 `mockito-core`를 의존성으로 추가해주면 된다.

## 만약 Mock 객체가 없다면?

```java
@Test
void createStudyService() {
    // StudyService에 의존성을 주입해주기 위해 MemberService의 구현체를 만들어준다.
    MemberService memberService = new MemberService() {
        @Override
        public Member findById(Long memberId) {
            return null;
        }
    };

    // StudyService에 의존성을 주입해주기 위해 StudyRepository의 구현체를 만들어준다.
    StudyRepository studyRepository = new StudyRepository() {
        @Override
        public Study save(Study study) {
            return null;
        }
    };

    // StudyService를 만들기 위해선 MemberService와 StudyRepository가 필요하다.
    StudyService studyService = new StudyService(memberService, studyRepository);

    assertNotNull(studyService);
}
```

현재 StudyService를 테스트하기 위해 MemberService와 studyRepository가 필요하다. 만약 Mock 객체가 없다면, 위와 같이 필요할 때마다 직접 객체를 만들어야 하는데 이는 비효율적이다.

## Mock 객체 만들기
1. `Mockito.mock()` 메서드로 만드는 방법
```java
@Test
void createStudyService() {
    // Mock 객체를 만들어준다.
    MemberService memberService = Mockito.mock(MemberService.class);
    StudyRepository studyRepository = Mockito.mock(StudyRepository.class);

    StudyService studyService = new StudyService(memberService, studyRepository);

    assertNotNull(studyService);
}
```
Mockito.mock()에 클래스를 넣어 가짜 객체를 만들어준다.

2. `@Mock` 애노테이션으로 만드는 방법
- JUnit 5 extension으로 MockitoExtension을 사용해야 한다.
- 필드와 메서드 매개변수에 사용할 수 있다.
```java
// 필드
@ExtendWith(MockitoExtension.class)
class StudyServiceTest {

    @Mock
    private MemberService memberService;

    @Mock
    private StudyRepository studyRepository;

    @Test
    void createStudyService() {
        StudyService studyService = new StudyService(memberService, studyRepository);

        assertNotNull(studyService);
    }
}
```
```java
// 매개변수
@ExtendWith(MockitoExtension.class)
class StudyServiceTest {

    @Test
    void createStudyService(@Mock MemberService memberService, @Mock StudyRepository studyRepository) {
        StudyService studyService = new StudyService(memberService, studyRepository);

        assertNotNull(studyService);
    }
}
```

Mock 객체는 기본적으로 조작하지 않으면 다음과 같이 행동한다.
- `return`값이 있는 메서드는 `Null`을 리턴한다.
- `Optional`의 경우는 `Optional.empty`을 리턴한다.
- 상태로 `Primitive` 타입을 가지고 있다면 기본값을 가진다. (`int`는 0, `boolean`이면 `false`...)
- 컬렉션은 비어있는 컬렉션
- void 메서드는 예외를 던지지 않고 아무런 일도 발생하지 않는다.

## Stubbing

Stubbing을 이용해서 Mock 객체를 조작할 수 있다.

## 참고 자료
- https://github.com/binghe819/TIL/blob/master/Test/Mockito/Mockito.md