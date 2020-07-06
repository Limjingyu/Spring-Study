# 02. 스프링 프레임워크 핵심 기술
## 섹션 1. IoC 컨테이너와 빈
### IoC 컨테이너 1부: 스프링 IoC 컨테이너와 빈
* Inversion of Control: 의존 관계 주입(Dependency Injection)이라고도 하며, 어떤 객체가 사용하는 의존 객체를 직접 만들어 사용하는게 아니라, 주입 받아 사용하는 방법
* 스프링 IoC 컨테이너 : 빈 설정 소스로 부터 빈 정의를 읽어들이고, 빈을 구성하고 제공 [BeanFactory](https://docs.spring.io/spring-framework/docs/5.0.8.RELEASE/javadoc-api/org/springframework/beans/factory/BeanFactory.html)
* 빈 : 스프링 IoC 컨테이너가 관리 하는 객체
  * 등록이유?
    * 의존성을 주입해주기 위해
    * 싱글톤으로 관리하기 위해
    * 라이프사이클(->찾아보기)

### IoC 컨테이너 2부: ApplicationContext와 다양한 빈 설정 방법
* [ApplicationContext](https://docs.spring.io/spring-framework/docs/5.0.8.RELEASE/javadoc-api/org/springframework/context/ApplicationContext.html) : ApplicationEventPublisher, BeanFactory, EnvironmentCapable, HierarchicalBeanFactory, ListableBeanFactory, MessageSource, ResourceLoader, ResourcePatternResolver
  * [참고](https://cbw1030.tistory.com/65?category=1067183)
  ```
  BeanFactory는 스프링 설정파일(~~.xml)에 등록된 bean 객체를 생성하고 관리하는 가장 기본적인 컨테이너 기능만을 제공한다. 그리고 컨테이너가 구동될 때 객체를 생성하는 것이 아니라 클라이언트로부터의 요청에 의해서만 객체를 생성한다. 이것은 lay loading이라고도 한다.
  ApplicationContext는 BeanFactory를 확장하여 트랜잭션 관리나 메시지 기반의 다국어 처리 등 다양한 기능을 지원한다. 또한 컨테이너가 구동되는 시점에 bean에 등록되어 있는 클래스들을 객체화하는 즉시 로딩 방식으로 동작한다. 대표적으로 이 ApplicationContext를 구현한 클래스는 ClassPathXmlApplicationContext, FileSystemXmlApplicationContext, XmlWebApplicationContext 등이 있다.
  ```
* 빈 등록 방법
  1. xml 파일 설정
  2. xml, ConponentScan
  3. java, @configuration, @Bean
  4. java @ComponentScan
  5. @SpringBootConfiguration
* @SpringBootApplication = @SpringBootConfiguration + @EnableAutoConfiguration + @ComponentScan + ...
  * @SpringBootConfiguration: 스프링 부트의 설정을 나타내는 어노테이션이다. 스프링의 @Configuration을 대체하며 스프링 부트 전용 어노테이션이다. 테스트 어노테이션을 사용할 때 계속 이 어노테이션을 찾기 때문에 스프링 부트에서는 필수 어노테이션이다.
  * @EnableAutoConfiguration: 자동 설정의 핵심 어노테이션이다. 클래스 경로에 지정된 내용을 기반으로 설정 자동화를 수행한다.
  * @ComponentScan: basePackages 프로퍼티 값에 별도의 경로를 설정하지 않으면 해당 어노테이션이 위치한 패키지가 루트 경로가 된다.
  * [참고](https://velog.io/@adam2/SpringBoot-%EC%9E%90%EB%8F%99-%ED%99%98%EA%B2%BD-%EC%84%A4%EC%A0%95AutoConfiguration)

### IoC 컨테이너 3부: @Autowire
* @Autowire : 의존관계를 자동설정할 때 사용하며 타입을 이용하여 의존하는 객체를 삽입해 준다. (해당 타입의 빈객체가 존재하지 않거나 또는 2개 이상 존재할 경우 스프링은 예외를 발생시킨다.)
  * [참고](https://galid1.tistory.com/512)
  * 주입방법? 
    * 생성자
    * 세터
    * 필드
  * 유니크하지않으면..
    * @Primary(추천 - type safe)
    * @Qualifier
      * (tip: @Qualifier가 @Primary보다 강력)
    * List로 모두 주입받기
* @Resource : @Autowired와 흡사하지만 @Autowired(by type), @Resource(by name)으로 연결한다는 점이 다르다.

### IoC 컨테이너 4부: @Component와 컴포넌트 스캔
* @ComponentScan : 실제 스캐닝은 ConfigurationClassPostProcessor라는 BeanFactoryPostProcessor에 의해 처리 됨.
  * @Component : Spring에서 관리되는 객체임을 표시하기 위해 사용하는 가장 기본적인 annotation이다. 즉, scan-auto-detection과 dependency injection을 사용하기 위해서 사용되는 가장 기본 어노테이션이다.
  * @Controller : 프레젠테이션 레이어.  MVC 코드에 사용되는 어노테이션이다. @RequestMapping 어노테이션을 해당 어노테이션 밑에서만 사용 가능
  * @Repository :퍼시스턴스(persistence) 레이어. 영속성을 가지는 속성(파일, 데이터베이스 등).
  * @Service : 서비스 레이어. 비즈니스 로직이나 respository layer 호출하는 함수에 사용된다. 다른 어노테이션과 다르게 @Component에 추가된 기능은 없다. 하지만 나중에 Spring 측에서 추가적인 exception handling을 해줄 수도 있으니 비즈니스 로직에는 해당 어노테이션을 사용하자.
  * [참고](https://happyer16.tistory.com/entry/Spring-annotation-ServiceControllerComponent-%EC%B0%A8%EC%9D%B4), [참고2](https://namocom.tistory.com/421)
* 펑션을 사용한 빈 등록..? -> @ComponentScan을 대체하는거 아님.
* 빈 라이프사이클 : 객체생성 -> 의존설정 -> 초기화 -> 소멸

### IoC 컨테이너 5부: 빈의 스코프
* 스프링은 직접 싱글톤 형태의 오브젝트를 만들고 관리하는 기능을 제공
  * 싱글톤(default) : 스프링 컨테이너에 하나의 빈 객체만 존재
  * 프로토타입(proxyMode) : 빈을 사용할때마다 새로운 객체를 생성

### IoC 컨테이너 6,7부: Environment 1,2부. 프로파일 / 프로퍼티
* 결국 알파,베타,스테이징,리얼 환경에 설정을 구분할때 사용 -> [정리](https://umbum.dev/1039)
* @profile("")
  * @Configuration에 쓰고 빈등록
  * @Component에 직접
* 프로퍼티 우선순위
  1. ServletConfig 매개변수
  2. ServletContext 매개변수
  3. JNDI (java:comp/env/)
  4. JVM 시스템 프로퍼티 (-Dkey="value")
  5. JVM 시스템 환경 변수
  6. 사용자 프로퍼티 설정파일
* @PropertySource


### IoC 컨테이너 7부: MessageSource
* 메세지 다국화 (필요할때 찾아보고 쓰면 될듯)
* messagess.properties / messagess_ko_kr.properties / ..

### IoC 컨테이너 8부: ApplicationEventPublisher
* 이벤트 프로그래밍에 필요한 인터페이스 제공. 옵저버 패턴 구현체.
* @EventListener
* @Async @EnableAsync
* 옵저버 패턴 찾아보기
* 스프링이 제공하는 기본 이벤트
  * ContextRefreshedEvent: ApplicationContext를 초기화 했더나 리프래시 했을 때 발생.
  * ContextStartedEvent: ApplicationContext를 start()하여 라이프사이클 빈들이 시작
신호를 받은 시점에 발생.
  * ContextStoppedEvent: ApplicationContext를 stop()하여 라이프사이클 빈들이 정지
신호를 받은 시점에 발생.
  * ContextClosedEvent: ApplicationContext를 close()하여 싱글톤 빈 소멸되는 시점에
발생.
  * RequestHandledEvent: HTTP 요청을 처리했을 때 발생.
  
### IoC 컨테이너 9부: ResourceLoader
* 리소스를 읽어오는 기능을 제공하는 인터페이스
* 리소스 읽어오기
  * 파일 시스템에서 읽어오기
  * 클래스패스에서 읽어오기
  * URL로 읽어오기
  * 상대/절대 경로로 읽어오기

## 섹션 2. Resource / Validation
### Resource 추상화
* 구현체
  * UrlResource: java.net.URL 참고, 기본으로 지원하는 프로토콜 http, https, ftp, file, jar.
  * ClassPathResource: 지원하는 접두어 classpath:
  * FileSystemResource
  * ServletContextResource: 웹 애플리케이션 루트에서 상대 경로로 리소스 찾는다.
  * ...
* 주요 메소드
  * getInputStream()
  * exitst()
  * isOpen()
  * getDescription(): 전체 경로 포함한 파일 이름 또는 실제 URL

### Validation 추상화
* 애플리케이션에서 사용하는 객체 검증용 인터페이스
		* Spring MVC에서 주로 사용되지만 다른 레이어에서도 사용 가능한 일반적인 인터페이스
		* Bean Validation (https://docs/jboss.org/hibernate/beanvalidation/spec/2.0/api)
	* Validator 만들기
		* Spring에 들어있는 패키지 가져올 것
		* support : 인자로 넘어오는 클래스가 검증하고자 하는 클래스인지 확인하도록 구현
		* validate : 검증 로직 구현
			 * ValidationUtils를 이용한 예제
				public class EventValidator implements Validator {
					@Override
					public boolean supports(Class<?> clazz) {
						return Event.class.equals(clazz);
					}
					@Override
					public void validate(Object target, Errors errors) {
						ValidationUtils.rejectIfEmptyOrWhitespace(errors, "title", "Default error msg");
					}
				}
		 	 * title 필드가 empty/whitespace면 errors에 에러를 담아준다.
			  * ValidationUtils를 이용하지 않는 예제
			@Override
			public void validate (Object target, Errors errors) {
				Event event = (Event) target;
				if (event.getTitle() == null) {
					errors.reject(String errorCode, object[] errorArgs, String defaultMessage);
				}
			}
* 최근에는 Validator를 직접 만들지 않고, spring boot를 사용한다면 LocalValidatorFactoryBean(스프링 2.0.5 이상부터 빈으로 자동 등록, 스프링이 제공)을 사용할 수 있다. 
		* 지금껏 사용해오던 방식인데, validation을 할 클래스의 필드에 @NotEmpty/@Size/@Min/@Max/@Email 등등을 달아주기만 하면 위에서 처럼 Validator를 구현한 EventValidator를 사용하지 않고도 간단한 검증 로직은 만들어 낼 수 있다.
		* 더 복잡한 validation은 직접 구현해야 한다
		@Component
		public class AppRunner implements ApplicationRunner {
			@Autowired
			Validator validator; // 위에서 설명한 Spring이 제공하는 Validator.
			
			public void run(ApplicationArguments args) throws Exception {
				Event event = new Event();
				Errors errors = new BeanPropertyBindingResult(event, "event"); // 직접 사용할 일은 없다
				validator.validate(event, errors);
				 // event에 @NotNull등과 같은 validation 어노테이션을 달아두었고, errors.hasErrors()가 true를 리턴한다. 
			}
		}
 


--- 
## TODO
* 빈 라이프사이클 상세하게 분석해보기(소멸 기준은?)
