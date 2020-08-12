# 03. 스프링 부트 개념과 활용
## 섹션 0. 소개
* 학습 목표
  * 스프링 부트의 핵심 원리를 이해합니다.
  * 스프링 부트가 제공하는 주요 기능을 사용할 수 있습니다.
  * 스프링 부트를 사용하여 웹 애플리케이션을 개발할 수 있습니다.
  * 스프링 부트를 사용하여 여러 데이터 기술과 연동하는 애플리케이션을 개발할 수 있습니다.
  * 스프링 부트 애플리케이션의 운영 정보를 관리하고 모니터링 할 수 있습니다.

## 섹션 1. 스프링 부트 시작하기
### 스프링 부트 소개
* https://spring.io/projects/spring-boot

### 스프링 부트 시작하기
```xml
<? xml version = "1.0"encoding = "UTF-8"?> 
<project  xmlns = "http://maven.apache.org/POM/4.0.0"xmlns  : xsi = "http : //www.w3 .org / 2001 / XMLSchema-instance " 
	xsi : schemaLocation = "http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd " > 
	<modelVersion> 4.0.0 </ modelVersion>

	<groupId> com.example </ groupId> 
	<artifactId> myproject </ artifactId> 
	<version> 0.0.1-SNAPSHOT </ version>

	<!-Spring Boot에서 기본값 상속-> 
	<parent> 
		<groupId> org.springframework.boot </ groupId> 
		<artifactId> spring-boot-starter-parent </ artifactId> 
		<version> 2.0.3.RELEASE < / version> 
	</ parent>

	<!-웹 애플리케이션에 대한 일반적인 종속성 추가-> 
	<dependencies> 
		<dependency> 
			<groupId> org.springframework.boot </ groupId> 
			<artifactId> spring-boot-starter-web </ artifactId> 
		</ dependency> 
	</ dependencies>

	<!-실행 파일로 패키지-> 
	<build> 
		<plugins> 
			<plugin> 
				<groupId> org.springframework.boot </ groupId> 
				<artifactId> spring-boot-maven-plugin </ artifactId> 
			</ plugin > 
		</ plugins> 
	</ build>

</ project>
```

### 스프링 부트 프로젝트 생성기
* https://start.spring.io/

### 스프링 부트 프로젝트 구조
* 메인 애플리케이션 위치 : 기본 패키지만들고 그 밑에

![프로젝트구조](./img/프로젝트구조.PNG)

## 섹션 2. 스프링 부트 원리
### 의존성 관리 이해
* 버전을 명시하지 않아도 알아서 가져온다. (pom.xml parent의 버전에 따라 필요한 버전을 알아서 넣어준다!)
* 내가 특별히 원하는 버전이 있거나 부트에서 관리하지 않는 의존성은 버전을 명시해줘야한다.
```xml
<!-- pom.xml -->
<parent>
	<groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>2.0.3.RELEASE</version>
</parent>

<!-- spring-boot-starter-parent-*.xml : Dependency management, UTF-8 source incoding, plugin configuration, resource filtering(.properties/.yml), ...-->
<parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-dependencies</artifactId>
        <version>2.0.3.RELEASE</version>
        <relativePath>../../spring-boot-dependencies</relativePath>
</parent>

<!-- spring-boot-dependencies-*.xml : 버전정보 있음 -->

```

### 의존성 관리 응용
* https://mvnrepository.com/ 참고
* 버전 관리 해주는 의존성 추가 : pom.xml - <dependencies> 에 추가 
* 버전 관리 안해주는 의존성 추가 : pom.xml - <dependencies> 에 추가 (버전 명시 필수)
* 기존 의존성 버전 변경하기 : pom.xml에 <properties> 에 버전 명시 (덮어쓰는 느낌)

### 자동 설정 이해
* @EnableAutoConfiguration (@SpringBootApplication 안에 숨어 있음)
* 빈은 사실 두 단계로 나눠서 읽힘
	* 1단계: @ComponentScan
	* 2단계: @EnableAutoConfiguration
* @ComponentScan - 해당 어노테이션을 달고있는 패키지 기준으로만 빈으로 등록
	* @Component
	* @Configuration @Repository @Service @Controller @RestController
* @EnableAutoConfiguration
	* spring.factories 파일에 들어있는 내용 모두 자동으로 빈 등록.(org.springframework.boot.autoconfigure.EnableAutoConfiguration)
	* 위에 등록된 애들은 @Configuration 어노테이션을 무조건 달고 있다.
	* but, 모든 애들이 빈으로 등록되는건 아니고, @ConditionalOnXxxYyyZzz 와 같이 조건에 맞을 때만 빈으로 자동 등록해준다.

### 자동 설정 만들기 1부: Starter와 AutoConfigure
* 커스텀한 자동설정 패키지 만들기
	* Xxx-Spring-Boot-Autoconfigure 모듈: 자동 설정
	* Xxx-Spring-Boot-Starter 모듈: 필요한 의존성 정의
	* 그냥 하나로 만들고 싶을 때는?
		* Xxx-Spring-Boot-Starter -> 해당이름으로 프로젝트 만들어서 pom.xml에 정의하여 사용(소스코드가 없는 프로젝트가 될 수도 있음)
* 구현 방법
	1. Configuration 프로젝트 생성
	2. 의존성 추가
	```xml
	<dependencies>
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-autoconfigure</artifactId>
		</dependency>
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-autoconfigure-processor</artifactId>
			<optional>true</optional>
		</dependency>
	</dependencies>
	<dependencyManagement>
		<dependencies>
			<dependency>
				<groupId>org.springframework.boot</groupId>
				<artifactId>spring-boot-dependencies</artifactId>
				<version>2.0.3.RELEASE</version>
				<type>pom</type>
				<scope>import</scope>
			</dependency>
		</dependencies>
	</dependencyManagement>
	```
	3. @Configuration 파일 작성
	4. src/main/resource/META-INF에 spring.factories 파일 만들기
	5. spring.factories 안에 자동 설정 파일 추가
	```
	org.springframework.boot.autoconfigure.EnableAutoConfiguration=\
	FQCN,\
	FQCN
	```
	6. mvn install
	7. ~SNAPSHOP.jar을 사용할 프로젝트에서 의존성추가
* 덮어쓰기 문제 : ComponentScan -> AutoConfiguration 이 수행되서 그렇다 (->다음강의)

### 자동 설정 만들기 2부: @ConfigurationProperties
* 덮어쓰기 방지하기 : @ConditionalOnMissingBean 어노테이션을 써주면 빈이 없을때만 수행한다.
* 빈 재정의 수고 덜기
	* Configuration 프로젝트
		* ~Propertiese 클래스에 @ConfigurationProperties(“holoman”)를 달아줌.
		* ~Configuration 빈에 @EnableConfigurationProperties(HolomanProperties.class)
	* Application 프로젝트 : application.properties에 정의
	* 의문??? : Configuration 프로젝트 Application 프로젝트의 application.properties 값을 어떻게 읽는거지? (밑에얘가 해주는건가..?)
	```xml
	<dependency>
		 <groupId>org.springframework.boot</groupId>
		 <artifactId>spring-boot-configuration-processor</artifactId>
		 <optional>true</optional>
	</dependency>
	```
* **@Conditional~ 어노테이션 찾아보기** 

### 내장 웹 서버 이해
* 스프링 부트는 서버가 아니다. 툴일 뿐이다. 내장된 톰캣을 사용하는 것! (서버는 톰캣,네트,제티,..)
	* 톰캣 객체 생성
	* 포트 설정
	* 톰캣에 컨텍스트 추가
	* 서블릿 만들기
	* 톰캣에 서블릿 추가
	* 컨텍스트에 서블릿 맵핑
	* 톰캣 실행 및 대기
* 이 모든 과정을 보다 상세히 또 유연하고 설정하고 실행해주는게 바로 스프링 부트의 자동 설정.
	* ServletWebServerFactoryAutoConfiguration (서블릿 웹 서버 생성)
		* TomcatServletWebServerFactoryCustomizer (서버 커스터마이징)
	* DispatcherServletAutoConfiguration
		* 서블릿 만들고 등록
* 의문??? : 부트쓰면 서버에 따로 톰캣이 없어도 되는..??

### 내장 웹 서버 응용 1부 : 컨테이너와 포트
* 다른 서블릿 컨테이너로 변경 (톰캣 -> ~) 
* 웹 서버 사용 하지 않기
	* properties에 `spring.main.web-application-type=none` 추가
