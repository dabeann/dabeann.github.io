---
layout: post
title: SpringBoot 스케쥴러(Scheduler)
subtitle: 
categories: SpringBoot
tags: [scheduler]
---
## 사용 계기
나는 일정 시간마다 동작하는 기능을 구현해야 했다.  
바로 아래의 동작이다.  
> **매일 0시**에 유저들의 오늘의 명언 설정하기  
  
여러 자료를 찾아보니, 스프링부트에서 Scheduler기능을 제공한다고 한다.  
따라서 이 기능을 사용해 구현하기로 결정했다.

## 스케쥴러 사용

### 클래스 생성
Scheduler를 관리할 클래스를 만든다.  
```java
@Component
@EnableScheduling
@RequiredArgsConstructor
public class ContentsScheduler {

}
```
- 스케쥴러 클래스는 스프링 빈에 등록되어야 하기에 `@Component`를 붙인다.
- 스케쥴러가 동작하도록 해주는 어노테이션 `@EnableScheduling`을 붙인다.
  - main이 위치한 Application 클래스에 붙여도 된다.

### 스케쥴러 등록(@Scheduled) 및 주기 설정
```java
@Component
@EnableScheduling
@RequiredArgsConstructor
public class ContentsScheduler {

    private final ContentsService contentsService;

    // 매일 0시 실행
    @Scheduled(cron = "0 0 0 * * *")
    public void updateTodayQuotation() {
        contentsService.updateTodayQuotation();
    }
}
```
- 주기적으로 실행될 메소드들을 생성하고 `@Scheduled`를 붙인다.
- corn 값
  - 순서대로 **"초 분 시 일 월 요일"** 을 의미한다.
  - *을 적을 경우 모든 조건을 의미한다.

---
[참고](https://velog.io/@minji/%EC%8A%A4%ED%94%84%EB%A7%81%EB%B6%80%ED%8A%B8-%EC%8A%A4%EC%BC%80%EC%A5%B4%EB%9F%AC-%EC%82%AC%EC%9A%A9%ED%95%98%EA%B8%B0%EB%A7%A4%EB%8B%AC-1%EC%9D%BC-%EB%8F%99%EC%9E%91-%EB%A7%A4%EC%9D%BC-%EB%8F%99%EC%9E%91)