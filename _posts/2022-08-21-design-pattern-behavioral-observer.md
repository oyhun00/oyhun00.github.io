---
title: Observer Pattern
author: yong
date: 2022-08-21 11:58:00 +0800
categories: [Design Pattern, Behavioral Pattern]
tags: [Design Pattern]
---

## ✏️ Observer Pattern (옵저버 패턴) 이란?

> 옵서버 패턴은 객체의 상태 변화를 관찰하는 관찰자들, 즉 옵저버들의 목록을 객체에 등록하여 상태 변화가 있을 때마다 메서드 등을 통해 객체가 직접 목록의 각 옵저버에게 통지하도록 하는 디자인 패턴이다. 주로 분산 이벤트 핸들링 시스템을 구현하는 데 사용된다. - 위키백과

상태가 변하는 특정 객체를 관찰자(옵저버)들이 관찰(구독)하고, 특정 객체에서 상태의 변화가 나타날 때 자신을 구독하고 있는 옵저버들에게 상태가 변화 됐음을 알려주는(발행) 발행/구독 패턴이다. 객체의 상태 변화를 감지하기 위해 사용될 수 있는 [polling](https://ko.wikipedia.org/wiki/%ED%8F%B4%EB%A7%81_(%EC%BB%B4%ED%93%A8%ED%84%B0_%EA%B3%BC%ED%95%99)) 방식을 지양하거나, 상대 클래스나 객체에 의존하지 않고 상태 변화를 통보하기 위해 유용한 패턴이다. 대표적으로 자바스크립트에서 onClick과 같은 이벤트 핸들러들을 예로 들을 수 있다.

## 🛴 Input의 이벤트 핸들러 구현하기

텍스트를 입력할 수 있는 `input`에서 입력을 감지하는 핸들러를 구현해보자. 입력된 값을 저장하는 `Input` 클래스와 `Input`의 `onChangeValue` 메서드가 실행될 때, 즉, 값이 입력되었을 때, 이를 감지하는 `OnChangeHandler` 클래스를 작성해보자.

<img src="/observer-before-1-uml.png" alt="UML">

`Input` 클래스에서 값을 입력하는 `onChangeValue` 메서드가 실행되면 `value` 값을 저장하고, `OnChangeHandler`의 `detect` 메서드를 실행함으로써 입력된 값을 감지하도록 한다.

```java
public class Input {
  private String value;
  private OnChangeHandler onChangeHandler;

  public void setOnChangeHandler(OnChangeHandler onChangeHandler) {
    this.onChangeHandler = onChangeHandler;
  }

  public void onChangeValue(String value) {
    this.value = value;
    onChangeHandler.detect();
  }

  public String getInputValue() {
    return value;
  }
}
```
{: file='Input.class'}

```java
public class OnChangeHandler {
  private Input input;

  public OnChangeHandler(Input input) {
    this.input = input;
  }

  public void detect() {
    System.out.println("Value change detection: " + input.getInputValue());
  }
}
```
{: file='OnChangeHandler.class'}

```java
public static void main(String[] args) {
  Input input = new Input();

  OnChangeHandler onChangeHandler = new OnChangeHandler(input);

  input.setOnChangeHandler(onChangeHandler);
  input.onChangeValue("Entered text");
  input.onChangeValue("Edited text");
}
```
{: file='Main.class'}

```console
Value change detection: Entered text
Value change detection: Edited text
```

의도했던 대로 값을 입력하면 입력된 값을 감지하고 해당 텍스트를 출력하게 되었다.


## 💣 문제점

하지만 만약 우리가 `input` 을 클릭했을 때, 더블 클릭 했을 때, 마우스 휠을 사용할 때의 이벤트를 실행하는 핸들러들을 추가하고 싶다면? 혹은 입력 핸들러가 아닌 클릭 핸들러로 변경하고 싶다면 `Input` 클래스를 수정해야 할 필요가 있다.


작성중