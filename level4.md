모잇지에서는 사용자에게 피드백을 주는 방법 중 하나로 토스트를 사용합니다. 특히 "저장되었습니다", "복사되었습니다"와 같은 간단한 알림을 보여줄 때 토스트는 매우 효과적인 방법입니다.

이번 아티클에서는 모잇지 프론트엔드 팀에서 토스트 상태관리를 어떻게 선언적으로 구현했는지, 그리고 실제로 어떻게 사용하고 있는지 공유해드리려고 합니다.


## 선언적 프로그래밍에 대한 착각과 토스트 상태관리

많은 개발자들이 토스트 상태관리를 구현할 때 "함수로 추상화하면 선언적"이라는 착각에 빠지곤 합니다. 하지만 단순히 함수를 사용한다고 해서 무조건 선언적인 프로그래밍을 구사하는 것은 아닙니다.

### 토스트 상태관리에서의 흔한 착각

가장 간단한 예부터 살펴봅시다. 토스트를 표시하는 함수를 절차적으로 작성하면 이렇게 됩니다.

```tsx
function showToast(message: string) {
  const toastElement = document.createElement('div');
  toastElement.textContent = message;
  toastElement.style.position = 'fixed';
  toastElement.style.top = '20px';
  toastElement.style.left = '50%';
  toastElement.style.transform = 'translateX(-50%)';
  toastElement.style.backgroundColor = '#4CAF50';
  toastElement.style.color = 'white';
  toastElement.style.padding = '12px 24px';
  toastElement.style.borderRadius = '4px';
  toastElement.style.zIndex = '9999';
  
  document.body.appendChild(toastElement);
  
  setTimeout(() => {
    toastElement.style.opacity = '0';
    setTimeout(() => {
      document.body.removeChild(toastElement);
    }, 300);
  }, 3000);
}
```

위 코드는 함수를 사용하고는 있지만 "먼저 DOM 요소를 생성하고, 그 다음 스타일을 적용하고, 마지막에 타이머를 설정한다"는 시간적 순서에 집중하고 있습니다. 즉, 함수를 사용했다고 해서 절차적 사고에서 벗어난 것이 아닙니다.

선언적인 토스트 상태관리는 동작의 시간적 순서에서 벗어나 상태 간의 관계를 기술하는 것에 집중해야 합니다.

### 토스트에서의 "어떻게" vs "무엇"

토스트 상태관리에서 핵심 구분 기준은 명확합니다. 절차적 코드는 "어떻게(How) 단계별로 토스트를 표시할 것인가"에 집중하고, 선언적 코드는 "무엇을(What) 원하는 상태 관계인가"에 집중합니다.

```tsx
// 절차적 사고 - "어떻게"에 집중
function handleSave() {
  // 1단계: 데이터 저장
  saveData();
  
  // 2단계: 토스트 메시지 설정
  setToastMessage('저장되었습니다!');
  
  // 3단계: 토스트 표시
  setIsToastVisible(true);
  
  // 4단계: 타이머 설정
  setTimeout(() => {
    setIsToastVisible(false);
  }, 3000);
}

// 선언적 사고 - "무엇"에 집중
function handleSave() {
  saveData();
  showToast('저장되었습니다!'); // "저장 완료 → 토스트 표시" 관계 선언
}
```

모잇지는 이렇게 동작에 집중하여 추상화된 토스트 상태관리를 선언적인 코드로 생각하고 있습니다.

여기에서 한 걸음 더 나아가서 `showToast` 함수 내부의 상태 업데이트를 살펴봅시다.

```tsx
const showToast = useCallback((newMessage: string) => {
  setMessage(newMessage);
  setIsVisible(true);
}, []);
```

이 상태 업데이트도 선언적인 코드로 볼 수 있습니다. 메시지 설정과 가시성 변경의 관계를 추상화하고 있기 때문입니다.

실제로 React의 상태 업데이트가 추상화하는 로직을 그대로 드러내면 아래와 같이 나타낼 수 있습니다.

```tsx
// React 상태 업데이트의 내부 로직 (단순화)
const setState = (newState) => {
  // 1. 이전 상태와 새 상태 비교
  // 2. 변경사항 감지
  // 3. 리렌더링 스케줄링
  // 4. 컴포넌트 트리 업데이트
  // 5. DOM 조작
};
```

