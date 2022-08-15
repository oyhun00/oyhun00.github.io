---
title: Strategy Pattern
author: yong
date: 2022-07-08 11:58:00 +0800
categories: [Design Pattern, Behavioral Pattern]
tags: [Design Pattern]
---

## ✏️ Strategy Pattern (전략패턴) 이란?

> 전략 패턴은 실행 중에 알고리즘을 선택할 수 있게 하는 행위 소프트웨어 디자인 패턴이다. 특정한 계열의 알고리즘들을 정의하고, 각 알고리즘을 캡슐화하며, 이 알고리즘들을 해당 계열 안에서 상호 교체가 가능하게 만든다. - 위키백과

즉, 각각의 알고리즘(전략)들을 공통의 인터페이스를 구현하는 클래스로 캡슐화하고, 동적으로 전략을 쉽게 바꿀 수 있도록 설계하는 패턴이다.

## 🕹️ 게임 캐릭터 구현하기

RPG 게임을 개발하며, 전사, 마법사 이렇게 2가지 직업들을 구현해야하는 상황이다. 각각의 직업은 스킬도 모두 다를 것이다. 전사는 검을 휘두르고 걸어다니며 이동하고, 마법사는 파이어 볼을 발사하고 순간이동으로 이동한다. 현재까지의 상황을 코드로 작성해보자.

```java
@import
 public abstract class Adventurer {
  private String job;

  public Adventurer(String job) { this.job = job; }
  public String getJob() { return job; }

  public abstract void skill();
  public abstract void move();
}
```
{: file='Adventurer.class'}

```java
@import
public class Warrior extends Adventurer {
  public Warrior(String job) { super(job); }
  
  public void skill() { System.out.println("검 휘두르기."); }
  public void move() { System.out.println("열심히 걷기."); }
}
```
{: file='Warrior.class'}

```java
@import
public class Magician extends Adventurer {
  public Magician(String job) { super(job); }
  
  public void skill() { System.out.println("파이어볼 발사하기."); }
  public void move() { System.out.println("텔레포트로 이동하기."); }
}
```
{: file='magician.class'}

```java
@import
public class Main {
  public static void main(String[] args) {
    Adventurer warrior = new Warrior("전사");
    Adventurer magician = new Magician("마법사");

    System.out.println("나는 " + warrior.getJob());
    warrior.attack();
    warrior.move();

    System.out.println("나는 " + magician.getJob());
    magician.attack();
    magician.move();
  }
}
```
{: file='Main.class'}

```console
나는 전사
검 휘두르기.
열심히 걷기.
나는 마법사
파이어볼 발사하기.
텔레포트로 이동하기.
```

이렇게 전사와 마법사 캐릭터가 구현되었다. 하지만 만약 전사의 공격 방식이 마법사의 파이어볼로 변경되고, 마법사의 공격 방식이 전사의 검 휘두르기로 변경해달라는 요청이 온다면? 또, 마검사라는 직업이 추가되어 공격 방식은 기존 전사의 검 휘두르기와 이동 방식은 마법사의 텔레포트로 구현해야 된다면 다음과 같이 구현하게 될 것이다.


```java
@import
public class Warrior extends Adventurer {
  public Warrior(String job) { super(job); }
  
  public void skill() { System.out.println("파이어볼 발사하기."); }
  public void move() { System.out.println("열심히 걷기."); }
}
```
{: file='Warrior.class'}

```java
@import
public class Magician extends Adventurer {
  public Magician(String job) { super(job); }
  
  public void skill() { System.out.println("검 휘두르기."); }
  public void move() { System.out.println("텔레포트로 이동하기."); }
}
```
{: file='magician.class'}

```java
@import
public class MagicWarrior extends Adventurer {
  public MagicWarrior(String job) { super(job); }
  
  public void skill() { System.out.println("검 휘두르기."); }
  public void move() { System.out.println("텔레포트로 이동하기."); }
}
```
{: file='magicWarrior.class'}

## 💣 코드의 문제점

각 직업의 공격, 이동 스킬을 수정, 변경하기 위해 각각 클래스의 코드를 수정해야하는데, 이때 SOLID 원칙에서 **개방 폐쇄 원칙(Open Close Principle)**에 위배하게 된다. 또한 중복되는 코드가 나타나고, 마검사라는 직업이 추가되면서 마검사의 `skill()` 메소드와 `move()` 메소드의 내용이 중복된다.

