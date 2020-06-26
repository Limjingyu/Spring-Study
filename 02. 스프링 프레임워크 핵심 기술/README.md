# 02. 스프링 프레임워크 핵심 기술
## 섹션 1. IoC 컨테이너와 빈
### IoC 컨테이너 1부: 스프링 IoC 컨테이너와 빈
* 빈등록이유?
  * 의존성을 주입해주기 위해
  * 싱글톤으로 관리하기 위해(싱글톤이란?프로토타입이란?)
  * 라이프사이클(->찾아보기)
### IoC 컨테이너 2부: ApplicationContext와 다양한 빈 설정 방법
* BeanFactory
* 빈 등록 방법
  1. xml 파일 설정
  2. xml, ConponentScan
  3. java, @configuration, @Bean
  4. java @ComponentScan
  5. @SpringBootConfiguration
### IoC 컨테이너 3부: @Autowire
* 주입방법?
  * 생성자
  * 세터
  * 필드
* 유니크하지않으면..
  * @Primary(추천 - type safe)
  * @Qualifier
  * List로 모두 주입받기
### IoC 컨테이너 4부: @Component와 컴포넌트 스캔
* @ComponentScan
* 펑션을 사용한 빈 등록..? -> @ComponentScan을 대체하는거 아님.
### IoC 컨테이너 5부: 빈의 스코프
* 싱글톤
* 프로토타입(proxyMode)
### IoC 컨테이너 6부: Environment 1부. 프로파일
* @profile("")
  * @Configuration에 쓰고 빈등록
  * @Component에 직접
### IoC 컨테이너 6부: Environment 2부. 프로퍼티
* 알파,베타,스테이징,리얼 환경에 설정을 구분할때.. (sample찾기)
### IoC 컨테이너 7부: MessageSource
* 메세지 다국화
* sample (messagess.properties / messagess_ko_kr.properties)
### IoC 컨테이너 8부: ApplicationEventPublisher
* @EventListener
* @Async @EnableAsync
* 옵저버 패턴 찾아보기
### IoC 컨테이너 9부: ResourceLoader
ㅎ

## 섹션 2. Resource / Validation
### Resource 추상화
ㅎ
### Validation 추상화