위와 같이, `setState`는 상태 비교, 변경사항 감지, 리렌더링 스케줄링 등의 작업을 "상태 업데이트"로 추상화합니다. 이런 관점에서 봤을 때, `setState`는 선언적인 코드입니다.

코드의 관점을 벗어나면 보다 재미있는 예시를 생각할 수 있습니다.

"토스트를 3초간 보여라"라고 하는 말을 생각합시다. 여기에서

* "3초"는 "1초의 3배만큼의 시간"을 추상화한 것입니다.
* "보여라"는 "가시성을 true로 설정하고, UI를 렌더링하고, 애니메이션을 실행한다"를 추상화한 것입니다.
* "토스트"는 "특정 위치에 특정 스타일을 가진 UI 컴포넌트"를 추상화한 것입니다.

그래서 "토스트를 3초간 보여라"는 사실 "1초의 3배만큼의 시간 동안, 특정 위치에 특정 스타일을 가진 UI 컴포넌트의 가시성을 true로 설정하고, UI를 렌더링하고, 애니메이션을 실행해라"라는 말을 추상화한, 선언적인 말로 볼 수 있을 것입니다.

## Before와 After 비교

토스트 상태관리를 도입하기 전과 후의 코드를 비교해보면, 선언적 추상화의 효과를 더 명확하게 볼 수 있습니다.

### Before: 명령적인 토스트 구현

토스트 상태관리를 도입하기 전에는 각 컴포넌트마다 토스트 상태를 직접 관리해야 했습니다.

```tsx
// 각 컴포넌트마다 반복되는 토스트 상태 관리
function SaveButton() {
  const [isToastVisible, setIsToastVisible] = useState(false);
  const [toastMessage, setToastMessage] = useState('');

  const handleSave = () => {
    // 저장 로직
    saveData();
    
    // 토스트 표시 로직 (반복됨)
    setToastMessage('저장되었습니다!');
    setIsToastVisible(true);
    
    // 자동 숨김 타이머 (반복됨)
    setTimeout(() => {
      setIsToastVisible(false);
    }, 3000);
  };

  return (
    <>
      <button onClick={handleSave}>저장</button>
      
      {/* 토스트 UI도 각각 구현 (반복됨) */}
      {isToastVisible && (
        <div style={{
          position: 'fixed',
          top: '20px',
          left: '50%',
          transform: 'translateX(-50%)',
          backgroundColor: '#4CAF50',
          color: 'white',
          padding: '12px 24px',
          borderRadius: '4px',
          zIndex: 9999,
          animation: 'slideDown 3s ease-in-out forwards'
        }}>
          {toastMessage}
        </div>
      )}
    </>
  );
}
```

위 코드에서는 토스트를 사용할 때마다 다음과 같은 문제들이 발생했습니다:

1. **중복 코드**: 각 컴포넌트마다 동일한 상태 관리 로직이 반복됩니다.
2. **일관성 부족**: 토스트 스타일이나 동작이 컴포넌트마다 다를 수 있습니다.
3. **유지보수 어려움**: 토스트 동작을 변경하려면 모든 컴포넌트를 수정해야 합니다.
4. **복잡성 증가**: 토스트 표시를 원하는 컴포넌트가 복잡해집니다.

### After: 선언적인 토스트 상태관리

토스트 상태관리를 도입한 후에는 코드가 훨씬 간단하고 선언적으로 변했습니다.

```tsx
// 토스트 상태관리 Hook
function useToastState() {
  const [isVisible, setIsVisible] = useState(false);
  const [message, setMessage] = useState('');

  const showToast = useCallback((newMessage: string) => {
    setMessage(newMessage);
    setIsVisible(true);
  }, []);

  const hideToast = useCallback(() => {
    setIsVisible(false);
  }, []);

  useEffect(() => {
    if (isVisible) {
      const timer = setTimeout(() => {
        hideToast();
      }, 3000);
      return () => clearTimeout(timer);
    }
  }, [isVisible, hideToast]);

  return { isVisible, message, showToast, hideToast };
}

// 사용하는 컴포넌트들 (매우 간단해짐)
function SaveButton() {
  const { showToast } = useToastActionsContext();

  const handleSave = () => {
    saveData();
    showToast('저장되었습니다!');  // 단 한 줄!
  };

  return <button onClick={handleSave}>저장</button>;
}
```

