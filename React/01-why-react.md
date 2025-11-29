# React 왜 만들어졌고, 왜 사용할까?

리액트가 만들어진 배경을 알기 위해서는 웹의 발전 과정을 먼저 이해해야 해요.

초기의 웹은 단순히 정적인 HTML 문서로만 이루어져 있었어요.
그런데 점점 “버튼 클릭 시 팝업 띄우기”, “화면 일부만 바꾸기” 같은 동적인 기능이 필요해졌죠.

그래서 JavaScript라는 언어가 등장했고, 우리는 아래와 같은 방식으로 **DOM(Document Object Model)을 직접 조작**하기 시작했어요.

```JavaScript
document.getElementById("title").innerText = "안녕하세요!";
```
이런 식으로 **“무엇을, 언제, 어떻게 바꿔라”** 라고 구체적으로 명령하는 방식을 **명령형(Imperative)** 프로그래밍이라고 해요.

## 명령형 방식의 한계와 상태의 복잡성

처음에는 이런 방식이 간단했지만, 웹이 점점 복잡해지면서 **상태(state)** 관리가 어려워졌어요.
> 예를 들어, `로그인 여부`, `장바구니에 담긴 상품 목록`, `사용자가 입력 중인 텍스트` 같은 데이터가 모두 상태(state)예요.

이 상태들이 여러 곳에서 동시에 바뀌다 보니, 화면(UI)과 데이터의 일관성이 깨지고 코드가 엉키기 시작했어요. 이런걸 스파게티 코드라고 부르죠.

jQuery는 이런 과정을 조금 더 편하게 만들어줬지만, 여전히 개발자가 **“무엇을, 어떻게 바꿀지” 직접 명령**해야 했습니다. 즉, 패러다임 자체는 명령형 그대로였던 거예요.

### 명령형 코드 예시: 모든 절차를 직접 지시해요.

아래 코드는 토글 버튼을 구현할 때 UI 변경의 모든 절차를 개발자가 직접 명령해야 하는 명령형 방식이에요.
```JavaScript
function setupImperativeUI() {
  // 1. 필요한 DOM 요소를 직접 찾아 변수에 저장
  const appDiv = document.getElementById('app');
  const button = document.getElementById('toggleButton');
  
  // 초기 상태 설정
  let isOn = false;

  // 2. 버튼 클릭 시, 상태에 따라 UI를 변경하는 구체적인 단계를 '명령'함
  button.addEventListener('click', () => {
      isOn = !isOn; // 상태 변경

      // **핵심:** if/else 문으로 텍스트, 배경색, 버튼 텍스트를 모두 직접 업데이트 명령
      if (isOn) {
          appDiv.textContent = 'ON';
          appDiv.style.backgroundColor = 'blue';
          button.textContent = '끄기'; 
      } else {
          appDiv.textContent = 'OFF';
          appDiv.style.backgroundColor = 'gray';
          button.textContent = '켜기';
      }
  });
  
  // 초기 렌더링을 위한 명령 실행 (옵션)
  appDiv.textContent = 'OFF';
  appDiv.style.backgroundColor = 'gray';
}

// DOM이 완전히 로드된 후에 실행
window.addEventListener('DOMContentLoaded', setupImperativeUI);
```
---

##  선언형(Declarative) 패러다임

페이스북 개발자들은 이런 문제를 해결하기 위해 다른 방식으로 접근했어요.
> “UI가 어떻게 바뀌는지 명령하지 말고, 지금 어떤 모습이어야 하는지 선언하자!”

이게 바로 선언형(Declarative) 패러다임이에요.

리액트는 이 철학을 기반으로 만들어졌고, UI를 재사용 가능한 조각인 **컴포넌트(Component)** 단위로 나누어 관리합니다. 
개발자는 각 컴포넌트가 특정 상태일 때 어떤 모습이어야 하는지만 선언하면, 그 상태에 따라 실제 DOM을 업데이트하는 일은 리액트가 대신 처리해요.

