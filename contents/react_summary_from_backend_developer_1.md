---
date: '2022-04-22'
title: '[React] Backend 개발자가 admin을 만들기 위해 하는 react 정리 1편'
categories: ['react', 'korean']
summary: 'component와 props, 그리고 props.children'
thumbnail: './images/react_icon.png'
---

안녕하세요. 오늘은 React로 admin 개발을 하면서 공부했던 React를 사용하기 위해서 배웠던 내용을 정리하려고 합니다. 같은 방법에 대해서 고민하셨던 부분이 있으셨다면 이 포스팅으로 해결하셨으면 좋겠습니다. 아마 아주 기초적인 부분들이 주를 이룰것으로 보이기때문에.. 적절히 걸러서 보시는게 좋을 수 있습니다. :)

오늘 정리할 내용은 react에서 component들 사이에 데이터 전달의 기본이 되는 props와 event에 대해서입니다.

## component(컴포넌트)와 데이터 전달

**React에서 Component는 UI를 이루는 HTML을 반환하는 작은 단위**입니다. Component는 한번 만들면 원하는 곳에서 재사용하는 것이 가능합니다. Java에서의 class 또는 메서드라고 생각하시면됩니다. class와 메서드(함수)가 한번 정의해두면 원해는 곳에서 불러서 사용할 수 있듯이 **Component는 UI를 원하는 곳에 불러서 배치할 수 있는 것**입니다.

컴포넌트는 전체적으로 Tree 구조를 가집니다. 그리고 또 다른 컴포넌트에게 데이터를 넘기는 것이 가능합니다. 먼저 아래에서 컴포넌트 개념을 코드로 알아보겠습니다. react 코드는 기본적으로 아래와 같은 구조를 가집니다. 아래 처럼 코딩을 먼저하면 Details라는 하나의 컴포넌트가 완성 된 것입니다. 

```javascript
import React from "react";
import "./Details.css";

const Details = () => {
  return <p>HI SABARADA</p>;
};

export default Details;
```

이렇게 선언된 Details라는 컴포넌트는 아래처럼 사용할 수 있습니다. 아래를 보시면 `div` 태그 안쪽에 `Details` 태그가 선언된 것을 보실 수 있습니다. 이것이 우리가 직접 선언한 위의 Details 컴포넌트를 사용하는 방법입니다. 

```javascript
import "./App.css";
import Details from "./components/Details";

function App() {
  return (
    <div className="App">
      <Details></Details>; // Details 컴포넌트를 사용하는 것
    </div>
  );
}

export default App;
```

이렇게 선언하게되면 아래와 같은 결과를 얻을 수 있습니다.

![react_1.png](images/react_1.png)

## props

컴포넌트간의 데이터를 전달하는 방법중 가장 기본이되는 props에 대해서 알아보도록하겠습니다. props는 상위 컴포넌트에서 본인이 포함하고 있는 하위 컴포넌트에 데이터를 전달하는 방법입니다. 코드를 보기전에 props를 사용함에 있어서 주의해야할 점을 먼저 짚어봅시다.

- **props는 읽기 전용**입니다. 
  - **함수 컴포넌트나 클래스 컴포넌트 모두 컴포넌트의 자체 props를 수정해서는 안 됩니다.**
  - 즉, 순수 함수 
  - 수정을 하고 싶다면 props를 먼저 copy하고 이를 수정하면 됩니다. 

props를 이용할 수 있도록 위에서 보았던 코드를 수정해보았습니다. 아래 코드에 Details 함수에 props라는 파라미터가 추가되었고 props에서 `title`을 가져와서 데이터로 넣어주고 있습니다. 

```javascript
import React from "react";
import "./Details.css";

const Details = (props) => {
  return <p>{props.title}</p>;
};

export default Details;
```

이와 맞춰 상위 컴포넌트에서는 Details에 `title` 속성을 추가해서 내려보내고 있습니다. 이렇게 내려보내기 때문에 Details 함수에서는 이를 이용할 수 있게 된 것을 알 수 있습니다.

```javascript
import "./App.css";
import Details from "./components/Details";

function App() {
  return (
    <div className="App">
      <Details title="나는 Karol 입니다."></Details>;
    </div>
  );
}

export default App;
```

![react_2.png](images/react_2.png)

## props.children에 대해서 알아보자

추가적으로 props의 children에 대해서 알아보도록 하겠습니다. children은 props에 이미 정의된 변수입니다. 하는 역할은 props가 가지고 있는 하위 tags들 나타내 주는 역할을 합니다. **Wrapper(래핑)를 만들때 주로 이용**합니다.

아래 코드를 보시면 Details 상위에 Card라는 새로운 커스텀 컴포넌트를 추가했습니다. 이것의 목적은 Card가 Details를 래핑하도록 하는 역할을 하고자합니다.

```javascript
import "./App.css";
import Details from "./components/Details";
import Card from "./UI/Card";

function App() {
  return (
    <div className="App">
      <Card>
        <Details title="나는 Karol 입니다."></Details>;
      </Card>
    </div>
  );
}

export default App;
```

그리고 이에 맞는 Card 컴포넌트를 만들었습니다. 하는 역할은 보시면 아시겠지만 css를 래핑하는 역할이라고 보시면됩니다.

```javascript
import classes from "./Card.module.css";

const Card = (props) => {
  return <div className={classes.card}></div>;
};

export default Card;
```

그렇다면 실행해보겠습니다. 아래와 같이 내부 태그의 내용이 노출되지 않는것을 확인하실 수 있습니다. **왜냐하면 Card 내부에는 내부 태그에 대한 정보가 없기 때문입니다. 이때 사용할 수 있는것이 props.children 입니다.**

![react_3](images/react_3.png)

Card의 코딩을 아래와 같이 바꾸도록 하겠습니다. **props.children을 가져와서 해당 태그(Card)의 내부에 있는 tag를 가져와서 대입시켜주는 역할**을 합니다. 아래처럼 바꿨습니다.

```javascript
import classes from "./Card.module.css";

const Card = (props) => {
  return <div className={classes.card}>{props.children}</div>;
};

export default Card;
```

이렇게 바꾸고 실행했을 때 정상적으로 원하는 대로 노출된 것을 알 수 있습니다.

![react_4](images/react_4.png)

## 마무리

오늘은 이렇게 component, props 그리고 props.children을 정리해보는 시간을 가져보았습니다.

감사합니다.

## 참조

[1] https://matchgroup.udemy.com/course/react-the-complete-guide-incl-redux/learn/lecture/25600888#overview

[2] https://ko.reactjs.org/docs/components-and-props.html

[3] https://stackoverflow.com/questions/49706823/what-is-this-props-children-and-when-you-should-use-it

[4] https://www.w3schools.com/react/react_components.asp