# Spring Security #
## Spring Security 구조 ##
- ### SecurityContextHolder -> SecurityContext -> Authentication -> Principal & GrantAuthority ###
- ### SecurityContextHolder ###
  - SecurityContextHolder는 SecurityContext를 제공하는 static method (getContext)를 지원
- ### SecurityContext ###
  - SecurityContext는 접근 주체와 인증에 대한 정보를 담고있는 context
  - Authentication을 담고 있다.
- ### Authentication ###
  - Principal 과 GrantAuthority를 제공
  - 인증이 이루어지면 해당 Authentication이 저장
- ### Principal GrantAuthority ###
  - Principal
    - 유저에 대한 정보
    - 대부분 Principal 로 UserDetails 를 반환
  - GrantAuthority
    - ROLE_ADMIN, ROLE_USER 등 Principal이 가지고 있는 권한
    - prefix로 ROLE_ 이 필요
    - 인증 이후 인가를 할 때 사용
    - 권한은 여러가지이기 때문에 Collection<GrantedAuthority> 형태로 제공


## ThreadLocal ##
- 요청 1개 Thread 1개가 생성된다.
- ThreadLocal을 사용하면 스레드마다 고유한 공간을 만들어 그곳에 Security Context를 저장 요청 1개, Threaed 1개, Security Context 1개
- MODE_THREADLOCAL
  - 기본 설정
  - ThreadLocalSecurityContextHolderStrategy 사용
  - ThreadLocal 을 사용하여 같은 Thread dksdptj SecurityContext를 공유
- MODE_INHERITABLETHREADLOCAL
  - Inheritable ThreadLocalSecurityContextHolderStrategy 사용
  - Inheritable ThreadLocal을 사용하여 자식 Thread 까지도 Security Context 공유
- MODE_GLOBAL
  - GlobalSecurityContextHolderStrategy 사용
  - 어플리케이션 전체에서 SecurityContext를 공유

## PasswordEncoder ##
### Password 관리 ###
- 회원가입할 때 password를 입력받으면 그 값을 암호화해서 저장
- 로그인할 때 입력받은 password와 회원가입할 때의 password를 비교할 수 있어야 함
### 절차 ###
- 회원가입할 때 password를 해시함수로 암호화해서 저장
- 로그인할 때 password가 들어오면 같은 해시함수로 암호화
- 저장된 값을 불러와서 2번의 암호화된 값과 비교
- 동이랗면 같은 암호로 처리

### PasswordEncoder 전략 ###
- NoOpPasswordEncoder
  - 평문으로 저장
- BcryptPasswordEncoder
  - Bcrypt해시함수 사용
  - 의도적으로 느리게 설정
  - 강도 설정 가능, 강도가 높을 수록 느리다
- StandardPasswordEncoder
- Pbkdf2PasswordEncoder
  - 미국표준기술연구소에 의해 승인된 알고리즘, 미국정부시스템에서도 사용
- ScryptPasswordEncoder
  - 해커가 무작위로 passowrd를 맞추려고 할 때 메모리 사용량을 늘리거나 줄여서 느린 공격을 실행할 수 밖에없음
  - 보안에 민감한 경우 사용

### Security Filter ###
- Spring Security 의 동작은 사실상 Filter로 동작
- 필터들은 제외할수도, 추가할 수도 있으며 필터의 동작하는 순서를 정해서 유기적으로 동작할 수 있다.
- SecurityContextPersistenceFilter
- Basic AuthenticationFilter
- UsernamePasswordAuthenticationFilter
- CsrfFilter
- RememberMeAuthenticationFilter
- AnonymousAuthenticationFilter
- FilterSecurityInterceptor
- ExceptionTranslationFilter
- 등
- doFilterInternal 에 브레이크포인트를 걸고 디버깅
- FilterOrderRegistration 에 필터순서 정의를 볼 수 있다.

### Filter 종류 ###
- SecurityContextPersistanceFilter
  - async요청에 대해서도 securityContext를 처리할 수 있는 WebAsyncManagerIntegrationFilter 다음으로 두번쨰로 실행됨
  - SecurityContext를 찾아와서 SecurityContextHolder에 넣어주는 필터
  - 로그인 시 쿠키를 주고 쿠키(JSessionID)와 함께 온 요청에 대해 세션으로 SecurityContext를 찾아 유저를 특정함