### 선언형 코드 예시: 결과만 선언하고 절차는 위임해요.
아래 코드는 State의 값에 따라 UI가 어떻게 보일지를 선언할 뿐, 직접 DOM을 조작하는 명령이 없어요.
```JavaScript
import React, { useState } from 'react';

function DeclarativeToggle() {
    // 1. 필요한 '상태(State)'를 정의합니다.
    const [isOn, setIsOn] = useState(false);

    // 2. 버튼 클릭 시, 상태를 변경하도록 '선언'합니다.
    const handleClick = () => {
        // "isOn을 반대로 바꿔라"고 React에게 요청합니다.
        setIsOn(!isOn);
        // **핵심:** 개발자는 이 이후의 DOM 업데이트 과정을 신경 쓰지 않아요.
    };

    // 3. 현재 State(isOn)에 기반하여 최종 UI가 '어떻게 보일지' 선언합니다.
    const displayStyle = {
        padding: '10px',
        color: 'white',
        // State 값에 따라 배경색이 결정될 것이라고 선언
        backgroundColor: isOn ? 'blue' : 'gray' 
    };
    
    return (
        <div>
            <div style={displayStyle}>
                {/* State 값에 따라 텍스트가 결정될 것이라고 선언 */}
                {isOn ? 'ON' : 'OFF'}
            </div>
            
            <button onClick={handleClick}>
                {/* State 값에 따라 버튼 텍스트가 결정될 것이라고 선언 */}
                {isOn ? '끄기' : '켜기'}
            </button>
        </div>
    );
}
export default DeclarativeToggle;
```

---

## 가상 DOM(Virtual DOM)

그렇다면 리액트는 어떻게 이런 “선언형 UI”를 효율적으로 구현할까요? 바로 **가상 DOM(Virtual DOM)** 이라는 개념 덕분이에요.

가상 DOM은 실제 브라우저의 DOM이 아니라, 메모리에 존재하는 자바스크립트 객체 형태의 트리 구조예요. 즉, 화면의 UI 상태를 가볍게 복제해 저장해 두는 거죠.

컴포넌트의 `props`나 `state`가 변경될 때마다 리액트는 새로운 가상 DOM을 만들고, **이전 가상 DOM과 비교(diffing)** 를 수행합니다. 이 과정은 마치 틀린 그림 찾기처럼 변경된 부분을 찾아내는 단계예요.

비교가 끝나면, 리액트는 변경된 부분만 모아 한 번의 DOM 접근으로 한꺼번에 업데이트합니다. 이걸 **배치 업데이트(batch update)** 라고 해요. 여러 변경 작업을 모아서 처리함으로써 DOM 접근 횟수를 최소화하는 거죠.

이는 마치 효율적으로 쇼핑하는 것과 같습니다.

필요한 물건이 생각날 때마다 마트에 가는 대신(DOM에 여러 번 접근), 집에서 **쇼핑 리스트(가상 DOM)** 를 완벽하게 작성한 뒤 마트(실제 DOM)에 단 한 번만 방문하여 모든 것을 처리하는 방식과 같아요.

### 성능보다 예측 가능한 일관성에 집중하자

사실 단 하나의 DOM 요소를 바꾸는 것만 놓고 보면, 가상 DOM을 거치지 않고 직접 변경하는 게 더 빠를 수도 있습니다.

왜냐면 가상 DOM은 **트리를 비교하는 과정(diffing)** 에서 CPU와 약간의 메모리를 소모하는 **오버헤드**가 있기 때문이에요.

하지만 리액트의 목적은 “가장 빠른 한 번의 업데이트”가 아니라 수많은 상태 변화 속에서도 일관되고 예측 가능한 UI 성능을 유지하는 것이에요.

> 즉, 리액트는 가장 빠른 길을 찾기보다
>
> “어디가 가장 덜 막히는 길인지” 알려주는 네비게이션에 가깝습니다.

DOM은 브라우저가 실제로 화면을 그리는 구조이기 때문에, 이걸 직접 조작하면 매번 **레이아웃 재계산(Layout)** 과 **다시 그리기(Paint)** 가 일어나요. 이게 바로 렌더링 병목의 핵심이에요.

리액트는 이런 복잡한 DOM 변경 과정을 대신 계산해 주고, “언제, 어떤 DOM을 바꿀지” 자동으로 결정해 줍니다.

즉, 약간의 연산(CPU)과 메모리를 사용해서 DOM 접근이라는 가장 비싼 비용을 최소화하는 거예요. 결국 리액트는 약간의 오버헤드를 감수하고, 더 큰 성능 저하를 방지하는 기술이라고 할 수 있습니다.
