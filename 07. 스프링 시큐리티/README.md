# 06. 스프링 시큐리티
## 스프링 시큐리티 : 폼 인증
### 폼 인증 예제 살펴보기
* 특정 url들은 인증 없이도 접근이 가능하고, 또 다른 url들은 인증이 되어이어야 접근 가능하며 인증이 안되어있는 경우 login page로 redirection 됨.
* 로그인/로그아웃 화면을 spring security가 기본 제공하는 것 외에 커스터마이징 하는 방법, 또, 설정을 커스터마이징 하는 방법, spring security가 sevlet 기반의 spring web mvc와 맞물려 동작하는지 등을 알아볼 예정
 
### 스프링 웹 프로젝트 만들기
* https://start.spring.io/
    * 위 페이지로 spring boot application을 gradle로 만들었을 때, dependency-management plugin이 추가됨. 덕분에 dependency 등록 시 version을 명시하지 않아도 된다.
* thymeleaf의 장점
    * web server를 띄우지 않고도 웹 페이지의 모양새를 확인 가능함
* code
```html
<!-- index.html, 동일하게 title만 바꿔서 admin, dashboard, info html 생성 -->
<!DOCTYPE html>
<html lang="en" xmlns:th="http://www.thymeleaf.org">
<head>
    <meta charset="UTF-8">
    <title>Index</title>
</head>
<body>
    <h1 th:text="${message}">Hello</h1>
</body>
</html>
```
```java
// SampleController
@Controller
public class SampleController {
 
    @GetMapping("/")
    public String index(Model model, Principal principal) {
        // To separate whether status is login
        if (principal == null) {
            model.addAttribute("message", "Hello Spring Security");
        } else {
            model.addAttribute("message", "Hello, " + principal.getName());
        }
        return "index";
    }
 
    @GetMapping("/info")
    public String info(Model model) {
        model.addAttribute("message", "Info");
        return "info";
    }
 
    @GetMapping("/dashboard")
    public String dashboard(Model model, Principal principal) {
        model.addAttribute("message", "Hello, " + principal.getName());
        return "dashboard";
    }
 
    @GetMapping("/admin")
    public String admin(Model model, Principal principal) {
        model.addAttribute("message", "Hello, " + principal.getName());
        return "admin";
    }
}
```
 
### 스프링 시큐리티 연동
* spring boot의 도움을 받아 spring-boot-starter-security dependency를 추가하면 자동설정이 됨
    * 이제 index.html만 접속을 해도 login 페이지로 리다이렉션 됨
    * default
        * user name : user
        * password : console에 찍히는 Using generated security passrword : XXXX 의 값
* spring security 추가 시 해결한 문제
    * 인증 가능
    * 현재 사용자 정보를 알 수 있음
* 현재 문제점
    * 인증 없이 접근 가능한 URL 설정 필요
    * user 하나 뿐
    * pw가 로그에 남음
 
### 스프링 시큐리티 설정하기
* WebSecurityConfigurerAdapter 상속받아 커스터마이징
* 아래 설정을 하면 index.html, info.html은 인증 없이 접근 가능해 진다 -> "인증 없이 접근 가능한 URL 설정 필요" 부분 해결
* code
```java
@Configuration
@EnableWebSecurity
public class SecurityConfig extends WebSecurityConfigurerAdapter {
    @Override
    protected void configure(HttpSecurity http) throws Exception {
        http.authorizeRequests()
                // can access /, /info pages without any authentication
                .mvcMatchers("/", "/info").permitAll()
                // /admin page should have "ADMIN" role
                .mvcMatchers("/admin").hasRole("ADMIN")
                // Any other pages need authentication
                .anyRequest().authenticated()
                .and()
                // use form login
                .formLogin()
                .and()
                // use http basic auth
                .httpBasic()
        ;
    }
}
```
 
