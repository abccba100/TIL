## 1. useForm 개념


### 1.1 useForm이란?

input 값을 state로 계속 들고 있으면서 렌더링 해주는 게 아닌, ref로 값을 가지고 있으면서  
렌더링을 효율적으로 해주는 React에서 제공하는 폼 관리 hook이다.

### 1.2 useForm을 쓰는 이유

```jsx
const [firstName, setFirstName] = useState('');

return (
	<input 
		value={firstName} 
		onChange={setFirstName(e.target.value)} 
	/>
)
```

입력 폼이 위처럼 적을 때는 useState로 구현하더라도 큰 문제가 발생하지는 않는다.

```jsx
const [firstName, setFirstName] = useState('');
const [lastName, setLastName] = useState('');
const [gender, setGender] = useState('M');
const [phoneNumber, setPhoneNumber] = useState('01012345678');
```

하지만, 이렇게 입력 폼이 많아지게 된다면 useState로 관리해야 할 상태가 많아지며,  
예를 들어 `firstName` 이라는 상태만 변하더라도 `firstName` , `lastName` , `gender` , `phoneNumber` 를 모두 다시 렌더링해야 한다.

- 조금 엉뚱하지만 여기서 하나의 생각이 든 게 있다.
    - 경우 1
    
    ```jsx
    const [firstName, setFirstName] = useState('');
    
    return (
    	<input value={firstName} onChange={setFirstName(e.target.value)} />
    	<span>요소1</span>
    	<span>요소2</span>
    	<span>요소3</span>
    )
    ```
    
    - 경우 2
    
    ```jsx
    const [firstName, setFirstName] = useState('');
    const [lastName, setLastName] = useState('');
    const [gender, setGender] = useState('M');
    const [phoneNumber, setPhoneNumber] = useState('01012345678');
    
    return (
    	<input value={firstName} onChange={setFirstName(e.target.value)} />
    	<input value={lastName} onChange={setLastName(e.target.value)} />
    	<input value={gender} onChange={setGender(e.target.value)} />
    	<input value={phoneNumber} onChange={setPhoneNumber(e.target.value)} />
    )
    ```
    

위와 같은 경우에는
‘경우 1에서 폼이 1개 더라도 state가 변할 때 전체가 리렌더링 되니 경우 2에서 리렌더링 되는 거랑 같지 않을까? 그러면 useState를 많이 써도 문제가 없는 거 아닌가?’ 라는 생각이 들어서 찾아보니

두 경우 모두 폼 state가 변할 때 전체가 리렌더링 되는 건 맞지만,  
이전과 변경된 점이 있는지 계산할 때 일반적인 요소들에 비해서 state를 이용하는 요소들은 계산 비용이 무겁다고 한다.

### 1.3 useForm의 해결 방안

해결 방안을 알기전에 `controlled` , `uncontrolled` 가 뭔지 알아야 하는데,  
그 전에 또 알아야 할 게 `<input>`은 React가 리렌더링을 안해줘도 value가 변하면 알아서 DOM에 반영된다.

- **controlled**
    1. 사용자가 값을 입력한다.
    2. `onChange={(e) => setValue(e.target.value)}` 로 value를 변경 시킨다. ← setValue로 인해 리렌더링 된다.
    3. `value={value}` 로 input의 value를 덮어 씌운다.
- **uncontrolled**
    1. 사용자가 값을 입력한다.
    2. DOM value가 변경된다.

useState는 **controlled** 방식을, useForm은 **uncontrolled** 방식을 이용한다.

그러면 딱 감이 오는 게 useFom 방식은 우리가 useState를 사용해서 직접 리렌더링 시켜주지 않고 DOM이 알아서 value를 변경 시키게 하니까 폼의 잦은 value 변경에도 불구하고 다른 요소들의 불필요한 렌더링이 일어나지 않는다.

하지만, 우리는 UI에 보이는 것 뿐만 아니라 직접 input에 있는 value를 사용해야 한다.  
따라서 useForm은 DOM에 직접적으로 접근할 수 있는 ref를 이용해서 input에 있는 value를 꺼내온다.

## 2. useForm에 넘길 수 있는 프로퍼티


### 2.1 mode

`mode`는 언제 유효성 검사를 언제 실행할지 정할 수 있다.

여기서 말하는 검사는 email 검사 같은 것일 수도 있고, 입력 범위 검사일 수도 있다.  
mode는 언제 검사를 할지만 결정할 뿐 어떤 검사를 할지는 다른 프로퍼티가 결정한다.

- 여기서 또 든 생각이 있다.
    
    ‘왜 검사 시점을 정하는 게 중요할까?’ 라는 생각이 들어서 찾아보니  
    검사를 진행한 후 이전 값과 다르면 리렌더링을 해서 검사 시점이 중요하다고 한다.
    

```jsx
useForm({
	mode: 'onSubmit'
})
```

위와 같이 코드를 작성하면 되고 mode에 들어갈 수 있는 값들은 아래와 같다.

