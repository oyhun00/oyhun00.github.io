---
title: JavaScript Memory
author: yong
date: 2023-08-15 11:58:00 +0800
categories: [JavaScript, Core]
tags: [JavaScript]
---

# JavaScript Memory
자바스크립트에서 메모리 관리는 자바스크립트 엔진에서 자동적으로 관리해준다. 하지만 자바스크립트를 사용하는 개발자라면 어느 정도 이해는 해놔야 될 것 같아 정리하였다.

# 🔹자바스크립트 메모리 생명 주기

자바스크립트 엔진은 변수나 함수 등을 생성할 때, 메모리를 할당하고 필요하지 않으면 해제한다. 메모리 할당은 말 그대로 메모리에 공간에 할당하는 행위이며, 해제는 할당된 공간을 비워 다른 메모리를 할당한다. 

<img src="/jsgc/1.png" alt="javascript-gc.png">

### Allocate (할당)

자바스크립트에서 생성한 객체에 필요한 메모리를 할당한다.

### Use (사용)

코드에서 명시적으로 수행되는 작업으로. 변수를 사용하거나 읽는 작업 하며 메모리를 읽고 쓴다.

### Release (해제)

자바스크립트 엔진에서 사용하지 않는 변수나 함수를 해제한다.

# 🔹Heap and Stack

자바스크립트 엔진에는 데이터를 저장하는 힙과 스택이라는 공간이 존재한다.

## Stack - 정적 메모리 할당

스택은 자바스크립트에서 정적 데이터를 저장하는데 사용하는 자료 구조이다. 자바스크립트에서 정적 데이터는 원시 값 (number, string, boolean, undefined, null)과 객체나 함수를 가리키는 참조이다.

정적 데이터는 크기가 변하지 않고 정적임으로 엔진에서 고정된 크기의 메모리를 사전에 할당한다. 그리고 이를 실행 직전에 메모리를 할당한다 해서 정적 메모리 할당이라 한다.

<img src="/jsgc/2.png" alt="javascript-gc.png">

## Heap - 동적 메모리 할당

힙은 스택과 달리 고정된 크기의 메모리를 할당하지 않는다. 필요에 따라 동적으로 공간을 할당하게 되고 이를 동적 메모리 할당이라 한다. 대표적으로 오브젝트나 객체를 힙에 저장하게 된다.

# 🔹자바스크립트의 참조

자바스크립트에서 모든 변수는 스택을 가장 먼저 가르킨다. 이때 원시 값이 아닌 경우에는 스택에는 힙에 존재하는 객체에 대한 참조가 저장하게 된다.

힙은 스택과 달리 정렬에 대한 규칙이 존재하지 않아 스택에 대한 참조를 유지해야 한다.

<img src="/jsgc/3.png" alt="javascript-gc.png">

# 🔹자바스크립트 가비지 콜렉션

자바스크립트 엔진에서 메모리 할당 해제는 가비지 콜렉터가 담당한다. 엔진이 특정 변수나 함수가 더 이상 사용되지 않음을 판단하면 이를 해제해준다.

하지만 확실하게 메모리가 여전히 사용중인지 아닌지를 판별할 수 없다. 이는 더 이상 필요하지 않은 공간을 정확한 타이밍에 가비지를 콜렉션할 수 있는 완벽한 알고리즘이 존재할 수 없다고 한다.

완벽하게 정확한 시점에서 메모리를 해제할 수 있진 않지만, 어느 정도 목표를 달성할 수 있는 가장 많이 사용되는 두 가지 가비지 콜렉션 알고리즘이 있다.

## Reference-counting Garbage Collection

이 알고리즘은 가리키는 참조가 없는 객체를 수집한다.

<img src="/jsgc/4.gif" alt="javascript-gc">

`hobbies` 변수는 참조가 존재하는 객체이기 때문에 힙에서만 유지되고 있다.

하지만 이 알고리즘은 순환 참조를 고려하지 않고 있다.

```tsx
let son = {
    name: 'John',
};

let dad = {
    name: 'Johnson',
};

son.dad = dad;
dad.son = son;

son = null;
dad = null;
```

<img src="/jsgc/5.png" alt="javascript-gc.png">

`son` 과 `dad` 객체가 서로를 참조하기 때문에, 알고리즘이 이 변수들이 사용되고 있는지를 판별할 수 없게 된다.

## Mark and Sweep Algorithm

이 알고리즘은 순환 참조에 대한 문제점을 해결할 수 있다. 주어진 객체에 대한 참조를 계산하는 대신, 루트 객체에서 도달할 수 있는지를 감지한다. 여기서 루트는 브라우저에서 `window` 객체, Nodejs에서 `global` 객체를 가리킨다.