선언적인 토스트 상태관리를 도입한 후에는 다음과 같은 개선사항들을 얻을 수 있었습니다:

1. **코드 중복 제거**: 토스트 관련 로직이 한 곳에 집중되어 중복이 사라졌습니다.
2. **일관성 보장**: 모든 토스트가 동일한 스타일과 동작을 가집니다.
3. **유지보수 용이**: 토스트 동작을 변경하려면 한 곳만 수정하면 됩니다.
4. **간단한 사용법**: 토스트를 표시하려면 `showToast('메시지')` 한 줄만 작성하면 됩니다.

## Context를 통한 상태와 액션 분리

모잇지에서는 토스트의 상태와 액션을 Context로 분리하여 관리하고 있습니다.

```tsx
// 상태만 분리 (읽기 전용)
const ToastStateContext = createContext({
  isVisible: boolean;
  message: string;
});

// 액션만 분리 (함수들)
const ToastActionsContext = createContext({
  showToast: (message: string) => void;
  hideToast: () => void;
});
```

**이렇게 분리한 이유는 무엇일까요?**

먼저 토스트를 여러 곳에서 사용한다면 각각의 토스트를 중복해서 개발할 필요 없이 한 번만 개발하면 되기 때문에 효율적일 것입니다. 또한 토스트의 동작이 변경된다고 하더라도, 한 곳에서만 바꾸면 다른 화면들에 모두 반영되기 때문에 빨리 수정할 수 있을 것입니다.

**수정하기 어려운 지점은 없을까요?**

화면마다 토스트가 조금씩 다르다면, 공통화된 것이 오히려 코드의 복잡함을 가져올 수도 있습니다. 예를 들어서, 어떤 페이지에서는 토스트의 위치가 다르거나, 애니메이션이 다를 수 있습니다. 또, 다른 페이지에서는 토스트가 자동으로 사라지는 시간이 다를 수 있습니다.

아래와 같이 `ToastProvider`에서 바뀔 수 있는 부분이 많다면, 내부 구현과 인터페이스도 복잡해지고, 쓰는 쪽에서도 불편할 것입니다.

```tsx
<ToastProvider
  duration={5000}
  position="top-right"
  animation="slide-in"
  backgroundColor="#4CAF50"
  textColor="#FFFFFF"
  /* 많은 옵션들 ... */
  onShow={/* ... */}
  onHide={/* ... */}
>
  {children}
</ToastProvider>
```

이처럼 모잇지에서는 선언적인 토스트 상태관리가 항상 좋은 것이 아니라, 앞으로 제품이 어떻게 변화할지, 비즈니스 요구사항이 어떻게 되는지에 따라서 달라질 수 있다고 생각하고 있습니다. 앞으로 토스트의 어떤 부분이 수정될지 예측하고, 이에 따라 적절한 추상화 레벨을 따르는 코드를 작성할 필요가 있습니다.

## 현실적인 개선 방향

현재 토스트 상태관리를 더 선언적으로 개선할 수 있는 현실적인 방법들을 살펴봅시다.

### 1단계: 타입 안전성 강화

현재 코드를 기반으로 한 점진적 개선부터 시작해봅시다.

```tsx
// 현재 코드 개선 - 1단계
interface ToastState {
  isVisible: boolean;
  message: string;
}

const useToastState = () => {
  const [toastState, setToastState] = useState<ToastState>({
    isVisible: false,
    message: '',
  });

  const showToast = useCallback((message: string) => {
    setToastState({ isVisible: true, message });
  }, []);

  const hideToast = useCallback(() => {
    setToastState({ isVisible: false, message: '' });
  }, []);

  useEffect(() => {
    if (toastState.isVisible) {
      const timer = setTimeout(hideToast, 3000);
      return () => clearTimeout(timer);
    }
  }, [toastState.isVisible, hideToast]);

  return { ...toastState, showToast, hideToast };
};
```

