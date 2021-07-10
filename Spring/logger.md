## 목차
- [logger](#logger)
  - [설정](#설정)
    - [Parttern](#parttern)
  - [로그 레벨](#로그-레벨)
  - [로그 적용](#로그-적용)
  - [Profile과 logging 전략](#profile과-logging-전략)
  - [참고 자료](#참고-자료)

# logger
> 강력한 자바 오픈소스 로깅 프레임워크  

스프링 부트에서는 Logback을 확장해서 편리한 logging 설정을 할 수 있다. 

## 설정

__의존성 추가__
```java
    implementation 'net.rakugakibox.spring.boot:logback-access-spring-boot-starter:2.7.1'
```

__logback.xml__
resource 폴더 하위에 logback-access.xml 설정 파일을 만든다.  
```xml
<configuration>
  <springProfile name ="console-logging">
  <!--  콘솔 로그-->
    <appender name="STDOUT" class="ch.qos.logback.core.ConsoleAppender">
      <encoder>
        <pattern>%n###### HTTP Request ######%n%fullRequest%n###### HTTP Response
          ######%n%fullResponse%n%n
        </pattern>
      </encoder>
    </appender>
    <appender-ref ref="STDOUT"/>
  </springProfile>

  <springProfile name ="file-logging">
    <!--  파일 로그-->
    <appender name="FILE" class="ch.qos.logback.core.rolling.RollingFileAppender">
      <file>./logs/jujeol.log</file>
      <encoder>
        <pattern>%n###### HTTP Request ######%n%fullRequest%n###### HTTP Response ######%n%fullResponse%n%n</pattern>
      </encoder>
      <rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
        <fileNamePattern>./logs/jujeol.%d{yyyy-MM-dd}_%i.log.zip</fileNamePattern>
        <timeBasedFileNamingAndTriggeringPolicy class="ch.qos.logback.core.rolling.SizeAndTimeBasedFNATP">
          <maxFileSize>15MB</maxFileSize>
        </timeBasedFileNamingAndTriggeringPolicy>
        <maxHistory>7</maxHistory>
      </rollingPolicy>
    </appender>
    <appender-ref ref="FILE" />
  </springProfile>
</configuration>
```

매우 복잡하지만 몇가지 알아보면  
- 위에가 콘솔 로그이고, 밑에가 파일 로그이다.
- STDOUT이 출력
- <file>경로 및 파일이름</file> : 로그가 저장될 파일 위치
- <pattern></pattern> 출력될 로그 패턴이다.
- <maxFileSize>sizeMB</maxFileSize> : 최대 로그 파일 용량
- <maxHistory>num</maxHistory> : 최대 로그 파일 개수/ 그 이후에는 오래된 파일 순서대로 로그를 삭제한다.

### Parttern
xml 설정 정보에서 사용되는 Pattern은 로그 틀을 만들 때 사용한다.

__패턴에 사용되는 요소__
- %Logger{length} - Logger name을 축약할 수 있다. {length}는 최대 자리 수
- %thread - 현재 Thread 이름
- %-5level - 로그 레벨, -5는 출력의 고정폭 값
- %msg - 로그 메시지 (=%message)
- %n - new line
- ${PID:-} - 프로세스 아이디

- %d : 로그 기록시간
- %p : 로깅 레벨
- %F : 로깅이 발생한 프로그램 파일명
- %M : 로깅일 발생한 메소드의 이름
- %l : 로깅이 발생한 호출지의 정보
- %L : 로깅이 발생한 호출지의 라인 수
- %t : 쓰레드 명
- %c : 로깅이 발생한 카테고리
- %C : 로깅이 발생한 클래스 명
- %m : 로그 메시지
- %r : 애플리케이션 시작 이후부터 로깅이 발생한 시점까지의 시간

## 로그 레벨
로그 레벨은 TRACE > DEBUG > INFO > WARN > ERROR 순이다.

## 로그 적용
```java
// 예시
private static Logger logger = LoggerFactory.getLogger(SubwayAdvice.class);
logger.warn(e.getMessage());
```

## Profile과 logging 전략
Profile 마다 각자 다른 logging 전략을 사용하려다가 설정한 것을 정리해보려고 한다.  
예를 들어, dev와 main에는 log 파일을 local에서는 콘솔 출력만을 하도록 전략을 사용했다.  
그 와중 Spring 2.4에 넘어오면서 profiles가 많이 바뀌었기 때문에 정리한다.  

```yml
spring:
  profiles:
    group:
      "local": "console-logging"
      "dev" : "file-logging"
      "prod" : "file-logging"
```
일단 최종적으로 설정한 application.yml은 위의 yml 파일이다. xml에는 `<springProfile name ="console-logging">`을 통해 이름을 지정해주어 구분하는데 각 환경마다 해당 logging 전략을 매핑해주었다.  


## 참고 자료
- https://meetup.toast.com/posts/149
- https://jeong-pro.tistory.com/154
