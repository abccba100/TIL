## 1. 렌더링 과정


### 1.1 렌더링 과정

1. **HTML 파싱**
2. **CSS 파싱**
    
    HTML에서 link를 만나면 CSS 파싱을 하게 된다.
    
3. **DOM 생성**
4. **CSSOM 생성**
5. **Render Tree**
    
    DOM과 CSSOM을 결합 했기 때문에 경우에 따라서는 DOM과 일치하지 않을 수도 있다.
    ex) `display: none`
    
6. **Layout 계산 (Reflow 단계)**
    
    Render Tree에 있는 HTML과 CSS를 보고 위치, 크기, 배치를 계산을 한다.
    도면 작성이라고 생각하면 쉽다.
    
7. **paint (Repaint 단계)**
    
    CSS에 맞게 색칠을 해주는 단계이다.
    도면 작성 이후에 색칠 작업이라고 생각하면 쉽다.
    
8. 화면 출력 (Composite)
    
    여러 개의 레이터들을 합쳐서 최종 화면을 보여준다.
    

## 2. Reflow & Repaint


### 2.1 Reflow

Reflow는 Layout 단계를 다시 실행하는 것으로 위치, 크기, 배치를 다시 계산한다.

아래와 같이 어떤 요소의 크기가 바뀌게 되면,
주변 요소에도 영향을 줄 수 있기 때문에 Layout을 다시 계산(Reflow)한다.

```jsx
box.style.width = "300px";
```

### 2.2 Repaint

Repaint는 Paint 단계를 다시 실행하는 것으로 시각적 표현(색상, 배경, 테두리 등)을 다시 그리게 된다.

아래와 같이 시각적인 속성의 값이 변하게 되면,
Paint 단계를 다시 실행(Repaint)한다.

```jsx
box.style.backgroundColor = "blue";
```

### 2.3 Reflow와 Repaint의 관계

Reflow와 Repaint의 관계는 아래와 같다.

1. Reflow가 발생하면 Repaint도 대부분 같이 발생한다.
    
    왜냐하면 위치, 크기 등을 다시 계산했기 때문에 거기에 맞게 다시 색칠을 해야 하기 때문이다.
    
2. Repaint만 발생할 수 있다.
    
    위치, 크기는 변하지 않고 색상 같은 시각적 표현이 변한 것은 Repaint 거치게 된다.
    
3. Composite만 실행될 수 있다.
    
    transform은 실제 Layout 위치를 바꾸지 않고, 보이는 위치만 바꾸기 때문에 Composite만 실행된다.
    

### 2.4 비용 차이

계산 비용 크기는 보통 아래와 같다.

**Reflow > Repaint > Composite**

Reflow가 가장 비싼 이유는 요소 하나만 바뀌더라도 부모, 자식, 형제에게 영향을 줄 수 있어 모두를 계산해야 한다.  
Repaint는 배치가 그대로여도 픽셀을 다시 칠해야 하기 때문에 비용이 있는 편이다.  
Composite는 이미 그려진 레이어를 합치고 위치나 투명도 정도만 바꾸기 때문에 매우 저렴하다. 

## 3. 최적화


### 3.1 최적화 필요 이유

최적화가 필요한 이유는 만약 Reflow & Repaint 단계 없이 Composite 단계만 거칠 수 있다면,
많은 계산이 필요 없기 때문에 애미네이션이 부드러워지고 계산 비용을 아낄 수 있다.

### 3.2 최적화 방법

1. **transform 사용**
    
    가능하다면 transform을 사용하면 Reflow & Repaint 없이 Composite만 실행되도록 할 수 있다.
    
2. DOM 수정 최소화
    
    나쁜 예 → 브라우저가 계산을 여러 번 할 가능성이 있음
    
    ```jsx
    box.style.width = "100px";
    box.style.height = "100px";
    box.style.margin = "20px";
    ```
    
    좋은 예 → 가능하다면 DOM 계산을 최소화
    
    ```jsx
    box.style.cssText = `
      width:100px;
      height:100px;
      margin:20px;
    `;
    ```
    
3. **반복문 안에서 Layout을 읽지 않음**
    
    `offsetWidth` , `offsetHeight` , `clientWidth` , `getBoundingClientRect()`  같이 Layout을 읽는 것들은 브라우저에게 Layout 계산이 끝났는지 물어보는 것과 같다 따라서 강제로 Layout 계산을 시킬 수 있다.
    
4. **Fragment 사용**
    
    나쁜 예 → DOM 삽입 1000번
    
    ```jsx
    for(let i=0;i<1000;i++){
      document.body.appendChild(li);
    }
    ```
    
    좋은 예 → DOM 삽입 1번
    
    ```jsx
    const fragment =
      document.createDocumentFragment();
    
    for(let i=0;i<1000;i++){
      fragment.appendChild(li);
    }
    
    document.body.appendChild(fragment);
    ```
    

더 많은 방법들이 있지만, 너무 다양하기도 하고 어려운 것들도 있어서 꾸준히 찾아보면서 공부할 필요가 있어 보인다.

---

Reflow와 Repaint가 뭔지는 대충 알고 있었는데
Composite 단계가 있는 줄은 몰랐다 그냥 transform 같은 것도 다 Reflow 같은 걸로 처리하는 줄 알았는데
'Composite로 처리하는 걸 보고 브라우저가 내 생각보다는 더 똑똑하구나' 라는 생각이 들었다.