- **onChange**
    
    input의 값이 변경될 때 마다 검사를 실행한다.  
    사실 이러면 useState처럼 입력 때 마다 리렌더링이 일어나서 성능적으로 큰 의미가 없다.  
    UX적으로 좋을 수 있지만, 성능은 떨어진다.
    
- **onBluer**
    
    input이 포커스를 잃을 때 검사를 실행한다.
    
- **onTouched**
    
    처음 bluer 이벤트가 발생했을 때 검사를 실행하며, 이후로는 change 이벤트가 발생할 때마다 매번 실행한다.  
    첫 bluer 이벤트 전까지는 onBluer이고 이후론 onChange 라고 생각하면 될 거 같다.
    
- **onSubmit**
    
    submit 이벤트가 발생할 때 검사를 실행한다.  
    mode 의 기본 값이며 mode를 따로 작성하지 않으면 onSubmit으로 동작하게 된다.
    

### 2.2 defaultValues

`defaultValues`는 폼의 기본 값을 미리 설정할 수 있다.

```jsx
useForm({
	defaultValues: {
		firstName: '',
		lastNAme: ''
	}
})
```

위와 같이 코드를 작성할 수 있고, 주의할 점은 값에 undefined를 넣으면 안된다.

### 2.3 values

`values` 는 외부요소에 의해 업데이트 되는 값이다.  
이건 controlled랑 같은 느낌이지 않을까 싶다.

```jsx
const data = fetch('/api');

useForm({
	values: data
})
```

위와 같이 코드를 작성할 수 이고, 만약 오류날 경우에 대비한 코드는 아래와 같이 작성할 수 있다.

```jsx
const { data, errors } = fetch('/api');

useForm({
	values: data,
	errors
})
```

아무래도 위부의 값을 가져와서 쓰다 보니 오류가 날 수 있기에 오류날 경우를 위해서 위와 같이 쓰인다.

## 3. useForm 반환값


### 3.1 formState

formState는 useForm이 반환해주고 폼의 상태와 관련된 정보를 formState가 반환해준다.

```jsx
const {
  formState: {
    isSubmitting,
    isSubmitSuccessful,
    isValid,
    isDirty,
    errors,
  },
} = useForm();
```

- **isSubmitting**
    
    폼 제출 진행 중인지 여부를 알 수 있다.
    
    ```jsx
    <p>{isSubmitting ? "제출 중..." : "제출"}</p>
    ```
    
- **isSubmitSuccessful**
    
    제출이 에러 없이 성공적으로 끝났는지 알 수 있다.
    
    ```jsx
    {isSubmitSuccessful && <p>제출 성공!</p>}
    ```
    
- **isValid**
    
    폼 전체가 vaildation 규칙에 맞는지 알 수 있다.
    
    ```jsx
    <button disabled={!isValid}>제출</button>
    ```
    
- **isDirty**
    
    초기 값(defaultValues)에서 값이 하나라도 변경되었는지 알 수 있다.
    
    ```jsx
    {isDirty && <p>수정된 내용이 있습니다</p>}
    ```
    
- **erros**
    
    각 필드 별로 error의 정보를 설정할 수 있다.
    
    ```jsx
    <input {...register("email", { required: "필수 입력" })} />
    {errors.email && <p>{errors.email.message}</p>}
    ```
    

### 3.2 register

register에서는 위에서 말한 validation 규칙을 설정할 수 있다.

- **required**
    
    값이 반드시 있어야 한다는 규칙을 추가한다.
    
    ```jsx
    register("email", {
      required: "이메일은 필수입니다"
    })
    ```
    
- **minLength / maxLength**
    
    문자열의 길이를 제한할 수 있다.
    
    ```jsx
    register("username", {
      minLength: {
        value: 3,
        message: "3자 이상 입력하세요"
      },
      maxLength: {
        value: 10,
        message: "10자 이하로 입력하세요"
      }
    })
    ```
    
- **pattern**
    
    정규식을 이용해서 형식 검사를 할 수 있다.
    
    ```jsx
    register("email", {
      pattern: {
        value: /^[^\s@]+@[^\s@]+\.[^\s@]+$/,
        message: "이메일 형식이 아닙니다"
      }
    })
    ```
    
- **validate**
    
    커스텀 함수를 작성해서 통과(true), 실패(false 또는 문자열)을 설정할 수 있다.
    return에 실패를 위한 문자열을 작성하면 메시지로 사용된다.
    
    ```jsx
    register("password", {
      validate: (value) => {
        return value.includes("!") || "특수문자를 포함해야 합니다"
      }
    })
    ```
    
- **valueAsNumber**
    
    숫자를 입력 받아도 문자열로 처리되다 보니 이걸 숫자로 변경해준다.
    
    ```jsx
    register("age", {
      valueAsNumber: true
    })
    ```

---
useForm을 공부하면서 느낀 게 input으로 무조건 입력을 받는다고 해서 useForm을 쓰기 보다는 상황에 따라서 useState가 더 좋은 경우도 있는 것 같다.  
특히 디바운스를 이용하면 useForm을 쓰는 이유가 사라지는 경우도 있다보니 useForm을 사용할 떈, 쓰는 이유를 생각해 봐야 할 것 같다.  
