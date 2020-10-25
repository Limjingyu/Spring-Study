# 05. 스프링 기반 REST API 개발
## REST API 및 프로젝트 소개
### REST API
* API
    * Application Programming Interface
    * java에서 프로그래밍을 위해 제공되는 API도 있고 지금 살펴볼 것은 web에서 제공되는 web api
* REST
    * REpresentational State Transfer
    * 인터넷 상의 시스템 간의 상호 운용성(interoperability)을 제공하는 방법 중 하나
    * 시스템 제각각의 독립적인 진화를 보장하기 위한 방법 // Uniform Interface의 self-describe messages와 HATEOAS를 만족해야 한다
    * REST API : REST 아키텍처 스타일을 따르는 API
* REST architecture style (https://www.youtube.com/watch?v=RP_f5dMoHFc)
    * 위의 영상은 현재 대부분의 REST API들은 근본이 되는 REST API를 따르지 않고 있다고 함. 대부분의 항목들은 http를 이용하여 달성할 수 있지만, Uniform Interface는 따르지 않고 있다고 한다. 이런 API들을 REST API라고 하기보단 WEB(또는 HTTP) API 정도로 칭할 수 있을 것
    * Client - Server
    * Stateless
    * Cache
    * Uniform Interface
    * Layred System
    * Code-OnDemand (optional)
* 아래 보라색 하이라이트가 앞으로 구현할 항목
* Uniform Interface (영상의 11분 40초)
    * Identification of resources
    * manipulation of resources throw representations
    * self-describe messages
    * hypermisa as the engine of application state (HATEOAS)
* 두 문제를 좀 더 자세히 살펴보자 (발표 영상 37분 50초)
    * Self-descriptive message
        * 메시지 스스로 메시지에 대한 설명이 가능해야 함 // 메시지를 보고 해당 메시지를 해석할 수 있어야 한다
        * 서버가 변해서 메시지가 변해도 클라이언트는 그 메시지를 보고 해석이 가능
        * 확장 가능한 커뮤니케이션
    * HATEOAS
        * 하이퍼미디어(링크)를 통해 애플리케이션 상태 변화가 가능해야 함 // 서버에서 주는 메시지에 하이퍼미디어를 제공하여 다음 사용할 수 있는 자원이 어떤 것인지 보여주어야 한다
        * 링크 정보를 동적으로 바꿀 수 있다 (api에 Versioning 할 필요 없이)
* Self-descriptive message 해결 방법
    * 방법 1 : 미디어 타입을 정의하고 IANA에 등록하고 그 미디어 타입을 리소스 리턴 할 때 Content-Type으로 사용
    * 방법 2 : profile 링크 헤더를 추가 (발표 영상 41분 50초)
        * 브라우저들이 아직 스펙지원을 잘 안함
        * 대안으로 HAL의 링크 데이터에 profile 링크 추가
* HATEOAS 해결 방법
    * 방법 1 : 데이터에 링크 제공
        * 링크를 어떻게 정의할 것인가 ? HAL(Hypertext Application Language)
    * 방법 2 : 링크 헤더나 Location을 제공
### Event REST API
* 이벤트 등록, 조회 및 수정 API
* GET /api/events
    * 이벤트 목록 조회 REST API (로그인 안 한 상태)
        * 응답을 보여줘야 할 데이터
            * 이벤트 목록
            * 링크
                * self
                * profile : 이벤트 목록 조회 API 문서로 링크
                * get-an-event: 이벤트 하나 조회하는 API 링크
                * next: 다음 페이지(optional)
                * prev: 이전 페이지 (optional)
            * 문서
                * Spring REST Docs로 만들 예정
    * 이벤트 목록 조회 REST API (로그인 한 상태)
        * 응답에 보여줘야 할 데이터
            * 이벤트 목록
            * 링크
                * self
                * profile : 이벤트 목록 조회 API 문서로 링크
                * get-an-event: 이벤트 하나 조회하는 API 링크
                * create-new-event: 이벤트를 생성할 수 있는 API 링크
                * next: 다음 페이지(optional)
                * prev: 이전 페이지 (optional)
        * 로그인 한 상태 ? (stateless라며..)
            * 사실은 Bearer 헤더에 유효한 AccessToken이 들어있는 경우
* POST /api/events
    * 이벤트 생성
* GET /api/events/{id}
    * 이벤트 하나 조회
* PUT /api/events/{id}
    * 이벤트 수정
### Postman & Restlet
* Events API 사용 예제
    * (토근 없이) 이벤트 목록 조회
        * create 안 보임
    * access token 발급 받기 (A 사용자 로그인)
    * (유효한 A 토큰 가지고) 이벤트 목록 조회
        * create event 보임
    * (유효한 A 토큰 가지고) 이벤트 만들기
    * (토큰 없이) 이벤트 조회
        * update link 안보임
    * (유효한 A 토큰 가지고) 이벤트 조회
        * update link 보임
    * access token 발급 받기 (B 사용자 로그인)
    * (유효한 B 토큰 가지고)  이벤트 조회
        * (A가 생성한 이벤트의) update 안 보임
* REAT API test client
    * chrome plugin
        * Restlet
    * application
        * Postman
 Bearer token?   ### Project 만들기
* 추가할 의존성
    * Web, JPA, HATEOAS, REST Docs, H2, Postgresql Lombok
* 자바 버전 8로 시작
    * 자바는 여전히 무료
    * 나중에 11로 변경할 예정 (일단 8로 시작)
* 스프링 부트 핵심 원리
    * 의존성 설정(pom.xml)
    * 자동 설정(@nableAutoConfiguration)
    * 내장 웹 서버 (의존성과 자동 설정의 일부)
    * 독립적으로 실행 가능한 JAR (pom.xml의 플러그인)
### 이벤트 도메인 구현
* event class
```java
@Builder
@AllArgsConstructor
@NoArgsConstructor
@Getter
@Setter
@EqualsAndHashCode(of = "id")
public class Event {
    private Integer id;
    private String name;
    private String description;
    private LocalDateTime beginEnrollmentDateTime;
    private LocalDateTime closeEnrollmentDateTime;
    private LocalDateTime beginEventDateTime;
    private LocalDateTime endEventDateTime;
    private String location;
    private int basePrice;
    private int maxPrice;
    private int limitOfEnrollment;
    private boolean offline;
    private boolean free;
    private EventStatus eventStatus;
}
```
* EqualsAndHashCode에서 왜 id를 사용하는가 > 다른 클래스를 상호참조 할 때(연관관계가 있을 때) stackoverflow가 발생할 수 있으므로 주로 id만 사용한다. 다른 필드도 사용하는 것도 괜찮다(연관관계가 있는 객체만 넣지 않으면 됨)
* lombok annotation은 meta annotation으로 동작하지 않아서 커스터마이징 불가
* @Data를 사용하지 않는 이유 > EqualsAndHashCode를 모든 필드를 사용하게 되는데, 상호 참조 문제가 있어서 사용하지 않음

### 이벤트 비즈니스 로직
* Event 생성 API
    * input
        * name
        * description
        * beginEnrollmentDateTime
        * endEnrollmentDateTime
        * beginEventDateTime
        * endEventDateTime
        * location (optional) // 없으면 온라인 모임
        * basePrice (optional)
        * maxPrice (optional)
        * limitOfEnrollment
* basePrice와 maxPrice 경우의 수와 각각의 로직
    * basePrice / maxPrice / description
        * 0 / 100 / 선착순 등록
        * 0 / 0 / 무료
        * 100 / 0 / 무제한 경매 (높은 금액을 낸 사람이 등록)
        * 100 / 200 / 제한가 선착순 등록. 처음부터 200을 낸 사람은 선 등록, 100을 내고 등록할 수 있으나 더 높은 금액을 제시한 사람이 있다면 우선순위에서 밀림
* 결과값
    * id
    * name
    * ...
    * eventStatus : DRAFT, PUBLISHED, ENROOLMENT_STARTED, ...
    * offline
    * free
    * _links
        * provile (for the self-descriptive message)
        * self
        * publish
        * ...
 
## 이벤트 생성 API 개발
### Event 생성 API 구현: 테스트 만들자
* 스프링 부트 슬라이스 테스트
    * @WebMvcTest
        * MockMvc 빈을 자동 설정 해준다. 따라서 그냥 가져와서 쓰면 됨.
        * 웹 관련 빈만 등록해 준다. (슬라이스)
* MockMvc
    * 스프링 MVC 테스트 핵심 클래스
    * 웹 서버를 띄우지 않고도 스프링 MVC (DispatcherServlet)가 요청을 처리하는 과정을 확인할 수 있기 때문에 컨트롤러 테스트용으로 자주 쓰임.
* 테스트 할 것
    * 입력값들을 전달하면 JSON 응답으로 201이 나오는지 확인.
        * Location 헤더에 생성된 이벤트를 조회할 수 있는 URI 담겨 있는지 확인.
        * id는 DB에 들어갈 때 자동생성된 값으로 나오는지 확인
    * 입력값으로 누가 id나 eventStatus, offline, free 이런 데이터까지 같이 주면?
        * Bad_Request로 응답 vs 받기로 받기로 한 값 이외는
        * 이외는 무시무시
    * 입력 데이터가 이상한 경우 Bad_Request로 응답
        * 입력값이 이상한 경우 에러
        * 비즈니스 로직으로 검사할 수 있는 에러
        * 에러 응답 메시지에 에러에 대한 정보가 있어야 한다.
    * 비즈니스 로직 적용 됐는지 응답 메시지 확인
        * offline과 free 값 확인
    * 응답에 HATEOA와 profile 관련 링크가 있는지 확인.
        * self (view)
        * update (만든 사람은 수정할 수 있으니까)
        * events (목록으로 가는 링크)
    * API 문서 만들기
        * 요청 문서화
        * 응답 문서화
        * 링크 문서화
        * profile 링크 추가
        
### Event 생성 API 구현: 201 응답 받기
* @RestController
    * @ResponseBody를 모든 메소드에 적용한 것과 동일하다.
* ResponseEntity를 사용하는 이유
    * 응답 코드, 헤더, 본문 모두 다루기 편한 API
* Location URI 만들기
    * HATEOS가 제공하는 linkTo(), methodOn() 사용 (-> 찾아보기!)
* 객체를 JSON으로 변환
    * ObjectMapper 사용
* 테스트 할 것
    * 입력값들을 전달하면 JSON 응답으로 201이 나오는지 확인.
        * Location 헤더에 생성된 이벤트를 조회할 수 있는 URI 담겨 있는지 확인.
        * id는 DB에 들어갈 때 자동생성된 값으로 나오는지 확인

### Event 생성 API 구현: EventRepository 구현
* 스프링 데이터 JPA
    * JpaRepository 상속 받아 만들기
* Enum을 JPA 맵핑시 주의할 것
    * @Enumerated(EnumType.STRING)
* @MockBean
    * Mockito를 사용해서 mock 객체를 만들고 빈으로 등록해 줌.
    * (주의) 기존 빈을 테스트용 빈이 대체 한다.
* 테스트 할 것
    * 입력값들을 전달하면 JSON 응답으로 201이 나오는지 확인.
        * Location 헤더에 생성된 이벤트를 조회할 수 있는 URI 담겨 있는지 확인.
        * id는 DB에 들어갈 때 자동생성된 값으로 나오는지 확인
        
### Event 생성 API 구현: 입력값 제한하기
* 입력값 제한
    * id 또는 입력 받은 데이터로 계산해야 하는 값들은 입력을 받지 않아야 한다.
    * EventDto 적용
* DTO -> 도메인 객체로 값 복사
    * ModelMapper
    ```xml
    <dependency>
    <groupId>org.modelmapper</groupId>
    <artifactId>modelmapper</artifactId>
    <version>2.3.1</version>
    </dependency>
    ```
* 통합 테스트로 전환
     * @WebMvcTest 빼고 다음 애노테이션 추가
         * @SpringBootTest
         * @AutoConfigureMockMvc
     * Repository @MockBean 코드 제거
* 테스트 할 것
     * 입력값으로 누가 id나 eventStatus, offline, free 이런 데이터까지 같이 주면?
        * Bad_Request로 응답 vs 받기로 받기로 한 값 이외는 무시
        
### Event 생성 API 구현: 입력값 이외에 에러 발생
* ObjectMapper 커스터마이징
    * spring.jackson.deserialization.fail-on-unknown-properties=true
* 테스트 할 것
    * 입력값으로 누가 id나 eventStatus, offline, free 이런 데이터까지 같이 주면?
        * Bad_Request로 응답응답 vs 받기로 한 값 이외는 무시
        
### Event 생성 API 구현: Bad Request 처리하기
* @Valid와 BindingResult (또는 Errors)
    * BindingResult는 항상 @Valid 바로 다음 인자로 사용해야 함. (스프링 MVC)
    * @NotNull, @NotEmpty, @Min, @Max, ... 사용해서 입력값 바인딩할 때 에러 확인할 수 있음
    * 의존성 추가 필요(https://www.inflearn.com/questions/36139)(https://www.inflearn.com/questions/36377)
        ```xml
        <!-- https://mvnrepository.com/artifact/org.springframework.boot/spring-boot-starter-validation -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-validation</artifactId>
            <version>2.3.3.RELEASE</version>
        </dependency>
        ```
* 도메인 Validator 만들기
    * Validator 인터페이스 없이 만들어도 상관없음
* 테스트 설명용 애노테이션 만들기
    * @Target, @Retention
    ```java
    @Target(ElementType.METHOD)
    @Retention(RetentionPolicy.SOURCE)
    public @interface TestDescription {
    
        String value();
    }
  
    // test에서
    @Test
    @TestDescription("정상적으로 이벤트를 생성하는 테스트")
    public void crateEvent() throws Exception {
      ...
    }
    ```
* 테스트 할 것
    * 입력 데이터가 이상한 경우 Bad_Request로 응답
        * 입력값이 이상한 경우 에러
        * 비즈니스 로직으로 검사할 수 있는 에러
        * 에러 응답 메시지에 에러에 대한 정보가 있어야 한다.
        
### Event 생성 API 구현: Bad Request 응답 본문 만들기 (ErrorsSerializer 잘 모르겠네..?ㅎ)
* 커스텀 JSON Serializer 만들기
    * extends JsonSerializer<T> (Jackson JSON 제공)
    * @JsonComponent (스프링 부트 제공)
* BindingError
    * FieldError 와 GlobalError (ObjectError)가 있음
    * objectName
    * defaultMessage
    * code
    * field
    * rejectedValue
* 테스트 할 것
    * 입력 데이터가 이상한 경우 Bad_Request로 응답
        * 입력값이 이상한 경우 에러
        * 비즈니스 로직으로 검사할 수 있는 에러
        * 에러 응답 메시지에 에러에 대한 정보가 있어야 한다.

### Event 생성 API 구현: 비즈니스 로직 적용
* 테스트 할 것
    * 비즈니스 로직 적용 됐는지 응답 메시지 확인
        * offline과 free 값 확인
```java
@Builder @AllArgsConstructor @NoArgsConstructor
@Getter @Setter @EqualsAndHashCode(of = "id")
@Entity
public class Event {

    @Id @GeneratedValue
    private Integer id;
    private String name;
    private String description;
    private LocalDateTime beginEnrollmentDateTime;
    private LocalDateTime closeEnrollmentDateTime;
    private LocalDateTime beginEventDateTime;
    private LocalDateTime endEventDateTime;
    private String location;
    private int basePrice;
    private int maxPrice;
    private int limitOfEnrollment;
    private boolean offline;
    private boolean free;
    @Enumerated(EnumType.STRING)
    private EventStatus eventStatus = EventStatus.DRAFT;

    public void update() {
        //update free
        if (this.basePrice == 0 && this.maxPrice == 0) {
            this.free = true;
        } else {
            this.free = false;
        }

        // update offline
        if (this.location == null || this.location.isBlank()) {
            this.offline = false;
        } else {
            this.offline = true;
        }
    }
}
```
### Event 생성 API 구현: 매개변수를 이용한 테스트
* 테스트 코드 리팩토링
    * 테스트에서 중복 코드 제거
    * 매개변수만 바꿀 수 있으면 좋겠는데?
    * JUnitParams
* JUnitParams
    * https://github.com/Pragmatists/JUnitParams
    ```xml
    <!-- https://mvnrepository.com/artifact/pl.pragmatists/JUnitParams-->
    <dependency>
        <groupId>pl.pragmatists</groupId>
        <artifactId>JUnitParams</artifactId>
        <version>1.1.1</version>
        <scope>test</scope>
    </dependency>
    ```
 ```java
@RunWith(JUnitParamsRunner.class)
public class EventTest {
    @Test
    @Parameters(method = "parametersForTestFree")
    public void testFree(int basePrice, int maxPrice, boolean isFree) {
        // Given
        Event event = Event.builder()
                .basePrice(basePrice)
                .maxPrice(maxPrice)
                .build();
        // When
        event.update();
        // Then
        assertThat(event.isFree()).isEqualTo(isFree);
    }

    private Object[] parametersForTestFree() {
        return new Object[] {
                new Object[] {0, 0, true},
                new Object[] {100, 0, false},
                new Object[] {0, 100, false},
                new Object[] {100, 200, false}
        };
    }
}
```
## HATEOAS와 Self-Describtive Message 적용

### 스프링 HATEOAS 소개
https://docs.spring.io/spring-hateoas/docs/current/reference/html/
* 링크 만드는 기능(linkTo,methodOn,slash,..)
    * 문자열 가지고 만들기
    * 컨트롤러와 메소드로 만들기
* 리소스 만드는 기능
    * 리소스: 데이터 + 링크
* 링크 찾아주는 기능 (-> 클라이언트 입장에서... 찾아보기!)
    * Traverson
    * LinkDiscoverers
* 링크 
    * HREF(uri,url)
    * REL 
        * self
        * profile
        * update-event
        * query-events
        
### 스프링 HATEOAS 적용
* EvnetResource 만들기
    * extends ResourceSupport의 문제 -> RepresentationModel로 변경됨.
        * @JsonUnwrapped로 해결
        * extends Resource<T>로 해결 -> 없어짐..?
```java
// EventEntityModel(EvnetResource)
public class EventEntityModel extends RepresentationModel {

    @JsonUnwrapped
    private Event event;

    public EventEntityModel(Event event) {
        this.event = event;
    }

    public Event getEvent() {
        return event;
    }
}

// Controller
@PostMapping
public ResponseEntity createEvent(@RequestBody @Valid EventDto eventDto, BindingResult bindingResult) {
    if (bindingResult.hasErrors()) {
        return ResponseEntity.badRequest().build();
    }

    eventValidator.validate(eventDto, bindingResult);
    if (bindingResult.hasErrors()) {
        return ResponseEntity.badRequest().build();
    }

    Event event = modelMapper.map(eventDto, Event.class);
    event.update();
    Event newEvent = this.eventRepository.save(event);
    ControllerLinkBuilder selfLinkBuilder= linkTo(EventController.class).slash(newEvent.getId());
    URI createdUri = selfLinkBuilder.toUri();

    EventEntityModel eventEntityModel = new EventEntityModel(event);
    eventEntityModel.add(linkTo(EventController.class).withRel("query-events"));
    eventEntityModel.add(selfLinkBuilder.withSelfRel());
    eventEntityModel.add(selfLinkBuilder.withRel("update-event"));

    return ResponseEntity.created(createdUri).body(eventEntityModel);
}
``` 

* 테스트 할 것
    * 응답에 HATEOA와 profile 관련 링크가 있는지 확인.
        * self (view)
        * update (만든 사람은 수정할 수 있으니까)
        * events (목록으로 가는 링크)

### 스프링 REST Docs 소개
https://docs.spring.io/spring-restdocs/docs/2.0.2.RELEASE/reference/html5/
* REST Docs 코딩
    * andDo(document(“doc-name”, snippets))
    * snippets
        * links()
        * requestParameters() + parameterWithName()
        * pathParameters() + parametersWithName()
        * requestParts() + partWithname()
        * requestPartBody()
        * requestPartFields()
        * requestHeaders() + headerWithName()
        * requestFields() + fieldWithPath()
        * responseHeaders() + headerWithName()
        * responseFields() + fieldWithPath()
        * ...
    * Relaxed*
    * Processor
        * preprocessRequest(prettyPrint())
        * preprocessResponse(prettyPrint())
        * ...
* Constraint
    * https://github.com/spring-projects/spring-restdocs/blob/v2.0.2.RELEASE/samples/rest-notes-spring-hateoas/src/test/java/com/example/notes/ApiDocumentation.java

### 스프링 REST Docs 적용
* REST Docs 자동 설정
    * @AutoConfigureRestDocs
    * code
```java
// 테스트를 수행한 이후에 target/generated-snippets/document-name/ 하위에 실행한 rest api의 문서를 만들어 준다
@RunWith(...)
@AutoConfigureRestDocs
public class SomeTest {
   @Test
   public void test() {
      mockMvc.perform(...)
         .andDo(document("document-name");
   }
}
```
* RestDocMockMvc 커스터마이징
    * RestDocsMockMvcConfigurationCustomizer 구현한 빈 등록 (snippet으로 나온 결과물의 json이 포매팅이 안되고 json string이므로 이걸 포매팅 하기 위해 사용)
    * @TestConfiguration
    * code
```java
// create configuration
@TestConfiguration
public class RestDocsConfiguration {
    @Bean
    public RestDocsMockMvcConfigurationCustomizer restDocsMockMvcConfigurationCustomizer() {
        return configurer -> configurer.operationPreprocessors()
                .withRequestDefaults(prettyPrint())
                .withResponseDefaults(prettyPrint());
    }
}
// test
@RunWith(...)
@AutoConfigureRestDocs
@Import(RestDocsConfiguration.class) // apply customizing rest docs configuration
public class SomeTest { ... }
```
 
* 테스트 할 것
    * API 문서 만들기
        * 요청 문서화
        * 응답 문서화
        * 링크 문서화
        * profile 링크 추가
 
### 스프링 REST Docs 각종 문서 조각 생성하기
* 요청 필드 문서화, 요청/응답의 필드들에 대한 설명 추가
    * requestFields() + filedWithPath()
    * responseFields() + fieldWithPath()
    * requestHeaders() + headerWithName()
    * responseHeaders() + headerWithName()
    * links() + linkWhtRel()
    * relaxedResponseFields() // 인자에 정의된 필드에 대해서만 검사하고 추가로 들어오는 필드는 검증에서 무시, links가 response body에 들어오게 되어서 responseFields()를 사용하면 에러 발생
* 테스트 할 것
    * API 문서 만들기
        * 링크 문서화
            * self
            * query-events
            * update-event
            * profile 링크 추가
        * 요청 헤더 문서화
        * 요청 필드 문서화
        * 응답 헤더 문서화
        * 응답 필드 문서화
 
### 스프링 REST Docs 문서 빌드
* 스프링 REST Docs
    * pom.xml / build.gradle에 asciidoctor plugin 추가
    * main/asciidoc/index.adoc 추가 : https://gitlab.com/whiteship/natural/-/blob/master/src/main/asciidoc/index.adoc
    * mvn package
        * test 실행 후 packaging
        * test 실행 중 문서 조각들이 생성됨
        * asciidoctor plugin이 spring boot가 지원하는 static directory에 문서 페이지를 두어 view에서 볼 수 있게 해준다 (target/generated-docs/index.html이 생성됨)
    * gradle에서 사용하기
        * plugin documents :https://asciidoctor.github.io/asciidoctor-gradle-plugin/development-3.x/user-guide/
        * 설정 방법
   ```groovy
   // build.gradle
   plugins { id 'org.asciidoctor.jv.convert' version '3.1.0' }
   asciidoctor {
      sourceDir file('src/main/asciidoc') // 기본 directory 위치는 src/docs/asciidoc이고, 강의에서는 src/main/asciidoc/index.adoc을 두기 때문에 수정, reference : https://asciidoctor.github.io/asciidoctor-gradle-plugin/development-3.x/user-guide/#_task_configuration
   }
   ```
* document 생성
    * ./gradlew asciidoctor
* 테스트
    * profile 링크 추가
    * code
   ```java
   //EventController
   @PostMapping
   public ResponseEntity createEvent(...) {
      ...
      eventResource.add(new Link("/docs/index.html#resources-events-create).withRel("profile"));
      ...
   }
   //EventControllerTest
   @Test
   public void createEvent() {
      ...
      document(
         links(
            ...
            linkWithRel("profile").description("Link to profile");
         ),
         ...
         responseFields(
            ...
            filedWithPath("_links.profile.href").description("Link to profile");
         )
   }
   ```
### 테스트용 DB와 설정 분리하기
* 테스트 할 때 계속 H2를 사용해도 좋지만, 애플리케이션 서버를 실행할 때 PostgreSQL을 사용하도록 변경하자
* /scripts.md 참고 (https://gitlab.com/whiteship/natural/-/blob/master/scripts.md)
    * PostgreSQL 드라이버 의존성 추가
        * groupId: org.postgresql, name: postgresql
    * Docker로 PostgreSQL 컨테이너 실행
        * docker run --name ndb -p 5432:5432 -e POSTGRES_PASSWORD=pass -d postgres
    * Docker container 접속
   ```bash
   docker exec -ti ndb bash
   su - postgres
   psql -d postgres -U postgres
   \l
   \dt
   ```
* datasource 설정
```
# application.properties
spring.datasource.username=postgres
spring.datasource.password=pass
spring.datasource.url=jdbc:postgresql://localhost:5432/postgres
spring.datasource.driver-class-name=org.postgresql.Driver
```
* hibernate 설정
```
spring.jpa.hibernate.ddl-auto=create-drop
spring.jpa.properties.hibernate.jdbc.lob.non_contextual_creation=true
spring.jpa.properteis.hibernate.format_sql=true
loging.level.org.hibernate.SQL=DEBUG
loging.level.org.hibernate.type.descriptor.sql.BasicBinder=TRACE
```
* 애플리케이션 설정과 테스트 설정 중복은 어떻게 줄일 것인가?
    * test code에서 @ActiveProfile("test") 이용
    * test/java/resourcees/application-test.properties 생성
```
#application-test.properties
spring.datasource.username=sa
spring.datasource.password=
spring.datasource.url=jdbc:h2:mem:testdb
spring.datasource.driver-class-name=org.h2.Driver
spring.datasource.hikari.jdbc-url=jdbc:h2:mem:testdb
spring.jpa.properties.hibernate.dialect=org.hibernate.dialect.H2Dialect
```

### API 인덱스 만들기
* 인덱스 만들기
    * 다른 리소스에 대한 링크 제공
    * 문서화
* code
```java
// IndexController.java
@GetMapping("/api")
public ResourceSupport root() {
   ResourceSupport index = new ResourceSupport();
   index.add(linkTo(EventController.class).withRel("events"));
   return index;
}
// ErrorsResource.java, To add link of index page when handling Errors
public class ErrorsResource extends EntityModel<Errors> {
    public ErrorsResource(Errors content, Link... links) {
        super(content, links);
        add(linkTo(methodOn(IndexController.class).index()).withRel("index"));
    }
}
// ErrorControllor.java
... // in createEvent method
if (errors.hasErros()) {
   return badRequest(errors);
}
private ResponseEntity badRequest(Errors errors) {
    return ResponseEntity.badRequest().body(new ErrorsResource(errors));
}
// EventControllerTest.java
... // in badRequest test
mockMvc.perform(post(...))
   ...
.andExpect(jsonPath("_links.index").exist()
```
## 이벤트 조회 및 수정 REST API 개발

### 이벤트 목록 조회 API 구현
* 페이징, 정렬 어떻게 하지?
    * 스프링 데이터 JPA가 제공하는 Pageable
* page link 추가, Page<Event>에 안에 들어있는 Event들을 각각의 리소스로 변환
    * PagedResourceAssembler<T> 사용하기
    * PagedResourceAssembler를 이용하여 PagedModel을 생성하면 page에 관련된 links(현재 페이징의 앞/뒤 링크)를 생성해 준다
    * assembler.toModel() 의 두 번째 인자에 람다식으로 각 Event를 변환해 줄 수 있는데 이 기능으로 Event -> EventResource로 변환하여 links를 추가한다
    * code
   ```java
   // in EventController
   @GetMapping
   public ResponseEntity queryEvents(Pageable pageable, PagedResourcesAssembler<Event> assembler) {
       Page<Event> page = eventRepository.findAll(pageable);
       PagedModel<EntityModel<Event>> pagedResources = assembler.toModel(page, e -> new EventResource(e)
   );

       return ResponseEntity.ok(pagedResources);
   }
   ```
 
* 테스트 할 때 Pageable parameter 제공하는 방법
    * page: 0부터 시작
    * size: default 20
    * sort: property, peoperty(,ASC|DESC)
 
* 테스트 할 것
    * Event 목록 Page 정보와 함께 받기
        * content[0].id 확인
        * pageable 경로 확인
    * Sort와 Paging 확인
        * 30개를 만들고 10개 사이즈로 두 번쨰 페이지를 조회하면 이전/다음 페이지로 가는 링크 필요
        * 이벤트 이름 순 정렬
        * page 관련 링크
    * Event를 EventResource로 변환해서 받기
        * 각 이벤트 마다 self
    * 링크 확인
        * self
        * profile
        * (create)
    * 문서화
 
### 이벤트 조회 API 구현
* 테스트 할 것
    * 조회하는 이벤트가 있는 경우 이벤트 리소스 확인
        * 링크
            * Self
            * Profile
            * (update)
        * 이벤트 데이터
    * 조회하는 이벤트가 없는 경우 404 응답 확인
 
### 이벤트 수정 API 구현
* 테스트 할 것
    * 수정하려는 이벤트가 없는 경우 404 NOT_FOUND
    * 입력 데이터(데이터 바인딩)가 이상한 경우에 400 BAD_REQUEST
    * 도메인 로직으로 데이터 검증 실패하면 400 BAD_REQUEST
    * (권한이 충분하지 않은 경우에 403 FORBIDDEN)
    * 정상적으로 수정한 경우에 이벤트 리소스 응답
        * 200 OK
        * 링크
        * 수정한 이벤트 데이터
 
### 테스트 코드 리팩토링
* 여러 컨트롤러 간의 중복 코드 제거하기
    * 클래스 상속을 이용하는 방법
    * @Ignore annotation으로 테스트로 간주되지 않도록 설정
    * BaseControllerTest에 공통으로 사용할 annotation, fields를 정의하고 이걸 상속하여 다른 테스트에 사용. BaseControllerTest는 test가 아니므로 @Ignore annotation을 붙여준다

## REST API 보안 적용
### Account 도메인 추가
* OAuth2로 인증하려면 일단 Account부터
    * Id
    * Email
    * Password
    * Roles
* AccountRoles
    * ADMIN, USER
* JPA mapping
    * @Table
* JPA enumeration collection mapping
    * @ElementCollection(fech = FetchType.EAGER)
    * @Enumerated(EnumType.STRING)
    * Private Set<AccountRole> roles;
* Event에 owner 추가
    * @ManyToOne
    * Account manager 
 
### 스프링 시큐리티 적용
* 스프링 시큐리티
    * 웹 시큐리티 (Filter 기반 시큐리티)
    * 메소드 시큐리티
    * 이 둘 다 Security Interceptor를 사용
        * 리소스에 접근을 허용할 것이냐 말것이냐를 결정하는 로직이 들어있음

* SecurityContextHolder가 인증 정보를 가지고 있고 꺼내 쓸 수 있다
* 인증 정보가 없다면 AuthentificationManger를 이용하여 로그인을 해야한다
    * 예를 들어 basic auth라면 사용자가 입력한 username, password를 인코딩한 값을 헤더에 넣어서 전달해주는데, UserDetailsService에서 username에 매칭되는 password를 db 등에서 가져와서 PasswordEncoder로 검사를 해서 매칭이 되면 로그인이 되는 것이고, 이를 SecurityContextHolder에 저장한다
* 로그인이 되어있다면 사용자의 권한을 AccessDecisionManager로 확인한다. 확인은 Role을 이용한다
* 의존성 추가
    * group:'org.springframework.security.oauth.boot', name:'spring-security-oauth2-autoconfigure'
    * 테스트가 다 깨지게 된다. 401 Unauthorized에 의해
* UserDetailsService 구현
    * 예외 테스트하기
        * expected
        * @Rule ExpectException
        * try-catch
    * asserThat(collection).extracting(GrantedAuthority::getAuth
    * code
   ```java
   // AccountService
   @Service
   public class AccountService implements UserDetailsService {

       @Autowired
       AccountRepository accountRepository;

       @Override
       public UserDetails loadUserByUsername(String username) throws UsernameNotFoundException {
           Account account = accountRepository.findByEmail(username)
                   .orElseThrow(() -> new UsernameNotFoundException(username));
           return new User(account.getEmail(), account.getPassword(), authorities(account.getRoles()));
       }

       private Collection<? extends GrantedAuthority> authorities(Set<AccountRole> roles) {
           return roles.stream()
                   .map(r -> new SimpleGrantedAuthority("ROLE_" + r.name()))
                   .collect(Collectors.toSet());
       }
   }
   ```
 
### 예외 테스트
* @Test(SomeExpectedException.class)
    * 예외 타입만 확인 가능
* try-catch
    * 예외 타입과 메시지 확인 가능
    * 다만 코드가 복잡해짐
* @Rule ExpectedException
    * 코드는 간결하면서 예외 타입과 메시지 모두 확인 가능
* code
```java
	@Rule
	public ExpectedException expectedException = ExpectedException.none();
	…
	@Test
	public void findByUsernameFail() {
	    // Expected
	    String username = "random@email.com";
	    expectedException.expect(UsernameNotFoundException.class);
	    expectedException.expectMessage(Matchers.containsString(username));
	
	    // When
	    accountService.loadUserByUsername(username);
}
```

### 스프링 시큐리티 기본 설정
* 시큐리티 필터를 적용하기 않음...
    * /docs/index.html
* 로그인 없이 접근 가능
    * GET /api/events
    * GET /api/events/{id}
* 로그인 해야 접근 가능
    * 나머지 다...
    * POST /api/events
    * PUT /api/events/{id{
    * ...
* 스프링 시큐리티 OAuth 2.0
    * AuthorizationServer: OAuth2 토큰 발행(/oauth/token) 및 토큰 인증(/oauth/authorize)
        * Oder 0 (리소스 서버 보다 우선 순위가 높다.)
    * ResourceServer: 리소스 요청 인증 처리 (OAuth 2 토큰 검사)
        * Oder 3 (이 값은 현재 고칠 수 없음)
* 스프링 시큐리티 설정
    * @EnableWebSecurity
    * @EnableGlobalMethodSecurity
    * extends WebSecurityConfigurerAdapter
    * PasswordEncoder: PasswordEncoderFactories.createDelegatingPassworkEncoder()
    * TokenStore: InMemoryTokenStore
    * AuthenticationManagerBean
    * configure(AuthenticationManagerBuidler auth)
        * userDetailsService
        * passwordEncoder
    * configure(HttpSecurity http)
        * /docs/**: permitAll
    * configure(WebSecurty web)
        * ignore 
            * /docs/**
            * /favicon.ico
    * PathRequest.toStaticResources() 사용하기
```java
@Configuration
@EnableWebSecurity
public class SecurityConfig extends WebSecurityConfigurerAdapter {

    @Autowired
    AccountService accountService;

    @Autowired
    PasswordEncoder passwordEncoder;

    @Bean
    public TokenStore tokenStore() {
        return new InMemoryTokenStore();
    }

    @Bean
    @Override
    public AuthenticationManager authenticationManagerBean() throws Exception {
        return super.authenticationManagerBean();
    }

    @Override
    protected void configure(AuthenticationManagerBuilder auth) throws Exception {
        auth.userDetailsService(accountService)
                .passwordEncoder(passwordEncoder);
    }

    @Override
    public void configure(WebSecurity web) throws Exception {
        web.ignoring().mvcMatchers("/docs/index.html");
        web.ignoring().requestMatchers(PathRequest.toStaticResources().atCommonLocations());
    }
}
```
### 스프링 시큐리티 폼 인증 설정
```java
// config
@Override
protected void configure(HttpSecurity http) throws Exception {
    http
            .anonymous()
                .and()
            .formLogin()
                .and()
            .authorizeRequests()
                .mvcMatchers(HttpMethod.GET, "/api/**").anonymous()
                .anyRequest().authenticated()
            ;
}

//AccountService
@Service
public class AccountService implements UserDetailsService {

    @Autowired
    AccountRepository accountRepository;

    @Autowired
    PasswordEncoder passwordEncoder;

    public Account saveAccount(Account account) {
        account.setPassword(this.passwordEncoder.encode(account.getPassword()));
        return this.accountRepository.save(account);
    }
    ...
}

// + test수정, ApplicationRunner로 유저추가
```

* 익명 사용자 사용 활성화
* 폼 인증 방식 활성화
    * 스프링 시큐리티가 기본 로그인 페이지 제공
* 요청에 인증 적용
    * /api 이하 모든 GET 요청에 인증이 필요함. (permitAll()을 사용하여 인증이 필요없이 익명으로 접근이 가능케 할 수 있음)
    * 그밖에 모은 요청도 인증이 필요함.

### 스프링 시큐리티 OAuth 2 설정: 인증 서버 설정
```xml
<dependency>
    <groupId>org.springframework.security</groupId>
    <artifactId>spring-security-test</artifactId>
    <version>${spring-security.version}</version>
    <scope>test</scope>
</dependency>
```
* 토큰 발행 테스트
    * User
    * Client
    * POST /oauth/token
        * HTTP Basic 인증 헤더 (클라이언트 아이디 + 클라이언트 시크릿)
        * 요청 매개변수 (MultiValuMap<String, String>)
            * grant_type: password
            * username
            * password
        * 응답에 access_token 나오는지 확인
* Grant Type: Password
    * Granty Type: 토큰 받아오는 방법
    * 서비스 오너가 만든 클라이언트에서 사용하는 Grant Type
    * https://developer.okta.com/blog/2018/06/29/what-is-the-oauth2-password-grant
* AuthorizationServer 설정
    * @EnableAuthorizationServer
    * extends AuthorizationServerConfigurerAdapter
    * configure(AuthorizationServerSecurityConfigurer security)
        * PassswordEncode 설정
    * configure(ClientDetailsServiceConfigurer clients)
        * 클라이언트 설정
        * grantTypes
            * password
            * refresh_token
        * scopes
        * secret / name
        * accessTokenValiditySeconds
        * refreshTokenValiditySeconds
    * AuthorizationServerEndpointsConfigurer
        * tokenStore
        * authenticationMaanger
        * userDetailsService
 ```java
@Configuration
@EnableAuthorizationServer
public class AuthServerConfig extends AuthorizationServerConfigurerAdapter {

    @Autowired
    PasswordEncoder passwordEncoder;

    @Autowired
    AuthenticationManager authenticationManager;

    @Autowired
    AccountService accountService;

    @Autowired
    TokenStore tokenStore;
    @Override
    public void configure(AuthorizationServerSecurityConfigurer security) throws Exception {
        security.passwordEncoder(passwordEncoder);
    }

    @Override
    public void configure(ClientDetailsServiceConfigurer clients) throws Exception {
        clients.inMemory()
                .withClient("myApp")
                .authorizedGrantTypes("password", "refresh_token")
                .scopes("read", "write")
                .secret(this.passwordEncoder.encode("pass"))
                .accessTokenValiditySeconds(10 * 60)
                .refreshTokenValiditySeconds(6 * 10 * 60)
                ;
    }

    @Override
    public void configure(AuthorizationServerEndpointsConfigurer endpoints) throws Exception {
        endpoints.authenticationManager(authenticationManager)
            .userDetailsService(accountService)
            .tokenStore(tokenStore)
        ;
    }
}

//Test
public class AuthServerConfigTest extends BaseControllerTest {

    @Autowired
    AccountService accountService;

    @Test
    @TestDescription("인증 토큰을 발급 받는 테스트")
    public void getAuthToken() throws Exception {
        String username = "jingyu.im@nhnpayco.com";
        String password = "920307";
        Account account = Account.builder()
                .email(username)
                .password(password)
                .roles(Set.of(AccountRole.ADMIN, AccountRole.USER))
                .build();
        this.accountService.saveAccount(account);

        String clientId = "myApp";
        String clientSecret = "pass";

        this.mockMvc.perform(post("/oauth/token")
                    .with(httpBasic(clientId, clientSecret))
                    .param("username", username)
                    .params("password", password)
                    .param("grant_type", "password")
        )
                .andDo(print())
                .andExpect(status().isOk())
                .andExpect(jsonPath("access_token").exists());
    }
}
```

### 스프링 시큐리티 OAuth 2 설정: 리소스 서버 설정
* 테스트 수정
    * GET을 제외하고 모두 엑세스 토큰을 가지고 요청 하도록 테스트 수정
* ResourceServer 설정
    * @EnableResourceServer
    * extends ResourceServerConfigurerAdapter
    * configure(ResourceServerSecurityConfigurer resources)
        * 리소스 ID
    * configure(HttpSecurity http)
        * anonymous
        * GET /api/** : permit all
        * POST /api/**: authenticated
        * PUT /api/**: authenticated
        * 에러 처리
            * accessDeniedHandler(OAuth2AccessDeniedHandler())

```java
@Configuration
@EnableResourceServer
public class ResourceServerConfig extends ResourceServerConfigurerAdapter {

    @Override
    public void configure(ResourceServerSecurityConfigurer resources) throws Exception {
        resources.resourceId("event");
    }

    @Override
    public void configure(HttpSecurity http) throws Exception {
        http
                .anonymous()
                    .and()
                .authorizeRequests()
                    .mvcMatchers(HttpMethod.GET, "/api/**")
                        .anonymous()
                    .anyRequest()
                        .authenticated()
                    .and()
                .exceptionHandling()
                    .accessDeniedHandler(new OAuth2AccessDeniedHandler())
                ;
    }
}


// TEST
    @Before
    public void setUp() {
        this.eventRepository.deleteAll();
        this.accountRepository.deleteAll();
    }

    private String getBearerToken() throws Exception {
        return "Bearer " + getAccessToken();
    }

    private String getAccessToken() throws Exception {
        String username = "jingyu.im@nhnpayco.com";
        String password = "920307";
        Account account = Account.builder()
                .email(username)
                .password(password)
                .roles(Set.of(AccountRole.ADMIN, AccountRole.USER))
                .build();
        this.accountService.saveAccount(account);

        String clientId = "myApp";
        String clientSecret = "pass";

        ResultActions perform = this.mockMvc.perform(post("/oauth/token")
                    .with(httpBasic(clientId, clientSecret))
                    .param("username", username)
                    .params("password", password)
                    .param("grant_type", "password")
        );
        String resultString = perform.andReturn().getResponse().getContentAsString();
        Jackson2JsonParser jackson2JsonParser = new Jackson2JsonParser();
        return jackson2JsonParser.parseMap(resultString).get("access_token").toString();
    }

// ex
mockMvc.perform(post("/api/events")
        .header(HttpHeaders.AUTHORIZATION, getBearerToken())
        .contentType(MediaType.APPLICATION_JSON)
        .accept(MediaTypes.HAL_JSON)
        .content(objectMapper.writeValueAsString(eventDto)))
    .andDo(print())

```

### 문자열을 외부 설정으로 빼내기
* 기본 유저 만들기
    * ApplicationRunner
        * Admin
        * User
* 외부 설정으로 기본 유저와 클라이언트 정보 빼내기
    * @ConfigurationProperties

```java
@Component
@ConfigurationProperties(prefix = "my-app")
@Getter @Setter
public class AppProperties {
    @NotEmpty
    private String adminUsername;

    @NotEmpty
    private String adminPassword;

    @NotEmpty
    private String userUsername;

    @NotEmpty
    private String userPassword;

    @NotEmpty
    private String clientId;

    @NotEmpty
    private String clientSecret;
}

//properties
my-app.admin-username=admin@email.com
my-app.admin-password=admin
my-app.user-username=user@email.com
my-app.user-password=user
my-app.client-id=myApp
my-app.client-secret=pass
```
### 이벤트 API 점검
* 토큰 발급 받기
    * POST /oauth/token
    * BASIC authentication 헤더
        * client Id(myApp) + client secret(pass)
    * 요청 본문 폼
        * username: admin@email.com
        * password: admin
        * grant_type: password
* 토큰 갱신하기
    * POST /oauth/token
    * BASIC authentication 헤더
        * client Id(myApp) + client secret(pass)
    * 요청 본문 폼
        * token: 처음에 발급받았던 refersh 토큰
        * grant_type: refresh_token
* 이벤트 목록 조회 API
    * 로그인 했을 때
        * 이벤트 생성 링크 제공
* 이벤트 조회
    * 로그인 했을 때
        * 이벤트 Manager인 경우에는 이벤트 수정 링크 제공 

### 스프링 시큐리티 현재 사용자
* SecurityContext
    * 자바 ThreadLocal 기반 구현으로 인증 정보를 담고 있다.
    * 인증 정보 꺼내는 방법: Authentication authentication =
* SecurityContextHolder.getContext().getAuthentication();
* @AuthenticationPrincipal spring.security.User user
    * 인증 안한 경우에 null
    * 인증 한 경우에는 username과 authorities 참조 가능
* spring.security.User를 상속받는 클래스를 구현하면
    * 도메인 User를 받을 수 있다.
    * @AuthenticationPrincipa me.whiteship.user.UserAdapter
    * Adatepr.getUser().getId()
* SpEL을 사용하면
    * @AuthenticationPrincipa(expression=”account”) me.whiteship.user.Account
* 커스텀 애노테이션을 만들면
    * @CurrentUser Account account
    ```java
    @Target(ElementType.PARAMETER)
    @Retention(RetentionPolicy.RUNTIME)
    @AuthenticationPrincipal(expression = "account")
    public @interface CurrentUser { } 
    ```

    ```java
    //Controller
    @GetMapping
    public ResponseEntity queryEvents(Pageable pageable, PagedResourcesAssembler<Event> assembler,
                                      @CurrentUser Account account) {
    //        Authentication authentication = SecurityContextHolder.getContext().getAuthentication();
    //        User principal = (User)authentication.getPrincipal();
    
        Page<Event> page = this.eventRepository.findAll(pageable);
        var pageEntityModel = assembler.toModel(page);
    //        var pageEntityModel = assembler.toModel(page, e -> new EventEntityModel(e)); // TODO _links.self ...
        pageEntityModel.add(new Link("/docs/index.html#resources-events-list").withRel("profile"));
        if(account != null) {
            pageEntityModel.add(linkTo(EventController.class).withRel("create-event"));
        }
        return ResponseEntity.ok(pageEntityModel);
    }
    
    //AccountAdapter
    public class AccountAdapter extends User {
    
        private Account account;
    
        public AccountAdapter(Account account) {
            super(account.getEmail(), account.getPassword(), authorities(account.getRoles()));
            this.account = account;
        }
    
        private static Collection<? extends GrantedAuthority> authorities(Set<AccountRole> roles) {
            return roles.stream()
                    .map(r -> new SimpleGrantedAuthority("ROLE_" + r.name()))
                    .collect(Collectors.toSet());
        }
    
        public Account getAccount() {
            return account;
        }
    }
    
    // AccountService
    
    ```

* 엇? 근데 인증 안하고 접근하면..?
    * expression = "#this == 'anonymousUser' ? null : account"
    * 현재 인증 정보가 anonymousUse 인 경우에는 null을 보내고 아니면 “account”를 꺼내준다.
* 조회 API 개선
    * 현재 조회하는 사용자가 owner인 경우에 update 링크 추가 (HATEOAS)
* 수정 API 개선 현재 사용자가 이벤트 owner가 아닌 경우에 403 에러 발생

### Events API 개선: 출력값 제한하기
* 생성 API 개선
    * Event owner 설정
    * 응답에서 owner의 id만 보내 줄 것.
    ```json
    {
        "id" : 4, 
        "name" : "test 3PISM1Ju", 
        "description" : "test event", 
        ... 
        "free" : false,
        "eventStatus" : "DRAFT", 
        "owner" : { "id" : 3, "email" : "keesun@email.com", "password" : "{bcrypt}$2a$10$3z/rHmeYsKpoOQR3aUq38OmZjZNsrGfRZxSnmpLfL3lpLxjD5/JZ6", "roles" : [ "USER", "ADMIN" ]  }, 
    }
    ```
    * JsonSerializer<User> 구현
    * @JsonSerialize(using) 설정
```java
//AccountSerializer
public class AccountSerializer extends JsonSerializer<Account> {
    @Override
    public void serialize(Account account, JsonGenerator jsonGenerator, SerializerProvider serializerProvider) throws IOException {
        jsonGenerator.writeStartObject();
        jsonGenerator.writeNumberField("id", account.getId());
        jsonGenerator.writeEndObject();
    }
}

// Event
@ManyToOne
@JsonSerialize(using = AccountSerializer.class)
private Account manager;
```

## 보강
### 깨진 테스트 살펴보기
* EventControllerTests.updateEvent()
    * 깨지는 이유: NullPointerException
    * 해결 방법: 코드 수정
* EventControllerTests.getEvent()
    * 깨지는 이유: NullPointerException
    * 해결 방법: 코드 수정
* DemoApplicationTests
    * 깨지는 이유: 테스트에서 H2가 아닌 PostgreSQL 사용하려 하지만, PostgreSQL이 동작중이지 않음.
    * 해결 방법: 해당 테스트에서 H2를 사용하도록 test 프로파일 설정.

### 스프링부트 업그레이드