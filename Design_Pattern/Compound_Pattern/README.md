# Compound Pattern
단일 작업을 수행하는데에 함께 동작하는 여러 컴포넌트들을 생성합니다.

---

시스템에서 서로 속하는 컴포넌트들이 자주 존재합니다. 그것들은 상태와 로직을 함께 공유하면서 서로 의존적입니다. 메뉴, 드롭다운, 셀렉트 컴포넌트에서 주로 볼 수 있습니다. Compound component pattern은 작업을 수행하는데 함께 동작하는 컴포넌트들을 생성하는데 도움이 됩니다.

---

## Context API

예시를 봅시다. 다람쥐 이미지들 목록이 있습니다. 다람쥐 이미지들을 보여주는 것 외에도, 이미지를 수정하거나 삭제할 수 있도록 버튼을 추가하고 싶습니다. `FlyOut` 컴포넌트를 구현하여 사용자가 컴포넌트를 토글할 때 목록을 보여줄 수 있습니다.

```js
import React from "react";
import Icon from "./Icon";

const FlyOutContext = React.createContext();

export function FlyOut(props) {
  const [open, toggle] = React.useState(false);

  return (
    <div className={`flyout`}>
      <FlyOutContext.Provider value={{ open, toggle }}>
        {props.children}
      </FlyOutContext.Provider>
    </div>
  );
}

function Toggle() {
  const { open, toggle } = React.useContext(FlyOutContext);

  return (
    <div className="flyout-btn" onClick={() => toggle(!open)}>
      <Icon />
    </div>
  );
}

function List({ children }) {
  const { open } = React.useContext(FlyOutContext);
  return open && <ul className="flyout-list">{children}</ul>;
}

function Item({ children }) {
  return <li className="flyout-item">{children}</li>;
}

FlyOut.Toggle = Toggle;
FlyOut.List = List;
FlyOut.Item = Item;
```

`FlyOut` 컴포넌트 내부에서, 기본적으로 세 가지 컴포넌트가 존재합니다.

- 토글 버튼과 목록이 있는 `FlyOut` wrapper
- `List`를 토글하는 Toggle button
- 메뉴 아이템의 목록을 포함하는 `List`

React의 Context API와 Compound component pattern을 함께 사용하면 이 예제를 완벽하게 구현할 수 있습니다!

우선, `FlyOut` 컴포넌트를 생성합니다. 이 컴포넌트는 **상태**를 가지고 있고, 내부 모든 **children**에게 toggle 값을 줄 수 있는 `FlyOutProvider`를 반환합니다.

```js
const FlyOutContext = createContext();

function FlyOut(props) {
  const [open, toggle] = useState(false);

  const providerValue = { open, toggle };

  return (
    <FlyOutContext.Provider value={providerValue}>
      {props.children}
    </FlyOutContext.Provider>
  );
}
```

상태가 있는 `FlyOut` 컴포넌트는 이제 자식 컴포넌트들에게 `open`과 `toggle` 값을 전달할 수 있습니다.

`Toggle` 컴포넌트를 생성합니다. 이 컴포넌트는 사용자가 메뉴를 토글할 수 있도록 클릭할 수 있도록 단순히 컴포넌트를 렌더링합니다.

```js
function Toggle() {
  const { open, toggle } = useContext(FlyOutContext);

  return (
    <div onClick={() => toggle(!open)}>
      <Icon />
    </div>
  );
}
```

`Toggle`이 `FlyOutContext` provider에 실제로 접근할 수 있도록 하기 위해, `FlyOut`의 자식 컴포넌트로 `Toggle` 컴포넌트를 렌더링해야 합니다! 단순하게 이것을 자식 컴포넌트로 구현할 수 있지만, `Toggle` 컴포넌트를 `FlyOut` 컴포넌트의 속성으로 할당하여 구현할 수도 있습니다!

```js
const FlyOutContext = createContext();

function FlyOut(props) {
  const [open, toggle] = useState(false);

  return (
    <FlyOutContext.Provider value={{ open, toggle }}>
      {props.children}
    </FlyOutContext.Provider>
  );
}

function Toggle() {
  const { open, toggle } = useContext(FlyOutContext);

  return (
    <div onClick={() => toggle(!open)}>
      <Icon />
    </div>
  );
}

FlyOut.Toggle = Toggle;
```

즉, 다른 파일에서 `FlyOut` 컴포넌트를 사용하고 싶을 때, `FlyOut`을 import하기만 하면 됩니다!

```js
import React from "react";
import { FlyOut } from "./FlyOut";

export default function FlyoutMenu() {
  return (
    <FlyOut>
      <FlyOut.Toggle />
    </FlyOut>
  );
}
```

Just a toggle is not enough. We also need to have a List with list items, which open and close based on the value of open.

토글 하나만으로는 충분하지 않습니다. `open`값에 따라 열고 닫는 `List`가 필요합니다.

```js
function List({ children }) {
  const { open } = React.useContext(FlyOutContext);
  return open && <ul>{children}</ul>;
}

function Item({ children }) {
  return <li>{children}</li>;
}
```

