# HOC Pattern
재사용가능한 로직을 어플리케이션 전반에 걸쳐 컴포넌트에 props로 내려주기

<br />

어플리케이션에서 여러 개의 컴포넌트에 같은 로직을 사용하고 싶을 때가 종종 있습니다. 이 로직은 컴포넌트에 특정 스타일을 적용하거나, 인증이 필요하거나 전역 상태를 추가하는 기능이 포함될 수 있습니다.

여러 개의 컴포넌트에 같은 로직을 재사용할 수 있는 한 가지 방법은 **고차 컴포넌트** 패턴을 사용하는 것입니다. 이 패턴으로 어플리케이션 전반에 걸쳐 컴포넌트 로직을 재사용할 수 있습니다.

Higher Order Component (HOC)는 다른 컴포넌트를 받는 컴포넌트입니다. HOC는 인자로 전달한 컴포넌트에 적용할 어떤 로직을 가집니다. 로직을 적용한 후에, HOC는 다른 로직을 가진 요소를 반환합니다.

어플리케이션 내부의 여러 컴포넌트에 특정 스타일을 추가하고 싶다고 가정합시다. `style` 객체를 매번 만드는 대신에, `style` 객체를 전달한 컴포넌트에 추가하는 HOC를 만들 수 있습니다.

```js
function withStyles(Component) {
  return props => {
    const style = { padding: '0.2rem', margin: '1rem' }
    return <Component style={style} {...props} />
  }
}

const Button = () = <button>Click me!</button>
const Text = () => <p>Hello World!</p>

const StyledButton = withStyles(Button)
const StyedText = withStyles(Text)
```

`Button`과 `Text` 컴포넌트의 수정 버전인 `StyledButton과` `StyledText` 컴포넌트를 만들었습니다. 이들은 `withStyles` HOC 내부에서 추가한 스타일을 포함합니다.

Container/Presentational 패턴에서 사용했던 `DogImages` 예시를 봅시다. API로부터 가져온 dog 이미지 리스트를 렌더링하는 것 외에는 아무것도 하지 않습니다.

```js
import React from "react";
import { render } from "react-dom";

import useDogImages from "./useDogImages";
import "./styles.css";

export default function DogImages() {
  const dogs = useDogImages();

  return dogs.map((dog, i) => <img src={dog} key={i} alt="Dog" />);
}

function App() {
  return (
    <div className="App">
      <h1>
        Browse Dog Images{" "}
        <span role="img" aria-label="emoji">
          🐕
        </span>
      </h1>
      <DogImages />
    </div>
  );
}

render(<App />, document.getElementById("root"));
```

사용자 경험을 조금 향상시켜봅시다. 데이터를 패치할 동안, 화면에 "Loading..." 문구를 보여주려 합니다. `DogImages` 컴포넌트에 바로 데이터를 추가하는 대신에 로딩 로직을 추가하는 고차 컴포넌트를 사용할 수 있습니다.

`withLoader` HOC를 생성합니다. HOC는 컴포넌트를 인자로 받고, 컴포넌트를 반환해야 합니다. 이 경우, `withLoader` HOC는 데이터 패치가 완료될 떄까지 `Loading…`을 보여주어야 하는 요소를 받아야 합니다.

사용할 `withLoader` HOC의 최소 버전을 만들어봅시다!

```js
function withLoader(Element) {
  return props => <Element />;
}
```
이 경우, HOC는 데이터를 패치할 동안 `Loading…` 문구를 보여주는 로직이 추가된 함수형 컴포넌트 `props => {}` 요소를 반환합니다. 데이터가 패치되고 나면, prop으로 패치된 데이터를 전달해야 합니다.

```js
import React, { useEffect, useState } from "react";

export default function withLoader(Element, url) {
  return (props) => {
    const [data, setData] = useState(null);

    useEffect(() => {
      async function getData() {
        const res = await fetch(url);
        const data = await res.json();
        setData(data);
      }

      getData();
    }, []);

    if (!data) {
      return <div>Loading...</div>;
    }

    return <Element {...props} data={data} />;
  };
}
```

완벽합니다! 컴포넌트와 url을 인자로 가지는 HOC를 생성했습니다.

1. `useEffect` hook에서, `withLoader` HOC는 `url` 인자로 전달한 API 엔드포인트로부터 데이터를 패치합니다. 데이터가 아직 반환되지 않을 동안 `Loading...` 문구를 포함한 요소를 반환합니다.