### 2단계: 비즈니스 의도가 명확한 API

```tsx
// 2단계: 비즈니스 메서드 추가
const useToast = () => {
  const { showToast } = useToastActionsContext();
  
  return {
    showSuccess: (message: string) => showToast(message),
    showError: (message: string) => showToast(message),
    showInfo: (message: string) => showToast(message),
  };
};

// 사용할 때 더 명확한 의도 표현
function SaveButton() {
  const { showSuccess, showError } = useToast();

  const handleSave = async () => {
    try {
      await saveData();
      showSuccess('저장되었습니다!');  // 의도가 명확
    } catch (error) {
      showError('저장에 실패했습니다.');  // 의도가 명확
    }
  };

  return <button onClick={handleSave}>저장</button>;
}
```

### 3단계: 선택적 상태 모델링 (고급)

필요한 경우에만 더 엄격한 상태 모델링을 적용할 수 있습니다.

```tsx
// 3단계: 선택적 적용 - 복잡한 토스트가 필요한 경우에만
type StrictToastState = 
  | { type: 'hidden' }
  | { type: 'visible'; message: string };

const useStrictToastState = () => {
  const [state, setState] = useState<StrictToastState>({ type: 'hidden' });

  const showToast = useCallback((message: string) => {
    setState({ type: 'visible', message });
  }, []);

  const hideToast = useCallback(() => {
    setState({ type: 'hidden' });
  }, []);

  return {
    isVisible: state.type === 'visible',
    message: state.type === 'visible' ? state.message : '',
    showToast,
    hideToast,
  };
};
```

## 선언적 사고의 본질

선언적 프로그래밍의 핵심은 "상태에서 관계로"의 사고 전환에 있습니다.

### 관계적 사고의 예시

토스트 상태관리에서 관계적 사고를 적용하면 다음과 같습니다:

```tsx
// 절차적 사고: 시간적 순서에 집중
const handleUserAction = () => {
  // 1. 액션 실행
  performAction();
  // 2. 성공 여부 확인
  if (actionSucceeded) {
    // 3. 토스트 표시
    showToast('성공했습니다!');
  }
};

// 관계적 사고: 상태 간 관계에 집중
const handleUserAction = () => {
  performAction();
  // "성공 상태 → 토스트 표시" 관계를 선언
  showToast('성공했습니다!');
};
```

### 상태 모델링의 철학

토스트 상태관리에서 상태 모델링은 다음과 같은 철학을 따릅니다:

1. **가능한 상태를 명시적으로 선언**: 토스트는 "숨김" 또는 "표시됨" 상태만 존재
2. **불가능한 상태를 원천 차단**: "표시됨"이면서 "메시지 없음" 상태는 존재하지 않음
3. **상태 전환을 관계로 표현**: "표시됨" 상태는 3초 후 "숨김" 상태로 자동 전환

```tsx
// 상태 관계를 선언적으로 표현
const useToastState = () => {
  const [state, setState] = useState<'hidden' | { type: 'visible'; message: string }>('hidden');

  const showToast = useCallback((message: string) => {
    // 상태 전환 관계 선언
    setState({ type: 'visible', message });
  }, []);

  const hideToast = useCallback(() => {
    // 상태 전환 관계 선언
    setState('hidden');
  }, []);

  // 자동 전환 관계 선언
  useEffect(() => {
    if (state !== 'hidden') {
      const timer = setTimeout(() => setState('hidden'), 3000);
      return () => clearTimeout(timer);
    }
  }, [state]);

  return {
    isVisible: state !== 'hidden',
    message: state !== 'hidden' ? state.message : '',
    showToast,
    hideToast,
  };
};
```

## 마치며

토스트 상태관리에서도 선언적 프로그래밍은 도구가 아니라 사고방식이며, 문법이 아니라 철학입니다. 이 철학을 이해하고 실무에 적용할 때, 우리는 더 나은 토스트 상태관리를 구현할 수 있게 됩니다.

이렇게 모잇지에서는 토스트 상태관리를 선언적으로 추상화하여 개발자가 토스트의 복잡한 내부 구현을 신경 쓰지 않고, 단순히 "무엇을 보여줄지"에만 집중할 수 있도록 만들었습니다.
