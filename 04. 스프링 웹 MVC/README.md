# 04. 스프링 웹 MVC
## 섹션 0. 소개
## 섹션 1. 스프링 MVC 동작 원리

### 스프링 MVC 동작원리 정리
* 결국엔 (굉장히 복잡한) 서블릿.
    = DispatcherServlet
* DispatcherServlet 초기화
    1. 특정 타입에 해당하는 빈을 찾는다.
    2. 없으면 기본 전략을 사용한다. (DispatcherServlet.properties)
* 스프링 부트 사용하지 않는 스프링 MVC
    * 서블릿 컨네이너(ex, 톰캣)에 등록한 웹 애플리케이션(WAR)에 DispatcherServlet을 등록한다.
        * web.xml에 서블릿 등록
        * 또는 WebApplicationInitializer에 자바 코드로 서블릿 등록 (스프링 3.1+, 서블릿 3.0+)
    * 세부 구성 요소는 빈 설정하기 나름.
* 스프링 부트를 사용하는 스프링 MVC
    * 자바 애플리케이션에 내장 톰캣을 만들고 그 안에 DispatcherServlet을 등록한다.
        * 스프링 부트 자동 설정이 자동으로 해줌.
    * 스프링 부트의 주관에 따라 여러 인터페이스 구현체를 빈으로 등록한다.
    
## 섹션 2. 스프링 MVC 설정
### 스프링 MVC 구성 요소 직접 빈으로 등록하기
@Configuration을 사용한 자바 설정 파일에 직접 @Bean을 사용해서 등록하기
```java
@Configuration
@ComponentScan
public class WebConfig {

    @Bean
    public HandlerMapping handlerMapping() {
        RequestMappingHandlerMapping handlerMapping = new RequestMappingHandlerMapping();
        handlerMapping.setInterceptors();
        handlerMapping.setOrder(Ordered.HIGHEST_PRECEDENCE);
        return handlerMapping;
    }

    @Bean
    public HandlerAdapter handlerAdapter() {
        RequestMappingHandlerAdapter handlerAdapter = new RequestMappingHandlerAdapter();
        handlerAdapter.setArgumentResolvers(~);
        handlerAdapter.setMessageConverters(~);
        return handlerAdapter;
    }
}
```
-> 이렇게 설정하는 것은 상당히 low한 방법.. 쓸일 없을껄?ㅎ <br/>
-> 요렇게 설정하면, 설정한건 여기꺼 쓰이고, 설정하지 않은건 DispatcherServlet의 기본 설정을 따른다.

### @EnableWebMvc
애노테이션 기반 스프링 MVC를 사용할 때 편리한 웹 MVC 기본 설정
```java
@Configuration
@ComponentScan
@EnableWebMvc
public class WebConfig {
    
}
```
-> @EnableWebMvc > DelegatingWebMvcConfiguration > WebMvcConfigurationSupport 에 빈 설정이 되어 있다. <br/>
-> 기본 interceptor, 의미있는 우선순위(order), 더 많은 컨버터 .. 등이 추가로 설정되어 있음. 더 좋다. 
-> Delegate : 하나의 객체가 다른 객체를 대신해서 동작 또는 조정할 수 있는 기능을 제공

### WebMvcConfigurer 인터페이스
@EnableWebMvc가 제공하는 빈을 커스터마이징할 수 있는 기능을 제공하는 인터페이스
```java
@Configuration
@EnableWebMvc
public class WebConfig implements WebMvcConfigurer {
    @Override
    public void configureViewResolvers(ViewResolverRegistry registry) {
        registry.jsp("/WEB-INF/", ".jsp");
    }
}
```
-> WebMvcConfigurer를 상속받아 사용가능. (스프링 3.1부터)
-> 지금까지 이게 부트 없이 사용하는 방법. 이후에 부트사용해서..
-> 스프링에서 제공하는지 부트에서 제공하는지 경계를 알고 사용하는게 도움이 된다...ㅎ

### 스프링 부트의 스프링 MVC 설정
![17](./img/17.PNG)
* 스프링 부트의 “주관”이 적용된 자동 설정이 동작한다.
    * JSP 보다 Thymeleaf 선호
    * JSON 지원
    * 정적 리소스 지원 (+ 웰컴 페이지, 파비콘 등 지원)
