## 1. 지속 가능한 컴포넌트


### 1.1 지속 가능한 컴포넌트란?

말은 어렵게 써뒀지만, 나는 지속 가능한 컴포넌트를 좋은 컴포넌트라고 생각한다.  
그리고 내가 생각하기 좋은 컴포넌트란 변경 사항이 있더라도 유연하게 변경이 가능하며, 한 가지의 역할만 하는 컴포넌트이다.

하지만, 이번에 지속 가능한 컴포넌트에 대해 공부를 하면서 기준이 조금 수정 되었다.
바뀐 기준은 다음과 같다.

1. **Headless 기반으로 추상화한다.**
2. **한 가지 역할만 가진다. → 컴포넌트를 조합해서 사용한다.**
3. **도메인을 분리한다.**

## 2. Headless 기반의 추상화


### 2.1 Headless란?

Headless는 UI없이 데이터 관리 같은 기능을 하는 함수이다.  
보통은 커스텀 훅으로 짜여져서 쓰인다.

아래와 같이 UI 없이 데이터 관리 혹은 로직만 제공한다.

```jsx
function useModal() {
  const [isOpen, setIsOpen] = useState(false);

  return {
    isOpen,
    open: () => setIsOpen(true),
    close: () => setIsOpen(false),
  };
}
```

### 2.2 추상화란?

추상화는 내부적으로 복잡한 기능을 숨기고 간단한 인터페이스로 제공하는 것을 말한다.  
이것도 보통 커스텀 훅으로 짜여져서 쓰인다.

아래에 나오는 예시 코드가 매우 복잡하다고 가정해보자.

추상화 전에는 아래와 같이 복잡한 코드를 직접 써야한다.

```jsx
const [value, setValue] = useState("");

const onChange = (e) => {
  setValue(e.target.value);
};
```

하지만, 추상화 후에는 훅을 사용하는 개발자가 `useInput()` 만 사용하면 된다.  
내부적으로 어떻게 돌아가는지 몰라도 된다. 그렇기에 **추상화에 목적은 복잡성을 숨기는 것이 된다.**

```jsx
const { value, onChange } = useInput();
```

### 2.3 Headless 기반의 추상화란?

Headless 기반의 추상화는 UI를 제공하지 않으면서 복잡성을 숨긴 추상화된 인터페이스(커스텀 훅)를 제공하는 것을 말한다.

- Headless 기반의 추상화를 하는 이유
    
    UI를 사용하지 않기 때문에 UI와 데이터 관리 & 로직을 분리할 수 있으며, 추상화 되었기 때문에 복잡성을 띄지 않아 사용하기 편하다.
    
- Headless 기반의 추상화를 쉽게 설명하자면
    
    대부분의 커스텀 훅은 추상화 때문이다. 그렇다면 추상화된 커스텀 훅에서 UI를 제공하지 않는다면 그게 Headless 기반의 추상화가 되는 것이다.
    

## 3. 컴포넌트 조합


### 3.1 컴포넌트를 조합하지 않는 경우

- 컴포넌트를 조합해서 사용하지 않으면 어떤 일이 발생할까?
    
    너무 쉽지만, 컴포넌트를 조합하지 않고 하나의 컴포넌트가 모든 역할을 맡게 하면 하나의 컴포넌트가 맡게 되는 일이 많아지고 결국 거대해지고 변경에 유연해지지 못한다.
    

아래 코드는 하나의 컴포넌트가 모든 역할을 다 하게 된다.  
만약 아래와 같은 변경 사항이 생길 경우 유연하게 대처하지 못한다.

- label이 ‘React Framework’가 아니라 다른 걸 사용하고 싶다.
- InputButton이 아니라 다른 기능(버튼 + 모달)이 필요하다.
- select 되는 부분에 다른 컴포넌트가 들어가게 하고 싶다.

이런 이유들 때문에 컴포넌트를 각각 역할에 맞게 분리하고 조합해서 사용해야 한다.