* 포트
	* properties에 `server.port=7070`
	* 랜덤 포트 `server.port=0`
	* ApplicationListner<ServletWebServerInitializedEvent> 리스너 추가로 포트번호 꺼내기.. (필요할까?)
	
### 내장 웹 서버 응용 2부 : HTTPS와 HTTP2
* **https://opentutorials.org/course/228/4894 정주행+정리 하기**
* HTTPS 설정하기
	* 키스토어 만들기
	* HTTP는 못쓴다. HTTPS를 적용했기때문에 하나만 가능. 
		* 되게하려면? [HTTP 커넥터는 코딩으로 설정하기](https://github.com/spring-projects/spring-boot/tree/v2.0.3.RELEASE/spring-boot-samples/spring-boot-sample-tomcat-multi-connectors)
* HTTP2 설정 : server.http2.enable=true 
	* https가 설정이 되있어야 http2를 설정할 수 있음
	* 사용하는 서블릿 컨테이너 마다 다름.
		* Undertow : 별도 설정 필요 없음
		* 톰캣
			* 버전 8.5.x : 복잡하다... 하지마라!
			* 버전 9.0.x, JDK9 이상에서는 설정없이 사용가능

### 톰캣 HTTP2
* (위에 써놓은 내용)
* **HTTP2란??**

### 독립적으로 실행 가능한 JAR
* spring-boot-maven-plugin의 기능
* mvn clean package # clean : target 하위 디렉토리 결과물 삭제, package : jar 생성
* jar 결과물이 가진 의존성
	* 의존성인 jar들을 전부 가지고 있게 된다
	* target의 결과물을 까보면, app/BOOT-INFO/lib/ 하위에 의존성 jar들이 들어있다
	* spring boot는 의존성 jar를 그대로 읽을 수 있도록 org.springframework.boot.loader.jar.JarFile을 이용한다
	* 의존성 jar를 실행하는 것은 org.springframework.boot.loader.Launcher를 이용한다
	* (참고) 과거에는 uber jar를 이용했는데, jar를 읽을 수 없어서 각 jar들을 다 풀어서 사용했다고 한다

### 스프링 부트 원리 정리
* 의존성 관리
	* spring-boot-starter-web 은 parent를 이용하여 버전 관리 가능
* 자동 설정
* 내장 웹 서버
stand-alone application을 만드는 것이 spring boot의 목적, 내장 웹 서버 기능들이 들어있는 이유도 그 중 하나

## 섹션 3. 스프링 부트 활용
### 스프링 부트 활용 소개
* 스프링 부트 핵심 기능
	* SpringApplication
	* 외부 설정
	* 프로파일
	* 로깅
	* 테스트
	* Spring-Dev-Tools
* 각종 기술 연동
	* 스프링 웹 MVC
	* 스프링 데이터
	* 스프링 시큐리티
	* REST API 클라이언트
	* 다루지 않은 내용들

### SpringApplication
* SpringApplication.run(App.class, args) 로는 커스터마이즈 하기가 어렵다. 인스턴스를 직접 만들면 커스터마이즈 할 수 있다
* FailureAnalyzers : 에러에 대한 로깅을 포매팅해준 것, spring boot에 기본적으로 등록되어 있는 기능
* 배너
	* https://docs.spring.io/spring-boot/docs/current/reference/html/spring-boot-features.html#boot-features-banner
	* 커스터마이즈 방법
		* application 실행할 때 SPRING 큰 글자 모양을 보여주는 것인데, resources/banner.txt를 만들어서 텍스트를 변경 가능
			* ${spring-boot.version} : 버전 정보를 찍을 수 있다. 
			* banner.txt를 다른 위치에 두려면 application.properties에 spring.banner.location=classpath:... 로 가능
		* 코딩으로 하려면 아래와 같이 하자
		```java
		SpringApplication app = new SpringApplication(App.class);
		app.setBanner((env, soureClass, out) -> {
			out.println("==========");
			out.println("whiteship");
			out.println("==========");
		}
		app.run(args);
		```
		* banner.txt가 코딩으로 설정한 것보다 우선순위가 높다
* SpringApplicationBuilder
	* 빌더 패턴 제공
	* 이 방법도 커스터마이징 가능
	```java
	psvm() {
		new SpringApplicationBuilder()
			.source(App.class)
			.run(args);
	}
	```
* ApplicationEvent 등록
	* spring, spring boot 이벤트를 여러 가지 상황에서 제공 : 애플리케이션이 시작될 떄, application context를 만들었을 떄/ 리프레시 될 떄, 애플리케이션이 다 떠서 준비가 되었을 때/ 실패했을 때 등
	* 이벤트 리스너 만들기
	```java
	@Component // listener가 bean으로 등록되면 자동으로 리스너를 실행해 준다. 다만 ApplicationContext가 만들어지기도 전에 발생하는 이벤트는 직접 SpringApplication 인스턴스에 등록해야 한다
	public class SampleListener implements ApplicationListener<ApplicationStartingEvent> {
		@Override
		public void onApplicationEvent(ApplicationStartingEvent event) {
			sout("ApplicationStarting");
		}
	}
	```
	* 근데 위의 이벤트는 ApplicationContext가 만들어지기도 전에 발생하는 이벤트여서 리스너가 동작하지 않는다. 이런 경우에는 bean으로 생성하는 것으로는 자동으로 실행되지 않고 SpringApplication 인스턴스를 만들고, .addListeners(new SampleListener())로 등록해주어야 한다(빈으로 등록하는 것은 의미가 없다)
* WebApplicationType 설정
	* WebApplicationType은 NONE, SERVLET, REACTIVE 세 가지가 있다
	* Spring MVC가 있다면 SERVLET으로 동작
	* Spring Webflux가 있다면 REACTIVE로 동작
	* 둘 다 없다면 NONE
	* 둘 다 있다면 SERVLET이 우선순위가 높다. 그래서 REACTIVE를 쓰려면 직접 설정해주어야 한다
* application arguments 사용하기
	* application arguments : -- 옵션으로 들어오는 것 (intellij에서는 Program arguments)
	* vm option : -D 로 들어오는 것
	* application argument 찍어보기
	```java
	@Component
	public class AppArgs {
		public AppArgs(ApplicationArguments args) {
			sout("foo:" + args.containsOption("foo")); // vm option -D로 설정한 값이었고, false가 찍힘
			sout("bar:" + args.containeOption("bar")); // application arguments로 설정한 값이고, true가 찍힘
		}
	}
	```
* application이 실행된 뒤 추가로 어떤 작업이 필요할 때
	* ApplicationRunner (추천) 또는 CommandLineRunner
		* ApplicationRunner가 더 좋다
		```java
		@Component
		public class AppRunner implements ApplicationRunner {
			@Override
			public void run(ApplicationArguments args) {
				sout("foo:" + args.containsOption("foo"));
				sout("bar:" + args.containeOption("bar"));
			}
		}
		```
		* CommandLineRunner를 이용하면 메소드 구현의 인자가 String ...args로 들어와서 처리하기에 별로..
	* @Order()
		* ApplicationRunner를 여러 개 설정했다면 각 순서를 설정할 수 있다
		* 숫자가 적은 것이 우선순위가 높다
* (참고) 디버그 로그 레벨을 남기는 방법 : 실행 옵션에서 VM options에 -Ddebug를 주거나, Program arguments에 --debug를 사용하면 된다. 또한 디버그 레벨을 찍으면 자동 설정이 된 것들을 보여주고, 자동설정이 동작하지 않았다면 그 이유도 보여준다

### 외부 설정
* application.properties 는 spring에서 설정 파일로 사용하는 규약, key-value로 사용
* 우선순위 : https://docs.spring.io/spring-boot/docs/current/reference/html/spring-boot-features.html#boot-features-external-config
	* command line arguments : java -jar <jar-file-name> --<key>=<value> # 우선순위가 4순위라 application.properties(우선순위 15)를 오버라이팅 한다
	* test용 properties 설정하기
		* 방법 1
			* test/resources/application.properties에 설정
			* intellij의 command+;를 눌러서 Modules 탭에 TestResources 탭, TestResource Folders에 설정
			* 빌드 시 test code도 classpath에 들어가고 test/resources/application.properties가 main/resources/application.properties를 대체한다
			```java
			@RunWith(SpringRunner.class)
			@SpringBootTest
			public class ApplicationTests {
				@Autowired
				Environment env; // org.springframework.core.env.Environment
				@Test
				public void test() {
					assertThat(env.getProperty("keesun.name"))
							.isEqualTo("keesun"); // test/resoucres/application.properties의 설정파일을 사용
				}
			}
			```
			* 문제점 : main/resouces/application.properties를 덮어쓰기 때문에 main에만 설정하고 test에 설정하지 않았을 경우 main의 설정파일에 설정한 값이 사라지게 된다
		* 방법2
			* 우선순위 3
			* @SpringBootTest(properties = "keesun.name=keesun2")
		* 방법3
			* 우선순위 2
			* @TestPropertySource(properties = "keesun.name=keesun3") annotation을 test class 위에 작성
		* 방법1은 파일 자체를 덮어써서 main/resouces/application.properties의 설정값들이 사라지지만, 방법 2,3은 main/resouces/application.properties는 그대로 있고 새로 설정한 값만 덮어쓰게 되어 문제가 없다
		* 방법 2,3에서 프로퍼티가 너무 많다면
			* properties = {"key=value", "key2=value2"}
			* 파일로 관리
				* test/resources/test.properties 생성
				* key=value 들 작성
				* @TestPropertySource(location="classpath:/test/properties")로 변경
		* (참고) application.properties에서 랜덤값 생성하기
			* 우선순위 11에 해당하는 RandomValuePropertySouce 이다
			* ${random.int} ${random.int(1000,2000)} ${random.uuid} 등
		* (참고) commandline properties를 사용하지 못하게 하려면 SpringApplication.setAddCommandLineProperties(false);
* application.properties를 놓을 수 있는 위치와 우선순위
	* 숫자가 작은 값이 우선순위가 높다
		1. file:./config/ : project root/config/ 하위에 application.properties를 둠
		2. file:./ : project root 또는 jar파일의 실행 위치에 둠
		3. classpath:/config/ 에 둠
		4. classpath:/ 에 둠
	* 다만 테스트에서 @TestPropertySouce를 사용중이라면 이 application.properties의 위치에 따른 우선순위보다는 외부설정 우선순위를 따라가게 된다

* 타입-세이프 프로퍼티 @ConfigurationProperties
	```java
	@Component
	@ConfigurationProperties("keesun")
	public class KeesunProperties {
		private String name;
		private int age;
		private String fullName;
		//getter/setter
	}
	// (참고)이 때 intellij에서 "Spring Boot Configuration Annotation Processor not found in classpath" 라는 에러를 주는데 application.properties에서 intellij가 자동완성을 하기위한 메타데이터를 가져올 수 있는 플러그인을 넣어줘야 한다. (spring-boot-configuration-processor)
	// 원래는 @ConfigurationProperties 어노테이션을 처리할 수 있게끔 main class 위에 @EnableConfigurationProperties(KeesunProperties.class)를 등록해주어야 하는데 내장되어 있다
	// 사용하는 곳에서는 @Autowired KeesunProperties properties; 로 받아서 쓴다
	```
	* 여러 프로퍼티를 묶어서 읽어올 수 있음
	* (설정 파일의 값들을) 빈으로 등록해서 다른 빈에 주입할 수 있음
		* @EnableConfigurationProperties
		* @Component (주로 쓰는 방법)
		* @Bean
	* 융통성 있는 바인딩
		* https://docs.spring.io/spring-boot/docs/current/reference/html/spring-boot-features.html#boot-features-external-config-relaxed-binding
		* 아래와 같은 케이스들을 전부 바인딩해줌
			* context-path(케밥)
			* context_path(언더스코어)
			* contextPath(캐멀)
			* CONTEXTPATH
	* 프로퍼티 타입 컨버전
		* @DurationUnit
			* 시간 정보를 받을 때 사용
			```java
			// application.properties
			keesun.sessionTimeout = 25
			// KeesunProperties class
			@DurationUnit(ChronoUnit.SECONDS)
			private Duration sessionTimeout = Duration.ofSeconds(30); // set default 30 sec
			```
			* 그런데 25s를 사용하면 @DurationUnit annotation을 사용하지 않아도 된다
			```java
			keesun.sessionTime=25s
			private Duration sessionTimeout = Duration.ofSeconds(30);
			```
		* spring이 제공하는 conversion service를 이용하여 변환해 준다
		* ex. keesun.age = 100 // string
			private int age; // 자동으로 형 변환하여 integer로 주입
	* 프로퍼티 값 검증
		* @Validated
		* JSR-303 사용 가능 (@NotNull, ...)
		```java
		@Component
		@ConfigurationProperties("keesun")
		@Validated
		public class KeesunProperties {
			@NotEmpty
			private String name;
			@Size(min=0, max=100)
			private int age;
			...
		}
		```
	* (참고) Third-party Configuration : 이미 존재하는 설정을 빈으로 등록하여 사용하는 방법https://docs.spring.io/spring-boot/docs/current/reference/html/spring-boot-features.html#boot-features-external-config-3rd-party-configuration

### 프로파일
* @Profile annotation은 어디에?
	* @Configuration
	* @Component
* 어떤프로파일을 활성화 할 것인가
	* spring.profiles.active
		* 이 값도 역시 프로퍼티이고, 프로퍼티 우선순위 적용을 전부 받는다
* 어떤 프로파일을 추가할 것인가
	* spring.profiles.include
		* 추가할 프로파일을 지정
		* 만약 application-prod.properties에서 spring.profiles.include=proddb를 지정하면, application-proddb.properties 설정도 읽어들인다
* 프로파일용 프로퍼티
	* application-{profile}.properties
	* 기본적으로 application.properties가 application-{profile}.properties보다 우선순위가 낮아서 application-{profile}.properties에 설정한 값이 application.properties에 설정된 값들을 덮어쓴다

### 로깅
* 로깅 퍼사드 vs 로거
	* 로깅 퍼사드 : Commons Logging(spring boot에서 사용), SLF4j
		* 실제 로깅을 하는게 아니라 추상화 해둔 인터페이스
		* 프레임워크를 만들때는 주로 로깅 퍼사드를 사용하여 코딩
		* 일반 애플리케이션을 만드는 곳에서도 로깅 퍼사드를 사용해도 문제는 없다
		* 로깅 퍼사드를 사용하는 장점은 로거를 바꿔낄 수 있다
	* 로거 : JUL, Log4J2, Logback(spring boot가 사용)
* spring 5에 로거 관련 변경 사항
	* Spring-JCL
		* spring 5부터는 Spring-JCL가 개입하면서 spring boot가 최종적으로 Commons Logging -> SLF4j -> Logback으로 사용하게 된다
		* (참고) spring-boot-starter-logging 의존성에서 보면 log4j-to-slf4j, jul-to-slf4j라는 의존성이 들어있는데 이 의존성들이 java util 이나 log4j를 slf4j api로 보내도록 한다
		* pom.xml에 exclusion 안해도 됨
* 스프링 부트 로깅
	* reference : https://docs.spring.io/spring-boot/docs/current/reference/html/spring-boot-features.html#boot-features-logging
	* 기본 포맷 : 날짜/시간/log level/pid/method/full package/log
	* --debug (일부 핵심 라이브러리만 디버깅 모드로)
	* --trace (전부 다 디버깅 모드로)
	* 컬러 출력 : spring.output.ansi.enabled = always/detect/never
	* 파일 출력 : logging.file 또는 logging.path 를 선택하여 사용. 이 파일은 10MB마다 rotate됨
	* 로그 레벨 조정 : logging.level.<package> = <log level> // 로그 레벨을 패키지마다 설정 가능. 직접 만든 패키지가 아니고 외부에서 제공하는 패키지를 설정해도 된다
* 커스터마이징
	* 보통 업무에서 사용할 때는 사내 로깅 시스템에 연동을 해야해서 이런 방법을 사용할 수 있다
	* 커스텀 로그 설정 파일 사용하기
		* Logback : logback-spring.xml 파일 추가 (classpath에 추가해야 한다. 즉, resources/logback-spring.xml)// https://docs.spring.io/spring-boot/docs/current/reference/html/howto.html#howto-configure-logback-for-logging
		* Log4J2 : log4j2-spring.xml 파일 추가
		* JUL (비추) : logging.properties
		* Logback extension
			* 프로파일 <springProfile name="프로파일">
			* Environment property <springProperty> // environment를 로깅할 때.
	* 로거를 Log4j2로 변경하기
		* tomcat을 jetty로 바꾼 방식과 동일하게 exclusion을 사용하여 기존의 것(spring-boot-starter-logging)을 제거하고 다른 것(spring-boot-starter-log4j2)을 추가하면 된다
		* https://docs.spring.io/spring-boot/docs/current/reference/html/howto.html#howto-configure-log4j-for-logging

### 테스트
* 시작은 spring-boot-starter-test를 추가하는 것 부터
	* test scope로 의존성 추가
	* junit, json-path(json assertion), mockito(mockup) 등 테스트에 사용되는 의존성이 다 들어온다
* @SpringBootTest
	* @RunWith(SpringRunner.class)와 같이 써야 한다
	* @SpringBootApplication 을 찾아가서 모든 bean scan 후에 @MockBean만 찾아서 교체해준다. 
	* webEnvironment
		* MOCK : mock servlet environment, 내장 톰캣 구동하지 않음. Mocking DispatcherServlet을 사용하므로 MockMvc를 사용해야 한다.
		```java
		@RunWith(SpringRunner.class)
		@SpringBootTest(webEnvironment = SpringBootTest.WebEnvironment.MOCK) // servlet을 mocking한 DispatcherServlet이 뜬다
		@AutoConfigureMockMvc // 위의 Mock DispatcherServlet에 요청을 보내려면 MockMvc, MockMvcClient를 사용해야 한다. 이를 위해 이 어노테이션과 아래의 MockMvc를 사용하면 된다
		public class SampleControllerTest {
			@Autowired
			MockMvc mockMvc;
			@Test
			public void hello() throws Exception {
				mockMvc.perform(get("/hello"))
					.andExpect(status().isOk()) // 기대되는 결과값
					.andExpect(content().string("hello suhyeon")) // 기대되는 컨텐츠 내용
					.andDo(print());
				// 어떤 클래스의 어떤 메소드를 사용했는지 등등도 검사할 수 있다 
		}
		```
		* RANDOM_PORT, DEFINED_PORT : 내장 톰캣을 사용, test용 RestTemplate이나 test용 WebClient(for webflux)를 사용해야 한다
		```java
		@RunWith(SpringRunner.class)
		@SpringBootTest(webEnvironment = SpringBootTest.WebEnvironment.RANDOM_PORT) // 실제 내장 톰캣을 띄우고 거기에 요청을 보내어 테스트한다
		public class SampleControllerTest {
			@Autowired
			TestRestTemplate testRestTemplate;
			@MockBean
			SampleService mockSampleService; // ApplicationContext에 들어있는 빈SampleController.sampleService를 모킹하여 controller layer에서만 테스트를 진행할 수 있다
			@Test
			public void hello() {
				when(mockSampleService.getName()).thenReturn("sh"); // 결과값을 모킹한다
				String result = testRestTemplate.getForObject("/hello", String.class);
				assetThat(result).isEqualsTo("hello sh");
			}
		}
		```
		* NONE: 서블릿 환경 제공 안 함
* @MockBean
	* ApplicationContext에 들어있는 빈을 Mock으로 만든 객체로 교체함
	* 모든 @Test마다 자동으로 리셋
* WebClient
	* spring webflux (asynchronous) 를 이용할 때 사용. 비동기로 동작하여 응답이 오면 콜백으로 처리한다
	* spring-boot-starter-webflux 의존성 필요
	```java
	public class SampleControllerTest {
		@Autowired
		WebTestClient webTestClient;
		@MockBean
		SampleService mockSampleService;
		@Test
		public void hello() {
			when(mockSampleService.getName()).thenReturn("suhyeon");
			webTestClient.get().uri("/").exchange()
				.expectStatus().isOk()
				.expectBody(String.class).isEqualsTo("hello suhyeon");
	}
	```
* 슬라이스 테스트
	* SpringBootTest가 모든 빈을 찾아서 작업하므로 좀 더 간단하게 하려면 이 방법을 사용
	* 레이어 별로 잘라서 테스트하고 싶을 때
	* @JsonTest // 가지고있는 모델이 어떤 모양의 json으로 나가는지 확인하는 테스트, https://docs.spring.io/spring-boot/docs/current/reference/html/spring-boot-features.html#boot-features-testing-spring-boot-applications-testing-autoconfigured-json-tests
	* @WebMvcTest // controller 하나만 테스트하기
	```java
	@RunWith(SpringRunner.class)
	@WebMvcTest(SampleController.class) // web과 관련된 빈들만(@Controller 등) 빈으로 등록이 된다. (@Component 하위의 어노테이션들의 구분이 필요했던 이유라고 생각됨)
	public class Test {
		@MockBean
		SampleService sampleService; // @Service는 web과 관련된 빈이 아니라서 mock으로 등록해야한다
		@Autowired
		MockMvc mockMvc;
		...
	}
	```
	* @WebFluxTest
	* @DataJpaTest // @Repository 들만 빈으로 등록
...

### 테스트 유틸
* OutputCapture 
	* 가장 유용할 것
	* 로그를 비롯한 콘솔에 찍히는 모든 것을 캡쳐
	* 로그 메세지가 어떻게 찍혔는지 테스트 가능하다
	```java
	@RunWith(SpringRunner.class)
	public class Test {
		@Rule
		public OutputCapture outputCapture = new OutputCapture();
		@Test
		public void test() {
			...
			assertThat(outputCapture.toString())
				.contains("holoman") // logger의 내용
				.contains("skip");        // sout의 내용도 가능
		}
	}
	```
* TestPropertyValues
* TestRestTemplate
* ConfigFileApplicationContextInitializer

### Spring-Boot-Devtools
: 말그대로 dev용..! (굳이 사용할 필요가 있을까 싶음.. 그냥 껐다 키자 )
* 캐시 설정을 개발 환경에 맞게 변경.
* 클래스패스에 있는 파일이 변경 될 때마다 자동으로 재시작.
    * 직접 껐다 켜는거 (cold starts)보다 빠른다. 왜?
    ```java
    스프링부트는 두개의 클래스로더를 사용한다. base classloader는 우리가 변경하지 않는 의존성(third-part jars)을 로드한다. restart classloader는 우리가 개발하는 애플리케이션을 로드한다. Devtools에 의해 애플리케이션이 다시시작 될 때 restart classloader는 제거되고 재생성된다. 반면, base classloader에서 로드된 외부 의존성은 제거되지 않고 다시 사용되기 때문에 일반적으로 cold restart보다 훨씬 빠르다.
    ```
    * 릴로딩 보다는 느리다. (JRebel 같은건 아님)
    * 리스타트 하고 싶지 않은 리소스는? spring.devtools.restart.exclude
    * 리스타트 기능 끄려면? spring.devtools.restart.enabled = false
* 라이브 릴로드? 리스타트 했을 때 브라우저 자동 리프레시 하는 기능
    * 브라우저 플러그인 설치해야 함.
    * 라이브 릴로드 서버 끄려면? spring.devtools.liveload.enabled = false
* 글로벌 설정
    * ~/.spring-boot-devtools.properties
* 리모트 애플리케이션


### 스프링 웹 MVC 1부: 소개
 : 스프링에서의 MVC개념 위주가 아니라 부트가 제공해주는 MVC 기능 기준으로 설명
 : 우리가 아무런 설정없이 어노테이션으로만 MVC 개발이 가능 -> WebMvcAutoConfiguration가 내부적으로 설정해줌! (@AutoConfiguration)
* 스프링 웹 MVC
    * https://docs.spring.io/spring/docs/5.0.7.RELEASE/spring-framework-reference/web.html#spring-web
* 스프링 부트 MVC
    * 자동 설정으로 제공하는 여러 기본 기능 (앞으로 살펴볼 예정)
* 스프링 MVC 확장 : @Configuration + implements WebMvcConfigurer
    * 기본 제공 설정 사용 + 확장
* 스프링 MVC 재정의 : @Configuration + @EnableWebMvc
    * 기본 제공 설정 사용 X(사용할 일 없을듯)

### 스프링 웹 MVC 2부: HttpMessageConverters
 : HTTP 요청 본문을 객체로 변환하거나, 객체를 HTTP 응답 본문으로 변환할 때 사용
* https://docs.spring.io/spring/docs/5.0.7.RELEASE/spring-framework-reference/web.html#mvc-config-message-converters
* 스프링 프레임워크에서 제공하는 인터페이스
* 추가 설명
    * 요청/반환타입에 따라 json/string 각 컨버터가 알아서 선택되어 사용됨 
    * @ReuqestBody : @RequestBody는 컨트롤러가 요청을 받을 때 본문에 들어있는 데이터를 객체로 받을 수 있으며, 스프링이 객체로 변환
    * @ResponseBody : @RequestBody를 사용하면 객체를 문자열로 변환 (@RestController를 사용하면 생략가능) 
    ```java
    // 컨트롤러
    @PostMapping("/users/create")
    public User craate(@RequestBody User user) {
        return user;
    }
      
    // 테스트
    @Test
    public void createUser_JSON() throws Exception {
        String userJson = "{\"username\":\"jingyu\", \"password\":\"123\"}";
        mockMvc.perform(post("/users/create")
                .contentType(MediaType.APPLICATION_JSON_UTF8)
                .accept(MediaType.APPLICATION_JSON_UTF8)
                .content(userJson))
            .andExpect(status().isOk())
            .andExpect(jsonPath("$.username", is(equalTo("jingyu"))))
            .andExpect(jsonPath("$.password", is(equalTo("123"))));
    }
    ```
### 스프링 웹 MVC 3부: ViewResolve
* ViewResolver? 들어오는 요청의 accept 헤더에 따라 클라이언트가 원하는 형식으로 응답을 제공하는 역할 달라지게 세팅
* 뷰 리졸버 설정 제공
* 
* TEST : 응답을 xml형식으로 주기
    ```java
    @Test
    public void createUser_XML() throws Exception {
        String userJson = "{\"username\":\"jingyu\", \"password\":\"123\"}";
        movckMvc.perform(post("/users/create")
                .contentType(MediaType.APPLICATION_JSON_UTF8)
                .accept(MediaType.APPLICATION_XML)
                .content(userJson))
            .andExpect(status().isOk())
            .andExpect(xpath("/User/username").string("jingyu"))
            .andExpect(xpath("/User/password").string("123"));
    }
    ```
    ```xml
    // HttpMessageConvertersAutoConfiguration
    <dependency>
        <groupId>com.fasterxml.jackson.dataformat</groupId>
        <artifactId>jackson-dataformat-xml</artifactId>
        <version>2.9.6</version>
    </dependency>
    ```
 
### 스프링 웹 MVC 4부: 정적 리소스 지원
 : 동적으로 생성하지 않고, 이미 만들어진 리소스를 제공하는 것 (에러페이지같은거에 쓸듯?)
 : 정적 리소스를 처리하는 작업은 ResourceHttpRequestHandler가 처리. 얘가 Last-Modified 헤더를 보고 304 응답을 보냄 (바뀌면 다시 200)
정적 리소스 맵핑 “/**”
* 기본 리소스 위치
    * classpath:/static
    * classpath:/public
    * classpath:/resources/
    * classpath:/META-INF/resources
* 기본리소스 경로 수정/추가 방법
    * 맵핑 설정 변경 가능 
        * ex) spring.mvc.static-path-pattern=/static/** : “/hello.html” => /static/hello.html 
    * 리소스 찾을 위치 변경 가능 (spring.mvc.static-locations) 
    * 내가 원하는 리소스핸들러 추가 : WebMvcConfigurer의 addRersourceHandlers로 커스터마이징 할 수 있음 (권장)
    ```java
    @Override
    public void addResourceHandlers(ResourceHandlerRegistry registry) {
      registry.addResourceHandler("/m/**")
          .addResourceLocations("classpath:/m/")
          .setCachePeriod(20);
    }
    ```
    
### 스프링 웹 MVC 5부: 웹JAR
 : 클라이언트에서 사용하는 javascript 라이브러리를 jar파일로 디펜던시를 추가하는 것. ex) jQuery,bootstrap,view.js,..
 : https://mvnrepository.com 에서 필요한 의존성 찾아서 넣어주면 사용 가능!
* 웹JAR 맵핑 “/webjars/**”
* 버전생략하고 사용하기? : webjars-locator-core 의존성 추가
    * 실무에서 버전없이 사용하는게 좋은건지 의문...
    
### 스프링 웹 MVC 6부: index 페이지와 파비콘
* 웰컴 페이지 : 루트 경로로 접근했을때 뜨는 페이지 화면
    * index.html 찾아 보고 있으면 제공.
    * index.템플릿 찾아 보고 있으면 제공.
    * 둘 다 없으면 에러 페이지.
*파비콘 : 
    * favicon.ico
    * 파이콘 만들기 https://favicon.io/
    * 파비콘이 안 바뀔 때?
        * https://stackoverflow.com/questions/2208933/how-do-i-force-a-favicon-refresh
        
### 스프링 웹 MVC 7부: Thymeleaf
 : 템플릿엔진? 주로 view를 만드는데 사용. 동적인 컨텐츠를 만드는데 사용. email,invoice 양식 등에도 사용
* 스프링 부트가 자동 설정을 지원하는 템플릿 엔진
    * FreeMarker
    * Groovy
    * Thymeleaf
    * Mustache
* JSP를 권장하지 않는 이유
    * JAR 패키징 할 때는 동작하지 않고, WAR 패키징 해야 함.
    * Undertow는 JSP를 지원하지 않음.
    * 스프링부트는 독립적으로 실행 가능한 내장 톰캣으로 빠르게 앱을 배포하길 원하는데, JSP는 이러한 스프링부트의 지향점과 어긋나기 때문에 스프링부트에서 권장하지 않음
* Thymeleaf 사용하기
    * Thymeleaf란?
        * Thymeleaf는 스프링 부트가 자동 설정을 지원하는  웹 템플릿 엔진. HTML문서에 HTML5 문법으로 서버쪽 로직을 수행하고 적용시킬 수 있음.
        * HTML 디자인에 전혀 영향을 미치지 않고 웹 템플릿 엔진을 통해 HTML을 생성할 수 있음
        * 독자적으로 HTML을 생성하기 때문에 테스트 시 렌더링 결과를 확인하기 좋음
        * -> Thymleaf는 서블릿 컨테이너가 개입하지 않는 서블릿 컨테이너에 독립적인 엔진. Thymleaf는 독자적으로 최종적인 view를 완성하며 랜더링되는 결과까지도 확인
    * https://www.thymeleaf.org/
    * https://www.thymeleaf.org/doc/articles/standarddialect5minutes.html < 읽어보기
    * 의존성 추가: spring-boot-starter-thymeleaf
    * 템플릿 파일 위치: /src/main/resources/template/
    * 예제 : https://github.com/thymeleaf/thymeleafexamples-stsm/blob/3.0-master/src/main/webapp/WEB-INF/templates/seedstartermng.html

### 스프링 웹 MVC 8부: HtmlUnit
 : html을 단위테스트하기 위한 툴 (html을 전문적으로 테스트할때 사용하는.. 굳이..ㅎ)
 : 화면에 폼 출력 테스트, 특정 브라우저 테스트, html안의 엘리먼트/태그 등으로 가져와서 테스트
* HTML 템플릿 뷰 테스트를 보다 전문적으로 하자.
    * http://htmlunit.sourceforge.net/
    * http://htmlunit.sourceforge.net/gettingStarted.html
    * 의존성 추가
    * @Autowire WebClient
    
### 스프링 웹 MVC 9부: ExceptionHandler
 : 부트에는 기본적인 에러핸들러가 등록되어 있다. = BasicErrorController(http/json 응답 모두 지원)
* 스프링 @MVC 예외 처리 방법
    * @ExceptionHandler(*Controller.class) : 각 클래스에서 구현. 
    * @ControllerAdvice를 : 전역에서 모든 예외를 처리하기 위해서는 별도의 클래스를 생성하고 @ControllerAdvice를 지정해야 한다.
* 커스텀 에러 페이지 : ErrorController를 상속하는 클래스를 만들어서 빈으로 등록
    * 상태 코드 값에 따라 에러 페이지 보여주기
        * 경로) src/main/resources/static|template/error/
        * ex) 404.html, 5xx.html, ..
    * -> ErrorViewResolver 구현
    
### 스프링 웹 MVC 10부: Spring HATEOAS
 : REST API를 만들떄, 서버가 리소스를 제공할떄, 리소스와 연관되어 있는 링크정보까지 같이 제공을 하고, 클라이언트는 이 정보를 바탕으로 리소스에 접근.
 : ex) ....? 뭔말이야
 ```
 HATEOAS를 쓰는 이유는 다음과 같은 기존 REST API의 단점을 보완하기 위해서입니다.
 
 1. REST API는 앤드포인트 URL이 정해지고 나면 이를 변경하기 어렵다는 단점이 있습니다. 만일 API의 URL을 변경하게 되면 모든 클라이언트의 URL까지 수정해야하기 때문에 번거로워지므로 기존 다른 API를 지속적으로 추가하게 됩니다. 따라서 URL 관리가 어렵게 됩니다. 
 2. 전달받은 정적 자원의 상태에 따른 요소를 서버 단에서 구현하기 어렵기 때문에 클라이언트 단에서 이 부분에 대한 로직을 처리해야 합니다.
 
 위 단점들을 links 요소를 통해 href 값의 형태로 보내주기 때문에 자원 상태에 대한 처리를 링크에 있는 URL을 통해 처리할 수 있게됩니다.
 (_links는 HATEOAS를 구현하기 위해 스프링 부트에서 생성한 JSON name입니다. 그리고 그 뒤의 self는 자기 참조를 뜻하는 것을 JSON을 통해서 나타낸 것입니다.)
    
 출처: https://engkimbs.tistory.com/780?category=767865 [새로비]
 ```
* Hypermedia As The Engine Of Application State
    * 서버: 현재 리소스와 연관된 링크 정보를 클라이언트에게 제공한다.
    * 클라이언트: 연관된 링크 정보를 바탕으로 리소스에 접근한다.
    * 연관된 링크 정보
        * Relation
        * Hypertext Reference
    * spring-boot-starter-hateoas 의존성 추가
    * https://spring.io/understanding/HATEOAS
    * https://spring.io/guides/gs/rest-hateoas/
    * https://docs.spring.io/spring-hateoas/docs/current/reference/html/
* ObjectMapper 제공
    * spring.jackson.*
    * Jackson2ObjectMapperBuilder
* LinkDiscovers 제공
    * 클라이언트 쪽에서 링크 정보를 Rel 이름으로 찾을때 사용할 수 있는 XPath 확장 클래스

* ? : import org.springframework.hateoas.Resource; 없..


### 스프링 웹 MVC 11부: CORS
* SOP과 CORS
    * SOP(Single-Origin Policy) : 기본적으로는 origin이 다른 경우 리소스를 요청할 수 없다
    * CORS(Cross-Origin Resource Sharing) : 서로 다른 origin 끼리 리소스를 공유
    * Origin? 아래 3개의 조합이 하나의 origin
        * URI 스키마 (http, https)
        * hostname (whiteship.me, localhost)
        * 포트 (8080, 18080)
* 스프링 MVC @CrossOrigin : 스프링에서 CORS를 지원하기 위해서 원래는 여러가지 설정을 해야하지만, 스프링부트에서 그러한 설정을 해주기 때문에 @CrossOrigin을 바로 사용할 수 있음
    * https://docs.spring.io/spring/docs/5.0.7.RELEASE/spring-framework-reference/web.html#mvc-cors
    * @Controller나 @RequestMapping에 추가하거나
    * WebMvcConfigurer 사용해서 글로벌 설정

### 스프링 데이터 : 인메모리 데이터베이스
* spring-boot-starter-jdbc 의존성 추가
* 지원하는 인-메모리 DB
    * H2(추천, 콘솔 때문에)
    * HSQL
    * Derby
* Spring-JDBC가 클래스패스(의존성)에 있으면 자동 설정이 필요한 빈을 설정 해 줌
    * DataSourceAutoConfiguration
    * JdbcTemplateAutoConfiguration
    * h2 db가 클래스패스에 들어있고 datasource 설정을 하지 않으면 spring boot가 알아서 in-memory db를 설정 해 준다
* 인-메모리 DB 기본 연결 정보 확인하는 방법
    * URL: "testdb"
    * username: "sa"
    * password: ""
* H2 console 사용하는 방법
    * spring-boot-devtools를 추가하거나
    * spring.h2.console.enabled=true만 추가
        * springboot 2.3부터 인메모리 db 설정이 매번 새로운 이름으로 만들어지도록 되어있어서 spring.datasource.generate-unique-name=false 을 application.properties에 추가해주어야 jdbc:h2:mem:testdb인 고정값으로 사용 가능하다
    * localhost:8080/h2-console로 접속 (이 path도 변경 가능)
* 실습 코드
    * CREATE TABLE USER (ID INTEGER NOT NULL, name VARCHAR(255), PRIMARY KEY (id))
    * INSERT INTO USER VALUES (1, 'keesun')
    * java
  ```java
    @Component
    public class H2Runner implements ApplicationRunner {
        @Autowired
        DataSource dataSource; // javax.sql.DataSource
        @Autowired
        JdbcTemplate jdbcTemplate;
        @Override
        public void run(ApplicationArguments args) throws Exception {
            // 아래와 같이 사용하면 connection이라는 리소스를 try 블록 내에서 사용하고 정리해 줌. 명시적으로 finally를 적을 필요가 없음
            try (Connection conn = dataSource.getConnection()) {
                System.out.println(conn.getMetaData().getURL());
                System.out.println(conn.getMetaData().getUserName());
                Statement statement = conn.createStatement();
                String sql = "CREATE TABLE USER (ID INTEGER NOT NULL, name VARCHAR(255), PRIMARY KEY (id))";
                statement.executeUpdate(sql);
            }
            // 위의 방식으로 사용하는 jdbc api보다 jdbcTemplate을 이용하면 try/catch, 리소스 반환 등의 구현이 잘 되어있어서 이 방법이 더 좋다
            jdbcTemplate.execute("INSERT INTO USER VALUES (1, 'keesun')");
        }
    }
  ```

### 스프링 데이터 : MySQL
* 지원하는 DBCP // 아래 세 가지의 dbcp가 classpath에 있다면 HikariCP를 기본으로 사용
    * HikariCP (default)
        * https://github.com/brettwooldridge/HikariCP#frequently-used (db관련된 설정을 볼 수 있다)
        * (참고) connection pool의 개수가 많더라도 실제 실행 가능한 connection은 cpu 개수만큼만 가능하다
    * TomcapCP
    * Commons DBCP2
* DBCP설정
    * spring.datasource.hikari.*
    * spring.datasource.tomcat.*
    * spring.datasource.dbcp2.*
* MySQL connector 의존성 추가
    * mysql-connector-java
* MySQL 추가 (docker 사용)
    * docker run -p 3306:3306 --name mysql_boot -e MYSQL_ROOT_PASSWORD=1 -e MYSQL_DATABASE=springboot -e MYSQL_USER=suhyeon -e MYSQL_PASSWORD=pass -d mysql
    * docker exec -i -t mysql_boot bash
    * mysql -u root -p
* MySQL용 Datasource 설정
    * spring.datasource.url=jdbc:mysql://localhost:3306/springboot?useSSL=false # 설정하지 않으면 in-memory db를 사용하려고 한다
    * spring.datasource.username=suhyeon
    * spring.datasource.password=pass
    * 운영 DB용 설정
        * spring.datasource.testWhileIdel=true
        * spring.datasource.validationQuery=SELECT 1
* MySQL은 상용으로 쓰기 위해 라이센스를 구매해야하고, GPL이므로 사용하는 코드의 공개 의무가 있다

### 스프링 데이터 : PostgreSQL
* 의존성 추가
    * postgresql
* application.properties 수정
```sh
		spring.datasource.url=jdbc:postgresql://localhost:5432/springboot
		spring.datasource.username=root
		spring.datasource.password=pass
```
* docker run -p 5432:5432 -e POSTGRES_PASSWORD=pass -e POSTGRES_USER=root -e POSGTRES_DB=springboot --name postgres_boot -d postgres
```sh
	docker exec -i -t postgres bash
	su - postgres
	psql springboot
	#db 조회
	\list
	#table 조회
	\dt
	#query
	SELECT *F ROM account;
```

### 스프링 데이터 : 스프링 데이터 JPA 소개
* ORM(Object-Relational Mapping)과 JPA(Java Persistence API)
    * 객체와 릴레이션을 맵핑할 때 발생하는 개념적 불일치를 해결하는 프레임워크
    * http://hibernate.org/orm/what-is-an-orm
    * JPA: ORM을 위한 자바(EE) 표준
* Spring Data JPA
    * Repository 빈 자동 생성
    * 쿼리 메소드 자동 구현
    * @EnableJpaRepositories(스프링 부트가 자동으로 설정해 줌)
    * 계층 : SpringDataJpa -> JPA -> Hibernate -> Datasource(jdbc)

### 스프링 데이터 : 스프링 데이터 JPA 연동
* Spring Data JPA 의존성 추가
    * spring-boot-starter-data-jpa
* Spring Data JPA 사용하기
    * @Entity class 만들기
  ```java
    @Entity
    public class Account {
        @Id @GeneratedValue
        private Long id;

        private String username;

        private String password;
        //getter, setter, equals, hashCode. Lombok을 사용하는 것도 추천
    }
  ```
    * Repository 만들기
			public interface AccountRepository extends JpaRepository<Account, Long> {
			}
* Spring Data repository test
    * H2 DB를 테스트 의존성에 추가하기
    * @DataJpaTest (슬라이트 테스트)작성
  ```java
    @RunWith(SpringRunner.class)
    @DataJpaTest // repository bean들만 생성하여 테스트. in-memory db를 필요로 한다.
    public class AccountRepositoryTest {
        @Autowired
        DataSource dataSource;
        @Autowired
        JdbcTemplate jdbcTemplate;
        @Autowired
        AccountRepository accountRepository;
        @Test
        public void di() {
            Account account = new Account();
            account.setUsername("suhyeon");
            account.setPassword("pass");

            Account newAccount = accountRepository.save(account);
            assertThat(newAccount).isNotEmpty();
            Account nonExistingAccount = accountRepository.findByUsername("asdf");
            assertThat(nonExistingAccount).isEmpty();
        }
    }
  ```
    * @SpringBootTest를 이용한 integration test를 이용한다면?
        * 모든 bean들을 생성하여 느림
        * in-memory db를 사용하지 않고 직접 db를 띄워둔 상태에서 테스트를 진행해야 함
        * 그리고 application.properties에 설정된 db의 값들이 테스트에 사용되어 변경됨

### 스프링 데이터 : 데이터베이스 초기화
* JPA를 사용한 데이터베이스 초기화
    * 아래 두 가지 설정을 주어야 초기화 가능
    * spring.jpa.hibernate.ddl-auto
        * create-drop : 애플리케이션이 끝나고 스키마를 지움
        * create : 지우고 새로 만듦
        * update : 기존 스키마는 그대로 두고 추가된 것만 추가해 줌 (기존 데이터 유지)
        * validate : ddl false로 두어야 함. 운영용으로 사용할 때. schema가 entity와 일치하는지 확인해 줌
    * spring.jpa.generate-ddl=true로 설정해주어야 동작
* ddl, dml 로깅하기
    * spring.jpa.show-sql=true
* SQL 스크립트를 사용한 데이터베이스 초기화
    * resources 하위에 schema.sql 또는 schema-${platform}.sql를 생성하여 DDL을 적어두면 애플리케이션 실행 시 실행 됨
    * ${platform} 값은 spring.datasource.platform으로 설정 가능

### 스프링 데이터 : 데이터베이스 마이그레이션
* Flyway와 Liquibase가 대표적인데 Flyway를 사용
    * sql file들을 버전 관리 할 수 있다
* https://docs.spring.io/spring-boot/docs/2.0.3.RELEASE/reference/htmlsingle/#howto-execute-flyway-database-migrations-on-startup
* 의존성 추가
    * org.flywaydb:flyway-core
* 마이그레이션 디렉토리
    * db/migration 또는 db/migration/{vendor} // postgers/mysql/...
    * spring.flyway.locations로 변경 가능
* 마이그레이션 파일 이름
    * V숫자__이름.sql
    * V는 반드시 대문자
    * 숫자는 순차적으로 (타임스탬프 권장)
    * 숫자와 이름 사이에 언다바 두 개
    * 이름은 가능한 서술적으로
* 스크립트 파일은 적용된 이후 절대 건드리면 안된다

### 스프링 데이터 : Redis
* 캐시, 메세지 브로커, 키/밸류 스토어 등으로 사용 가능
* 의존성 추가
    * spring-boot-starter-data-redis // spring boot 자동설정이 등록을 해줌
* Redis 설치 및 실행(도커)
    * docker run -p 6379:6379 --name redis_boot -d redis
    * docker exec -it redis_boot redis-cli
* spring data redis
    * https://projects.spring.io/spring-data-redis/
    * StringRedisTemplate 또는 RedisTemplate
    * extends CrudRepository
    * code
  ```java
    @Component
    public class RedisRunner implements ApplicationRunner {
        @Autowired
        StringRedisTemplate redisTemplate;
        @Override
        public void run(ApplicationArguments args) {
            ValueOperations<String, String> values = redisTemplate.opsForValue();
            values.set("suhyeon", "kim");
            values.set("springboot", "2.0");
            values.set("hello","world");
        }
    }
  ```
* Redis 주요 커맨드
    * https://redis.io/commands
    * keys *
    * get {key}
    * hgetall {key}
    * hget {key} {column}
* 커스터마이징
    * spring.redis.*
        * port, url, ...
* repository 생성
  ```java
    //entity
    @RedisHash("accounts")
    public class Account {
        @Id
        private String id;
        private String username;
        private string email;
    }
    //repository, CrudRepository는 redis특화가 아닌 spring jpa에서 제공하는 거의 최상위 계층의 인터페이스
    public interface AccountRepository extends CrudRepository<Account, String> {
    }
  ```

### 스프링 데이터 : MongoDB
* MongoDb는 JSON기반의 도큐먼트 데이터베이스
* 의존성 추가
    * spring-boot-starter-data-mongodb
* MongoDB설치 및 실행 (도커)
    * docker run -p 27017:27017 --name mongo_boot -d mongo
    * docker exec -i -t mongo_boot bash
    * mongo
* spring data mongo db
    * MongoTemplate
    * MongoRepository
    * 내장형 MongoDB(테스트용)
        * de.flapdoodle.embed:de.flapdoodle.embed.mongo
    * @DataMongoTest
    * 명령어
        * db : db 조회
        * use test : test라는 db 사용
        * db.accounts.find({}) : accounts collection 조회
* code
    * 이번엔 ApplicationRunner를 main이 있는 클래스에 람다식으로 구현
  ```java
    @SpringBootApplication
    public class SpringbootmongoApplication {
        @Autowired
        MongoTemplate mongoTemplate;
        @Autowired
        AccountRepository accountRepository;
        psvm()
        @Bean
        public ApplicationRunner applicationRunner() {
            return args -> {
                Account account = new Account();
                account.setEmail("asdf@gmail.com");
                account.setUsername("aaa");
                mongoTemplate.insert(account);
                // or
                accountRepository.insert(account);
            };
        }
    }
    //entity
    @Document(collection = "accounts")
    public class Account {
        @Id
        private String id;
        private String username;
        private String email;
        // getter, setter
    }
    // repository도 사용 가능
    public interface AccountRepository extends MongoRepository<Account, String>{}
  ```

* 내장형 mongo db를 사용하는 테스트 코드
  ```java
    @RunWith(SpringRunner.class)
    @DataMongoTest
    public class AccountRepositoryTest {
        @Autowired
        AccountRepository accountRepository;
        @Test
        public void findByEmail() {
            Account account = new Account();
            account.setUsername("suhyeon");
            account.setEmail("suhyeon@gmail.com");

            accountRepository.save(account);
            Optional<Account> byId = accountRepository.findById(account.getId());
            assertThat(byId).isNotEmpty();

            Optional<Account> byEmail = accountRepository.findByEmail(account.getEmail());
            assertThat(byEmail).isNotEmpty();
        }
    }
  ```

### 스프링 데이터 : Neo4j
* Neo4j는 노드간의 연결 관계를 영속화 하는데 유리한 그래프 데이터 베이스
* 의존성 추가
    * spring-boot-starter-data-neo4j
* Neo4j 설치 및 실행(도커)
    * docker run -p 7474:7474 -p 7687:7687 -d --name neo4j_boot neo4j
    * http://localhost:7474/browser
* spring data Neo4J
    * Neo4jTemplate(deprecated)
    * SessionFactory
    * Neo4jRepository
* code
  ```java
    @Component
    public class Neo4jRunner implements ApplicationRunner {
        @Autowired
        SessionFactory sessionFactory;
        @Override
        public void run(ApplicationArguments args) {
            Account account = new Account();
            account.setUsername("suhyeon");
            account.setEmail("aaa@gmail.com");

            Session session = sessionFactory.openSession();
            session.save(account);
            sessionFactory.close();
        }
    }
    // entity
    @NodeEntity
    public class Account {
        @Id @GeneratedValue
        private Long id;
        private String username;
        private String email;
        // getter, setter
    }
  ```

### 스프링 데이터 : 정리
* https://docs.spring.io/spring-boot/docs/current-SNAPSHOT/reference/htmlsingle/#boot-features-sql

### 스프링 시큐리티 1부: Starter-Security
 : spring-boot-starter-security 의존성 추가해주면 사용 가능
 : 의존성을 추가하면 모든 요청에 인증을 필요로 함
    302 from Login 관련 인증정보가 없을때, 자동으로 로그인화면으로 redirect (user/임시pw)
    401
* 스프링 시큐리티
    * 웹 시큐리티
    * 메소드 시큐리티
    * 다양한 인증 방법 지원 :LDAP, 폼 인증, Basic 인증, OAuth, ...
* 스프링 부트 시큐리티 자동 설정
    * SecurityAutoConfiguration
    * UserDetailsServiceAutoConfiguration
    * spring-boot-starter-security
        * 스프링 시큐리티 5.* 의존성 추가
    * 모든 요청에 인증이 필요함.
    * 기본 사용자 생성
        * Username: user
        * Password: 애플리케이션을 실행할 때 마다 랜덤 값 생성 (콘솔에 출력 됨.)
        * spring.security.user.name
        * spring.security.user.password
    * 인증 관련 각종 이벤트 발생
        * DefaultAuthenticationEventPublisher 빈 등록
        * 다양한 인증 에러 핸들러 등록 가능
* 스프링 부트 시큐리티 테스트
    * https://docs.spring.io/spring-security/site/docs/current/reference/html/test-method.html
    * @WithMockUser
    ```java
    @RunWith(SpringRunner.class)
    @WebMvcTest(HomeController.class)
    public class HelloControllerTest {
    
        @Autowired
        MockMvc mockMvc;
    
        @Test
        public void hello_without_user() throws Exception {
            mockMvc.perform(get("/hello"))
                    .andDo(print())
                    .andExpect(status().isUnauthorized());
        }
    
        @Test
        @WithMockUser
        public void hello() throws Exception {
            mockMvc.perform(get("/hello"))
                    .andDo(print())
                    .andExpect(status().isOk())
                    .andExpect(view().name("hello"));
        }
    
        @Test
        @WithMockUser
        public void my() throws Exception {
            mockMvc.perform(get("/my"))
                    .andDo(print())
                    .andExpect(status().isOk())
                    .andExpect(view().name("my"));
        }
    }
    ```
  
### 스프링 시큐리티 2부: 시큐리티 설정 커스터마이징
1. 웹 시큐리티 설정
    ```java
    @Configuration
    public class WebSecurityConfig extends WebSecurityConfigurerAdapter
    {
     @Override
     protected void configure(HttpSecurity http) throws Exception {
         http.authorizeRequests()
             .antMatchers("/", "/hello").permitAll()
             .anyRequest().authenticated()
             .and()
             .formLogin()
             .and()
             .httpBasic();
     }
    }
    ```
2. UserDetailsServie 구현
    * Account DTO/Repository/Service 구현
    * https://docs.spring.io/spring-security/site/docs/current/reference/htmlsingle/#jc-authentication-userdetailsservice
    ```java
    @Service
    public class AccountService implements UserDetailsService {
    
        @Autowired
        private AccountRepository accountRepository;
    
        @Autowired
        private PasswordEncoder passwordEncoder;
    
        public Account createAccount(String username, String password) {
            Account account = new Account();
            account.setUsername(username);
            account.setPassword(passwordEncoder.encode(password));
            return accountRepository.save(account);
        }
    
        @Override
        public UserDetails loadUserByUsername(String username) throws UsernameNotFoundException {
            Optional<Account> byUserName = accountRepository.findByUserName(username);
            Account account = byUserName.orElseThrow(() -> new UsernameNotFoundException(username));
            return new User(account.getUsername(), account.getPassword(), authorities());
        }
    
        private Collection<? extends GrantedAuthority> authorities() {
            return Arrays.asList(new SimpleGrantedAuthority("ROLE_USER"));
        }
    }
    ```

3. PasswordEncoder 설정 및 사용
    * https://docs.spring.io/spring-security/site/docs/current/reference/htmlsingle/#core-services-password-encoding
    ```java
    @Bean
        public PasswordEncoder passwordEncoder() {
            return PasswordEncoderFactories.createDelegatingPasswordEncoder();
        }
    ```
  
### 스프링 REST 클라이언트 1부: RestTemplate과 WebClient
 : 해당내용은 부트에 기능은 아님. 단지 부트에서는 쉽게 쓸수있도록 기본적으로 빈을 등록을 해줌.
* RestTemplate
    * Blocking I/O 기반의 Synchronous API
    * RestTemplateAutoConfiguration
    * 프로젝트에 spring-web 모듈이 있다면 RestTemplateBuilder를 빈으로 등록해 줍니다.
    * https://docs.spring.io/spring/docs/current/spring-framework-reference/integration.html#rest-client-access
* WebClient
    * Non-Blocking I/O 기반의 Asynchronous API
    * WebClientAutoConfiguration
    * 프로젝트에 spring-webflux 모듈이 있다면 WebClient.Builder를 빈으로 등록해줍니다.
    * https://docs.spring.io/spring/docs/current/spring-framework-reference/web-reactive.html#webflux-client


### 스프링 REST 클라이언트 2부: 커스터마이징
* RestTemplate
    * 기본으로 java.net.HttpURLConnection 사용.
    * 커스터마이징
        * 로컬 커스터마이징
        * 글로벌 커스터마이징
            * RestTemplateCustomizer
            * 빈 재정의
* WebClient
    * 기본으로 Reactor Netty의 HTTP 클라이언트 사용.
    * 커스터마이징
        * 로컬 커스터마이징
        * 글로벌 커스터마이징
            * WebClientCustomizer
            * 빈 재정의
```java
@SpringBootApplication
public class WebclientApplication {

    public static void main(String[] args) {
        SpringApplication.run(WebclientApplication.class, args);
    }

    @Bean
    public WebClientCustomizer webClientCustomizer() {
        return webClientBuilder ->  webClientBuilder.baseUrl("http://localhost:8080");
    }

    @Bean
    public RestTemplateCustomizer restTemplateCustomizer() {
        return restTemplate -> restTemplate.setRequestFactory(new HttpComponentsClientHttpRequestFactory());
    }
}
```
### 그밖에 다양한 기술 연동
* 캐시
* 메시징
* Validation
* 이메일 전송
* JTA
* 스프링 인티그레이션
* 스프링 세션
* JMX
* 웹소켓
* 코틀린
* ...


## 섹션 5. 스프링부트 운영
스프링 부트는 애플리케이션 운영 환경에서 유용한 기능을 제공합니다. 스프링 부트가 제공하는 엔드포인트와 메트릭스 그리고 그 데이터를 활용하는 모니터링 기능에 대해 학습합니다.

### 스프링 부트 Actuator 1부: 소개
 : https://docs.spring.io/spring-boot/docs/current/reference/htmlsingle/#production-ready-endpoints
* 의존성 추가
    * spring-boot-starter-actuator
* 애플리케이션의 각종 정보를 확인할 수 있는 Endpoints(사용자나 디바이스같은 IT 서비스의 최종 목적지)
    * 다양한 Endpoints 제공.
    * JMX 또는 HTTP를 통해 접근 가능 함.
    * shutdown을 제외한 모든 Endpoint는 기본적으로 활성화 상태.
    * 활성화 옵션 조정
        * management.endpoints.enabled-by-default=false
        * management.endpoint.info.enabled=true

### 스프링 부트 Actuator 2부: JMX와 HTTP
* JConsole 사용하기
    * https://docs.oracle.com/javase/tutorial/jmx/mbeans/
    * https://docs.oracle.com/javase/7/docs/technotes/guides/management/jconsole.html
* VisualVM 사용하기
    * https://visualvm.github.io/download.html
* HTTP 사용하기
    * /actuator
    * health와 info를 제외한 대부분의 Endpoint가 기본적으로 비공개 상태
    * 공개 옵션 조정
        * management.endpoints.web.exposure.include=*
        * management.endpoints.web.exposure.exclude=env,beans

### 스프링 부트 Actuator 3부: 스프링 부트 어드민
 : https://github.com/codecentric/spring-boot-admin
* 스프링 부트 Actuator UI 제공
* 어드민 서버 설정
    ```xml
    <dependency>
        <groupId>de.codecentric</groupId>
        <artifactId>spring-boot-admin-starter-server</artifactId>
        <version>2.0.1</version>
    </dependency>
    ```
    @EnableAdminServer
* 클라이언트 설정
    ```xml
    <dependency>
        <groupId>de.codecentric</groupId>
        <artifactId>spring-boot-admin-starter-client</artifactId>
        <version>2.0.1</version>
    </dependency>
    ```
    spring.boot.admin.client.url=http://localhost:8080 
    management.endpoints.web.exposure.include=*
    
    
## 섹션 6. 마무리
* 스프링 부트 원리
    * 의존성 관리
    * 자동 설정
    * 내장 웹 서버
    * JAR 패키징
* 스프링 부트 활용
    * 스프링 부트 핵심 기능
    * 다양한 기술 연동
* 스프링 부트 운영
    * Actuator
    * 스프링 부트 어드민
---


---