* 스프링 MVC 커스터마이징
    * application.properties < 요 방식이 요즘 주로 사용하는 방식!
    * @Configuration + Implements WebMvcConfigurer: 스프링 부트의 스프링 MVC 자동설정 + 추가 설정
    * @Configuration + @EnableWebMvc + Imlements WebMvcConfigurer : 스프링 부트의스프링 MVC 자동설정 사용하지 않음.

-> EnableAutoConfiguration에 기술되어 있는 모든 자동설정들이 자동으로 등록된다. (부트 수업에 있었던 내용) <br/>
-> 자동설정된 내용을 기반으로 `application.properties`에 prefix만 맞춰서 설정해주면 커스터마이징해서 사용 가능!
 
### 스프링 부트에서 JSP 사용하기
```
“If possible, JSPs should be avoided. There are several known limitations when using them with embedded servlet containers.” 
```
* 제약 사항
    * JAR 프로젝트로 만들 수 없음, WAR 프로젝트로 만들어야 함
    * Java -JAR로 실행할 수는 있지만 “실행가능한 JAR 파일”은 지원하지 않음
    * 언더토우(JBoss에서 만든 서블릿 컨테이너)는 JSP를 지원하지 않음
    * Whitelabel 에러 페이지를 error.jsp로 오버라이딩 할 수 없음.

* 의존성 추가
```xml
<dependency>
    <groupId>javax.servlet</groupId>
    <artifactId>jstl</artifactId>
</dependency>
<dependency>
    <groupId>org.apache.tomcat.embed</groupId>
    <artifactId>tomcat-embed-jasper</artifactId>
    <scope>provided</scope>
</dependency>
```
* 태그 선언
```html
<%@ taglib prefix="c" uri="http://java.sun.com/jsp/jstl/core"%>
```
* application.properties
```text
spring.mvc.view.prefix=/WEB-INF/jsp
spring.mvc.view.suffix=.jsp=
```

-> JAR/WAR 찾아보기 / maven... 프로젝트 빌드/배포 과정 살펴보기 - pom.xml에 core는 jar고, 나머지는 war...(with kksshh525)
-> 로컬에서 띄울때는 war exploded (war랑 차이?)

### WAR 파일 배포하기
![19](./img/19.PNG)

### WebMvcConfigurer 1부 Formatter 추가하기
* Formatter
    * Printer: 해당 객체를 (Locale 정보를 참고하여) 문자열로 어떻게 출력할 것인가
    * Parser: 어떤 문자열을 (Locale 정보를 참고하여) 객체로 어떻게 변환할 것인가

* 포매터 추가하는 방법 1
    * WebMvcConfigurer의 addFormatters(FormatterRegistry) 메소드 정의
* 포매터 추가하는 방법 2 (스프링 부트 사용시에만 가능 함)
    * 해당 포매터를 빈으로 등록

### ᄃ도메인 클래스 컨버터 자동 등록
* 스프링 데이터 JPA는 스프링 MVC용 도메인 클래스 컨버터를 제공합니다.
* 도메인 클래스 컨버터
    * 스프링 데이터 JPA가 제공하는 Repository를 사용해서 ID에 해당하는 엔티티를 읽어옵니다.
* 의존성 설정
```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-jpa</artifactId>
</dependency>
<dependency>
    <groupId>com.h2database</groupId>
    <artifactId>h2</artifactId>
</dependency>
```
* 엔티티 맵핑
```java
@Entity
public class Person {
    @Id @GeneratedValue
    private Integer id;
...
```
* 리파지토리 추가
```java
public interface PersonRepository extends JpaRepository<Person, Integer> {
}
```
* 테스트 코드 수정
    * 테스트용 이벤트 객체 생성
    * 이벤트 리파지토리에 저장
    * 저장한 이벤트의 ID로 조회 시도

### 핸들러 인터셉터 1부: 개념


### 핸들러 인터셉터 2부: 만들고 등록하기


### 리소스 핸들러

