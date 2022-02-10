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