```jsx
function ReactFramerworkSelector({ defaultValue }) {
  const [isOpen, openk, close] = useBoolean();
  const [selected, change] = useState(defaultValue);

  return (
    <>
      <InputButton label="React Framework" value={selected} onClick={open} />
      {isOpen ? (
        <Options onClose={close}>
          {options.map((value) => {
            return (
              <Button
                selected={selected === value}
                onClick={() => change(value)}
              >
                {value}
              </Button>
            );
          })}
        </Options>
      ) : null}
    </>
  );
}
```

### 3.2 컴포넌트를 조합하는 경우

- 왜 Dropdown 컴포넌트를 만들었을까?
    
    Dropdown 컴포넌트를 만듦으로서 props를 받아 여러 곳에서 사용될 수 있게 되었다.
    
    - ~~label이 ‘React Framework’가 아니라 다른 걸 사용하고 싶다.~~ → 해결
    - ~~InputButton이 아니라 다른 기능(버튼 + 모달)이 필요하다.~~ → 해결
    - ~~select 되는 부분에 다른 컴포넌트가 들어가게 하고 싶다.~~ → 해결

- 왜 Select 컴포넌트를 만들었을까?
    
    Select 컴포넌트를 만들었기 때문에 상위 컴포넌트에서 Dropdown 컴포넌트의 내부 구조를 몰라도 되고 props만 넘겨서 간단하게 사용할 수 있다.
    

```jsx
// Select 라는 함수 만들어서 컴포넌트 분리
function Select({ label, trigger, value, onChange, options }: Props) {
	return (
		<Dropdown label={label} value={value} onChange={onChange}>
			<Dropdown.Trigger as={trigger} />
			<Dropdown.Menu>
				{options.map(option => (
					<Dropdown.Item>{option}</Dropdown.Item>
				})}
			</Dropdown.Menu>
	);
}
```

FrameworkSelect는 Select 컴포넌트를 사용하게 되는 상위 컴포넌트가 되면서 데이터 관리를 하게 된다.

```jsx
// Select 함수가 필요한 컴포넌트에서 활용
function FrameworkSelect() {
  const {
    data: { frameworks },
  } = useFrameworks();
  const [selected, change] = useState();
  
  return (
    <Select
    	trigger={<InputButton value={selected} />}
			value={selected}
			onChange={change}
			options={frameworks}
	/>
  );
}
```

## 4. 도메인 분리


### 4.1 도메인 분리해야 하는 이유

도메인을 분리한다는 건 특정 역할을 하는 컴포넌트에서 도메인 맥락을 제거해서 일반적인 인터페이스를 가지는 컴포넌트로 만든다는 걸 말한다.

FrameworkSelect에서 도메인을 분리하게 전에는 Framework라는 특정 도메인에 포함되어 있어 재사용하기 어려웠다.

하지만, 아래 코드처럼 도메인을 분리하게 되면 컴포넌트 인터페이스가 일반적이게 바뀌게 된다.  
**컴포넌트 인터페이스는 일반적일 수록 다른 개발자들이 이해하고 사용하기 쉽다.**

```jsx
// 특정 도메인을 제거하고 일반적인 역할을 할 수 있도록 인터페이스 변경한 코드
interface Props {
  options: Array<{ label: string }>;
  value?: string[];
  onChange?: (selecteds: string[]) => void;
  valueAs?: (value?: string[]) => string;
}
```

---

처음에 지속 가능한 컴포넌트에 대한 것을 보고 배울 때는 지금까지 해온 것이랑 크게 다를 게 없다고 생각했지만,  
정작 제대로 공부하니 내가 지금까지 짜온 코드가 매우 부끄럴 정도로 품질이 좋지 않았다는 걸 깨닳았다.  
그리고 이제는 컴포넌트가 무엇인지 다시 한 번 생각하게 해주어서 막막했던 한 부분을 해결한 기분이다.
