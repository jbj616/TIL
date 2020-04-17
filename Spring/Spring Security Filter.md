## Spring Security 학습

![](https://github.com/spring-guides/top-spring-security-architecture/raw/master/images/security-filters.png)

<출처 : Spring.io>

스프링 시큐리티는 필터를 사용해 인증과 요청 단위 권한 부여를 수행한 때 Client측으로 요청이 들어요면 Filter를 지나가게 되는 때 FilterChainProxy라는 필터에서 요청을 전달받아 Spring Security Filters를 수행한다. 이 때 요청은 모든 필터를 통과하고 난후 컨트롤러에 도달하기 때문에 요청으로 부터 안전하게 처리할 수 있으며 Service에 도달하기 전에 AOP를 통해서 메소드 단위의 권한 부여를 수행할 수 있다.



### Spring Security Filter Chain

> https://docs.spring.io/spring-security/site/docs/3.0.x/reference/security-filter-chain.html

수많은 필터를 거치게 되다 그 중 몇가지 중요한? 필터를 선정해보았다.

- SecurityContextPersistenceFilter

  - 요청이 들어오면 SecurityContextRepository에서 SecurityContext를 불러와 SecurityContextHolder에서 SecurityContext를 설정한다. 이 때 SecurityContext에 대한 변경사항을 HttpSession에 복사여 사용할 수 있다.

- UsernamePasswordAuthenticationFilter

  - 인증이 시작되는 곳

- ExceptionTranslationFilter

  - 스프링 시큐리티의 예외를 해석하는 책임

- FilterSecurityInterceptor

  - 허용된 요청인지 결정하는 작업을 AccessDesicionManager에게 위힘하여 접근 허용을 확인
  - 허가 거부시 예외를 발생시켜 URI 접근을 막는다

  

### 공용 리소스에 대한 filter Debug

```bash
o.s.security.web.FilterChainProxy : /register at position 1 of 11 in additional filter chain; firing Filter: 'WebAsyncManagerIntegrationFilter'
o.s.security.web.FilterChainProxy : /register at position 2 of 11 in additional filter chain; firing Filter: 'SecurityContextPersistenceFilter'
```

- 요청이 WebAsyncManagerIntegrationFilter와 SecurityContextPersistenceFilter를 통과한다

```bash
w.c.HttpSessionSecurityContextRepository : HttpSession returned null object for SPRING_SECURITY_CONTEXT
w.c.HttpSessionSecurityContextRepository : No SecurityContext was available from the HttpSession: org.apache.catalina.session.StandardSessionFacade@2317acfa. A new one will be created.
```

- SecurityContext를 구성해야하는 데 존재 하지 않기 때문에 생성한다.

```bash
o.s.security.web.FilterChainProxy : /register at position 3 of 11 in additional filter chain; firing Filter: 'HeaderWriterFilter'
o.s.security.web.FilterChainProxy : /register at position 4 of 11 in additional filter chain; firing Filter: 'LogoutFilter'
```

- logout시 처리되는 요청인지 판단한다

```bash
o.s.security.web.FilterChainProxy : /register at position 5 of 11 in additional filter chain; firing Filter: 'UsernamePasswordAuthenticationFilter'
o.s.security.web.FilterChainProxy : /register at position 6 of 11 in additional filter chain; firing Filter: 'RequestCacheAwareFilter'
o.s.security.web.FilterChainProxy : /register at position 7 of 11 in additional filter chain; firing Filter: 'SecurityContextHolderAwareRequestFilter'
```

- 로그인 처리 URL과 일치 하지 않기 때문에 그냥 지나친다

```bash
o.s.security.web.FilterChainProxy : /register at position 8 of 11 in additional filter chain; firing Filter: 'AnonymousAuthenticationFilter'
 o.s.s.w.a.AnonymousAuthenticationFilter  : Populated SecurityContextHolder with anonymous token: 'org.springframework.security.authentication.AnonymousAuthenticationToken@58ada9f6: Principal: anonymousUser; Credentials: [PROTECTED]; Authenticated: true; Details: org.springframework.security.web.authentication.WebAuthenticationDetails@fffbcba8: RemoteIpAddress: 0:0:0:0:0:0:0:1; SessionId: 72AF38EE5C7AE6E7E5FBBC2DF06E80EF; Granted Authorities: ROLE_ANONYMOUS'
```

- 로그인 처리 즉, 인증을 하지 않았기 때문에 Anonymous 인증 토근을 생성해서 넘긴다

```bash
o.s.security.web.FilterChainProxy : /register at position 9 of 11 in additional filter chain; firing Filter: 'SessionManagementFilter'
```

- 세션 ID가 유효한지 판단한다

```bash
o.s.security.web.FilterChainProxy : /register at position 10 of 11 in additional filter chain; firing Filter: 'ExceptionTranslationFilter'
```

- 시큐리티 예외가 발생한 것을 잡는다

```bash
o.s.security.web.FilterChainProxy : /register at position 11 of 11 in additional filter chain; firing Filter: 'FilterSecurityInterceptor'
o.s.s.w.u.matcher.AntPathRequestMatcher  : Checking match of request : '/register'; against '/register'
o.s.s.w.a.i.FilterSecurityInterceptor    : Secure object: FilterInvocation: URL: /register; Attributes: [permitAll]
...
```

- 요청에 대한 판단이 이루어진다