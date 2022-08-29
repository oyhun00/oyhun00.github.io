---
title: 옵저버 패턴을 리엑트에서 사용해보기
author: yong
date: 2022-08-29 11:58:00 +0800
categories: [Design Pattern, React]
tags: [Design Pattern, Observer Pattern, React]
---

## 🔗 옵저버 패턴 알아보기

[https://oyhun00.github.io/posts/design-pattern-behavioral-observer/](https://oyhun00.github.io/posts/design-pattern-behavioral-observer/)

상태가 변하는 특정 객체를 관찰자(옵저버)들이 관찰(구독)하고, 특정 객체에서 상태의 변화가 나타날 때 자신을 구독하고 있는 옵저버들에게 상태가 변화 됐음을 알려주는(발행) 발행/구독 패턴이다. 저번에 작성한 옵저버 패턴을 기반으로 리액트에 적용해보자.

## 🛎️ 버튼 클릭 시, 여러 이벤트를 실행하는 인터랙션 만들기

사용자가 버튼을 클릭할 때, 알림창과, 토스트 알림 그리고 행동을 기록하기 위해 로그를 남기는 인터랙션을 구현해보자. 다음은 `Subject` 클래스와 인터페이스이다.

```typescript
interface Subject {
  subscribe(obs: Observer): void;
  unsubscribe(obs: Observer): void;
  notify(data: String): void;
}

class Observable implements Subject {
  private observers: Observer[];

  constructor() {
    this.observers = [];
  }

  subscribe(obs: Observer) {
    this.observers.push(obs);
  }

  unsubscribe(obs: Observer) {
    this.observers = this.observers.filter((observer) => observer !== obs);
  }

  notify(data: String) {
    this.observers.forEach((observer) => observer(data));
  }
}
```

`Subject` 클래스에서의 기능은 다음과 같다.

- **observers**: `Subject` 객체의 상태 변화를 관찰(구독)할 옵저버들이 담겨져 있다.
- **subscribe:** `Subject` 객체의 상태 변화를 관찰(구독)할 옵저버들을 추가한다.
- **unsubscribe:** `Subject` 객체의 상태 변화를 구독하고 있는 특정 옵저버를 제거(구독 해제)한다.
- **notify:** `Subject` 객체의 상태 변화를 관찰(구독)하고 있는 옵저버들에게 변화를 알린다

이제 `Subject` 객체를 구독할 `Observer` 를 리액트에서 구현해보자. 사용자가 버튼을 클릭하게 되면, `Subject` 클래스에서 구독하고 있는 옵저버들에게 변화를 알리는 `notify` 를 실행하여, 각 옵저버들에게 업데이트를 실행하도록 한다.

```typescript
const App = () => {
  const handleClick = () => {
    Observable.notify("Click !");
  };

  const eventAlert: Observer = (data: String) => {
    alert(data);
  };

  const eventLogger: Observer = (data: String) => {
    console.log(data);
  };

  const eventToastify: Observer = (data: String) => {
    toast(data);
  };

  Observable.subscribe(eventAlert);
  Observable.subscribe(eventLogger);
  Observable.subscribe(eventToastify);

  return (
    <>
      <button onClick={handleClick}>Button</button>
    </>
  );
};
```

`eventAlert`, `eventLogger`, `eventToastify` 함수는 `Subject` 객체를 구독할 Observer 가 된다. 이 옵저버들을 `subscribe` 를 통해 구독하게끔 하고, 버튼을 클릭하면 실행되는 `handleClick` 함수를 통해 업데이트를 실행하게 된다.
