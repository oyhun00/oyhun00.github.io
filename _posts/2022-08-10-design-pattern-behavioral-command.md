---
title: Command Pattern
author: yong
date: 2022-08-10 11:58:00 +0800
categories: [Design Pattern, Behavioral Pattern]
tags: [Design Pattern]
---

## ✏️ Command Pattern (커맨드 패턴) 이란?

> 커맨드 패턴(Command pattern)이란 요청을 객체의 형태로 캡슐화하여 사용자가 보낸 요청을 나중에 이용할 수 있도록 메서드 이름, 매개변수 등 요청에 필요한 정보를 저장 또는 로깅, 취소할 수 있게 하는 패턴이다. - 위키백과

즉, 이벤트가 발생할 때, 실행될 기능들을 캡슐화하고, 클래스를 변경하지 않도록 재사용성이 높은 클래스를 설계하는 패턴이다.

## 🎮 만능 버튼 구현하기

버튼이 눌리면, 특정 기능을 실행하는 프로그램을 구현한다. 우선 버튼을 누를 때, 램프의 불이 켜지는 프로그램을 구현해보자.

```java
public class Button {
  private Lamp lamp;

  public Button(Lamp lamp) {
    this.lamp = lamp;
  };

  public void pressed() {
    lamp.turnOn();
  };
}
```
{: file='Button.class'}

```java
public class Lamp {
  public void turnOn() {
    System.out.println("Lamp Turn On");
  };
}

```
{: file='Lamp.class'}

```java
public static void main(String[] args) {
  Lamp lamp = new Lamp();
  Button button = new Button(lamp);


  button.pressed();
}
```
{: file='Main.class'}

Button 생성자를 이용해 Lamp 객체를 전달하고, Button의 pressed 메서드가 실행되면 전달받은 Lamp 객체의 turnOn 메서드를 실행하여 기능을 수행하게 된다.

## 💣 문제점

버튼을 눌렀을 때, 램프의 불이 켜지는 기능 뿐만 아니라, 다른 기능을 추가하고 또 추가된 기능을 실행하고 싶다면?

예를 들어 램프 대신 알람 기능을 실행하고 싶다면, Button 클래스를 수정해야 한다.

```java
public class Alarm {
  public void start() {
    System.out.println("Alarm Start");
  }
}
```
{: file='Alarm.class'}

```java
public class Button {
  private Alarm alarm;

  public Button(Alarm alarm) {
    this.alarm = alarm;
  };

  public void pressed() {
    alarm.start();
  };
}
```
{: file='Button.class'}

```java
public static void main(String[] args) {
  Alarm alarm = new Alarm();
  Button button = new Button(alarm);


  button.pressed();
}
```
{: file='Main.class'}

정상적으로 버튼을 눌렀을 때, 알람의 기능은 수행될 것이다. 그러나 기능을 변경하기 위해 기존의 Button 클래스를 수정하는 것은 SOLID 원칙에서 **개방 폐쇄 원칙(Open Close Principle)**에 위배하게 된다. 따라서 버튼을 눌렀을 때, 실행될 기능을 분기 처리하기 위해 기능을 선택할 수 있게끔 설계해야 한다.

<br>

```java
enum Mode { LAMP, ALARM }

public class Button {
  private Lamp theLamp;
  private Alarm alarm;
  private Mode mode;

  public Button(Lamp theLamp, Alarm alarm) {
    this.theLamp = theLamp;
    this.alarm = alarm;
  };

  public void setMode(Mode mode) {
    this.mode = mode;
  }

  public void pressed() {
    if (mode.equals(Mode.LAMP)) {
      theLamp.turnOn();
    } else {
      alarm.start();
    }
  } 
}
```
{: file='Button.class'}

```java
public static void main(String[] args) {
  Lamp theLamp = new Lamp();
  Alarm alarm = new Alarm();
  Button button = new Button(theLamp, alarm);

  button.setMode(Mode.ALARM);
  button.pressed();

  button.setMode(Mode.LAMP);
  button.pressed();
}
```
{: file='Main.class'}

이제 버튼을 눌렀을 때, 분기 처리하여 특정 기능을 수행할 수 있도록 구현되었다. 하지만 알람 기능뿐만 아니라 보일러를 키는 기능, 에어컨을 키는 기능 등 여러가지 기능들이 추가된다면, 또 다시 Button 클래스를 수정해야할 것이고, 분기 처리하는 코드는 길어질 것이다. 이렇게 되면 재사용성 높은 설계라고 볼 순 없을 것이다.

## 💡 해결법

앞서 이야기한 코드의 문제점을 해결하기 위해, **커맨드 패턴**을 적용함으로써 해결할 수 있다.

Button 클래스를 수정하지 않고 사용하기 위해, pressed 메서드에서 기능을 직접 구현하는 대신, 버튼을 눌렀을 때, 실행되는 기능들을 캡슐화하여 외부에서 제공받도록 설계한다.
아래는 커맨트 패턴을 적용한 클래스 다이어그램이다.

<img src="/command-uml.png" alt="Command Patter UML">

이제 Button 클래스는 특정 기능을 실행할 때, Lamp나 Alarm의 메서드들을 직접 호출하지 않고, 정의된 Command 인터페이스를 통해 excute로 메서드를 호출한다. 그리고 Command 인터페이스를 상속받은 클래스들은 각각의 기능을 excute 메서드를 통해 기능을 구현한다.

```java
public interface Command {
  public abstract void execute();
}
```
{: file='Command.class'}

각 기능들의 메서드들을 직접 호출하지 않도록 Command 인터페이스를 작성한다.

<br>

```java
public class Button {
  private Command theCommand;

  public Button(Command theCommand) {
    this.theCommand = theCommand;
  }

  public void setCommand(Command newCommand) {
    this.theCommand = newCommand;
  }

  public void pressed() {
    theCommand.execute();
  }
}
```
{: file='Button.class'}

pressed 메서드를 호출하면, 주어진 Command의 excute 메서드를 호출하도록 한다.

<br>

```java
public class LampOnCommand implements Command {
  private Lamp lamp;

  public LampOnCommand(Lamp lamp) {
    this.lamp = lamp;
  };

  public void execute() {
    lamp.turnOn();
  }
}
```
{: file='LampOnCommand.class'}

```java
public class AlarmStartCommand implements Command {
  private Alarm alarm;

  public AlarmStartCommand(Alarm alarm) {
    this.alarm = alarm;
  }

  public void execute() {
    alarm.start();
  }
}
```
{: file='FireBallStrategy.class'}

앞서 설계한 인터페이스를 캡슐화한 각 기능들이 상속받아 인터페이스를 구현하고, Button 객체에 설정한다.

<br>

```java
public static void main(String[] args) {
    Lamp theLamp = new Lamp();
    Command lampOnCommand = new LampOnCommand(theLamp);

    Button button1 = new Button(lampOnCommand);
    button1.pressed();

    Alarm alarm = new Alarm();
    Command alarmOnCommand = new AlarmStartCommand(alarm);

    Button button2 = new Button(alarmOnCommand);
    button2.pressed();

    button2.setCommand(lampOnCommand);
    button2.pressed();
  }
```
{: file='Main.class'}

버튼을 눌렀을 때, 실행될 기능은 Command 인터페이스를 구현한 클래스의 객체를 Button 객체에 설정하여 사용할 수 있게 되었다. 이제 OCP를 위배하지 않고 다양한 기능을 구현할 수 있다.