2. 데이터가 패치되고 나면, 패치된 데이터를 `data`에 설정합니다. `data`가 더이상 `null`이 아니므로 HOC의 인자로 준 요소를 보여줍니다.

이제, 이 행동을 어떻게 어플리케이션에 적용하여 `Loading...` 표시를 `DogImages` 리스트에 실제로 보여줄 수 있을까요?

`DogImages.js` 에서, 더이상 일반 `DogImages` 컴포넌트를 export하지 않고, `withLoading` HOC로 "감싼" `DogImages` 컴포넌트를 export해야 합니다.

```js
export default withLoading(DogImages);
```

`withLoader` HOC는 또한 url이 데이터를 가져올 엔드포인트를 알 것으로 예상합니다. 이 경우 우리는 Dog API endpoint를 추가하고 싶습니다.

```js
export default withLoader(
  DogImages,
  "https://dog.ceo/api/breed/labrador/images/random/6"
);
```

`withLoader` HOC가 예시에서는 `DogImages`에 해당하는 별도의 `data` props를 가진 요소를 반환하므로 `DogImages` 컴포넌트의 `data` prop에 접근할 수 있습니다.

```js
const [data, setData] = useState(null);

// ...중략...

return <Element {...props} data={data} />;
```

완벽합니다! 이제 데이터가 패치될동안 `Loading...`을 화면에서 볼 수 있습니다.

Higher Order Component 패턴은 단일 공간에 모든 로직을 유지하면서 여러 컴포넌트에 동일한 로직을 제공할 수 있습니다. `withLoader` HOC는 인자로 받은 컴포넌트나 url을 신경쓰지 않습니다. 유효한 컴포넌트와 API 엔드포인트라면, API 엔드포인트에서 컴포넌트로 데이터를 전달할 뿐입니다.

<br />

## Composing

또한 여러 고차 컴포넌트들을 구성할 수도 있습니다. `DogImages` 리스트에 hover했을 때, `Hovering!` 텍스트 영역을 보여주는 기능을 추가해봅시다.

HOC의 인자로 전달한 요소에 `hovering` prop을 제공하는 HOC를 만들어야 합니다. 해당 prop에 따라, 사용자가 `DogImages` 리스트에 hover 했는지에 따라 조건적으로 텍스트 영역을 렌더링할 수 있습니다.

이제 `withHover` HOC를 `withLoader` HOC에 감쌉니다.

```js
export default withHover(
  withLoader(DogImages, "https://dog.ceo/api/breed/labrador/images/random/6")
);
```

`DogImages` 요소는 이제 `withHover`와 `withLoader`가 전달한 모든 props를 가집니다. 이제 `hovering` prop이 `true` 혹은 `false`인지에 따라 조건적으로 `Hovering!` 텍스트 영역을 렌더링할 수 있습니다.

