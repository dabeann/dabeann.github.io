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

## 본격적인 크롤링
어떻게 크롤링했는지 간단하게 정리해 보자.
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
Java로 크롤링 코드를 짜며 마주했던 대표적인 에러들을 정리해 보자.
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
![image](https://github.com/dabeann/dabeann.github.io/assets/127164905/bdf16f72-86f6-40a9-85b9-842d540a7ec1)
보통 가게는 오른쪽 사진과 같이 [홈 / 소식 / 메뉴 / … ] 로 바가 구성되어 있다.  
그래서 처음엔 `driver.findElement(By.xpath("//div[@class='flicking-camera']/*[3]")).click();` 코드만 썼다.  
바 에서 3번째 “메뉴” 를 클릭하기 위해! 3번째 값 클릭하도록 했다.  
  
왜 오류가 나나 찾아보니 [홈 / 메뉴 / … ] 로 구성된 가게들도 있었다.  
`"//div[@class='flicking-camera']//*[contains(text(), '메뉴')]"` 를 넣어 ‘메뉴’ 텍스트를 포함하는 것을 우선적으로 클릭하도록 하였다.  
근데 또 ‘메뉴’ 글씨만 클릭하게 하니깐 무슨 *안보여서 클릭못한다* .. 에러가 떠서 try catch문으로 둘 다 넣었다.
```java
try {
      // 텍스트를 포함하는 요소를 찾기 위한 XPath
      driver.findElement(By.xpath("//div[@class='flicking-camera']//*[contains(text(), '메뉴')]")).click();
} catch (Exception ex) {
     // 바에서 보통 3번째가 '메뉴'
     driver.findElement(By.xpath("//div[@class='flicking-camera']/*[3]")).click();
   }
```

### div class 이름
[Selenium으로 네이버 지도 크롤링하기](https://velog.io/@kimdy0915/Selenium%EC%9C%BC%EB%A1%9C-%EB%84%A4%EC%9D%B4%EB%B2%84-%EC%A7%80%EB%8F%84-%ED%81%AC%EB%A1%A4%EB%A7%81%ED%95%98%EA%B8%B0)  
처음에는 위 링크만 보고 따라했다. 하지만 class이름이 바뀐 것인지 모조리 에러가 났다.  
`.findElement(By.className("LDgIH"));` **LDgIH**라는 클래스 이름이 없었다.  
-> 찾을 부분 우클릭 -> 검사 를 클릭하게 되면 어느부분인지 html이 나온다. 이걸 토대로 클래스 이름을 찾는다.  
![image](https://github.com/dabeann/dabeann.github.io/assets/127164905/22063e3f-cb4d-4a11-a9eb-d574b18c2d33)  

### 가격 정보 class 이름
처음에는 `"//div[@class='GXS1X']/em"` 했었다.  
클래스 안에서 em tag 안에 가격이 있었기 때문이다.
![image](https://github.com/dabeann/dabeann.github.io/assets/127164905/eb16ce17-0395-4703-8e80-679e79824ca5)  
  
하지만 위와 같이 코드를 짜면 "변동" 가격이 아예 들어가지 않는다. "변동"은 em tag 안에 없기 때문이다.
![image](https://github.com/dabeann/dabeann.github.io/assets/127164905/c38292cb-a4d9-42b1-be30-1c89a924210c)
  
또한 가격이 "2500~2600"와 같은 범위형이면 값을 가져올 때 `priceElements` 리스트에 2500, 2600 두 개의 값이 따로 들어간다.  
-> 결국 메뉴 이름 리스트와 개수가 달라져서 index overflow가 발생한다.
![image](https://github.com/dabeann/dabeann.github.io/assets/127164905/09fdf04b-9767-4620-9b15-50801008934e)

→ `"//div[@class='GXS1X']/em"`  → `"//div[@class='GXS1X']”`로 바꾸니 해결됐다.  
```java
// 변동 가격이랑 123~124 이런 가격도 나오도록
List<WebElement> priceElements = driver.findElements(By.xpath("//div[@class='GXS1X']"));
```

### 다음페이지 버튼 클릭
이전페이지와 다음페이지의 클래스 이름이 같아서 한참 헤멨다.  
![image](https://github.com/dabeann/dabeann.github.io/assets/127164905/2277e33b-877e-4030-b378-f6a3c838a1db)
![image](https://github.com/dabeann/dabeann.github.io/assets/127164905/14fd6b37-0e12-49f1-ac52-057fd79f8a1f)  
따라서 “다음페이지” 라는 text를 가진 element를 클릭하도록 설정했다.  
```java
// 다음 페이지로 가는 버튼
WebElement nextButton = driver.findElement(By.xpath("//span[contains(text(), '다음페이지')]/.."));
```

## 전체 코드
나의 리포지토리 파일 링크이다.  
[크롤링 코드 전체](https://github.com/dajeongdev/Americanote/blob/develop/src/main/java/com/coffee/americanote/cafe/service/CrawlingCafe.java)  

## 후기
크롤링을 이번 프로젝트하며 처음 경험하였다. 
내가 알던 자바 코드와는 다르게 짜야해서 생소했다. 객체지향적으로 짤 수도 없고 ... 하는 내내 고민이 많았다.  
  
크롤링을 하는 동안은 내가 개발자가 아닌 느낌이 들었다. (자바 코드보다 웹 사이트에서 태그를 더 많이 봤다 ㅎㅎ)  
그래도 크롤링을 다 하고 정보를 db에 넣으니 정말 뿌듯했다~!  
이 코드를 나중에 재활용하지는 않을 것 같지만 내가 크롤링 코드를 짜며 힘들었던 부분을 정리해 보았다.  

다음에 크롤링할 기회가 온다면 더 빠르게 코드를 짤 수 있다 !!.
