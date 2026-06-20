---
title: "[코딩 자율학습 스프링 부트 3 자바 백엔드 개발 입문] 8장 게시글 삭제하기: Delete"
date: 2026-06-20 16:00:00 +0900
categories: [Study, Backend]
tags: [spring-boot, backend]
description: 코딩 자율학습 스프링 부트 3 자바 백엔드 개발 입문 8장의 핵심 개념 정리
---

## 1. 데이터 삭제 과정

게시판에서 글을 삭제하는 과정은 다음과 같다.

1. 클라이언트가 HTTP 메서드로 특정 게시글의 삭제를 요청한다.
2. 삭제 요청을 받은 컨트롤러는 리파지터리를 통해 DB에 저장된 데이터를 찾아 삭제한다. 이 작업은 기존 데이터가 있는 경우에만 수행된다.
3. 삭제가 완료됐다면 클라이언트를 결과 페이지로 리다이렉트한다.

결과 페이지로 리다이렉트할 때 클라이언트에 삭제 완료 메시지를 띄우기 위한 클래스가 `RedirectAttributes`이다.
`RedirectAttributes` 객체의 `addFlashAttribute()`라는 메서드는 리다이렉트된 페이지에서 사용할 일회성 데이터를 등록할 수 있다.

---

## 2. 데이터 삭제하기

### Delete 버튼 추가하기

[Delete] 버튼을 상세 페이지에 추가하기 위해 `show.mustache` 파일을 열어 [Edit] 버튼을 복사해 [Delete] 버튼을 만든다.

{% raw %}

```html
<a href="/articles/{{article.id}}/edit" class="btn btn-primary">Edit</a>
<a href="/articles/{{article.id}}/delete" class="btn btn-primary">Delete</a>
```

{% endraw %}

### Delete 요청을 받아 데이터 삭제하기

원래는 데이터를 삭제하므로 DELETE를 써야하지만 HTML에서는 POST와 GET을 제외한 다른 메서드를 제공하지 않는다. 그래서 지금은 GET 방식으로 삭제 요청을 받아 처리한다.

**delete() 메서드 기본 틀 만들기**

`ArticleController`의 `update()` 메서드 아래에 삭제 요청을 받아 처리할 `delete()` 메서드를 추가한다.
원래는 `@DeleteMapping("/articles/{id}/delete")`로 작성하면 되지만 DELETE 메서드를 지원하지 않으니 `@GetMapping("/articles/{id}/delete")`로 작성한다.

```java
public class ArticleController {
    (중략)
    public String update(ArticleForm form) {
        (중략)
    }

    @GetMapping("/articles/{id}/delete")
    public String delete() {
        log.info("삭제 요청이 들어왔습니다!"); // 메서드 동작 확인용 로그
        return null;
    }
}
```

**삭제할 대상 가져오기**

DB에 접근해 데이터를 처리할 때는 JPA의 리파지터리를 이용한다.

```java
@GetMapping("/articles/{id}/delete")
public String delete(@PathVariable Long id) { 
    log.info("삭제 요청이 들어왔습니다!"); // 메서드 동작 확인용 로그

    // 1. 삭제할 대상 가져오기
    Article target = articleRepository.findById(id).orElse(null); // 데이터 찾기
    log.info(target.toString()); // 데이터 존재 여부 확인용 로그

    // 2. 대상 엔티티 삭제하기
    if (target != null) {
        articleRepository.delete(target);
    }

    return null;
}
```

**결과 페이지로 리다이렉트하기**

리다이렉트는 return 문에 작성한다. 게시글을 삭제하면 목록 페이지로 돌아가야 한다. 따라서 `"redirect:/articles"`를 작성한다.

```java
// 3. 결과 페이지로 리다이렉트하기
return "redirect:/articles";
```

### 삭제 완료 메시지 남기기

`RedirectAttributes`를 활용하려면 `delete()` 메서드의 매개변수로 받아 와야 한다.

`ArticleController.java`

```java
public String delete(@PathVariable Long id, RedirectAttributes rttr) {
    (중략)

    if (target != null) {
        articleRepository.delete(target);
        rttr.addFlashAttributes("msg", "삭제됐습니다!");
    }
}
```

형식 : `객체명.addFlashAttributes(념겨_주려는_키_문자열, 넘겨_주려는_값_객체);`

헤더 안에 삭제 메시지를 출력하기 위해 `resources > templates > layouts` 디렉터리에서 `header.mustache` 파일을 연다.

{% raw %}

```html
(생략)
</nav>

{{#msg}} <!-- msg 사용 범위 설정 -->
<div class="alert alert-primary alert-dimissible"> <!-- 메시지 창 작성 -->
    {{msg}}
    <button type="button" class="btn-close" data-bs-dismiss="alert" aria-label="Close"></button>
</div>
{{/msg}}
```

{% endraw %}