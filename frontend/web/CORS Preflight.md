## 1. CORS Preflight

### 1.1 기본 개념

브라우저가 본 요청을 보내기 전에, 이 요청이 안전한지 서버에 먼저 확인하는 과정이다.  
모든 요청에 대해 실행되는 게 아니라, 조건을 벗어나는 요청에 대해서만 실행된다.  

- 약간 더 쉽게 말하자면
    
    브라우저가 서버한테 “나 이런 요청 보낼건데 너 이 요청 알고 있는 거 맞지?” 묻고  
    서버는 “응 나 그 출처 알고 있어” 라고 답하는 느낌이라고 생각하면 조금 더 와닿을 것이다.

## 2. Simple Request 조건

### 2.1 조건

아래 조건을 모두 만족하면 preflight 없이 바로 본 요청이 나간다.

- 메서드가 GET, POST, HEAD 중 하나
- Content-Type이 application/x-www-form-urlencoded, multipart/form-data, text/plain 중 하나
- 커스텀 헤더 없음

### 2.2 조건을 벗어나는 경우

아래와 같은 경우 preflight가 발생한다.

- PUT, DELETE 등의 메서드 사용
- Content-Type이 application/json
    
    HTML 폼이 기본으로 보내는 형식(urlencoded, multipart 등)이 아니라, 요즘 API 통신에서 흔히 쓰는 JSON 형식이라 브라우저가 "안전하다고 보장된 형식"으로 안 봐서 걸린다.
    
- Authorization 같은 커스텀 헤더 포함
    
    로그인 토큰 등을 담아 보내는 헤더처럼, 브라우저가 기본으로 허용하는 헤더 목록에 없는 헤더를 말한다.


- 조건의 기준이 뭘까?
    
    아주 간단하게 설명하면 과거 CORS가 없던 시절에 HTML 폼 태그로 만들 수 있었던 요청(GET, POST 등)은 딱히 막을 이유가 없어서 통과 시켜주는 것이고  
    폼으로는 못 만들던 요청(PUT, DELETE)는 preflight로 확인하는 것이다.

## 3. Preflight 동작 순서

### 3.1 사전 요청

브라우저가 본 요청 대신 **OPTIONS** 메서드로 먼저 서버에 요청을 보낸다.

OPTIONS는 실제 데이터를 주고받는 게 아니라, "이 요청 보내도 되냐"고 사전 확인만 하는 용도의 HTTP 메서드다.

### 3.2 서버 응답

서버가 아래 응답 헤더로 답한다.

```
Access-Control-Allow-Origin
Access-Control-Allow-Methods
Access-Control-Allow-Headers
```

각각 어느 출처(요청을 보낸 웹사이트 주소), 어떤 메서드, 어떤 커스텀 헤더를 허용하는지를 나타낸다.

### 3.3 브라우저 판단

브라우저가 이 응답을 보고 허용 여부를 판단한다.

1. **허용되는 경우**
    
    그제서야 진짜 요청(본 요청)을 보낸다.
    
2. **허용되지 않는 경우**
    
    본 요청 자체가 나가지 않고 콘솔에 CORS 에러가 뜬다.
    

## 4. Credentials 포함 시 주의점

쿠키, Authorization 헤더 등 인증 정보(credentials)를 포함하는 요청이면 Access-Control-Allow-Origin에 와일드카드(*, 모든 출처를 다 허용한다는 뜻)를 쓸 수 없다.

인증 정보가 실려 있는 요청까지 아무 출처에서나 받아버리면 보안 구멍이 생기기 때문이다. 그래서 정확한 출처를 명시해야 하고, 서버가 아래 헤더도 같이 응답해야 브라우저가 이를 받아들인다.

```
Access-Control-Allow-Credentials: true
```


---
이제까지 CORS 에러 뜨면 그냥 서버 설정 문제겠거니 하고 넘어갔는데  
preflight 단계에서 정확히 뭘 검증하는지 알고 나니 에러 메시지 보고 바로 원인 짚을 수 있을 것 같다.  
