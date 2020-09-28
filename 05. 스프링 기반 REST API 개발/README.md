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
 
