---
layout: post
date: 2024-04-08
title: "ApplicationContext 분석하기"
tags: [spring, analyze, ]
categories: [spring, ]
---


# ApplicationContext 분석하기


---


ApplicationContext.class는 빈들의 생성과 의존성 주입 등의 역할을 하는 DI 컨테이너이다.


## 궁금증?!


---


![](https://prod-files-secure.s3.us-west-2.amazonaws.com/d92da26b-2e8f-4fff-b257-9cc62cceb339/02fe3ceb-f6e5-4e3d-b784-5b2cdc23d954/Untitled.png?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Content-Sha256=UNSIGNED-PAYLOAD&X-Amz-Credential=AKIAT73L2G45HZZMZUHI%2F20240408%2Fus-west-2%2Fs3%2Faws4_request&X-Amz-Date=20240408T134303Z&X-Amz-Expires=3600&X-Amz-Signature=69a596fc7c944f950d1ad5bdb32637f97f6d912f183063fae3adf3a0f00b8eaa&X-Amz-SignedHeaders=host&x-id=GetObject)


매일 실행하는 스프링부트에 WebApplicationContext가 실행 된다.


도대체 WebApplicationContext는 어디에 사용될까?


## WebApplicationContext.Class


---


![](https://prod-files-secure.s3.us-west-2.amazonaws.com/d92da26b-2e8f-4fff-b257-9cc62cceb339/73532ca0-bcf7-4b87-8810-580bb42c580e/Untitled.png?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Content-Sha256=UNSIGNED-PAYLOAD&X-Amz-Credential=AKIAT73L2G45HZZMZUHI%2F20240408%2Fus-west-2%2Fs3%2Faws4_request&X-Amz-Date=20240408T134303Z&X-Amz-Expires=3600&X-Amz-Signature=0b69e3e9dd0f7532b241451c705cb12a7662ded473a21dfd239cb4fd4704b14b&X-Amz-SignedHeaders=host&x-id=GetObject)


WebApplicationContext은 ApplicationContext를 상속 하는 인터페이스이다.


그러면 ApplicationContext를 살펴보자


## ApplicationContext.Class


---


![](https://prod-files-secure.s3.us-west-2.amazonaws.com/d92da26b-2e8f-4fff-b257-9cc62cceb339/64de6025-3d6b-48c3-a114-5c7f36622000/Untitled.png?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Content-Sha256=UNSIGNED-PAYLOAD&X-Amz-Credential=AKIAT73L2G45HZZMZUHI%2F20240408%2Fus-west-2%2Fs3%2Faws4_request&X-Amz-Date=20240408T134303Z&X-Amz-Expires=3600&X-Amz-Signature=90a07151379f0d74139d72025f3a8dfb5c6435c87cb1356b8af22361e0f81ddf&X-Amz-SignedHeaders=host&x-id=GetObject)


ApplicationContext는 여러 인터페이스를 상속받고 있지만


그 중 가장 중요하게 봐야한 포인트는 `BeanFactory`를 상속 받고 있다.


# BeanFactory.Class


---


undefined


Beanfactory 인터페이스는 빈의 생성, 보관, 제거 등의 기능을 담고있다.


하지만 Beanfactory는 단일 빈을 처리하기 위한 인터페이스 기능을 한다.


따라서 여러 빈을 관리하기 위한 인터페이스가 존재한다.


## 확장된 기능을 가진 인터페이스


---


[다이어그램](https://www.notion.so/ApplicationContext-ecd13581bf714082ba145203d1ac8562)을 살펴보면 여러 인터페이스를 확인 할 수 있다.


`ListableBeanFactory` : 여러 빈을 처리하기 위한 기능


`HierarchicalBeanFactory` : 여러 BeanFactory들 간의 계층(부모-자식) 관계를 설정하기 위한 기능


….등 있다.


## ApplicationContext의 또 다른 기능


---


undefined


ApplicationContext.class의 코드를 잘 살펴보면


@Autowired 처리를 위한 `AutowireCapableBeanFactory` 가 있다.


하지만 `ApplicationContext` 는 `AutowireCapableBeanFactory`를 상속받지 않는데도 불구하고 @Autowired를 처리할 수 있다.


그 이유는 `AutowireCapableBeanFactory`를 반환하는 인터페이스를 가지고 있다.


## AutowireCapableBeanFactory은 인터페이스다.


---


AutowireCapableBeanFactory은 @Autowired와 같은 기능을 사용하기 위한 인터페이스이다. 따라서 실질적으로 `진짜 빈`을 생성해주는 곳은 따로있다.


## 실제로 관리해주는 곳은 어디일까?


---


![](https://prod-files-secure.s3.us-west-2.amazonaws.com/d92da26b-2e8f-4fff-b257-9cc62cceb339/41af91c1-7892-4e31-83df-287afd226baa/Untitled.png?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Content-Sha256=UNSIGNED-PAYLOAD&X-Amz-Credential=AKIAT73L2G45HZZMZUHI%2F20240408%2Fus-west-2%2Fs3%2Faws4_request&X-Amz-Date=20240408T134303Z&X-Amz-Expires=3600&X-Amz-Signature=b5884682cd683110618d0025cfde0d8770a475ae63e65c765f728d028989ebcf&X-Amz-SignedHeaders=host&x-id=GetObject)


**DefaultListableBeanFactory.class**를 거슬러 가다보면 상위에 @Autowired 처리를 위한 인터페이스인 **AutowireCapableBeanFactory.class** 와 그에 대한 추상클래스 구현체인 **AbstractAutowireCapableBeanFactory.class**를 상속받고 있는것을 확인 할 수 있다.


## DefaultListableBeanFactory를 생성하는 곳은?


---


GenericApplicationContext.class의 소스를 확인해보자


undefined


GenericApplicationContext .class에서


`진짜 빈`들을 등록, 관리하는 **DefaultListableBeanFactory.class**의 생성을 확인 할 수 있다.


## GenericApplicationContext와 ApplicationContext관계


---


![](https://prod-files-secure.s3.us-west-2.amazonaws.com/d92da26b-2e8f-4fff-b257-9cc62cceb339/e09146b6-c84d-45bc-9830-36f57ad227df/Untitled.png?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Content-Sha256=UNSIGNED-PAYLOAD&X-Amz-Credential=AKIAT73L2G45HZZMZUHI%2F20240408%2Fus-west-2%2Fs3%2Faws4_request&X-Amz-Date=20240408T134303Z&X-Amz-Expires=3600&X-Amz-Signature=81fa08cabc29e012ddb0bb4b9d197179ae19d72ec71724cc5d8df62703d6e805&X-Amz-SignedHeaders=host&x-id=GetObject)


**GenericApplicationContext.class**는 **AbstractApplicationContext.class**를 상속받고


**AbstractApplicationContext.class**는 **ConfigurableApplicationContext.class**를 인터페이스로 받았다.


**ConfigurableApplicationContext.class**는 **ApplicationContext.class**를 상속받고있다.


# 결론


---

1. 서버 실행을 하면 **WebApplicationContext.class**가 초기화 된다.
2. **WebApplicationContext.class**의 부모인 **ApplicationContext.class**는 ****DI 컨테이너 역할을 한다.
3. **ApplicationContext.class**에서 **getAutowireCapableBeanFactory()** 메서드를 이용하여 @Autowired 기능을 사용 할 수 있다.
4. 진짜 빈 관리는 **GenericApplicationContext.class**에서 **DefaultListableBeanFactory.class**에서 진행한다.
