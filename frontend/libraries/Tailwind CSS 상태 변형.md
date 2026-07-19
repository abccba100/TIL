## 1. `hover:` / `focus-visible:`


### 1.1 hover

마우스를 올렸을 때 스타일을 바꾸고 싶으면 클래스 앞에 `hover:`를 붙인다.

```
<button className="bg-blue-500 hover:bg-blue-600">
  버튼
</button>
```

- `hover:bg-blue-600` ← `:hover { background-color: ... }`를 클래스로 표현한 것

### 1.2 focus랑 focus-visible이 다른 이유

포커스 스타일을 줄 때 `focus:` 말고 `focus-visible:`을 써야 하는 경우가 있다.

```
<button className="focus:outline-2 focus:outline-blue-500">버튼</button>
<button className="focus-visible:outline-2 focus-visible:outline-blue-500">버튼</button>
```

`focus:`는 마우스 클릭으로 포커스됐을 때도 스타일이 적용된다.

근데 버튼을 마우스로 클릭했을 때 파란 테두리(아웃라인)가 뜨는 건 대부분 사용자 입장에서 시각적으로 어색하고 불필요하다.

반면 `focus-visible:`은 키보드로 Tab 눌러서 포커스 이동했을 때만 적용된다.

- 마우스 클릭 → 포커스는 되지만 시각적으로 뭘 눌렀는지 이미 보여서 아웃라인이 굳이 필요 없음
- 키보드 탭 이동 → 지금 어디에 포커스가 있는지 시각적으로 보여줘야 함 (접근성)

그래서 `focus-visible:`을 쓰면 마우스 사용자한텐 안 보이고 키보드 사용자한테만 보이는, 딱 필요한 사람한테만 필요한 스타일을 줄 수 있다.

## 2. `group`


### 2.1 기본 사용법

부모 요소에 마우스를 올렸을 때 그 안에 있는 자식 요소의 스타일을 바꾸고 싶은 경우가 있다.

예를 들면 카드에 hover했을 때 카드 안의 제목 색이 같이 바뀌는 것.

```
<div className="group cursor-pointer rounded-lg border p-4 hover:border-blue-500">
  <h3 className="text-gray-900 group-hover:text-blue-600">카드 제목</h3>
  <p className="text-gray-500 group-hover:text-gray-700">카드 설명</p>
</div>
```

- 부모 `div`에 `group` ← "나를 부모로 지정한다"는 표시
- 자식 요소에 `group-hover:` ← "이 group으로 지정된 부모가 hover 상태일 때"

### 2.2 group이 실제로 해결하는 문제

순수 CSS로도 `.parent:hover .child { color: blue; }`처럼 쓰면 부모 hover에 따라 자식 스타일을 바꾸는 건 원래부터 가능하다.

하위 결합자(descendant combinator)랑 `:hover`를 조합한, CSS 초창기부터 있던 평범한 문법이다.

그럼 `group-hover:`가 해결하는 진짜 문제는 뭐냐면, 이 관계를 표현하려면 원래는 `.css` 파일에 이름 붙은 선택자(`.parent`, `.child`)를 새로 만들어야 한다는 점이다.

이건 Tailwind가 애초에 없애려던 방식(클래스 이름 짓기, 파일 분리)으로 되돌아가는 거라, utility-first 워크플로우가 깨진다.

`group` / `group-hover:`는 이 부모-자식 hover 관계를 커스텀 CSS 파일 없이 클래스만으로 표현할 수 있게 해주는 게 핵심이다.

"순수 CSS로 불가능해서"가 아니라 "utility-first 방식을 유지하려고" 만들어진 문법이라는 게 정확한 이유다.

## 3. `peer`


### 3.1 기본 사용법

`group`이 부모→자식 관계라면 `peer`는 형제 요소끼리 상태를 공유한다.

```
<input type="checkbox" className="peer" />
<span className="peer-checked:text-blue-500">동의함</span>
```

