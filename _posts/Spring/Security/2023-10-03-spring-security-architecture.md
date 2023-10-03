---
title: Spring Security Architecture
author: ryan park
date: 2023-10-03 +0800
categories: [Spring, Security]
tags: [spring, java]
# image:
#   path: https://github.com/kiryanchi/kiryanchi.github.io/assets/50714602/d21330ba-9c9e-49cd-aa73-bf900c7cc7ba
#   alt: clean code text image
---

# 배경
- Spring Security 6.1.4

# Spring Security

Spring Security는 인증(authentication)과 인가(authorization), 널리 알려진 공격에 대한 방어(protection aginst common attacks)를 제공하는 프레임워크다.

# A Review of Filters

Spring Security의 Servlet은 Servlet Filters를 기반으로 동작한다.
그래서 우리는 Security's Architecture를 알아보기전에 Filter에 대해 먼저 알아보자.

HTTP 요청이 들어왔을 때 일반적으로 다음 그림처럼 계층을 나눠 처리한다.

![Filter Chain](https://docs.spring.io/spring-security/reference/_images/servlet/architecture/filterchain.png)

사용자는 어플리케이션에 요청을 보낸다. 그럼 요청받은 `URI`를 기반으로 `HttpServletRequest`를 처리하는 `Filter` 인스턴스와 `Servlet`을 포함한 `FilterCahin` 컨테이너를 생성한다.
이때, Spring MVC 어플리케이션에서 `Servlet`은 `DispatcherServlet`의 인스턴스다.
대부분 하나의 처리(`HttpServletRequest`와 `HttpServletResponse`)에 한개의 `Servlet`이 생성된다.
하지만 `Filter`는 여러개 사용될 수 있다.

- 다운스트림 `Filter` 인스턴스 혹은 `Servlet` 인스턴스가 호출되는 것을 막을 수 있다. 이 경우, `Filter`는 `HttpServletResponse`를 작성한다.
- 다운스트림 `Filter` 인스턴스 혹은 `Servlet` 인스턴스에 사용되는 `HttpServletRequest` 혹은 `HttpServletResponse`를 수정할 수 있다.

```java
public void doFilter(ServletRequest request, ServletResponse response, FilterChain chain) {
	// do something before the rest of the application
    chain.doFilter(request, response); // invoke the rest of the application
    // do something after the rest of the application
}
```

`Filter`의 강력한 점은 `FilterChain`에서 나온다. `Filter`는 다운스트림 `Filter` 인스턴스 혹은 `Servlet` 인스턴스에만 영향을 끼치므로 `Filter`가 호출되는 순서는 **굉장히** 중요하다.

# DelegatingFilterProxy

# FilterChainProxy

# SecurityFilterChain

`SecurityFilterChain`은 [FilterChainProxy](#filterchainproxy)가 현재 요청에 무슨 Spring Security `Filter` 인스턴스가 호출되어야 하는지 결정할 때 사용된다.

![SecurityFilterChain](https://docs.spring.io/spring-security/reference/_images/servlet/architecture/securityfilterchain.png)

일반적으로 `SecurityFilterChain`에 등록된 [Security Filters](#security-filters)는 빈이다. 하지만 [DelegatingFilterProxy](#delegatingfilterproxy)가 아닌 [FilterChainProxy](#filterchainproxy)에 등록되어 있다.
`FilterChainProxy`는 `Servlet` 컨테이너 또는 [DelegatingFilterProxy](#delegatingfilterproxy)에 바로 등록할 수 있는 많은 이점을 제공한다.

1. Spring Security Servlet support의 시작점이다. 그래서 디버깅을 한다면 `FilterChainProxy`를 중단점으로 설정하면 된다.
2. `FilterChainProxy`는 Spring Security 사용의 핵심이기때문에, 보이지 않는 작업을 선택적으로 수행할 수 있다.
예를 들어, 메모리 누수를 방지하기 위해 `SecurityContext`를 삭제할 수 있다. 또한, 특정 타입의 공격들을 방어하기위해 Spring Security의 `HttpFirewall`를 적용할 수 있다.
3. `SecurityFilterChain`이 언제 호출되어야 하는지에 대한 유연함을 제공한다. Servlet Container 안에서, `Filter` 인스턴스들은 URL 하나에만 기반해 호출된다. 하지만 `FilterChainProxy`는 `RequestMatcher` 인터페이스를 활용해 `HttpServletRequest` 에 포함된 모든 것들을 기반해 호출할 수 있다.

사진을 보고 좀 더 자세히 이해해보자.

![Multiple SecurityFilterChain](https://docs.spring.io/spring-security/reference/_images/servlet/architecture/multi-securityfilterchain.png)

사진에서 `FilterChainProxy`는 가장 처음 패턴 매칭되는 `SecurityFilterChain`을 호출한다. 
만약 요청된 URL이 `/api/messages/`라면, `/api/**` 패턴을 가진 `SecurityFilterChain_0`와 매칭되어서 호출된다. `/**` 패턴을 가진 `SecurityFilterChain_n`과 매칭되긴 하지만 순서가 뒤에 있기 때문에 호출되지 않는다.

`SecurityFilterChain_0` 안에는 3개의 `Filter` 인스턴스들이 설정되어 있다. 반면에 `SecurityFilterChain_n`안에는 4개가 설정되어 있다. 이것은 굉장히 중요하다. 각각의 `SecurityFilterChain`은 독립적으로 구성되며 유니크하다.
실제로, 만약 특정 호출을 무시하고 싶다면 `SecurityFilterChain`은 0개의 `Filter` 인스턴스를 가질 수도 있다.

# Security Filters

Security Filter 들은 [SecurityFilterChain](#securityfilterchain) 을 구성하고 있다.
각각의 Filter 들은 인증(authentication), 인가(authorization), 보안 취약점 방어(exploit protection) 그 외의 수많은 다른 이유로 사용될 수 있다.
Filter 들은 적시에 호출될 수 있도록 특정 순서로 실행된다.
예들 들어, 인증을 수행하는 `Filter`는 인가를 수행하는 `Filter` 보다 앞에 호출되어야 한다.
일반적으로 Spring Security의 `Filter` 들의 순서를 알 필요는 없지만, 알아야할 때가 있다.


# 참고

- [Srping Security Architecture](https://docs.spring.io/spring-security/reference/servlet/architecture.html)