### 스프링 시큐리티 커스터마이징: 인메모리 유저 추가
* 처음 spring security를 디펜던시에 등록하여 사용할 때 등록되는 user와 그에 따른 pw를 logging하는 것
    * UserDetailsServiceAutoConfiguration에서 정의되어 있는 것이다. 
    * inMemoryUserDetailsManager에서 InMemoryUserDetailsManager를 생성하여 주고, pw는 random uuid를 생성하는 로직이 들어있다.
    * 생성에 사용되는 외부 설정이 spring.security 으로 되어있다.
* code
```java
// in SecurityConfig.java
@Override
protected void configure(AuthenticationManagerBuilder auth) throws Exception {
    auth.inMemoryAuthentication()
            // Prefix of {noop} sets encoding of password
            .withUser("suhyeon").password("{noop}123").roles("USER").and()
            .withUser("admin").password("{noop}!@#").roles("ADMIN");
}
```
* 문제 해결
    * 이제 여러 사용자와, admin이 존재하게 된다.
* 또 다른 문제
    * 사용자를 매번 코딩해서 추가해야 하는가? DB를 사용하자
 
### 스프링 시큐리티 커스터마이징: JPA 연동
* dependency
    * jpa
        * group : org.springframework.boot
        * name : org.springframework.boot:spring-boot-starter-data-jpa
    * h2
        * group : com.h2database
        * name : h2
        * scope : runtime // gradle : runtimeOnly
* code
```java
// UserDetailsService implementation
@Service
public class AccountService implements UserDetailsService {
    @Autowired
    private AccountRepository accountRepository;
 
    @Override
    // This method has duty that returns UserDetails by using username.
    public UserDetails loadUserByUsername(String username) throws UsernameNotFoundException {
        Account account = accountRepository.findByUsername(username);
        if (account == null){
            throw new UsernameNotFoundException(username);
        }
        return User.builder()
                .username(account.getUsername())
                .password(account.getPassword())
                .roles(account.getRole())
                .build();
    }
}
 
// Account entity
@Entity
public class Account {
    @Id
    @GeneratedValue
    private Integer id;
    @Column(unique = true)
    private String username;
    private String password;
    private String role;
    // getter and setter
}
 
 
// AccountRepository
public interface AccountRepository extends JpaRepository<Account, Integer> {
    Account findByUsername(String username);
}
 
// Remove configure(AuthenticationManagerBuilder auth) method in SecurityConfig
 
// Add temporary controller. It should not be used at real service!!
@RestController
public class AccountController {
    @Autowired
    private AccountRepository accountRepository;
 
    @GetMapping("/account/{role}/{username}/{password}")
    public Account createAccount(@ModelAttribute Account account) {
        return accountRepository.save(account);
    }
}
 
// And permit /account/** urls
```
* error : "There is no PasswordEncoder mapped for the id "null""
    * spring security 버전 5이상일 때 발생. password algorithm을 prefix로 지정해주어야 한다.
    * 여기서는 Account entity에 hardcoding하여 {noop}을 추가하도록 했지만 실제로는 PasswordEncoder를 이용하면 된다
* UserDetailsService 는 SecurityConfig에 overriding하여 userDetailsService에 setService를 해주어야 하지만 또 다른 방법으로 UserDetailsService를 상속받는 클래스가 빈으로 등록되면 자동으로 설정 해 준다.
 
### 스프링 시큐리티 커스터마이징: PasswordEncoder
* 기존 코드에서 {noop}을 제거 해 보자
    * spring security가 제공하는 PasswordEncoder를 사용
* code
```java
// PasswordEncoder bean 등록
@Bean
public PasswordEncoder passwordEncoder() {
  return NoOpPasswordEncoder.getInstance();
}
// AccountService에서 bean으로 등록된 PasswordEncoder 주입 & 사용
@Autowired
private PasswordEncoder passwordEncoder;
…
public Account createNew(Account accout) {
  account.encodePassword(passwordEncoder);
  return this.accountRepository.save(account);
}
// Account entity 수정
  public void encodePassword(PasswordEncoder passwordEncoder) {
  this.password = passwordEncoder.encode(this.password);
}
```
* 위와 같이 설정하면 이제 password 입력 시 {noop}을 넣어주지 않고도 PasswordEncoder mapped for the id "null" 에러가 발생하지 않음
* spring security 5버전 이하에서는 PasswordEncoder를 적용하지 않고도 이렇게 동작했는데 변경된 이유?
    * 기존 noop 을 사용하는 것에서 여러가지 인코딩을 제공하기 위함
    * 히스토리는 spring security docs를 확인
