---
title: Factory
author: yong
date: 2022-08-28 11:58:00 +0800
categories: [Design Pattern, Creational Pattern]
tags: [Design Pattern]
---

## ✏️ Factory (팩토리) 란?

**인스턴스 생성을 처리하는 클래스를 팩토리(Factory)**라 한다. 인스턴스의 생성을 서브 클래스에게 결정하도록 위임하는 방식이다. 팩토리를 사용하게 되면 객체 생성 작업을 팩토리로 캡슐화하여 변경이 필요할 시점에 여기저기 수정할 필요 없이, 팩토리 클래스 하나만 수정하여 유지보수가 용이하도록 도와준다.

## 🍕 피자집의 피자 제작 시스템 구성하기

피자집에서 피자를 주문 받고, 구워서 포장하는 피자 제작 시스템을 구성해보자.

```java
public void orderPizza(String type) {
  Pizza pizza;

	if (type.equals("Cheeze")) {
    pizza = new CheesePizza();
  } else if (type.equals("Veggie")) {
    pizza = new VeggiePizza();
  }

  pizza.prepare();
  pizza.bake();
  pizza.box();
}
```

간단하게 치즈 피자와 야채 피자를 제작하는 시스템을 구성했다. 하지만 피자의 종류는 두 가지만 존재하진 않을 것이다. 분명 피자의 종류가 추가되고, 야채 피자는 상대적으로 수요가 적어 메뉴에서 사라질 수도 있다. 이제 기존에 존재하던 야채 피자를 메뉴에서 빼고, 페페로니 피자와 치킨 피자를 추가해보자.

```java
public void orderPizza(String type) {
  Pizza pizza;

	if (type.equals("Cheeze")) {
    pizza = new CheesePizza();
  } /* else if (type.equals("Veggie")) {
    pizza = new VeggiePizza();
  }*/
  else if (type.equels("Pepperoni")) {
    pizza = new PepperoniPizza();
	} else if (type.equels("Chicken")) {
    pizza = new ChickenPizza();
	}

  pizza.prepare();
  pizza.bake();
  pizza.box();
}
```

이제 앞으로 새로운 피자 메뉴가 출시되거나, 특정 피자를 메뉴에서 빼기 위해선 코드를 변경하고, 삭제해야할 것이다. 이렇게 되면 추후에 유지보수가 어려워지고, 유연성이 떨어질 것이다. 저렇게 변경에 열려있는 코드를 **개방 폐쇄 원칙(OCP)**에 따라 **변경에 닫혀있도록** 수정해야한다.

## 💊 피자 객체를 생성하는 팩토리 만들기

우선 변화가 자주 일어나는 곳을 찾아서 캡슐화를 하려한다. 위 코드에서 볼 수 있듯이 변경되는 부분은 **인스턴스를 생성하는 구상 클래스를 선택하는** 분기문이다.

```java
if (type.equals("Cheeze")) {
  pizza = new CheesePizza();
} /* else if (type.equals("Veggie")) {
  pizza = new VeggiePizza();
} */
else if (type.equels("Pepperoni")) {
  pizza = new PepperoniPizza();
} else if (type.equels("Chicken")) {
  pizza = new ChickenPizza();
}
```

피자 인스턴스만 생성할 수 있도록 위 코드를 따로 분리한다.

```java
public class SimplePizzaFactory {
  public Pizza createPizza(String type) {
	  Pizza pizza = null;

    if (type.equals("Cheeze")) {
      pizza = new CheesePizza();
    } else if (type.equels("Pepperoni")) {
      pizza = new PepperoniPizza();
    } else if (type.equels("Chicken")) {
      pizza = new ChickenPizza();
    }

    return pizza;
  }
}
```

```java
public class PizzaStore {
  SimplePizzaFactory simplePizzaFactory;

  public PizzaStore(SimplePizzaFactory simplePizzaFactory) {
    this.simplePizzaFactory = simplePizzaFactory;
  }

  public void orderPizza(String type) {
    Pizza pizza;

    pizza = simplePizzaFactory.createPizza(type);
    pizza.prepare();
    pizza.bake();
    pizza.box();
  }
}
```

이제 피자 인스턴스를 생성하는 과정을 캡슐화함으로써 `PizzaStore` 클래스는 피자 생성 과정 따위 몰라도 된다. 단지 피자를 준비하고, 굽고, 포장하는 과정만 수행하면 피자를 만들 수 있게 되었다.

## 💩 변경되는 코드를 단지 분리했을 뿐인거 같은데?

이렇게 캡슐화해서 분리해서 얻는 이득이 있나 의문점을 가졌었지만, `SimplePizzaFactory` 팩토리 클래스를 사용하는 여러 클라이언트에서 사용할 수 있는 상황에서 피자를 생성하는 작업을 팩토리 클래스 하나만 수정하여 리소스를 줄일 수 있다. 하지만 여전히 인스턴스를 생성하는 과정에서 여러가지 문제가 발생할 수 있는데, 이 부분은 다른 생성 패턴을 활용하여 인스턴스를 생성하는 코드를 삭제할 것이다.

### 📚 Reference

헤드퍼스트 디자인패턴 - [에릭 프리먼, 엘리자베스 롭슨, 케이시 시에라, 버트 베이츠]