## 💡 해결법

앞서 이야기한 코드의 문제점을 해결하기 위해, **전략 패턴**을 적용함으로써 해결할 수 있다.

위에서 전략패턴을 “각각의 알고리즘(전략)들을 공통의 인터페이스를 구현하는 클래스로 캡슐화하고, 동적으로 전략을 쉽게 바꿀 수 있도록 설계하는 패턴” 이라고 설명하였다. 이제 각 직업의 공격과 이동 방식(**전략**)을 공통의 인터페이스로 정의해보자.

```java
@import
public interface AttackStrategy { public void attack(); }
```
{: file='AttackStrategy.class'}

```java
@import
public interface MovingStrategy { public void move(); }
```
{: file='MovingStrategy.class'}


이제 공격과 이동 스킬(전략)을 클래스로 구현한다.

```java
@import
public class SmashStrategy implements AttackStrategy {
  public void attack() { System.out.println("검 휘두르기."); }
}
```
{: file='SmashStrategy.class'}

```java
@import
public class FireBallStrategy implements AttackStrategy {
  public void attack() { System.out.println("파이어볼 발사하기."); }
}
```
{: file='FireBallStrategy.class'}

```java
@import
public class WalkingStrategy implements MovingStrategy {
  public void move() { System.out.println("열심히 걷기."); }
}
```
{: file='WalkingStrategy.class'}

```java
@import
public class TeleportStrategy implements AttackStrategy {
  public void attack() { System.out.println("텔레포트로 이동하기."); }
}
```
{: file='TeleportStrategy.class'}


각 직업의 공격과 이동 스킬(전략)은 인터페이스와 그 구현체로 분리했으니, 직업 클래스에서 공격과 이동 메소드는 제거한다.

```java
@import
public class Magician extends Adventurer {
  public Magician(String job) { super(job); }
}
```
{: file='Magician.class'}

```java
@import
public class Warrior extends Adventurer {
  public Magician(String job) { super(job); }
}
```
{: file='Warrior.class'}

```java
@import
public class MagicWarrior extends Adventurer {
  public MagicWarrior(String job) { super(job); }
}
```
{: file='MagicWarrior.class'}


그리고 Adventurer 클래스는 다음과 같이 수정한다.

```java
@import
public abstract class Adventurer {
  private String job;
  private AttackStrategy attackStrategy;
  private MovingStrategy movingStrategy;

  public Adventurer(String job) { this.job = job; }
  public String getJob() { return job; }

  public void attack() { attackStrategy.attack(); }
  public void move() { movingStrategy.move(); }

  public void setAttackStrategy(AttackStrategy attackStrategy) { this.attackStrategy = attackStrategy; }
  public void setMovingStrategy(MovingStrategy movingStrategy) { this.movingStrategy = movingStrategy; }
 }
```
{: file='Adventurer.class'}

`setAttackStrategy`, `setMovingStrategy` 을 통해 공격과 이동 전략을 주입 받고 `AttackStrategy`, `MovingStrategy` 인터페이스에서 공격과 이동 전략을 가져올 수 있다.

```java
@import
public class Main {
  public static void main(String[] args) {
    Adventurer warrior = new Warrior("전사");
    Adventurer magician = new Magician("마법사");
    Adventurer magicWarrior = new MagicWarrior("마검사");

    warrior.setAttackStrategy(new FireBallStrategy());
    warrior.setMovingStrategy(new WalkingStrategy());

    magician.setAttackStrategy(new SmashStrategy());
    magician.setMovingStrategy(new TeleportStrategy());

    magicWarrior.setAttackStrategy(new SmashStrategy());
    magicWarrior.setMovingStrategy(new TeleportStrategy());

    System.out.println("나는 " + warrior.getJob());
    warrior.attack();
    warrior.move();

    System.out.println("나는 " + magician.getJob());
    magician.attack();
    magician.move();

    System.out.println("나는 " + magicWarrior.getJob());
    magicWarrior.attack();
    magicWarrior.move();
  }
}
```
{: file='Main.class'}

이제 `setAttackStrategy` 와 `setMovingStrategy`의 전략을 통해 각 직업군 마다 스킬을 개방 폐쇄 원칙을 어기지 않고 변경할 수 있고, 중복되는 코드 없이 스킬을 추가하거나 변경할 수 있게 되었다.