* bcrypt로 암호화 하기 (noop은 추천하지 않는다)
```java
@Bean
public PasswordEncoder passwordEncoder() {
  return PasswordEncoderFactories.createDelegatingPasswordEncoder();
}
```
 
### 스프링 시큐리티 테스트 1부
* 지금까지 확인 방법은 직접 web browser를 통해 user를 만들고 로그인 한다음 확인을 해봄
* 이제 매번 확인하기 보다 source code를 이용하여 테스트를 만들어 봄
* dependency
    * group : org.springframework.security
    * name : spring-security-test
    * scope : test
    * version : spring boot dependencies가 spring security test의 버전을 가지고 있지 않아서 ${spring-security.version}을 이용하여 버전을 지정해 주면 spring security와 동일한 버전을 사용하게 된다
* code
```java
// AccountControllerTest
@RunWith(SpringRunner.class)
@SpringBootTest
@AutoConfigureMockMvc
public class AccountControllerTest {
  @Autowired
  MockMvc mockMvc;
 
  @Test
  @WithAnonymousUser
  public void index_anonymous() throws Exception {
    mockMvc.perform(get("/")
    //.with(anonymous()) // it can be replaced by @WithAnonymousUser annotation
    )
    .andDo(print())
    .andExpect(status().isOk());
}
 
  @Test
  @WithMockUser(username = "suhyeon", roles = "USER")
  public void index_user() throws Exception {
    mockMvc.perform(get("/")
    // Mocking user which is already login as a role of USER
    //.with(user("suhyeon").roles("USER")) // it can be replaced by @WithMockUser annotation
    )
    .andDo(print())
    .andExpect(status().isOk());
  }
 
  @Test
  @WithMockUser(username = "admin", roles = "ADMIN")
  public void index_admin() throws Exception {
    mockMvc.perform(get("/"))
    .andDo(print())
    .andExpect(status().isOk());
  }
 
  @Test
  @WithUser
  public void admin_user() throws Exception {
    mockMvc.perform(get("/admin"))
    .andDo(print())
    .andExpect(status().isForbidden());
  }
 
  @Test
  @WithMockUser(username = "admin", roles = "ADMIN")
  public void admin_admin() throws Exception {
    mockMvc.perform(get("/admin"))
    .andDo(print())
    .andExpect(status().isOk());
  }
}
// Custom Annotation, WithMockUser is meta annotation
@Retention(RetentionPolicy.RUNTIME)
@WithMockUser(username = "suhyeon", roles = "USER")
public @interface WithUser {
}
```
 
 
### 스프링 시큐리티 테스트 2부
* 폼 로그인/로그아웃 테스트
    * perform(formLogin())
    * perform(formLogin().user("admin").password("pass"))
    * perform(logout())
* spring security test가 제공하는 method
    * formLogin() : mockMvc.perform에서 form login을 사용하게 해준다
    * authenticated(), unauthenticated() : perform의 결과로 인증 여부를 확인할 때 사용된다
* Transactional annotation
    * test에서 중복된 데이터를 insert하는 테스트가 여러 개 있다면 테스트가 깨질 수 있는데, Transactional annotation이 테스트 후에 롤백을 시켜주어 해결 할 수 있다.
