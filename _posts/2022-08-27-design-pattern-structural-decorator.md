---
title: Decorator Pattern
author: yong
date: 2022-08-27 11:58:00 +0800
categories: [Design Pattern, Structural Pattern]
tags: [Design Pattern]
---

## ✏️ Decorator Pattern (데코레이터 패턴) 이란?

> 기본 기능에 추가할 수 있는 기능의 종류가 많은 경우, 각 추가 기능을 Decorator 클래스로 정의한 후 필요한 Decorator 객체를 조합함으로써 추가 기능의 조합을 설계하는 방식 - 위키백과

객체에 추가적인 기능을 서브 클래스를 생성하는 방식보다 훨씬 유연하게 동적으로 객체를 결합하며 추가할 수 있는 패턴이다.

## 🚘 도로를 표시하는 네비게이션 SW 만들기

UML

```java
public class RoadDisplay {
  public void draw() {
    System.out.println("Default Road Display");
  }
}
```

```java
public class RoadWithLaneDisplay extends RoadDisplay {
  public void draw() {
    super.draw();
    drawLane();
  }

  private void drawLane() {
    System.out.println("\tLane Display");
  }
}
```

```java
public static void main(String[] args) {
  RoadDisplay roadDisplay = new RoadDisplay();
  roadDisplay.draw();

  RoadDisplay roadWithLaneDisplay = new RoadWithLaneDisplay();
  roadWithLaneDisplay.draw();
}
```

```console
Default Road Display
Default Road Display
        Lane Display
```

기본적으로 이 네비게이션은 도로를 간단한 선으로 표시하고, 차선은 사용자가 선택하여 기능을 보여줄 수 있다. 기본적으로 보여주는 도로와 추가로 차선을 표시하여 제공하고 싶다면 RoadDisplay 클래스와 이를 상속받는 RoadDisplayWithLane 클래스를 설계해야 할 것이다.

## 💣 문제점

만약 도로의 차선 표시 기능 뿐만 아니라, 교통량과 교차로를 표시하는 기능을 제공하고, 이 기능을 동시에 보여주는 즉, 조합하기 위해선 다음과 같이 수많은 하위 클래스를 설계하고 추가적인 메소드들을 작성해야 할 것이다.

(UML)

```java
public class RoadWithLaneDisplay extends RoadDisplay() { ... }
public class RoadWithTrafficDisplay extends RoadDisplay() { ... }
public class RoadWithTrafficAndLaneDisplay extends RoadDisplay() { ... }
```

## 💡 해결법

이렇게 조합별로 클래스가 늘어나는 문제를 해결하기 위해 우리는 **데코레이터** 패턴을 적용하여 해결할 수 있다. 기본 기능인 `RoadDisplay` 객체에 부가적인 기능(Decorator)인 `Traffic` , `Lane` 들을 장식(Decorate)하여 해결해보자.

(UML)

각 부가적인 기능(Decorator) 클래스에서는 기본 기능인 RoadDisplay 클래스가 draw 메소드를 호출하도록 하고, 데코레이터들이 가지고 있는 기능은 직접 제공하도록 한다.

```java
public abstract class Display {
  public abstract void draw();
}
```

기능을 표시하는 `Display` 추상 클래스를 작성하여 `draw` 메소드를 서브 클래스에서 작성하도록 한다.

```java
public class RoadDisplay extends Display {
  public void draw() {
    System.out.println("Road Display");
  }
}
```

기본 기능인 도로 표시 기능을 구현한다. `draw` 메소드는 `Display` 추상 클래스에서 상속 받아 기능을 구현한다.

```java
public abstract class DisplayDecorator extends Display {
  private Display decoratedDisplay;

  public DisplayDecorator(Display decoratedDisplay) {
    this.decoratedDisplay = decoratedDisplay;
  };

  public void draw() {
    decoratedDisplay.draw();
  };
}
```

마찬가지로 Decorator 역시 `Display` 추상 클래스를 상속받는다. (내용 보충)

```java
public class LaneDecorator extends DisplayDecorator {
  public LaneDecorator(Display decoratedDisplay) {
    super(decoratedDisplay);
  }

  public void draw() {
    super.draw();
    drawLane();
  }

  private void drawLane() {
    System.out.println("\tLane Display");
  }
}

public class TrafficDecorator extends DisplayDecorator {
  public TrafficDecorator(Display decoratedDisplay) {
    super(decoratedDisplay);
  }

  public void draw() {
    super.draw();
    drawLane();
  }

  private void drawLane() {
    System.out.println("\tTraffic Display");
  }
}

public class CameraDecorator extends DisplayDecorator { ... }
```

`LaneDecorator`, `TrafficDecorator`, `CameraDecorator` 클래스는 실제로 기본 기능에 추가적으로 장식해줄 Decorator 이므로, `DisplayDecorator` 추상 클래스를 상속 받도록 한다. `DisplayDecorator` 에서 상속받은 `draw()` 메소드를 오버라이드하여, 각 부가 기능(Decorator)이 수행해야 할 기능만 추가로 작성한다.

```java
public static void main(String[] args) {
  Display roadDisplay = new RoadDisplay();
  roadDisplay.draw();

  Display roadWithLaneDisplay = new LaneDecorator(new RoadDisplay());
  roadWithLaneDisplay.draw();

  Display roadWithTrafficDisplay = new TrafficDecorator(new RoadDisplay());
  roadWithTrafficDisplay.draw();

  Display roadWithLaneAndTrafficDisplay = new RoadDisplay();
  roadWithLaneAndTrafficDisplay = new TrafficDecorator(roadWithLaneAndTrafficDisplay);
  roadWithLaneAndTrafficDisplay = new LaneDecorator(roadWithLaneAndTrafficDisplay);
  roadWithLaneAndTrafficDisplay.draw();
}
```
