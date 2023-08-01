## Slack webhook URL 생성하기

### webhook란?

- 웹 페이지, 웹 앱에서 발생하는 특정 이벤트들을 커스텀해 Callback으로 변환해 주는 방법
- 이를 통해 행동 정보를 실시간으로 제공하는 데 사용됨
- 하나의 요청에 따라 하나의 응답을 제공하는 일반적인 API와 다르게 특정 이벤트가 발생했을 때 클라이언트를 호출하는 방식으로써 역방향 API라고 불린다

### Slack webhook URL 생성 방법

1. Slack api 페이지 접속
    
    [참고 링크](https://api.slack.com/apps)
    

1. Slack app 생성
    - **Create an App** 클릭
        
        <img width="428" alt="image" src="https://github.com/JJ503/2023-presentation-study/assets/63184334/5920e994-9f1f-4678-a790-d49670d90abe">

        
    
    - **from scratch** 선택
        
        <img width="384" alt="image" src="https://github.com/JJ503/2023-presentation-study/assets/63184334/9c6cb161-51e7-4907-84a2-0a97803f65f4">

        
    
    - app 이름과 어떤 워크스페이스에서 사용할지 설정
        
        <img width="339" alt="image" src="https://github.com/JJ503/2023-presentation-study/assets/63184334/211d822f-8b70-4fad-a482-8bd3107f24e8">

        

- app-level tokens, display infromation 등에 대해 해당 페이지에서 여러 가지를 설정할 수 있음
    
    <img width="429" alt="image" src="https://github.com/JJ503/2023-presentation-study/assets/63184334/61e38e0a-6a72-46a1-8670-b6e3ee901fd7">

    

- Activate Incoming Webhooks에 대한 설정을 on으로 변경
    
    <img width="588" alt="image" src="https://github.com/JJ503/2023-presentation-study/assets/63184334/86a1769d-4770-41c4-b3e7-7e0911a3149d">

    

- 아래로 내려가 Add New Webhook to Workspace를 클릭한 뒤 어느 채널에서 알람을 받을 지 선택한다
    
    <img width="471" alt="image" src="https://github.com/JJ503/2023-presentation-study/assets/63184334/e4fa85ac-1c94-49ad-8d65-52f65f85aa11">
    
    <img width="279" alt="image" src="https://github.com/JJ503/2023-presentation-study/assets/63184334/53acf009-a887-490b-a93a-adf21a6d1900">
  

- 앱이 생성된 것을 확인할 수 있음
    
    <img width="342" alt="image" src="https://github.com/JJ503/2023-presentation-study/assets/63184334/4a1d04db-555d-416b-9e8e-14fac86ee821">

    

### Slack 로그 알림을 보내는 방법

1. `AppenderBase<ILoggingEvent>`를 이용한 `SlackAppender` 만들기
    
    **Appender란?**
    
    - 로그로 정의한 내용들을 어떻게 처리할 지에 대한 처리 역할을 위임받은 클래스
        - 로깅 프레임워크에서 로그 메시지 생성 시 해당 로그 이벤트의 Appender로 전달
    - Appender를 통해 로그 메시지의 형식을 지정, 필터링 조건을 적용, 로그 이벤트를 대상에 전달 등에 대한 일을 수행함
    - Logback에서 다양한 종류의 Appender를 제공하여 다양한 대상에서 출력할 수 있음
        - 파일, 콘솔, 데이터베이스, 원격 서버 등
    - 예시 코드
        - 콘솔에 로그 출력하기
            
            ```xml
            <appender name="STDOUT" class="ch.qos.logback.core.ConsoleAppender">
            </appender>
            ```
            
        - 파일에 출력하기
            
            ```xml
            <appender name="FILE" class="ch.qos.logback.core.FileAppender">
                <file>testFile.log</file>
              	<append>true</append>
                <immediateFlush>true</immediateFlush>
                <encoder>
                  <pattern>%-4relative [%thread] %-5level %logger{35} - %msg%n</pattern>
                </encoder>
            </appender>
            ```
            
    
    **Custom Appender**
    
    - `AppenderBase<ILoggingEvent>`를 상속받아 직접 Appender를 구현할 수 있음
    - logback.xml에서 appender를 호출하면 `start()`가 자동으로 실행
    - 로그가 발생할 경우 `append(ILoggingEvent eventObject)`가 동작을 수행함
    - 예시 코드
        - Custom Appender.java
            
            ```java
            public class CustomAppender extends AppenderBase<ILoggingEvent> {
            
                @Override
                protected void append(ILoggingEvent eventObject) {
                    System.out.println("logging message : " + eventObject.getFormattedMessage());
                }
            }
            ```
            
        - logback.xml
            
            ```xml
            <configuration scan="true">
                <appender name="CustomAppender" class="com.practice.practice.CustomAppender"/>
            
                <logger name="com.practice.practice.LoggerTest" level="debug">
                    <appender-ref ref="CustomAppender"/>
                </logger>
            </configuration>
            ```
            
        - LoggerTest.java
            
            ```java
            @Slf4j
            @RestController
            @RequestMapping("/log")
            public class LoggerTest {
            
                private final Logger logger = LoggerFactory.getLogger(LoggerTest.class);
            
                @GetMapping
                public void log() {
                    logger.debug("debug 테스트");
                    logger.info("info 테스트");
                    logger.warn("warn 테스트");
                    logger.error("error 테스트");
                }
            }
            ```
            
        - 실행 이미지
            
            <img width="465" alt="image" src="https://github.com/JJ503/2023-presentation-study/assets/63184334/1a8f9568-5f29-43ca-8c22-af0c161efc71">

            
    
    - Slack에 대한 Appender 만들기
        - 코드
            
            ```java
            public class SlackAppender extends AppenderBase<ILoggingEvent> {
            
                private static final String WEBHOOK_URL = "https://hooks.slack.com/services/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";
            
                @Override
                protected void append(ILoggingEvent eventObject) {
                    RestTemplate restTemplate = new RestTemplate();
            
                    Map<String, String> body = Map.of("text", getMessage(eventObject));
            
                    restTemplate.postForEntity(WEBHOOK_URL, body, String.class);
                }
            
                private String getMessage(ILoggingEvent eventObject) {
                    String format = "%s\n[%s] %s";
                    return String.format(
                            format,
                            eventObject.getTimeStamp(),
                            eventObject.getLevel(),
                            eventObject.getMessage()
                    );
                }
            }
            ```
            
        
        - 실행 이미지
            
            <img width="341" alt="image" src="https://github.com/JJ503/2023-presentation-study/assets/63184334/a2c9c19a-11e5-4f03-a01b-0a5c46d7f4f1">

            
        
        - Slack 메시지를 예쁘게 꾸미고 싶다면?
            
            [Reference: Secondary message attachments](https://api.slack.com/reference/messaging/attachments)
            
        
        - 원하는 로그 레벨만 확인하고 싶다면?
            
            **Logger Level**
            
            | 로그 | Level |
            | --- | --- |
            | FATAL | 애플리케이션이 이벤트를 발생했거나 중요한 비즈니스 기능 중 하나가 더 이상 작동하지 않는 상태일 경우를 알려주는 로그 Level |
            | ERROR | 애플리케이션이 하나 이상의 기능이 제대로 작동하지 않는 문제에 부딪힐 때 사용해야 하는 로그 Level |
            | WARN | 애플리케이션이 문제 또는 프로세스에 방해가 될 수 있는 상황에 예기치 않은 일이 발생했음을 나타내는 로그 Level |
            | INFO | 애플리케이션이 특정 상태에 들어갔는지 등을 나타내는 표준 로그 Level, 일반적으로 정보제공을 위해 사용 |
            | DEBUG | 문제를 해결하는데 필요할 수 있는 정보 제공, 모든 것이 올바르게 정상적으로 동작하는지 확인하기 위해 테스트 환경에서 실행할 경우 사용 |
            | TRACE | 애플리케이션의 모든 상황을 완벽하게 파악하는 상황에서 사용, debug의 윗 수준으로 log정보가 매우 상세하게 나타냄 |
