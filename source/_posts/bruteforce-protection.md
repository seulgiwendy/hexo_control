---
title: 브루트 포스 공격을 막아봅시다.(Spring Security에서의 예외 처리)
date: 2017-12-13 21:27:35
tags:
    - Java
    - Spring_Security
---

### 간단한 브루트 포스(Brute Force) 공격을 막아보자. 

말은 거창하지만 간단하다. 브루트 포스(Brute Force)란 비밀번호가 맞을 때까지 노가다로 때려맞히는, 아주 원초적이지만 효과적이고 또 돈도 안 드는 공격 방식이다. 

3G 통신이 상용화도 되기 전 대부분의 핸드폰들은 단순한 4자리의 암호 체게를 갖고 있었고, 기본 비밀번호는 4빵, 즉 0000 이었다. 친구들의 핸드폰을 탈취해 꼭 시도해봤던 일 중 하나는 이 네 자리의 암호를 때려맞혀 보는 일이었다. 

비밀번호는 숫자 네 자리로만 이뤄져있으니 단순한 계산으로 생각해보면 0000~9999의 숫자를 한 번씩만 시도해보면 언젠가는 비밀번호를 알아낼 수 있을 일이었다. 그러나 한 다섯번 쯤 시도하면 늘 뜨는 에러 메시지, *입력 횟수 초과! n분 후에 시도하세요.* 가 나의 원대한 도전을 늘 가로막았었다. 

#### 이런 식의 공격이 요즘 웹에 의미가 있나? 

결론부터 말하면 **의미없다고 생각한다.** 브루트 포스 공격이 통하기엔 이미 모던 웹 서비스들은 복잡도가 너무 높은 비밀번호를 요구하고 있다(주인도 까먹어서 문제다). 또 이런 비밀번호에 대한 유효성 검증을 회원가입 시에 프론트 단에서 구현해주기 때문에 브루트 포스 공격으로 비밀번호가 털릴 상황은 그렇게 많이 연출되지 않을 거라고 본다. 

그러나 스프링 시큐리티에서의 브루트 포스 방어는 Spring Boot 기반 프로젝트에서의 캐시 사용, Exception Handler 구현 등 연습해볼 만한 도전과제가 넘쳐나는 구현이다. 

따라서 나는 실 서비스를 위한 보안 목적이라기보단 학습 목적으로 이번 구현을 시작했음을 미리 밝혀둔다. 

#### 무엇을? 어디서? 

로그인 과정 상에서 Exception을 발생시켜 로그인 로직 자체를 멈추게 할 것이란 계획은 세웠다, 그렇다면 **무슨 Exception을 어디서 발생시키나?** 를 고민해야 한다.

