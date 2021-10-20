# Spring Security #
## Spring Security 구조 ##

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


### Filter 종류 ###
- UsernamePasswordAuthentication Filter
  - Form 데이터로 username, password 기반의 인증을 담당하는 필터
  - UsernamePasswordAuthenticationFilter => ProviedeManager(AuthenticationManager) 인증정보 제공 관리자 => AbstractUserDetailsAuthenticationProvider 인증 정보 제공 => DaoAuthenticationProvider 유저 정보 제공 => UserDetailsService 유저 정보 제공하는 Service
  
- ExceptionTranslationFilter
    - 인증이나 인가에 실패할 때 Exception을 처리해주는 Filter
    - AuthenticationException: 로그인페이지
    - AccessDeniedException: whitelabel error page, anonymous user라면 로그인페이지
    - 

## Spring Security Config ##