- BasicAuthenticationFilter
  - 직접 필터를 적용하면 따로 로그인하지 않아도 일회성으로 페이지를 불러온다. username과 password를 로그인데이터를 Base64로 인코딩해서 보내면 BaseAuthenticationFilter는 이것을 인증한다.
  - 세션이 필요없고 매번 인증이 이루어진다.
  - https를 사용해서 아이디와 비밀번호가 노출되지 않도록 한다.
- UsernamePasswordAuthentication Filter
  - Form 데이터로 username, password 기반의 인증을 담당하는 필터
  - UsernamePasswordAuthenticationFilter => ProviedeManager(AuthenticationManager) 인증정보 제공 관리자 => AbstractUserDetailsAuthenticationProvider 인증 정보 제공 => DaoAuthenticationProvider 유저 정보 제공 => UserDetailsService 유저 정보 제공하는 Service
  
  
- ExceptionTranslationFilter
    - 인증이나 인가에 실패할 때 Exception을 처리해주는 Filter
    - AuthenticationException: 로그인페이지
    - AccessDeniedException: whitelabel error page, anonymous user라면 로그인페이지
    - 

## Spring Security Config ##



## JWT ##
### 세션의 장단점 ###
- 장점
  - JSESSIONID 는 유의미한 값이 아니라 서버에서 세션정보를 찾는 key로만 활용된다. 따라서 탈취되었다고 해서 개인정보가 탈취된건 아님(단 세션하이재킹 공격의 위험이 있음)
- 단점
  - 서버에 세션 정보를 저장할 공간이 필요함
  - 서비스를 이용하는 유저가 많다면 저장할 공간이 많이 필요하게 됨
  - 분산 서버에서는 세션을 공유하기 어려움

### 토큰으로 인증하기 ###
유저가 로그인을 하면 서버에서 토큰을 생성한 뒤 저장하지 않고 토큰값을 내려줌
이 토큰 값을 유저가 요청을 할 때 같이 보내고 이 토큰을 해석하여 유저를 인증
세션에서는 JSESSION은 의미없는 값,
Token은 username 등을 포함
- 장점
  - 세션관리를 할 필요가 없어 별도의 저장소가 필요하지 않다.
  - 서버 분산, 클러스터 환경에서 확장성이 좋음
- 단점
  - 한 번 제공된 토큰은 회수가 어렵다.
  - 세션의 경우에는 세션을 삭제해버리면 브라우저의 JSESSIONID는 쓸모가 없다.
  - 토큰은 세션을 저장하지 않기 때문에 한 번 제공된 토큰을 회수할 수 없고 따라서 토큰의 유효기간을 짧게 한다.
  - 토큰에는 유저의 정보가 있기 때문에 안정성이 우려된다. 따라서 민감정보를 토큰에 포함시키면 안된다.
  
- JWT의 구조
- Header
  - JWT를 검증하는데 필요한 정보를 가진 객체
  - Signature에 사용한 암호화 알고리즘이 무엇인지, Key의 ID를 담고 있고 이 정보를 Json으로 변환한 후 UTF-8로 인코딩한 뒤 Base64 URL-Safe로 인코딩한 값
- Payload
  - 실질적으로 인증에 필요한 데이터를 저장
  - 데이터의 각각 필드들을 Claimdlfkrh gka
  - Clain에는 username을 포함한다.
  - 토큰 발행시간, 만료시간 등을 포함
  - Json으로 바꾼뒤 UTF-8로 인코딩하고 Base64로 인코딩한 데이터
- Signature
  - Signature은 Header와 Payload를 합친 뒤 비밀키로 Hash를 생성하여 암호화한다ㅣ
  - 헤더, 페이로드 값을 Secret key로 hashing하고 Base64로 변경

- Key Rolling
  - JWT 토큰 생성 메커니즘 상 Secret key가 노출되면 사실상 모든 데이터가 유출될 수 있다.
  - Secret Key를 여러개를 사용하고 수시로 추가하고 삭제해줘서 변경한다면 하나의 Secret Key가 노출되어도 다른 Secret key는 안전
  - JWT에 kid를 포함하여 제공하고 서버에서 토큰을 해석할 때 kid로 Secret key를 찾아서 Signature를 검증

