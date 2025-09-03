

## 개념

### Filter
- **Servlet 기반 웹 애플리케이션의 필터 메커니즘**
- 요청과 응답이 서블릿 컨테이너와 웹 애플리케이션 사이를 통과할 때 사전/사후 처리를 담당
- `javax.servlet.Filter` 인터페이스를 구현

### Interceptor
- **Spring MVC 프레임워크에서 제공하는 인터셉터 메커니즘**
- HandlerMapping과 HandlerAdapter 사이에서 요청을 가로채어 처리
- `org.springframework.web.servlet.HandlerInterceptor` 인터페이스를 구현

## 주요 차이점

| 구분 | Filter | Interceptor |
|------|---------|-------------|
| **실행 시점** | 서블릿 실행 전/후 | 컨트롤러 실행 전/후 |
| **처리 범위** | 모든 요청 (정적 리소스 포함) | 핸들러로 매핑되는 요청만 |
| **Spring Context 접근** | 제한적 | 완전한 접근 가능 |
| **예외 처리** | 서블릿 레벨 예외만 처리 | Spring의 예외 해결 전략 활용 |
| **설정 방식** | `@WebFilter` 또는 FilterRegistrationBean | `WebMvcConfigurer` 구현 |

## 실행 순서

```
요청 → Filter → DispatcherServlet → Interceptor → Controller → Interceptor → Filter → 응답
```

## 사용 사례

### Filter가 적합한 경우
- 인코딩/디코딩 처리
- 로그 기록
- 인증 및 권한 검사
- XSS, CSRF 보안 처리
- 압축/압축해제

### Interceptor가 적합한 경우
- 로그인 체크
- 권한 체크
- 프로그램 실행시간 계산
- 어드민 체크
- Controller로 넘어가는 정보의 가공

## 구현 예시

### Filter 구현
```java
@Component
public class LoggingFilter implements Filter {
    
    @Override
    public void doFilter(ServletRequest request, ServletResponse response, 
                        FilterChain chain) throws IOException, ServletException {
        
        HttpServletRequest httpRequest = (HttpServletRequest) request;
        System.out.println("Filter: " + httpRequest.getRequestURI());
        
        chain.doFilter(request, response); // 다음 필터 또는 서블릿으로 진행
        
        System.out.println("Filter: Response processed");
    }
}
```

### Interceptor 구현
```java
@Component
public class LoggingInterceptor implements HandlerInterceptor {
    
    @Override
    public boolean preHandle(HttpServletRequest request, HttpServletResponse response, 
                           Object handler) throws Exception {
        System.out.println("Interceptor preHandle: " + request.getRequestURI());
        return true; // false 반환 시 요청 중단
    }
    
    @Override
    public void postHandle(HttpServletRequest request, HttpServletResponse response, 
                          Object handler, ModelAndView modelAndView) throws Exception {
        System.out.println("Interceptor postHandle");
    }
    
    @Override
    public void afterCompletion(HttpServletRequest request, HttpServletResponse response, 
                              Object handler, Exception ex) throws Exception {
        System.out.println("Interceptor afterCompletion");
    }
}
```

### Interceptor 등록
```java
@Configuration
public class WebConfig implements WebMvcConfigurer {
    
    @Autowired
    private LoggingInterceptor loggingInterceptor;
    
    @Override
    public void addInterceptors(InterceptorRegistry registry) {
        registry.addInterceptor(loggingInterceptor)
                .addPathPatterns("/**")
                .excludePathPatterns("/static/**");
    }
}
```

## 성능 고려사항

### Filter
- 서블릿 컨테이너 레벨에서 동작하여 오버헤드 최소
- 모든 요청을 처리하므로 필요한 경우에만 사용

### Interceptor
- Spring 컨텍스트 내에서 동작하여 약간의 오버헤드 존재
- 세밀한 제어가 가능하여 복잡한 로직에 적합

## 결론

**Filter**는 웹 애플리케이션의 전체적인 전처리/후처리가 필요한 경우에 사용하고, **Interceptor**는 Spring MVC의 특정 핸들러에 대한 세밀한 제어가 필요한 경우에 사용하는 것이 적절합니다.

두 기술 모두 요청/응답 처리의 횡단 관심사를 효과적으로 분리할 수 있는 중요한 메커니즘입니다.