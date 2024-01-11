---
layout: post
title: Spring Security Jwt
subtitle: 
categories: backend
tags: [security, jwt]
---
## JWT
JSON Web Tokens are an open, industry standard RFC 7519 method for representing claims securely between two parties.  
Header, Payload, Signature 3개의 부분으로 구성되어 있다.  
[JWT.io](https://jwt.io/)

### Header
Signature를 해싱하기 위한 알고리즘 정보들이 담겨있다.

### Payload
Server와 Client가 주고받는, 시스템에서 실제로 사용될 정보에 대한 내용들을 담고있다.

### Signature
Token의 유효성 검증을 위한 문자열
이 문자열을 통해 서버에서는 이 token이 유효한 token인지를 검증할 수 있다.

### JWT 장점
- 중앙의 인증서버, 데이터 스토어에 대한 의존성 없음
- 시스템 수평 확장 유리
- Base64 URL Sage Encoding > URL, Cookie, Header 모두에 사용 가능

### JWT 단점
- Payload의 정보가 많아지면 네트워크 사용량 증가, 데이터 설계 고려 필요
- Token이 client에 저장, server에서 client의 token을 조작할 수 없음