> A well-known library used for composing HOCs is [recompose](https://github.com/acdlite/recompose). Since HOCs can largely be replaced by React Hooks, the recompose library is no longer maintained, thus won't be covered in this article.

<br />

## Hooks

어떤 경우에는, HOC 패턴을 React Hooks으로 대체할 수 있습니다.

`withHover` HOC를 `useHover` hook으로 바꿔봅시다. 고차 컴포넌트를 갖는 대신, `mouseOver`와 `mouseLeave`이벤트 리스너를 요소에 추가하는 hook을 export합니다. 더이상 HOC에서 그랬던 것처럼 요소를 전달하지 않아도 됩니다. 그 대신, hook에서 `ref`를 반환하여 `mouseOver`와 `mouseLeave` 이벤트를 얻도록 할 것입니다.

```js
import { useState, useRef, useEffect } from "react";

export default function useHover() {
  const [hovering, setHover] = useState(false);
  const ref = useRef(null);

  const handleMouseOver = () => setHover(true);
  const handleMouseOut = () => setHover(false);

  useEffect(() => {
    const node = ref.current;
    if (node) {
      node.addEventListener("mouseover", handleMouseOver);
      node.addEventListener("mouseout", handleMouseOut);

      return () => {
        node.removeEventListener("mouseover", handleMouseOver);
        node.removeEventListener("mouseout", handleMouseOut);
      };
    }
  }, [ref.current]);

  return [ref, hovering];
}
```

`useEffect` hook은 컴포넌트에 이벤트 리스너를 추가하고, 사용자가 현재 요소에 hover했는지에 따라 `hovering` 값을 `true` 혹은 `false`로 설정합니다. `ref`와 `hovering`는 hook에서 반환되어야 합니다. `ref`는 컴포넌트에 ref를 추가하여 `mouseOver`와 `mouseLeave`를 받기 위해, `hovering`은 `Hovering!` 텍스트 영역을 조건부로 렌더링하기 위해서 반환합니다.

`withHover` HOC로 `DogImages` 컴포넌트를 감싸는 대신, `useHover` hook을 `DogImages` 컴포넌트 내부에 바로 사용할 수 있습니다.

```js
function DogImages(props) {
  const [hoverRef, hovering] = useHover();

  return (
    <div ref={hoverRef} {...props}>
      {hovering && <div id="hover">Hovering!</div>}
      <div id="list">
        {props.data.message.map((dog, index) => (
          <img src={dog} alt="Dog" key={index} />
        ))}
      </div>
    </div>
  );
}
```

완벽합니다! `withHover` 컴포넌트를 `DogImages`로 감싸지 않고 컴포넌트 내부에  `useHover` hook을 사용할 수 있습니다.

---

일반적으로, React Hooks는 HOC 패턴을 대체하지는 않습니다.

> "In most cases, Hooks will be sufficient and can help reduce nesting in your tree." - [React Docs](https://reactjs.org/docs/hooks-faq.html#do-hooks-replace-render-props-and-higher-order-components)

React docs가 언급한대로, Hooks을 사용하는 것은 컴포넌트 트리의 깊이를 줄일 수 있습니다. HOC 패턴을 사용하면, 깊게 증첩된 컴포넌트 트리 구조가 되기 쉽상입니다.

```js
<withAuth>
  <withLayout>
    <withLogging>
      <Component />
    </withLogging>
  </withLayout>
</withAuth>
```

Hook을 컴포넌트에 바로 추가하면, 더이상 컴포넌트를 감쌀 필요가 없습니다.

고차 컴포넌트는 단일 공간에 모든 로직을 가진채로 다수 컴포넌트에 같은 로직을 쉽게 제공합니다. Hooks은 컴포넌트 내부에 커스텀 행동을 추가하는 것을 가능케 합니다. 하지만 HOC 패턴에 비해 여러 컴포넌트들이 Hook이 제공하는 행동에 의존할 경우, 버그가 발생할 위험이 증가됩니다.

### Best use-cases for a HOC:

- 동일하지만 사용자 맞춤형이 아닌 행동을 다수의 컴포넌트에 사용해야할 때 유용합니다.

- 별도의 맞춤형 로직 없이, 컴포넌트가 혼자서 동작할 수 있습니다.
### Best use-cases for Hooks:

- 각 컴포넌트마다 행동을 커스터마이징해야 할 때 유용합니다.
- 그 행동이 어플리케이션 전반에 걸쳐 퍼져나가지 않으며, 단 하나(혹은 몇몇 컴포넌트)에 한해 사용할 때 유용합니다.
- 그 행동이 컴포넌트에 많은 props를 추가할 때 유용합니다.

<br />

## Case Study

HOC 패턴에 의존하는 어떤 라이브러리들은 출시 후에 Hooks 지원을 추가하였습니다. [Apollo Client](https://www.apollographql.com/docs/react/)가 바로 그 예시입니다.

> "No experience with Apollo Client is needed to understand this example."

Apollo Client를 사용하는 한가지 방법은 `graphql()` 고차 컴포넌트를 사용하는 것입니다.

```js
import React from "react";
import "./styles.css";

import { graphql } from "react-apollo";
import { gql } from "apollo-boost";

const ADD_MESSAGE = gql`
  mutation AddMessage($message: String!) {
    addMessage(message: $message) {
      message
    }
  }
`;

class Input extends React.Component {
  constructor() {
    super();
    this.state = { message: "" };
  }

  handleChange = (e) => {
    this.setState({ message: e.target.value });
  };

  handleClick = () => {
    this.props.mutate({ variables: { message: this.state.message } });
  };

  render() {
    return (
      <div className="input-row">
        <input
          onChange={this.handleChange}
          type="text"
          placeholder="Type something..."
        />
        <button onClick={this.handleClick}>Add</button>
      </div>
    );
  }
}

export default graphql(ADD_MESSAGE)(Input);
```

`graphql()` HOC로, 고차 컴포넌트로 감싼 컴포넌트에서 사용할 수 있는 데이터를 클라이언트로부터 생성할 수 있습니다! 여전히 `graphql()` HOC를 사용할 수는 있지만, 몇가지 단점이 존재합니다.

컴포넌트가 여러 resolver들에 접근해야 할 때, 여러 `graphql()` 고차 컴포넌트를 구성해야합니다. 여러 개의 HOC들을 구성하는 것은 데이터가 어떻게 컴포넌트로 전달되는지 이해하기 어렵게 만듭니다. 이 경우, HOC의 순서가 중요한데, 코드를 리팩터링할 때 버그를 쉽게 유발할 수 있습니다.


Hook이 출시된 후에, Apollo는 Hooks 지원을 Apollo Client 라이브러리에 추가하였습니다. `graphql()` 고차 컴포넌트 대신, 라이브러리가 지원하는 Hook을 통해 개발자들은 데이터에 쉽게 접근할 수 있게 되었습니다.

`graphql()` 고차 컴포넌트 예시로 보았던 것과 정확히 동일한 데이터를 사용하는 예제를 봅시다. 이번에는 Apollo Client에서 제공하는 `useMutation` hook을 사용하여 컴포넌트에 데이터를 제공할 것입니다.

```js
import React, { useState } from "react";
import "./styles.css";

import { useMutation } from "@apollo/react-hooks";
import { gql } from "apollo-boost";

const ADD_MESSAGE = gql`
  mutation AddMessage($message: String!) {
    addMessage(message: $message) {
      message
    }
  }
`;

export default function Input() {
  const [message, setMessage] = useState("");
  const [addMessage] = useMutation(ADD_MESSAGE, {
    variables: { message }
  });

  return (
    <div className="input-row">
      <input
        onChange={(e) => setMessage(e.target.value)}
        type="text"
        placeholder="Type something..."
      />
      <button onClick={addMessage}>Add</button>
    </div>
  );
}
```

`useMutation` hook을 사용하여 데이터를 컴포넌트로 전달하기 위해 필요했던 코드의 양을 줄였습니다.

boilerplate의 감소뿐만 아니라, 여러개의 resolver들의 데이터를 컴포넌트에서 사용하는 것 또한 쉽습니다. 여러 개의 고차 컴포넌트를 구성하는 것 대신, 컴포넌트 내부에 여러 hook들을 작성하기만 하면 됩니다. 데이터가 컴포넌트로 전달되는 방법을 파악하는 것도 훨씬 쉽고, 컴포넌트를 리팩터링하거나 더 작은 단위로 쪼갤 때 개발자 경험을 향상시킵니다.

<br />

## Pros

HOC 패턴을 사용하면 단일 공간에 재사용가능한 모든 것을 담을 수 있습니다. 잠재적으로 새로운 버그를 매번 유발할 수 있는 코드를 계속해서 복제하면서 발생할 수 있는 버그가 어플리케이션 전역에 퍼지는 위험을 줄일 수 있습니다. 모든 로직을 한 공간에 둠으로써 DRY 원칙을 준수하며 관심사를 쉽게 분리할 수 있습니다.

## Cons

HOC가 컴포넌트로 전달하는 prop 명이 충돌할 수 있습니다.

```js
function withStyles(Component) {
  return props => {
    const style = { padding: '0.2rem', margin: '1rem' }
    return <Component style={style} {...props} />
  }
}

const Button = () = <button style={{ color: 'red' }}>Click me!</button>
const StyledButton = withStyles(Button)
```

이 경우, `withStyles` HOC는 `style` 이라는 prop을 우리가 전달한 컴포넌트에 추가합니다. 그러나, `Button` 컴포넌트는 이미 `style`이라는 prop을 가지고 있어, 이것을 덮어쓰게 됩니다! props을 병합하거나 이름을 다르게 함으로써 실수로 발생할 수 있는 이름 충돌을 해결해야 합니다.

```js
function withStyles(Component) {
  return props => {
    const style = {
      padding: '0.2rem',
      margin: '1rem',
      ...props.style
    }

    return <Component style={style} {...props} />
  }
}

const Button = () = <button style={{ color: 'red' }}>Click me!</button>
const StyledButton = withStyles(Button)
```

내부 자식 컴포넌트에 props를 전달하는 다수의 HOC를 사용할 때, 어떤 HOC가 어떤 prop에 책임이 있는지 알아내기 어렵습니다. 
이로 인해 어플리케이션의 디버깅과 확장이 어려워질 수 있습니다.