우선 누가 암호를 몇 번 틀렸는지 기록하기 위해 com.google.commons에 들어있는 [LoadingCache](https://google.github.io/guava/releases/19.0/api/docs/com/google/common/cache/LoadingCache.html)를 사용할 것이다. Key는 String으로, 횟수를 저장할 Value는 Integer로 저장할 것이다. 또 이 Cache 인스턴스를 들고 있으면서 특정 IP의 차단 여부를 판별하고, 로그인 성공 / 실패 이벤트가 생길 때마다 저장 내용을 조작할 LoginAttemptService를 구현할 것이다.

Exception은 RuntimeException을 상속받는 LoginAttemptExceedException을 만들어 줄 것이며, CustomUserDetailsService.loadByUsername()으로 유저 정보 질의가 들어왔을 때 해당 IP의 차단 여부를 체크해 Exception을 발생시키게 된다. 이를 위해 CustomUserDetailsService는 LoginAttemptService를 Autowire받게 된다. 

#### LoginAttemptService

로그인 실패 시도를 기록하고 차단 여부를 알려줄 서비스는 아래와 같이 구현한다. 

```
@Service
public class LoginAttemptService {

    private final int MAX_ATTEMPT = 2; //허용 시도횟수 두번. 
    private LoadingCache<String, Integer> attemptsCache;

    public LoginAttemptService() {
        super();
        attemptsCache = CacheBuilder.newBuilder()
                .expireAfterWrite(1, TimeUnit.DAYS).build(new CacheLoader<String, Integer>() {
                    @Override
                    public Integer load(String key) throws Exception {
                        return 0;
                    }
                });
    }

    public void loginSucceeded(String key) {
        attemptsCache.invalidate(key);
    }

    public void loginFailed(String key) {
        int attempts = 0;
        try {
            attempts = attemptsCache.get(key);
        } catch(ExecutionException e) {
            attempts = 0;
        }
        attempts++;
        attemptsCache.put(key, attempts);
    }

    public void resetAttempt(String key) {
        try {
            attemptsCache.put(key, 0);
        } catch(Exception e) {

        }
    }

    public boolean isBlocked(String key) {
        try {
            return attemptsCache.get(key) >= MAX_ATTEMPT;
        } catch (ExecutionException e) {
            return false;
        }
    }
}
```

LoadingCache의 .get()이 Exception을 발생시키는 점이 맘에 들지 않지만.... 그냥 Optional<T> 로 구현하면 더 나았을텐데. 하긴 LoadingCache는 인터페이스에 불과하므로 CacheBuilder를 쓰지 않고 내가 직접 구현해서 쓰는 것도 방법이 될 수 있겠다. 

#### IntegratedFormloginFailureHandler

클래스명이 쓸데없이 거창한 모습이다. 이 핸들러를 통해 로그인 에러 페이지도 커스터마이즈하고 싶었기에 클래스명을 이렇게 붙여 봤다. 

구현은 아래와 같이.... 

```
public class IntegratedFormloginFailureHandler implements AuthenticationFailureHandler {
    private static final Logger log = LoggerFactory.getLogger(IntegratedFormloginFailureHandler.class);

    @Override
    public void onAuthenticationFailure(HttpServletRequest req, HttpServletResponse res, AuthenticationException e) throws IOException, ServletException {
        if (isBruteForceEvent(e)) {
            log.error("brute force event detected from failure handler!");
            log.error("cause of exception : {}" , e.getCause().getClass().getName());
            res.sendRedirect("/captcha");//횟수 초과 시에 이동할 주소.
            return;
        }
        log.error("general sign-in fail event detected from failure handler!");
        res.sendRedirect("/signin");
    }

    private boolean isBruteForceEvent(AuthenticationException e) {
        return e.getCause() instanceof LoginAttemptExceedException;
    }
}
```

isBruteForceEvent()의 e.getCause()가 신경이 쓰일 거라 생각한다. 사실 이렇게 구현해야 한다는 것을 수십번의 삽질, 그리고 Throwable에 대한 학습 후 알아낼 수 있었다. 

우선 로그인 과정에서 Exception이 발생했을 때 콘솔에 어떤 메시지가 뜨는지를 살펴보자 

```
org.springframework.security.authentication.InternalAuthenticationServiceException: login attempt exceeds!
	at org.springframework.security.authentication.dao.DaoAuthenticationProvider.retrieveUser(DaoAuthenticationProvider.java:126)
	(...)
	at java.lang.Thread.run(Thread.java:748)
Caused by: codesquad.security.Exceptions.LoginAttemptExceedException: login attempt exceeds!
	at codesquad.security.CustomUserDetailsService.loadUserByUsername(CustomUserDetailsService.java:49)
	at org.springframework.security.authentication.dao.DaoAuthenticationProvider.retrieveUser(DaoAuthenticationProvider.java:114)
	... 53 common frames omitted
```

Stack Trace상에서 Caused By: 를 주목할 필요가 있겠다. 우선 Spring Security에서 사용자 인증 과정중에 발생한 모든 Exception은 InternalAuthenticationServiceException으로 Wrapping된 후에 던져진다는 점을 알 수 있었다. **이걸 알기 전까지 무수한 삽질을 반복해야 했다.** 이걸 모르고 Handler상에서 LoginAttemptExceedException을 바로 찾았으니 당연히 찾아지지 않았다. 

InternalAuthenticationServiceException을 뜯어보면 이 클래스는 AuthenticationServiceException을 상속받고 있고 이는 다시 AuthenticationException을 상속받은 예외다. 

스프링 시큐리티가 예외 처리용으로 제공하는 AuthenticationFailureHandler 인터페이스의 .onAuthenticationFailure() 메소드는 AuthenticationException을 인자로 받고 있는데, Spring Security 단에서 발생하는 모든 AuthenticationException과 이를 상속받은 예외들이 이 Handler를 통해 처리되게 되는 것이다. 

사용자 인증 과정에서 발생하는 **AuthenticationException을 상속받지 않은 모든 예외는 InternalAuthenticationServiceException으로 감싸진 채로 던져지므로** e.getCause()를 호출해 Cause Exception을 불러와야 우리가 구현한 LoginAttemptExceedException의 내용을 확인할 수 있는 것이다. 

아, 말로 써놓고 보니 어렵다. 이런 방식을 Chained Exception 이라고 하는 것 같은데 [이 글](https://docs.oracle.com/javase/tutorial/essential/exceptions/chained.html)을 참고해 보면 좋겠다. 

어려운 구현은 다 끝났고 남은 건 자질구레한 설정들이다. 

#### Spring Security의 SecurityConfig을 일부 변경. 

```
@Configuration
@EnableOAuth2Client
public class SecurityConfig extends WebSecurityConfigurerAdapter {

    private IntegratedFormloginFailureHandler failureHandler = new IntegratedFormloginFailureHandler();

    ...

     .formLogin()
                    .loginPage("/signin")
                    .loginProcessingUrl("/signin")
                    .successHandler(this.signinRedirectionHandler)
                    .failureHandler(this.failureHandler)//아까 공들여 만든 실패 핸들러를 등록시켜준다. 
                    .permitAll()
```

#### CustomUserDetailsService가 IP 차단 여부를 체크하도록 변경.

```
...

@Override
public UserDetails loadUserByUsername(String username) throws UsernameNotFoundException {
        log.debug("load username : {}" , username);

        String ip = getClientIP();
        if (loginAttemptService.isBlocked(ip)) {
            
            throw new LoginAttemptExceedException("login attempt exceeds!");
      
        }
        ...
    }

    //클라이언트의 IP를 알아내는 메소드

    private String getClientIP() { 
        String header = req.getHeader("X-Forwarded-For");
        if (header == null) {
            return req.getRemoteAddr();
        }
        return header.split(",")[0];
    }
```

**어때요, 참 쉽죠?**

리디렉션을 /captcha로 돌려주는 것에서 이미 짐작하셨겠지만 다음 구현과제는 Google의 [reCAPTCHA](https://www.google.com/recaptcha/intro/android.html)를 서비스에 적용해보는 것이다. 

#### 유용했던 참고자료

[Prevent Brute Force Authentication Attempts with Spring Security](http://www.baeldung.com/spring-security-block-brute-force-authentication-attempts)

[LoadingCache Javadoc](https://google.github.io/guava/releases/19.0/api/docs/com/google/common/cache/LoadingCache.html)

+포비의 힌트. 감사합니다~!



