# Container/Presentational Pattern
어플리케이션 로직과 뷰 로직을 분리하여 관심사를 분리하기

<br />

---

React에서 관심사를 분리할 수 있는 한가지 방법은 Container/Presentational 패턴을 사용하는 것입니다. 이 패턴으로 어플리케이션 로직에서 뷰를 분리할 수 있습니다.

---

<br />

6개의 dog 이미지를 패치하여 스크린에 렌더링하는 어플리케이션을 만들고 싶다고 가정합시다.

이상적으로, 프로세스를 다음 두 부분으로 나누어서 관심사를 분리하려고 합니다.

1. Presentational Components: 사용자에게 데이터를 어떻게 보여줄 것인가를 관리하는 컴포넌트. 예제에서는, dog 이미지들을 렌더링하는 리스트에 해당.

2. Container Components: 어떤 데이터가 사용자에게 보여야 하는지를 관리하는 컴포넌트. 예제에서는, dog 이미지들을 패치하는 것에 해당.

[![Watch the video](./images/preview01.png)](https://res.cloudinary.com/ddxwdqwkr/video/upload/v1609056518/patterns.dev/jspat-40_af2vga.mp4)

dog 이미지들을 패치하는 것은 어플리케이션 로직을 다루는 반면에, 이미지들을 전시하는 것은 뷰를 다룹니다.

<br />

## Presentational Component

presentational 컴포넌트는 props로 데이터를 받습니다. 주요 기능은 데이터를 수정하지 않고, 스타일을 포함하여, 원하는 방식으로 데이터를 보여주는 것입니다.

```js
import React from "react";

export default function DogImages({ dogs }) {
  return dogs.map((dog, i) => <img src={dog} key={i} alt="Dog" />);
}
```

`DogImages` 컴포넌트는 presentational 컴포넌트입니다.
Presentational 컴포넌트들은 보통 상태값을 가지고 있지 않는데(stateless)  즉, UI 목적으로 필요한 상태값이 아니라면 자신만의 React 상태값을 포함하지 않습니다. presentational 컴포넌트 자신은 받은 데이터를 변경하지 않습니다.

Presentational 컴포넌트들은 container 컴포넌트로부터 데이터를 받습니다.

<br />

## Container Components

container 컴포넌트의 주요 기능은 가지고 있는 데이터를 presentational 컴포넌트로 전달하는 것입니다. Container 컴포넌트 자체는 일반적으로 데이터를 관리하는 presentational 컴포넌트 외에 다른 구성 요소를 렌더링하지 않습니다. Container 컴포넌트 자체는 어떤 것도 렌더링하지 않으므로, 보통 스타일과 관련한 어떤 로직도 포함하지 않습니다.

예시에서 dog 이미지들을 `DogsImages` presentational 컴포넌트로 전달하려고 합니다. 그전에, 외부 API에서 이미지들을 패치해야 합니다. Container 컴포넌트를 생성하여 데이터를 패치하고 이 데이터를 화면에 이미지를 띄울 presentational 컴포넌트인 `DogImages`로 전달합니다.

```js
import React from "react";
import DogImages from "./DogImages";

export default class DogImagesContainer extends React.Component {
  constructor() {
    super();
    this.state = {
      dogs: []
    };
  }

  componentDidMount() {
    fetch("https://dog.ceo/api/breed/labrador/images/random/6")
      .then(res => res.json())
      .then(({ message }) => this.setState({ dogs: message }));
  }

  render() {
    return <DogImages dogs={this.state.dogs} />;
  }
}

```

이 두 컴포넌트를 함께 결합하면 애플리케이션 로직을 뷰와 분리할 수 있습니다.

[![Watch the video](./images/preview02.png)](https://res.cloudinary.com/ddxwdqwkr/video/upload/v1609056519/patterns.dev/jspat-45_budnfb.mp4)

<br />

## Hooks

많은 경우에, Container/Presentational 패턴은 React Hooks으로 대체할 수 있습니다. Hooks의 도입으로 많은 개발자들은 state를 제공하는 container 컴포넌트로 없이 statefulness를 쉽게 추가하게 되었습니다.

`DogImagesContainer` 컴포넌트로 데이터를 패칭하는 로직을 사용하는 대신에, 커스텀 hook에서 이미지들을 패칭하여 dogs 배열을 반환할 수 있습니다.

```js
export default function useDogImages() {
  const [dogs, setDogs] = useState([]);

  useEffect(() => {
    fetch("https://dog.ceo/api/breed/labrador/images/random/6")
      .then(res => res.json())
      .then(({ message }) => setDogs(message));
  }, []);

  return dogs;
}
```

이 hook을 사용함으로써, 데이터를 패치하여 `DogImages` presentational 컴포넌트로 전달하는 `DogImagesContainer` container 컴포넌트로 감쌀 필요가 없어졌습니다. 대신, `DogImages` presentational 컴포넌트에서 바로 이 hook을 사용하면 됩니다.

```js
import React from "react";
import useDogImages from "./useDogImages";

export default function DogImages() {
  const dogs = useDogImages();

  return dogs.map((dog, i) => <img src={dog} key={i} alt="Dog" />);
}
```

`useDogImages` hook을 사용해도 여전히 뷰를 어플리케이션 로직과 분리할 수 있습니다. `DogImages`컴포넌트에서 데이터를 수정하지 않고, `useDogImages` hook이 반환하는 데이터를 사용할 수 있습니다.

[![Watch the video](./images/preview03.png)](https://res.cloudinary.com/ddxwdqwkr/video/upload/v1609056518/patterns.dev/jspat-46_evhhpd.mp4)

Hook으로 Container/Presentational 패턴처럼 컴포넌트에서 뷰와 로직을 분리할 수 있습니다. container 컴포넌트 내부에 presentational 컴포넌트를 감싸는 별도의 레이어를 사용할 필요가 없어집니다.

## Pros

Container/Presentational 패턴을 사용하면 얻는 이점이 많습니다.

Container/Presentational 패턴은 관심사의 분리를 장려합니다. Presentational 컴포넌트는 순수 함수로 UI에 대한 책임이 갖는 반면에, container 컴포넌트는 어플리케이션의 데이터 상태에 대한 책임을 갖습니다. 이러한 점이 관심사를 쉽게 분리시킵니다.

Presentational 컴포넌트는 데이터를 수정하지 않고 단순히 보여주기 때문에 쉽게 재사용가능합니다. application 전반에서 다른 목적으로 presentational 컴포넌트를 재사용할 수 있습니다.

presentational 컴포넌트는 어플리케이션 로직을 변경하지 않기 때문에 디자이너와 같이 베이스 코드를 잘 몰라도 쉽게 수정할 수 있습니다. presentational 컴포넌트가 어플리케이션의 많은 부부에서 재사용된 경우, 변경은 app 전반에 걸쳐 일관되게 적용됩니다. 

presentational 컴포넌트는 보통 순수 함수이므로, 쉽게 테스트할 수 있습니다. 데이터 저장소(store)를 mocking할 필요 없이, 어떤 데이터에 기반하여 presentational 컴포넌트가 렌더링되는지 알고 있습니다.

## Cons

Container/Presentational 패턴은 어플리케이션 로직을 렌더링 로직에서 쉽게 분리할 수 있게 합니다. 그러나, Container/Presentational 패턴을 사용하지 않고 stateless한 함수형 컴포넌트를 클래스 컴포넌트에 다시 쓸 필요 없이, Hooks을 통해 같은 결과를 얻을 수 있습니다. 이젠 더 이상 상태값을 사용하기 위해 클래스 컴포넌트를 만들 필요가 없습니다.

여전히 Container/Presentational 패턴을 React Hooks와 더불어 사용할 수 있지만, 이 패턴은 작은 규모의 어플리케이션에서는 과할 수 있습니다.

---

## 첨언

[Presentational and Container Components - Dan Abramov](https://medium.com/@dan_abramov/smart-and-dumb-components-7ca2f9a7c7d0) 에서 2019년에 업데이트

> Update from 2019: I wrote this article a long time ago and my views have since evolved. In particular, I don’t **suggest splitting your components like this anymore**. If you find it natural in your codebase, this pattern can be handy. But I’ve seen it enforced without any necessity and with almost dogmatic fervor far too many times. **The main reason I found it useful was because it let me separate complex stateful logic from other aspects of the component. Hooks let me do the same thing without an arbitrary division.** This text is left intact for historical reasons but don’t take it too seriously.

2015년 당시, C/P 패턴을 쓰는 이유는 복잡한 상태 관련 로직을 컴포넌트에서 분리하기 위함이었다.
그러나 [Hook의 개요](https://ko.reactjs.org/docs/hooks-intro.html#motivation)에서 말하듯이, 더이상 해당 목적으로 C/P 패턴이 아닌 Hook을 사용하기를 권장한다.

> Hook은 계층의 변화 없이 상태 관련 로직을 재사용할 수 있도록 도와줍니다.

<details open>
<summary>hook이 나온 배경</summary>
<div style="background-color:#2e2e2e;color:#eee;padding:18px 20px 18px 0;margin-top:10px;">
<ul style="list-style:none;">
<li>1. 코드 재사용을 위한 고전 방법들(render props, HOC)은 재사용을 목적으로 하는 Wrapper 컴포넌트를 추가함으로써 컴포넌트 트리를 복잡하게 만듦</li>
<li>2. 각 생명주기에 여러 로직이 포함됨(componentDidMount에서 fetch + eventListener 등록, componentWillUnmount에서 cleanup...) -> 복잡해짐</li> 
<li>
3. 개념적으로 React 컴포넌트는 항상 함수에 더 가깝습니다. Hook은 React의 정신을 희생하지 않고 함수의 사용을 권장합니다. 

(Component는 입력값으로 props와 state를 받아 React Element를 출력하는 함수)

<span style="padding-left:10px;color:#bbb">👉 Conceptually, components are like JavaScript functions. They accept arbitrary inputs (called “props”) and return React elements describing what should appear on the screen.</span>
</li>
</ul>
</div>
</details>

<br />

### Q. 그렇다면 Hook이 C/P 패턴을 완전히 대체할까?

Hook은 C/P 패턴을 대체하는 것은 아니다.

개념적으로 C/P 패턴을 적용한 구조가 자연스러우면 C/P 패턴을 사용해도 좋다고 생각된다.

Hook은 로직을 재사용하는데 유용하고, C/P 패턴은 컴포넌트를 재사용하는데 유용하다. (이전에 도입했던 VAC 패턴 참고)

참고한 글 : [React Hook VS Container Component](https://yujonglee.com/socwithhooks.html)