* code
```java
@Test
@Transactional
public void login_success() throws Exception {
    String username = "suhyeon";
    String password = "123";
    createUser(username, password);
 
    mockMvc.perform(formLogin().user(username).password(password))
            .andExpect(authenticated());
}
 
@Test
@Transactional
public void login_fail() throws Exception {
    String username = "suhyeon";
    String password = "123";
    createUser(username, password);
 
    mockMvc.perform(formLogin().user(username).password("12345"))
            .andExpect(unauthenticated());
}
 
private Account createUser(String username, String password) {
    Account account = new Account();
    account.setUsername(username);
    account.setPassword(password);
    account.setRole("USER");
    return accountService.createNew(account);
}
```

## 스프링 시큐리티 : 아키텍처
### SecurityContextHolder와 Authentication
* https://docs.spring.io/spring-security/site/docs/5.1.5.RELEASE/reference/htmlsingle/#core-components 
* SecurityContextHolder
    * 인증을 담고 있는 곳
    * SecurityContext 제공, 기본적으로 ThreadLocal을 사용
    * 동일 Thread 내에서 수행되는 중이라면 authentication 정보를 그대로 사용 가능하고 만약 다른 Thread라면 SecurityContextHolder의 다른 전략을 이용해야 한다. 보통은 web service, servlet 기반이니 요청을 처리할 때 async가 아니라면 동일 thread내에서 처리된다
    * controller의 handler argument로 Principal을 받아오지 않아도 SecurityContextHolder를 이용하여 사용자의 권한 정보를 가져올 수 있다.
* SecurityContext
    * Authentication 제공

* Authentication
    * Principal과 GrantAuthority 제공
    * Principal
        * Authentication 내에 속해있는 객체. 접근은 SecurityContextHolder -> SecurityContext -> Authentication -> Principal
        * "누구" 에게 해당하는 정보
        * UserDetailsService에서 리턴한 그 객체
            * UserDetailsService
                * 유저 정보를 UserDetails 타입으로 가져오는 DAO interface
                * 인증 정보를 어디서 어떻게 가져올 지 명시하고 실제로 인증을 하는 것은 AuthenticationManager가 이 UserDetailsService가 리턴한 UserDetails 객체를 가지고 하는 것이다.
                * 구현은 마음대로 (강좌에서는 spring data jpa를 이용)
        * 객체는 UserDetails 타입
            * UserDetails
                * 애플리케이션이 가지고 있는 유저 정보와 스프링 시큐리티가 사용하는 Authentication 객체 사이의 어댑터
    * GrantAuthority
        * "ROLE_USER", "ROLE_ADMIN" 등 Principal이 가지고 있는 "권한"을 나타냄
        * 인증 이후, 인가 및 권한 확인 시 이 정보를 참조
 
### AuthenticationManager와 Authentication
* spring security에서 인증(Authentication)은 AuthenticationManager가 한다
* Authentication authenticate(Authentication authentication) throws AuthenticationException;
    * 인자로 받은 Authentication이 유효한 인증인지 확인하고 Authentication 객체를 리턴한다
    * 인증을 확인하는 과정에서 비활성 계정, 잘못된 pw, 잠긴 계정 등 여러 에러를 던질 수 있다
    * 보통 구현체는 ProviderManager.java를 사용하게 된다
* 인자로 받은 Authentication
    * (폼 인증의 경우)사용자가 입력한 인증에 필요한 정보(username, password)로 만든 객체
    * Authentication
        * Principal: "suhyeon"
        * Credentials: "123"
* 유효한 인증인지 확인
    * 사용자가 입력한 password가 UserDetailsService를 통해 읽어온 UserDetails 객체에 들어있는 password와 일치하는지 확인
    * 해당 사용자 계정이 잠겨있진 않은지 비활성 계정은 아닌지 등 확인
* Authentication 객체를 리턴
    * Authentication
        * Principal : UserDetailsService에서 리턴한 그 객체 (spring.security.principal.User)
        * Credentials:
        * GrantedAuthorities
 
