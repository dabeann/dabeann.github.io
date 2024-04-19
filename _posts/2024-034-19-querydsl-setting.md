---
layout: post
title: Spring Boot Querydsl setting
subtitle: 
categories: SpringBoot
tags: [querydsl]
---
## Querydsl 사용 이유
프로젝트에서 qeurydsl을 사용해야 하는 기능이 있었다.  
**동적 필터링**이었다. 가격과 여러 커피의 특징들로 검색하는 기능이다. 여러 검색 조건을 동시에 적용해야 하다 보니 qeurydsl이 필요했다.

## Spring Boot 버전
우리 프로젝트의 spring boot 버전은 **3.2.3**이다.

## Querydsl 설정부
build.gradle 파일에 다음과 같이 추가한다.
```java
dependencies{
  ...
  //querydsl 추가
  implementation 'com.querydsl:querydsl-jpa:5.0.0:jakarta'
  annotationProcessor "com.querydsl:querydsl-apt:5.0.0:jakarta"
  annotationProcessor "jakarta.annotation:jakarta.annotation-api"
  annotationProcessor "jakarta.persistence:jakarta.persistence-api"
  ...
  }
```
```java
// Querydsl 설정부
def generated = 'src/main/generated'

// querydsl QClass 파일 생성 위치를 지정
  tasks.withType(JavaCompile) {
  options.getGeneratedSourceOutputDirectory().set(file(generated))
  }

// java source set 에 querydsl QClass 위치 추가
  sourceSets {
  main.java.srcDirs += [ generated ]
  }

// gradle clean 시에 QClass 디렉토리 삭제
  clean {
  delete file(generated)
  }
```
또한 Qfile은 모두 .gitignore에 추가해준다.
```java
### querydsl Q files ###
  src/main/generated/
```

## Compile
- Gradle → Tasks → build → clean
- Gradle → Tasks → other → compileJava
- 컴파일하면 Q파일이 생성된다.

참고 : [블로그](https://velog.io/@daoh98/Query-DSL-Spring-boot-3.0-%EC%9D%B4%EC%83%81-Query-DSL-%EC%84%A4%EC%A0%95-%EB%B0%A9%EB%B2%95)