- 기준이 되는 요소에 `peer` ← "나를 peer로 지정한다"
- peer 바로 뒤에 오는 형제 요소에 `peer-checked:` ← "이 peer가 checked 상태일 때"

### 3.2 sr-only랑 같이 쓰는 커스텀 체크박스 패턴

실무에서는 기본 체크박스 UI를 그대로 안 쓰고 커스텀 디자인을 입히는 경우가 많은데, 이때 `peer`랑 `sr-only`를 같이 쓰는 패턴이 자주 나온다.

```
<input id="agree" type="checkbox" className="peer sr-only" />
<label
  htmlFor="agree"
  className="cursor-pointer rounded border px-3 py-1 peer-checked:bg-blue-500 peer-checked:text-white"
>
  동의합니다
</label>
```

- `sr-only` ← 요소를 시각적으로는 숨기지만 스크린리더에는 계속 읽히게 하는 유틸리티 (`display: none`으로 완전히 숨기면 접근성이 깨짐)
- 실제 체크박스는 `sr-only`로 숨기고, `label`을 커스텀 UI로 보여줌
- `label`이 `htmlFor`로 `input`이랑 연결돼 있어서 `label`을 클릭해도 체크박스가 토글됨
- 체크박스가 checked 되면 `peer-checked:`로 `label`의 스타일이 바뀜

### 3.3 peer가 실제로는 순수 CSS의 한계를 메꾸는 케이스

group과 다르게, peer가 다루는 "형제 상태를 보고 다른 형제 스타일을 바꾸는" 문제는 최근까지 순수 CSS 선택자만으로는 안 됐던 영역이다.

`:has()` 선택자가 생기면서 비로소 CSS로도 흉내 낼 수 있게 됐다.

```
.card:has(input:checked) label {
  color: blue;
}
```

`:has()`는 "괄호 안 조건에 맞는 후손을 가진 요소"를 선택한다.

위 코드는 "checked된 input을 포함한 카드 안의 label 색을 바꾼다"는 뜻으로, `peer-checked:`가 하는 일과 거의 같다.

다만 `:has()`는 브라우저 지원이 비교적 최근에야 자리 잡았고, 여전히 별도 CSS 선택자를 파일에 적어야 하니까 실무에서는 `peer`를 쓰는 게 utility-first 워크플로우에 더 맞는다.

## 4. 임의값(arbitrary value) 문법


### 4.1 기본 사용법

스케일에 없는 값이 필요할 때 대괄호로 직접 값을 넣을 수 있다.

```
<div className="w-[137px] top-[13px] bg-[#1da1f2]">...</div>
```

`w-[137px]`처럼 CSS 스펙에 있는 값이면 거의 다 대괄호 안에 넣을 수 있다.

다만 JIT가 이 문자열도 소스 코드에서 완전한 형태로 스캔해야 인식하기 때문에, 이전에 배운 동적 클래스명 문제( `w-[${size}px]`  같은 조합)는 여기서도 똑같이 적용된다.

### 4.2 variant랑 결합

임의값도 `hover:`, `md:` 같은 variant랑 결합해서 쓸 수 있다.

```
<div className="hover:bg-[#1da1f2] md:w-[300px]">...</div>
```

다만 스케일 값이 이미 있는 경우(`w-4`, `bg-blue-500` 등)는 임의값보다 스케일을 쓰는 게 낫다.

디자인 일관성이 스케일에서 나오는 거라, 임의값을 남발하면 그 의미가 없어지기 때문이다. 정말 스케일에 없는 값이 필요할 때만 예외적으로 쓰는 게 맞다.


---
Tailwind CSS에서는 CSS 파일을 만들어서 작성하는 것을 최소하기 위해서 만들어졌는데,
'어떻게 유틸리티 우선 방식으로 상태에 따라 자식이나 형제에 CSS를 입힐 수 있을까?' 라는 생각이 많이 들었는다.  
이것 역시 내 생각보다는 훨씬 간단하게 group, peer로 간단하게 해결해 버려서 진짜 Tailwind CSS를 만든 사람이 똑똑했구나라고 느끼게 된다.  