### ThreadLocal
* Java.lang 패키지에서 제공하는 thread 범위 변수. 즉, thread 수준의 데이터 저장소
    * 같은 thread 내에서만 공유
    * 따라서 같은 thread라면 해당 데이터를 메소드 매개변수로 넘겨줄 필요 없다
    * SecurityContextHolder의 기본 전략
* ThreadLocal sample code
```java
public class AccountContext { 
   private static final ThreadLocal<Account> ACCOUNT_THREAD_LOCAL = new ThreadLocal<>(); 
 
   public static void setAccount(Account account) { 
      ACCOUNT_THREAD_LOCAL.set(account); 
   } 
 
   public static Account getAccount() {
      return ACCOUNT_THREAD_LOCAL.get(); 
   }
} 
```
 
### Authentication과 SecurityContextHolder
* AuthenticationManager가 인증을 마친 뒤 리턴받은 Authentication 객체의 행방은?
* UsernamePasswordAuthenticationFilter
    * 폼 인증을 처리하는 시큐리티 필터
    * 인증된 Authentication 객체를 SecurityContextHolder에 넣어주는 필터
    * UsernamePasswordAuthenticationFilter.java
```java
Authentication attemptAuthentication(request, reponse) {
   …
   return this.getAuthenticationManager().authenticate(authRequest); // 인증 하는 부분. 위에서 말한 "인증은 AuthenticatinoManager가 한다"에 해당
}
// AbstractAuthenticationProcessingFilter.java , super class of UsernamePasswordAuthenticationFilter
doFilter() {
   …
   successfulAuthentication(request, response, chain, authResult);
}
successfulAuthentication(request, response, chain, authResult) {
   …
   SecurityContextHolder.getContext().setAuthentication(authResult);
}
```
* SecurityContextPersistenceFilter
    * SecurityContext를 HTTP session에 캐시(기본 전략)하여 여러 요청에서 Authentication을 공유하는 필터
    * SecurityContextRepository를 교체하여 세션을 HTTP session이 아닌 다른 곳에 저장하는 것도 가능
    * SecurityContextPersistenceFilter.java
```java
…
SecurityContext contextBeforeChainExecution = repo.loadContext(holder); // HttpSession에서 인증 데이터를 꺼내서 HttpSessionSecurityContextRepository에서 저장된 인증 정보를 가져오는 시도를 함
try {
   SecurityContextHolder.getContext().setAuthentication(authentication); // 로그인이 되기 전엔 NullAuthentication이고, 
   …
}
```
 
### 스프링 시큐리티 필터와 FilterChainProxy
* 스프링 시큐리티에서 필터를 관리하는 방법
* 스프링 시큐리티가 제공하는 필터들 
1. WebAsyncManagerIntergrationFilter 
2. SecurityContextPersistenceFilter 
3. HeaderWriterFilter
4. CsrfFilter
5. LogoutFilter
6. UsernamePasswordAuthenticationFilter 
7. DefaultLoginPageGeneratingFilter 
8. DefaultLogoutPageGeneratingFilter
9. BasicAuthenticationFilter
10. RequestCacheAwareFtiler
11. SecurityContextHolderAwareReqeustFilter 
12. AnonymouseAuthenticationFilter 
13. SessionManagementFilter 
14. ExeptionTranslationFilter 
15. FilterSecurityInterceptor 
* 이 모든 필터는 FilterChainProxy가 호출한다

* FilterChainProxy에서는 다음과 같은 과정으로 처리한다
1. 모든 SecurityFilterChain에서 url pattern에 매칭되는 FilterChain들을 걸러낸다
2. 각 FilterChain을 순회하며 실행한다
* SecurityFilterChain 리스트는 어디서 만드는건가?
    * 이전에 설정해 봤던 SecurityConfig extends WebSecurityConfigurerAdapter
    * 이 것을 여러개 생성하면 SecurityFilterChain에 여러 개 등록된다
        * SecurityFilterChain에서 url pattern을 걸러낼 때 http.antMather("url pattern")을 지정해주면 된다
        * 생성한 config의 @Order()를 통해 filter 순서도 결정된다
        * 사실 Order보다는 antMatcher로 핸들링하는 것이 더 좋다
 
