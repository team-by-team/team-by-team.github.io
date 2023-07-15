---
title: "LocalDateTime 원하는 Format으로 바인딩하기 (feat.@DateTimeFormat, @JsonFormat)"
description: "Spring에서 LocalDateTime 원하는 Format으로 바인딩하기"
date: 2023-07-01
update: 2023-07-01
tags:
- Spring
- LocalDateTime
---

> 해당 글은 우아한테크코스 5기 팀바팀 크루 [성하](https://github.com/sh111-coder)가 작성했습니다!


## 📘 0. 일정 등록 API Request Body
API 명세를 정할 당시에는

LocalDateTime이 요청으로 들어왔을 때 어떻게 Request Dto의 필드로 바인딩되는지 감이 안 잡혔습니다.



그래서 우선 API 명세를 확정한 이후에 부딪혀보자! 라고 마음을 다잡고 명세를 정했었습니다.



그렇게 일정 등록 API 명세가 확정된 채로, 일정 등록 API를 구현하게 됐습니다.



확정된 일정 등록 API의 Request Body는 다음과 같았습니다.
```http request
{
    "title": "4차 데모데이 회의 (아무것도 구현안함)",
    "startDateTime": "2023-07-13 14:00",
    "endDateTime": "2023-07-14 15:59"
}
```
startDateTime과 endDateTime을 LocalDateTime으로 바인딩할 때 공부한 내용을 공유하겠습니다! ✌🏻


## ❌ 1. 문제 상황

Request로 오는 String의 날짜 형식을 LocalDateTime에 어떻게 바인딩할지 찾아보니,



@DateTimeFormat을 사용하여 Request Dto 필드의 형식을 지정할 수 있었습니다.


그래서 다음과 같이 Request Dto를 생성하였습니다.

```java
public record ScheduleRegisterRequest(

        @NotBlank(message = "제목은 빈 값일 수 없습니다.")
        String title,

        @DateTimeFormat(pattern = "yyyy-MM-dd HH:mm")
        LocalDateTime startDateTime,

        @DateTimeFormat(pattern = "yyyy-MM-dd HH:mm")
        LocalDateTime endDateTime) {

}
```
그 후, 객체가 잘 바인딩이 되는지 API 테스트를 진행했습니다.

결과는, 다음과 같이 에러가 발생했습니다.

```java
java.time.format.DateTimeParseException: Text '2023-07-13 14:00' could not be parsed at index 10
	at java.base/java.time.format.DateTimeFormatter.parseResolved0(DateTimeFormatter.java:2052) ~[na:na]
	at java.base/java.time.format.DateTimeFormatter.parse(DateTimeFormatter.java:1954) ~[na:na]
	at java.base/java.time.LocalDateTime.parse(LocalDateTime.java:494) ~[na:na]
	at com.fasterxml.jackson.datatype.jsr310.deser.LocalDateTimeDeserializer._fromString(LocalDateTimeDeserializer.java:177) ~[jackson-datatype-jsr310-2.15.2.jar:2.15.2]
	at com.fasterxml.jackson.datatype.jsr310.deser.LocalDateTimeDeserializer.deserialize(LocalDateTimeDeserializer.java:81) ~[jackson-datatype-jsr310-2.15.2.jar:2.15.2]
	at com.fasterxml.jackson.datatype.jsr310.deser.LocalDateTimeDeserializer.deserialize(LocalDateTimeDeserializer.java:40) ~[jackson-datatype-jsr310-2.15.2.jar:2.15.2]
    
    ...
```
에러 로그를 보면, String으로 들어온 '2023-07-13 14:00' 데이터 바인딩 시 오류가 난 것을 확인할 수 있었습니다.



## 🔮 2. @JsonFormat으로 해결
결과적으로, **@DateTimeFormat** 대신 **@JsonFormat**을 사용하여 해결할 수 있었습니다.

```java
public record ScheduleRegisterRequest(
        @NotBlank(message = "제목은 빈 값일 수 없습니다.")
        String title,

        @JsonFormat(shape = JsonFormat.Shape.STRING, pattern = "yyyy-MM-dd HH:mm", timezone = "Asia/Seoul")
        LocalDateTime startDateTime,

        @JsonFormat(shape = JsonFormat.Shape.STRING, pattern = "yyyy-MM-dd HH:mm", timezone = "Asia/Seoul")
        LocalDateTime endDateTime) {

}
```

그렇다면, 왜 @DateTimeFormat 대신 @JsonFormat을 사용해야 했을까요?

두 어노테이션을 간단히 살펴봅시다.


## ✅ 3. @DateTimeFormat VS @JsonFormat

먼저 @DateTimeFormat을 살펴보면 다음과 같습니다.

```java
package org.springframework.format.annotation;

...

public @interface DateTimeFormat {
	...
}
```
패키지에서 볼 수 있듯이 **@DateTimeFormat**은 Spring의 어노테이션입니다.



그럼, **@JsonFormat**을 살펴볼까요?
```java
package com.fasterxml.jackson.annotation;

...

public @interface JsonFormat {
	...
}
```
패키지를 보면 **@JsonFormat**은 @RequestBody, @ResponseBody를 사용한 직렬화/역직렬화를 담당하는

Jackson 라이브러리의 어노테이션입니다.



Jackson 라이브러리에서는 내부적으로 pattern으로 지정한 형식을 LocalDateTime으로 변환해줍니다.

따라서, Jackson 라이브러리의 @JsonFormat은 날짜 형식이라면 자유롭게 사용이 가능합니다.



하지만, @RequestBody, @ResponseBody를 사용한 역직렬화/직렬화 시

@DateTimeFormat을 사용하려면 Jackson 라이브러리에 있는 어노테이션이 아니기 때문에

LocalDateTime의 기본 형식인 'yyyy-MM-dd'T'HH:mm:ss'으로 요청이 들어와야 바인딩됩니다.

(이때, @DateTimeFormat을 사용하지 않아도 바인딩이 됩니다.)


## ❓ 4. @RequestParam, @ModelAttribute에서는 무엇을 사용해야 할까?
위에서 설명한 상황은 @RequestBody, @ResponseBody 등

Jackson 라이브러리를 사용하여 역직렬화/직렬화를 하는 상황이었습니다.


그렇다면, Jackson 라이브러리가 관여하지 않는 @RequestParam, @ModelAttribute에서는 무엇을 사용해야 할까요?


Jackson 라이브러리가 사용되지 않기 때문에 **@JsonFormat은 무시되게 됩니다.**

**따라서, @DateTimeFormat을 사용하여 날짜 형식을 자유롭게 바인딩 할 수 있습니다.**


## 🎯 5. 결론

따라서, 결론적으로 @RequestBody, @ResponseBody를 사용한 역직렬화/직렬화 시

날짜 형식을 자유롭게 사용하기 위해서는

**@DateTimeFormat이 아닌, @JsonFormat을 사용해야합니다.**

@RequestBody, @ResponseBody를 사용한 역직렬화/직렬화 시에 Jackson 라이브러리를 사용하여 날짜를 변환하기 때문에,

**Jackson 라이브러리의 어노테이션인 @JsonFormat**을 사용해야 함을 알 수 있었습니다.

(물론, Request Body 날짜 형식이 LocalDateTime의 기본 형식인  'yyyy-MM-dd'T'HH:mm:ss'이라면

@DateTimeFormat, @JsonFormat 둘다 사용하지 않아도 그대로 바인딩이 가능합니다.)

반대로, Jackson 라이브러리가 사용되지 않는

@RequestBody, @ResponseBody의 경우에는 @DateTimeFormat을 사용해야 함을 알 수 있었습니다.

위의 결론을 요약하면 다음과 같습니다!

```java
* 기본 전제 : LocalDateTime의 기본 형식인 'yyyy-MM-dd'T'HH:mm:ss'이 아닐 때

1. @RequestBody, @ResponseBody : @JsonFormat 사용
2. @RequestParam, @ModelAttribute : @DateTimeFormat 사용
```


---

#### Reference
[Controller 메서드에서 LocalDateTime 타입의 변수 바인딩 받기](https://swampwar.github.io/2020/03/19/LocalDateTime-%EB%B3%80%EC%88%98%EB%B0%94%EC%9D%B8%EB%94%A9.html)
[SpringBoot에서 날짜 타입 JSON 변환에 대한 오해 풀기](https://jojoldu.tistory.com/361)
