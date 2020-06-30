# 01. 예제로 배우는 스프링 입문 (개정판)

## 섹션 1. 스프링 예제 프로젝트 PetClinic
## 섹션 2. 스프링 IoC
* 가장 상세한 설명은 [마틴 파울러의 아티클](https://www.martinfowler.com/articles/injection.html) 참고
* IoC란?
  * 제어권이 역전된 것
  * 일반적인 경우는 의존성을 사용하는 주체가 만들어서 사용하지만, IoC의 경우 그 의존성을 외부에서 주입(DI)하게 된다. 즉 의존성을 사용하는 주체가 만들지 않고 다른 누군가가 만들어서 의존성을 넣어주게 되는 것이 일반적인 제어와는 다르게 역전되었다고 한다
  * Spring framework에서는 빈을 통해서 DI를 제공한다
* Spring의 IoC
  * 주로 ApplicationContext를 사용
  * 빈을 생성, 빈 사이의 의존성을 연결, 생성한 빈들을 제공하는 역할
  * Spring IoC Container 안에 있는 객체(빈)들 끼리만 의존성 주입
  * 빈 객체는 Singleton으로 생성
    * 멀티스레드 환경에서 singleton으로 객체를 만드는 일이 까다로운데, IoC를 이용하여 해결 가능
* 빈
  * 일반적인 객체지만 IoC Container가 관리하는 객체
  * 빈 등록 방법
    * AnnotationProcessor : lifecycle callback들 중 @Component 어노테이션이 붙어있는 클래스의 인스턴스들을 찾아서 빈을 등록
    * @ComponentScan : 빈 생성 대상이 되는 패키지 위치를 가리키고, 이 하위 모든 패키지를 찾아본다
      * @Component
        * @Repository
        * @Service
        * @Controller
        * @Configuration
    * 직접 빈을 생성
      * 자바 설정 파일 이용 (요즘의 추세)
  * 빈 주입하기
    * Constructor
    * @Autowired
    * ApplicationContext에서 직접 사용 (사용x)
* 의존성 주입
  * @Autowired/@Inject를 붙이는 곳
  * 생성자
    * spring 4.3이상 부터 생성자가 하나이고 인자들이 빈으로 생성되어 있담녀 @Autowired 어노테이션 없이도 spring framework가 의존성 주입을 해준다
  * 필드
    * 필드 위에 @Autowired를 붙여주면 생성자 없이 DI를 받을 수 있다
  * Setter
    * Setter위에 @Autowired를 붙여주면 생성자 없이 DI를 받을 수 있다
  * 생성자 의존성 주입을 권장하는 이유는 필수 필드를 채워줘야 하는 경우 강제할 수 있는 장점이 있다. 다만 순환참조를 하고 있다면 생성자로 주입 불가하고 필드/setter 주입을 이용해야 한다
## 섹션 3. 스프링 AOP
* Aspect Oriented Programming
  * ex. @Transactional : AOP기반이며 Proxy pattern을 이용하여 구현됨
* AOP를 구현하는 방법
  * 컴파일 : A.java -> (AOP 추가) -> A.class
  * 바이트 코드 조작 : 컴파일된 자바 파일을 메모리 로드시에 AOP를 추가하는 방식, AspectJ의 방식 중 하나
  * 프록시 패턴 : spring AOP의 방식, 실제 구현은 원래의 클래스를 호출하지 않고 한 단계 래핑한 클래스를 호출하고 기존 코드의 앞뒤로 추가적인 작업을 할 수 있게 한다
## 섹션 4. 스프링 PSA
* Portable Service Abstraction
* Spring MVC
  * Servlet을 이용하여 API를 만들어 봤다면, HttpServlet 클래스를 상속받아 doGet/doPost등의 메소드를 오버라이드 하여 만들어야 했다
  * Spring MVC를 이용하면 servlet을 구현할 필요 없이 더 간단하게 코드 작성이 가능하다 (Spring MVC 내부 구현은 servlet을 구현)
  * Spring MVC에서 사용되는 서버를 바꿔가며 사용 가능한 것도 PSA에 해당
* Spring Transaction
  * @Transactional annoataion
  * JDBC 구현체를 코드 변경없이 다른 것으로 바꾸어 사용 가능하다
#### 참고
* 메이븐(Maven)은 알고 스프링(Spring)을 쓰는가?
  * 출처: https://jeong-pro.tistory.com/168 [기본기를 쌓는 정아마추어 코딩블로그]
* [Spring] 스프링 프레임워크(Spring Framework)란?
  * 출처: https://engkimbs.tistory.com/601?category=767795 [새로비]
* 키워드
```
- application context / Bean Factory
- @SpringBootApplication @ConponentScan
- IoC 싱글톤
- DI 생성자/필드/setter @Autowired/@inject
- @Component : Repository/Service/Controller/Configuration
- AOP 프록시패턴 어노테이션만들기
- PSA(Portable Service Abstraction) 
```