### DelegatingFilterProxy와 FilterChainProxy
* FilterChainProxy를 호출하는건 누가?
* DelegatingFilterProxy 
    * 일반적인 서블릿 필터
    * 서블릿 필터 처리를 스프링에 들어있는 특정 빈으로 위임하고 싶을 때 사용하는 서블릿 필터
    * 타겟 빈 이름을 설정한다
    * 스프링 부트 없이 스프링 시큐리티 설정할 때는  AbstractSecurityWebApplicationInitializer를 사용해서 등록 
    * 스프링 부트를 사용할 때는 자동으로 등록 된다(SecurityFilterAutoConfiguration)  FilterChainProxy 
* 보통 “springSecurityFilterChain” 이라는 이름의 빈으로 등록된다

* 이 전에 살펴본 15개의 Filter는 ServeltFilter의 interface를 따랐지만 실제로 servlet filter로 등록되는 것은 아니고 spring 내부에서 사용하는 것이다
* 정리하자면
    * SecurityContextHolder가 Authentication을 가지고 있음
    * AuthenticationManager가 Authentication을 인증
    * 인증된 Authentication을 여러 가지 Filter들이 SecurityContextHolder에 다시 넣어준다
    * 여러 가지 Filter들은 FilterChainProxy가 호출한다
    * FilterChainProxy는 DelegatingFilterProxy가 호출한다
 
### AccessDecisionManager 1부
* AccessControl을 다루는 인터페이스, 권한에 따른 인가에 대한 처리를 다룬다.
* Access Control 결정을 내리는 인터페이스로, 구현체 3가지를 기본으로 제공한다. 
    * AffirmativeBased(default): 여러 Voter중에 한명이라도 허용하면 허용. 기본 전략. 
    * ConsensusBased: 다수결 
    * UnanimousBased: 만장일치 
* AccessDecisionVoter 
    * 해당 Authentication이 특정한 Object에 접근할 때 필요한 ConfigAttributes(SecurityConfig에서 설정한 정책)를 만족하는지  확인한다. 
    * WebExpressionVoter: 웹 시큐리티에서 사용하는 기본 구현체, ROLE_Xxxx가  매치하는지 확인. 
    * RoleHierarchyVoter: 계층형 ROLE 지원. ADMIN > MANAGER > USER 
* vote의 결과를 확인해보려면 AccessDesisionManager 구현체(AffirmativeBased, ConsensusBased, UnanimousBased)에서 디버깅 해볼 수 있다
 
### AccessDecisionManager 2부
* AccessDecisionManager 또는 Voter를 커스터마이징 하는 방법
* 예제
    * 설정
        * ROLE_USER만 /user에 접근 가능하도록 설정
        * 그 결과 ROLE_ADMIN이 /user에 접근하면 403
    * 문제점
        * 문맥상 당연히 admin이 user에 접근하는 것이 가능해야 하는데 접근 권한이 없음
    * 해결
        * AccessDecisionManager를 커스터마이징
        * code
```java
// SecurityConfig.java
public AccessDecisionManager accessDecisionManager() {
    RoleHierarchyImpl roleHierarchy = new RoleHierarchyImpl();
    roleHierarchy.setHierarchy("ROLE_ADMIN > ROLE_USER");
 
    DefaultWebSecurityExpressionHandler handler = new DefaultWebSecurityExpressionHandler();
    handler.setRoleHierarchy(roleHierarchy);
 
    WebExpressionVoter webExpressionVoter = new WebExpressionVoter();
    webExpressionVoter.setExpressionHandler(handler);
    List<AccessDecisionVoter<? extends Object>> voters = Arrays.asList(webExpressionVoter);
 
    return new AffirmativeBased(voters);
}
protected void configure(HttpSecurity http) throws Exception {
   http.
   …
   .accessDecisionManager(accessDecisinoManager())
   …
}
```
