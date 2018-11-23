# .NET XSS 위협 완화

작성자 : gbkim1988@gmail.com

# 요약

Stored XSS 위협에 대한 방어

# 해결책

* 다중 레이어 기반의 위협 완화 전략
* 화이트리스트(whitelist) 기반의 입력값 통제 정책
* 잘못 구성된 HTML 혹은 악의적인 HTML 에 대한 처리 능력

## 1. 다중 레이어 기반의 위협 완화 전략이란?

레이어는 사용자 입력 값을 처리하는 각 처리 단계를 의미한다. 일반적인 서비스는 다음의 단계를 거치어 입력 값을 처리한다. 

**$사용자 입력 값 처리 단계**

1. 프론트엔드에서 사용자 입력 값을 서버로 전송하기 전에 javascript 를 이용하여 sanitize
2. 백엔드에서 사용자 입력 값을 DB 에 저장하기 전에 백엔드 로직을 통해 사용자 입력 값을 sanitize
3. DB 에서 사용자 입력 값을 출력할 때에 백엔드에서 검증 후 프론트엔드로 출력
4. 프론트엔드에서 데이터를 출력하기 전에 javascript 모듈을 통해 sanitize

## 2. 화이트리스트(whitelist) 기반의 입력 값 통제 정책이란?

화이트리스트(whitelist) 기반의 입력값 통제 정책이란, 사용자가 입력 값 중에 서비스의 안정성과 보안을 저해할 수 있는 요소들을 제거하기 위하여 입력 가능한 유형을 미리 정의하는 것을 의미한다. 개발자들은 사전에 수립된 정책을 기반으로 일관된 코드를 산출하여 입력 값을 검증 및 필터할 수 있으며 서비스 전반에서 입력 데이터에 대한 신뢰도를 높일 수 있다.

그렇다면, 사용자가 입력하는 데이터에는 어떤 것들이 있을까? 크게 두 가지로 분류가 가능하다. 사용자가 입력하는 데이터에 HTML, Markdown 기호 등이 입력되는 경우, 혹은 단순 파라미터 값이 대입되는 경우이다. 후자의 경우에 개발자들은 단순한 정책으로 필터링, 치환, 길이값 제한 등을 구현할 수 있지만 문제는 HTML, Markdown 등의 구조화된 입력값들이다.

(code-machina) TODO: 실제 데이터를 기반하여 어떤한 데이터가 검색되는지 확인해 보자.

**$입력 값 분류**
1. HTML 컨텐츠
2. Text 컨텐츠
3. 단순 파라미터
4. 혼합 컨텐츠 (Text, HTML)

이 중 HTML 컨텐츠에 대한 화이트리스트 정책을 적용하는 방안을 알아본다.

### 2.1 화이트리스트: HTML 컨텐츠

멀티이미지, 이미지 등을 제공받는 블로그 서비스의 경우 사용자 입력의 내용이 HTML로 구성되어 전송된다. 전송되는 데이터에는 많은 태그 정보가 포함되기 때문에 Sanitize 를 수행할 때 어려움을 느끼기 마련이다. 이를 단순한 문제로 치환하기 위해서는 다음과 같은 고민이 필요하다.

- 사용자 입력을 처리하기 위한 정책
  - 잘못 구성된 HTML 의 경우
  - 잘 구성된 HTML 의 경우
  - 악의적인 코드가 삽입된 HTML 의 경우
- 화이트리스트
  - Tag
  - Attribute
  - Allowed Host
- 블랙리스트
  - 악의적인 패턴 필터링

결과적으로 블랙리스트를 적용할 경우 작업 부하가 상당함을 코드를 계획하지 않고도 유추할 수 있다. 따라서, 화이트리스트 적용 방법에 대해 고민을 할 것이다. 화이트리스트는 사이트의 일관적인 정책을 세부적으로 수립하기 위해서 세세한 규칙을 수립해야 한다. 예를 들어 보자.

- www.example.com 도메인만을 src attribute 에 허용한다.
- src, href 의 기본적인 attribute 만을 허용하고 나머지는 허용하지 않는다.
- tag 는 img, iframe, a, p, strong 헤더만을 허용한다.

위의 규칙을 정립하고 적용하기 위해서는 Backend, Frontend 에서 다음과 같이 Dictionary 를 통한 정책 정의가 필수이다. 가독성을 위하여 python 코드로 정의한다.

```python
whitelist_policy = {
    'AllowedHost': [
        'www.example.com',
    ],
    'AllowedTag': [
        'a',
        'img',
        'iframe',
        'p',
        'strong',
    ], 
    'AllowedAttribute': [
        'src',
        'href'
    ]
}
```

화이트리스트 정책을 위와 같이 dictionary 로 정의하면 개발자의 입장에서 깔끔한 정책 정의를 제공할 수 있다.

`Pseudo Code`

```python
santizer = Sanitizer(policy=whitelist_policy)
# from parameter `arg1`
html = arg1['html'] 
sanitized = sanitzer.load(html)

return response.message(sanitized)
```

`decorator` 패턴을 적용 `Pseudo Code`

```python

from policy import whitelist_html_policy

@antixss(policy=whitelist_html_policy)
def my_blog_article_view(request):
    c = {}
    # .... skipping  ....
    return render(request, "my_blog_article_view.html", c)

```