<img src="/jsgc/6.png" alt="javascript-gc.png">

루트가 참조하는 객체들, 그리고 그 객체에서 참조하는 또 다른 객체들을 탐색하며 마크를 한다. 이러한 마크가 끝나면 가비지 컬렉터는 힙 내부 전체를 돌며 마크되지 않은 영역을 해제한다. 이러한 과정을 Sweep이라 한다. 이렇게 되면 순환 참조가 더 이상 문제되지 않는다.

# 🔹자바스크립트 GC의 단점

## 메모리 사용량

앞서 기술했듯이, 가비지 컬렉터가 정확한 타이밍에 가비지를 콜렉션 할 수 없다는 점을 보았을 때, 자바스크립트는 실제로 필요한 것보다 더 많은 메모리를 사용할 수 있다고 한다.

만약 메모리 효율을 최대로 끌어올려야 할 때, JS가 아닌 다른 Low-level 언어를 사용하는 것이 좋다.

# 🔹자바스크립트 메모리 누수

## 전역 변수

대표적으로 브라우저에서 `const` 나 `let` 키워드 대신 `var` 를 사용하거나 생략하는 경우, 엔진이 변수를 `window` 객체에 할당한다. 이는 `function` 으로 정의한 함수도 마찬가지다.

```tsx
user = getUser();

var secondUser = getUser();

function getUser() {
  return 'user';
}
```

전역 변수를 사용하게 된다면, 필요하지 않을 경우 메모리 공간을 확보해주는 것이 좋다. `null` 을 할당하여 메모리를 해제한다.

```tsx
window.users = null;
```

## 타이머 및 콜백

자바스크립트에서 `setInterval` 과 같은 타이머 함수를 선언하고 clear 하지 않으면 메모리 사용량이 증가한다. 특히 SPA에서 이벤트 리스너와 콜백을 동적으로 추가할 때 주의해야 한다.

```tsx

const object = {};
const intervalId = setInterval(function() {
    doSomething(object);
}, 2000);
```

위 코드는 2초마다 함수를 실행시키고, interval이 취소되지 않으면, 참조된 객체들은 가비지 컬렉터에 수집되지 않는다. 만약 더 사용하지 않는다면 다음과 같이 해제한다.

```tsx
clearInterval(intervalId);
```

SPA 같은 경우에서 특히나 주의해야하는 이유는 다른 페이지로 이동해도 여전히 백그라운드에서 실행 될 수 있기 때문이다.

## DOM 참조

자바스크립트에 DOM Element를 저장할 때 발생하게 된다.

```tsx
const elements = [];
const element = document.getElementById('button');

elements.push(element);

function removeAllElements() {
    elements.forEach((item) => {
        document.body.removeChild(document.getElementById(item.id))
    });
}
```

Element를 제거할 때, 배열에서도 제거해줘야 한다. 그렇지 않으면 가비지 컬렉터가 수집할 수 없다.

```tsx
const elements = [];
const element = document.getElementById('button');

elements.push(element);

function removeAllElements() {
    elements.forEach((item, index) => {
        document.body.removeChild(document.getElementById(item.id));

        elements.splice(index, 1);
    });
}
```

모든 DOM Element는 부모 노드에 대한 참조도 유지하기 때문에, 가비지 콜렉터가 Element의 부모와 자식을 수집하는 것을 방지할 수 있다.

## 📚 Reference

- https://felixgerschau.com/javascript-memory-management/#trade-offs
- [https://velog.io/@sejinkim/자바스크립트의-메모리-관리-설명](https://velog.io/@sejinkim/%EC%9E%90%EB%B0%94%EC%8A%A4%ED%81%AC%EB%A6%BD%ED%8A%B8%EC%9D%98-%EB%A9%94%EB%AA%A8%EB%A6%AC-%EA%B4%80%EB%A6%AC-%EC%84%A4%EB%AA%85)

해당 포스트는 위 레퍼런스를 통해 작성하였습니다. 그런데 Mark and Sweep Algorithm에서 도달할 수 없는 객체를 마크 처리하고, 마크 처리된 객체를 Sweep한다고 나와 있지만, JVM에선 참조할 수 없는 객체를 마크 처리한다는 점에서 차이점이 나타나 다음 포스트들을 참조하여 도달할 수 있는 객체를 마크 처리 하는것으로 수정하여 작성하였습니다. 만약 이 내용이 잘못된 내용이라면 피드백 부탁드립니다.

- https://v8.dev/blog/concurrent-marking#background
- https://ko.javascript.info/garbage-collection#ref-35
