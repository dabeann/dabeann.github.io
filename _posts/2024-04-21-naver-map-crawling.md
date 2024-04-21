---
layout: post
title: Naver map crawling with Java
subtitle: 
categories: SpringBoot
tags: [crawling]
---
## 크롤링
크롤링이란? 웹 페이지를 그대로 가져와서 거기서 데이터를 추출해 내는 행위다.  
웹 사이트에서 태그를 이용하여 프로그래밍적으로 데이터를 가져온다.

## 크롤링 계기
전국의 카페와 카페의 아메리카노 정보가 필요했다.  
외부 api를 찾는 것이 베스트였지만.. 카페와 커피까지 있는 api는 없었다.  
따라서 **특정 지역**을 한정해서 네이버 지도의 정보를 크롤링 하기로 결정했다!  

## Java 선택 이유
사실 **크롤링**이라 하면 대부분 Python을 이용한다.  
하지만 나는 Spring Boot를 항상 사용하고 Java에 익숙했기 때문에 Java를 선택하였다!  

## 크롤링 코드
### 크롤링할 정보
우선 내가 크롤링 해야할 정보는 다음과 같다.
**연남동 카페**를 검색한 후 나오는 카페들의
- 카페 이름
- 카페 주소
- 카페 위도
- 카페 경도
- 카페 대표 이미지

**해당 카페의**
- 커피 이름
- 커피 가격

### Web Driver
크롤링할 웹 사이트는 **크롬**으로 결정했다. 따라서 크롬 웹 드라이버가 필요하다.  
웹 드라이버는 컴퓨터가 브라우저를 제어할 수 있게 해준다.  
  
드라이버를 다운 받는 방법은 다음 사이트를 참고하였다. 자신의 크롬 버전과 알맞는 드라이버 버전을 다운받아야 한다.  
참고 : [크롬 드라이버 참고](https://kminito.tistory.com/78)

### 검색
```java
// 브라우저에서 url로 이동한다.
driver.get(URL);
// 브라우저에서 로딩될 때까지 잠시 기다린다.
Thread.sleep(2000);

// 네이버 지도 검색창에 원하는 단어 입력 후 엔터
WebElement inputSearch = driver.findElement(By.className("input_search"));
inputSearch.sendKeys(LOCATION + " " + KEYWORD);
Thread.sleep(1500);
inputSearch.sendKeys(Keys.ENTER);
Thread.sleep(1500);

// 데이터가 iframe 안에 있는 경우 (HTML 안 HTML) 이동
driver.switchTo().frame("searchIframe");
```
URL은 "https://map.naver.com/v5/" 이고, **private static final String**으로 선언해 주었다.

### 태그로 원하는 정보 가져오기
```java
String address = driver.findElement(By.cssSelector("span.LDgIH")).getText();
```
위 코드는 주소를 가져오는 역할을 한다.  
네이버 지도를 직접 들어가서 F12를 누른다. 그 후 원하는 정보의 태그를 찾아 작성하면 된다.  

### 글씨를 포함하는 버튼 정보 가져오기
```java
WebElement nextButton = driver.findElement(By.xpath("//span[contains(text(), '다음페이지')]/.."));
```
'다음페이지'라는 text를 포함하는 다음 페이지로 가는 버튼의 WebElement 정보를 가져오는 코드이다.

## 크롤링 이슈
### 네이버 주문이 가능한 가게
```java
// 메뉴 정보가 있는 요소를 찾아서 가져오기
List<WebElement> menuElements = driver.findElements(By.xpath("//div[@class='yQlqY']/span[@class='lPzHi']"));
// 변동 가격이랑 123~124 이런 가격도 나오도록
List<WebElement> priceElements = driver.findElements(By.xpath("//div[@class='GXS1X']"));
```
메뉴 이름과 가격 정보를 각각 리스트로 받아온다.  
**변동**가격이란, 가격에 숫자가 들어있지 않고 '변동'이라고 String으로 써져 있는 경우이다.  
**123~123**가격이란, 가격에 숫자가 들어있지 않고 '5000~10000'이라고 범위로(String) 써져 있는 경우이다.  
  
위와 같이 **클래스 이름**으로 정보를 가져오게 된다.  
대부분 가게의 경우 모두 저 태그 이름으로 하면 메뉴 정보를 가져올 수 있었다. 하지만 **예외**가 발생했다!  
![image](https://github.com/dabeann/dabeann.github.io/assets/127164905/da764b46-aec1-4a83-b466-44d0c48c5250)
위 사진과 같이 div의 class명이 다른 가게가 있었다. 이와 같은 가게(네이버 주문이 가능한 가게)는 메뉴 정보를 아무것도 불러오지 못한다.  
> 그래도 1페이지 검색 했을 경우, 네이버 주문이 가능한 가게는 1개였기에 그냥 무시하고 넘어가기로 결정했다.  

### 메뉴 버튼 클릭
ㅇㅇㅇ
