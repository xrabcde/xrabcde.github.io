---
layout: post
title: Book/모던자바인액션/ 12장. 새로운 날짜와 시간 API
date: 2021-03-23
excerpt: "모던자바인액션 12장 내용 공부"
tags: [book, modern_java_in_action]
comments: true
---

자바 8에서는 지금까지의 날짜와 시간 문제를 개선하는 새로운 날짜와 시간 API를 제공한다.
## LocalDate, LocalTime, Instant, Duration, Period 클래스
먼저 간단한 날짜와 시간 간격을 정의해보자.
### LocalDate와 LocalTime
LocalDate는 시간을 제외한 날짜를 표현하는 불변 객체다. 특히, LocalDate는 어떤 시간대 정보도 포함하지 않는다는 특징이 있다.
정적 팩토리 메서드 `of`로 LocalDate를 만들 수 있다.
```
LocalDate date = LocalDate.of(2017, 9, 21); //2017-09-21
int year = date.getYear(); //2017
Month month = date.getMonth(); //SEPTEMBER
int day = date.getDayOfMonth(); //21
DayOfWeek dow = date.getDayOfWeek(); //THURSDAY
int len = date.lengthOfMonth(); //31(3월의 일 수)
boolean leap = date.isLeapYear(); //false(윤년이 아님)
```
팩토리 메서드 `now`는 시스템 시계의 정보를 이용해 현재 날짜 정보를 얻는다.
```
LocalDate today = LocalDate.now();
```
get 메서드에 `TemporalField`를 전달해서 시간 정보를 얻는 방법도 있다.
```
int year = date.get(ChronoField.YEAR);
int month = date.get(ChronoField.MONTH_OF_YEAR);
int day = date.get(ChronoField.DAY_OF_MONTH);
```
다음처럼 내장 메서드 getYear(), getMonthValue(), getDayOfMonth()를 이용해 가독성을 높일 수 있다.
```
int year = date.getYear();
int month = date.getMonthValue();
int day = date.getDayOfMonth();
```
마찬가지로 `13:45:20` 같은 시간은 LocalTime 클래스로 표현할 수 있다. 
```
LocalTime time = LocalTime.of(13, 45, 20); //13:45:20
int hour = time.getHour(); //13
int minute = time.getMinute(); //45
int second = time.getSecond(); //20
```
`parse` 정적 메서드를 이용해 날짜와 시간 문자열로 LocalDate와 LocalTime 인스턴스를 만드는 방법도 있다.
```
LocalDate date = LocalDate.parse("2017-09-21");
LocalTime time = LocalTime.parse("13:45:20");
```
### 날짜와 시간 조합
LocalDateTime은 LocalDate와 LocalTime을 쌍으로 갖는 복합 클래스다.
즉, LocalDateTime은 날짜와 시간을 모두 표현할 수 있으며 직접 LocalDateTime을 만드는 방법도 있고, 날짜와 시간을 조합하는 방법도 있다.
```
//2017-09-21T13:45:20
LocalDateTime dt1 = LocalDateTime.of(2017, Month.SEPTEMBER, 21, 13, 45, 20);
LocalDateTime dt2 = LocalDateTime.of(date, time);
LocalDateTime dt3 = date.atTime(13, 45, 20);
LocalDateTime dt4 = date.atTtime(time);
LocalDateTime dt5 = time.atDate(date);
```
### Instant 클래스 : 기계의 날짜와 시간
사람은 보통 주, 날짜, 시간, 분으로 날짜와 시간을 계산하지만 기계의 관점에서는 연속된 시간에서 특정 지점을 하나의 큰 수로 표현하는 것이 가장 자연스러운 시간 표현 방법이다.
팩토리 메서드 `ofEpochSecond`에 초를 넘겨줘서 Instant 클래스 인스턴스를 만들 수 있다.
다음 네 가지 코드는 같은 Instant를 반환한다.
```
Instant.ofEpochSecond(3);
Instant.ofEpochSecond(3, 0);
Instant.ofEpochSecond(2, 1_000_000_000); //2초 이후의 1억 나노초(1초)
Instant.ofEpochSecond(4, -1_000_000_000); //4초 이전의 1억 나노초(1초)
```
### Duration과 Period 정의
지금까지 살펴본 인터페이스는 모두 Temporal 인터페이스를 구현한다.
이번에는 두 시간 객체 사이의 지속시간 duration을 만들어보자. Durationn 클래스의 정적 팩토리 메서드 `between`으로 두 시간 객체 사이의 지속시간을 만들 수 있다.
```
Duration d1 = Duration.between(time1, time2);
Duration d2 = Duration.between(dateTime1, dateTime2);
Duration d3 = Duration.between(instant1, instant2);
```
LocalDateTime은 사람이 사용하도록, Instant는 기계가 사용하도록 만들어진 클래스로 두 인스턴스는 서로 혼합할 수 없다.
또한, Duration 클래스는 초와 나노초로 시간 단위를 표현하므로 between 메서드에 LocalDate를 전달할 수 없다.
년, 월, 일로 시간을 표현할 때는 `Period` 클래스를 사용한다.
```
Period tenDays = Period.between(LocalDate.of(2017, 9, 11), LocalDate.of(2017, 9, 21));
```
## 날짜 조정, 파싱, 포매팅
`withAttribute` 메서드로 기존의 LocalDate를 바꾼 버전을 직접 간단하게 만들 수 있다.
```
LocalDate date1 = LocalDate.of(2017, 9, 21); //2017-09-21
LocalDate date2 = date1.withYear(2011); //2011-09-21
LocalDate date3 = date2.withDayOfMonth(25); //2011-09-25
LocalDate date4 = date3.with(ChronoField.MONTH_OF_YEAR, 2); //2011-02-25
```
선언형으로 LocalDate를 사용하는 방법도 있다.
```
LocalDate date1 = LocalDate.of(2017, 9, 21); //2017-09-21
LocalDate date2 = date1.plusWeeks(1); //2017-09-28
LocalDate date3 = date2.minusYears(6); //2011-09-28
LocalDate date4 = date3.plus(6, ChronoUnit.MONTHS); //2012-03-28
```
### TemporalAdjusters 사용하기
지금까지 살펴본 날짜 조정 기능은 간단한 것들이었다. 다음 주 일요일, 돌아오는 평일, 어떤 달의 마지막 날 등 좀 더 복잡한 날짜 조정 기능이 필요할 때는 TemporalAdjuster를 사용할 수 있다.
```
LocalDate date1 = LocalDate.of(2014, 3, 18); //2014-03-18
LocalDate date2 = date1.with(nextOrSame(DayOfWeek.SUNDAY)); //2014-03-23
LocalDate date3 = date2.with(lastDayOfMonth()); //2014-03-31
```
### 날짜와 시간 객체 출력과 파싱
날짜와 시간 관련 작업에서 포매팅과 파싱은 서로 떨어질 수 없는 관계다. 이 때 가장 중요한 클래스는 `DateTimeFormatter` 이다.
```
LocalDate date = LocalDate.of(2014, 3, 18);
String s1 - date.format(DateTimeFormatter.BASIC_ISO_DATE); //20140318
String s2 = date.format(DateTimeFormatter.ISO_LOCAL_DATE); //2014-03-18
```
반대로 날짜나 시간을 표현하는 문자열을 파싱해서 날짜 객체를 다시 만들 수 있다.
```
LocalDate date1 = LocalDate.parse("20140318", DateTimeFormatter.BASIC_ISO_DATE);
LocalDate date2 = LocalDate.parse("2014-03-18", DateTimeFormatter.ISO_LOCAL_DATE);
```
## 다양한 시간대와 캘린더 활용 방법
새로운 클래스를 이용하면 서머타임(DST)와 같은 복잡한 사항이 자동으로 처리된다.
### 시간대 사용하기
표준 시간이 같은 지역을 묶어서 시간대 규칙 집합을 정의힌다. ZoneRules 클래스에는 약 40개 정도의 시간대가 있다.
```
ZoneId romeZone = ZoneId.of("Europe/Rome");
ZoneId zoneId = TimeZone.getDefault().toZoneId();
```
위의 코드에서처럼 ZoneId의 새로운 메서드인 toZoneId로 기존의 TimeZone 객체를 ZoneId 객체로 변환할 수도 있다.
```
LocalDate date = LocalDate.of(2014, Month.MARCH, 18);
ZonedDateTime zdt1 = date.atStartOfDay(romeZone);
LocalDateTime dateTime = LocalDateTime.of(2014, Month.MARCH, 18, 13, 45);
ZonedDateTime zdt2 = dateTime.atZone(romeZone);
Instant instant = Instant.now();
ZonedDateTime zdt3 = instant.atZone(romeZone);
```