`List` 컴포넌트는 `open`의 true/false 여부에 따라 자식 컴포넌트를 렌더링합니다. `Toggle`와 마찬가지로 `List`와 `Item`을 `FlyOut`컴포넌틐의 속성으로 정의합니다. 

```js
const FlyOutContext = createContext();

function FlyOut(props) {
  const [open, toggle] = useState(false);

  return (
    <FlyOutContext.Provider value={{ open, toggle }}>
      {props.children}
    </FlyOutContext.Provider>
  );
}

function Toggle() {
  const { open, toggle } = useContext(FlyOutContext);

  return (
    <div onClick={() => toggle(!open)}>
      <Icon />
    </div>
  );
}

function List({ children }) {
  const { open } = useContext(FlyOutContext);
  return open && <ul>{children}</ul>;
}

function Item({ children }) {
  return <li>{children}</li>;
}

FlyOut.Toggle = Toggle;
FlyOut.List = List;
FlyOut.Item = Item;
```

이제 `List`와 `Item` 또한 `FlyOut`의 속성으로 사용할 수 있습니다! 이 경우, 사용자에게 **Edit**과 **Delete**,  두 가지 선택사항을 보여주어야 합니다. 두 개의 `FlyOut.Item`을 렌더링하는 `FlyOut.List`를 구현하여 하나는 **Edit**으로, 다른 하나는 **Delete** 옵션으로 사용합니다.

```js
import React from "react";
import { FlyOut } from "./FlyOut";

export default function FlyoutMenu() {
  return (
    <FlyOut>
      <FlyOut.Toggle />
      <FlyOut.List>
        <FlyOut.Item>Edit</FlyOut.Item>
        <FlyOut.Item>Delete</FlyOut.Item>
      </FlyOut.List>
    </FlyOut>
  );
}
```

좋습니다! `FlyOutMenu` 자체에 어떠한 상태값도 추가하지 않고 전체 `FlyOut` 컴포넌트를 구현하였습니다.

Compound pattern은 컴포넌트 라이브러리를 설계할 때 유용합니다. [Semantic UI](https://react.semantic-ui.com/modules/dropdown/#types-dropdown)와 같은 UI 라이브러리들을 사용할 때 이 패턴을 자주 볼 수 있습니다.

## [React.Children.map](https://reactjs.org/docs/react-api.html#reactchildrenmap)

컴포넌트에 children을 매핑해서 Compound Component pattern을 직접 구현할 수도 있습니다. 추가 props에 `open`과 `toggle` 속성을 복제함으로써 해당 엘리먼트들에 추가할 수 있습니다.

```js
export function FlyOut(props) {
  const [open, toggle] = React.useState(false);

  return (
    <div>
      {React.Children.map(props.children, child =>
        React.cloneElement(child, { open, toggle })
      )}
    </div>
  );
}
```

모든 children 컴포넌트들은 복제되어 `open`과 `toggle` 값을 받습니다. 이전 예제처럼 Context API를 사용할 필요 없이도 props로 해당 값들에 접근할 수 있습니다.

```js
export function FlyOut(props) {
  const [open, toggle] = React.useState(false);

  return (
    <div className={`flyout`}>
      {React.Children.map(props.children, child =>
        React.cloneElement(child, { open, toggle })
      )}
    </div>
  );
}
```

## Pros

Compound component들은 내부에 자신만의 상태를 관리하고, 이것들을 몇몇 자식 컴포넌트들 사이에서 공유할 수 있습니다. compound component를 구현할 때, 우리가 직접 상태를 관리할 필요가 없습니다.

compound component를 import할 때, 해당 컴포넌트에서 사용할 수 있는 자식 컴포넌트들을 명시적으로 import할 필요 없습니다.

```js
import { FlyOut } from "./FlyOut";

export default function FlyoutMenu() {
  return (
    <FlyOut>
      <FlyOut.Toggle />
      <FlyOut.List>
        <FlyOut.Item>Edit</FlyOut.Item>
        <FlyOut.Item>Delete</FlyOut.Item>
      </FlyOut.List>
    </FlyOut>
  );
}
```

## Cons

`React.Children.map`을 사용하여 값들을 주입하면, 컴포넌트 중첩이 제한적입니다. 부모 컴포넌트에 직접적으로 설정한 자식 컴포넌트들만이 `open`과 `toggle` props를 사용할 수 있습니다. 즉, 다른 컴포넌트로 해당 컴포넌트들을 감쌀 수 없습니다.

```js
export default function FlyoutMenu() {
  return (
    <FlyOut>
      {/* This breaks */}
      <div>
        <FlyOut.Toggle />
        <FlyOut.List>
          <FlyOut.Item>Edit</FlyOut.Item>
          <FlyOut.Item>Delete</FlyOut.Item>
        </FlyOut.List>
      </div>
    </FlyOut>
  );
}
```

`React.cloneElement`로 엘리먼트를 복제하는 것은 shallow merge입니다. 전달한 새로운 props와 기존에 이미 존재하던 props는 합쳐집니다. 만약에 `React.cloneElement` 메소드로 전달하는 props명과 기존에 존재하던 props명이 같다면 결국 이름 충돌이 발생합니다. props가 얕게 병합되었으므로, 해당 prop의 값이 전달한 최근 값으로 덮어쓰여집니다.