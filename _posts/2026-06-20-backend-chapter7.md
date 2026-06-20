---
title: "[코딩 자율학습 스프링 부트 3 자바 백엔드 개발 입문] 7장 게시글 수정하기: Update"
date: 2026-06-20 13:00:00 +0900
categories: [Study, Backend]
tags: [spring-boot, backend]
description: 코딩 자율학습 스프링 부트 3 자바 백엔드 개발 입문 7장의 핵심 개념 정리
---

## 1. 데이터 수정 과정

데이터 수정 실습
1. <수정 페이지> 만들고 기존 데이터 불러오기
    a) <상세 페이지>에서 [Edit] 버튼을 클릭한다.
    b) 요청을 받은 컨트롤러는 해당 글의 id로 DB에서 데이터를 찾아 가져온다.
    c) 컨트롤러는 가져온 데이터를 뷰에서 사용할 수 있도록 모델에 등록한다.
    d) 모델에 등록된 데이터를 <수정 페이지>에서 보여준다. 그러면 사용자가 내용을 수정할 수 있는 상태가 된다.
2. 데이터를 수정해 DB에 반영한 후 결과를 볼 수 있게 <상세 페이지>로 리다이렉트하기
    a) 폼 데이터(수정 요청 데이터)를 DTO에 담아 컨트롤러에서 받는다.
    b) DTO를 엔티티로 변환한다.
    c) DB에서 기존 데이터를 수정 데이터로 갱신한다.
    d) 수정 데이터를 <상세 페이지>로 리다이렉트한다.

## 2. <수정 페이지> 만들기

### <상세 페이지>에 Edit 버튼 만들기

상세 페이지인 `show.mustache` 파일을 연다. `</table>` 아래 에 `<a>` 태그를 추가하고 href 속성 값으로 연결하려는 URL인 "/articles/{{article.id}}/edit"를 작성한다.
```html
</table>

<a href="/articles/{{article.id}}/edit">Edit</a>
<a href="/articles">Go to Article List</a>
```

보통 article의 사용 범위를 {{#article}}{{/article}} 형식으로 지정한 경우에는 {{id}}만 써도 되지만 범위를 따로 지정하지 않았다면 점(.)을 사용해 {{article.id}}라고 표시해야 한다.

### Edit 요청을 받아 데이터 가져오기

Edit 요청을 받을 컨트롤러를 만들어야 한다.

**edit() 메서드 기본 틀 만들기**
`ArticleController.java` 파일을 열고 index() 메서드 아래에 edit() 메서드를 추가한다.
1. 수정 요청을 받아 처리할 edit() 메서드를 작성하고 반환할 수정 페이지를 articles 디렉터리 안에 `edit.mustache` 파일로 설정한다.
2. URL 요청을 받는 @GetMapping()을 작성한다. 괄호 안에는 `show.mustache` 파일에서 "/articles/{{article.id}}/edit" 주소로 연결 요청했으므로 이 URL인 "articles/{id}/edit"를 작성한다. **이때 변수는 {id}라고 작성하는 것을 유의한다.** (컨트롤러에서 URL 변수를 사용할 때는 하나({})만 사용한다.)
```java
@GetMapping("/articles")
public String index(Model model) {
    (중략)
}
@GetMapping("/articles/{id}/edit") // URL 요청 접수
public String edit() { // 메서드 생성 및 뷰 페이지 설정
    // 뷰 페이지 설정하기
    return "articles/edit";
}
```

**수정할 데이터 가져오기**

DB에 있는 기존 데이터를 불러오는 코드를 작성한다.
1. DB에서 데이터를 가져올 때는 리파지터리를 이용한다. 따라서 articleRepository의 findById(id) 메서드로 데이터를 찾아 가져온다. 만약 데이털르 찾지 못하면 null을 반환하고 데이터를 찾았다면 Article 타입의 articleEntity로 저장한다.
2. 변수 id는 메서드의 매개변수로 받아 오고 자료형은 Long으로 작성한다. GetMapping() 어노테이션의 URL 주소에 있는 id를 받아오는 것이므로 데이터 타입 앞에 @PathVariable 어노테이션을 추가한다.

```java
@GetMapping("/articles/{id}/edit")
public String edit(@PathVariable Long id) {
    // 수정할 데이터 가져오기
    Article articleEntity = articleRepository.findById(id).orElse(null);
    // 뷰 페이지 설정하기
    return "articles/edit";
}
```

### 수정 폼 만들기

`,src > main > resources > templates > articles`에 `edit.mustache` 파일을 만든다.
`<form>` 태그는 `new.mustache` 파일 부분을 복사한다.
{% raw %}
```html
{{>layouts/header}}
{{#article}}
<form class="container" action="" method="post"> 
    (중략)
    <input type="text" class="form-control" name="title" value="{{title}}">
    (중략)
    <textarea class="form-control" rows="3" name="content">{{content}}</textarea>
    <a href="articles/{{article.id}}">Back</a>
</form>
{{/article}}
{{>layouts/footer}}
```
{% endraw %}

## 3. 수정 데이터를 DB에 갱신하기

클라이언트와 서버 간 처리 흐름의 4가지 기술
- MVC(Model-View-Controller) : 서버 역할을 분담해 처리하는 기법
- JPA(Java Persisitence API) : 서버와 DB 간 소통에 관여하는 기술
- SQL(Structured Query Language) : DB 데이터를 관리하는 언어
- HTTP(HyperText Transfer Protocol) : 데이터를 주고받기 위한 통신 규약

### HTTP 메서드

**프로토콜(Protocol)** : 컴퓨터 간에 원활하게 통신하기 위해 사용하는 전 세계 표준, 기기 간에 각종 신호 처리 방법, 오류 처리, 암호, 인증 방식 등을 규정하고 있어 이를 따라야만 오류나 지연없이 원활하게 통신이 가능하다.

- FTP(File Transfer Protocol)
- SMTP(Simple Mail Transfer Protocol)
- HTTP(HyperText Transfer Protocol)

HTTP는 웹 서비스에 사용하는 프로토콜로 클라이언트의 다양한 요청을 메서드를 통해 서버로 보내는 역할을 한다. 대표적인 메서드로는 다음과 같다.
- POST : 데이터 생성 요청
- GET : 데이터 조회 요청
- PATCH(PUT) : 데이터 수정 요청
- DELETE : 데이터 삭제 요청
데이터의 생성, 조회, 수정, 삭제는 데이터 관리에서 가장 기본이 되는 동작으로 CRUD(Create Read Update Delete)라고도 한다.
