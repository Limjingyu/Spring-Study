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
* @Conditional~ 어노테이션 찾아보기 

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

### 톰캣 HTTP2

### 독립적으로 실행 가능한 JAR

### 스프링 부트 원리 정리


## 섹션 3. 스프링 부트 활용

## 섹션 4. 스프링 부트 운영

## 섹션 5. 마무리

---
