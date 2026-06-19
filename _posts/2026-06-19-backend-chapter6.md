---
title: "[코딩 자율학습 스프링 부트 3 자바 백엔드 개발 입문]6장 게시판 내 페이지 이동하기"
date: 2026-06-19 20:00:00 +0900
categories: [Study, Backend]
tags: [spring-boot, backend]
description: 코딩 자율학습 스프링 부트 3 자바 백엔드 개발 입문 6장의 핵심 개념 정리
---

# 1. 링크와 리다이렉트란

보통 웹 게시판 내 페이지 이동은 다음과 같다.

![게시판 내 페이지 이동](/assets/img/posts/page-move.png)

## 링크(link)

미리 정해 놓은 요청을 간편히 전송하는 기능, 보통 페이지 이동을 위해 사용한다.
HTML의 `<a>` 태그 혹은 `<form>` 태그로 작성하며, 클라이언트가 링크를 통해 어느 페이지로 이동하겠다고 요청하면 서버는 결과 페이지를 응답한다.

## 리다이렉트(redirect)

클라이언트가 보낸 요청을 마친 후 계속해서 처리할 다음 요청 주소를 재지시한다.
이를 통해 분리된 기능을 하나의 연속적인 흐름으로 연결할 수 있다.
리다이렉트를 지시받은 클라이언트는 해당 주소로 다시 요청을 보내고 서버는 이에 대한 결과를 응답한다.

# 2. 링크와 리다이렉트를 이용해 페이지 연결하기

## 기존에 만든 서비스의 문제점

1. 새 글을 작성하기 위한 링크 없음
2. 새 글을 작성한 후 다시 목록으로 돌아가는 [뒤로가기]가 없음

전반적으로 게시판을 구성하는 페이지 간 연결고리가 없어 불편하다.
이를 링크와 리다이렉트를 적용해 해결해보자.

## 새 글 작성 링크 만들기

`src > main > resources > templates > articles > index.mustache` 에 다음 코드를 작성한다.

```html
</table>

<a href="/articles/new">New articles</a>

{{>layouts/footer}}
```

## `<입력 페이지>` → `<목록 페이지>` 돌아가기

입력 페이지의 뷰 파일인 new.mustache를 열어 코드의 [Submit] 버튼 아래에 링크를 추가한다.

```html
<button type="submit" class="btn btn-primary">Submit</button>
<a href="/articles">Back</a>
</form>
```

## `<입력 페이지>` → `<상세 페이지>` 이동하기

새 글을 작성하고 상세 페이지로 이동할 수 있게 리다이렉트를 적용한다.

입력 페이지에서 데이터를 전송하면 ArticleController의 createArticle() 메서드에서 폼 데이터를 받는다.
createArticle() 메서드는 포스트(Post) 방식으로 "/articles/create"라는 URL 요청을 받아 폼 데이터를 처리한다.
리다이렉트는 return 값에 정의한다.

형식 : `return "redirect:URL_주소"`;

\+ 연산자를 사용해 id에 따라 URL 주소가 달라지게 할 수 있다.

```java
@PostMapping("/articles/create")
public String createArticle(ArticleForm form) {
    (중략)
    // 2. 리파지터리로 엔티티를 DB에 저장
    Article saved = articleRepository.save(article);
    (중략)
    return "redirect:/articles/" + saved.getId();
}
```

getId() 메서드를 사용하려면 해당 게터(Getter) 메소드가 있어야 한다.

### 방법 1.

```java
public class Article {
    (중략)
    public Long getId() { // 주의: 데이터 타입을 String -> Long 변경해야 함
        return id;
    }
}
```

### 방법 2.

```java
@Entity
@Getter // 롬복으로 게터 추가
public class Article {
    (중략)
    // getId() 메서드 삭제
    /* public Long getId() {
        return id;
    }*/ 
}
```

## `<상세 페이지>` → `<목록 페이지>` 돌아가기

상세 페이지의 URL인 `articles/{id}`를 받는 컨트롤러의 메서드는 ArticleController의 show() 메서드이다.
이 메서드의 return 문을 보면 show라는 머스테치 파일(뷰 파일)을 반환한다.

```java
@GetMapping("articles/{id}")
public String show(@PathVariable Long id, Model model) {
    (중략)
    return "articles/show";
}
```

따라서 show.mustache 파일에 링크를 추가하면 된다.

```html
</table>

<a href="/articles">Go to Article List</a>

{{>layouts/footer}}
```

## `<목록 페이지>` → `<상세 페이지>` 이동하기

제목을 클릭하면 해당 글의 `<상세 페이지>`로 이동한다.

목록 페이지의 뷰 파일인 index.mustache 에서 게시글 제목에다가 링크를 걸어준다.

```html
{{#articleList}}
<tr>
    <th>{{id}}</th>
    <td><a href="/articles/{{id}}">{{title}}</a></td>
    <td>{{content}}</td>
</tr>
{{/articleList}}
```
