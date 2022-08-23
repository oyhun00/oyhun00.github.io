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

## 🙋‍♂️ 특정 유저의 정보 조회하기

어느 웹 어플리케이션에서 유저 목록을 확인할 수 있는 테이블이 있다. 이 테이블에서 특정 유저를 클릭하면 해당하는 유저 ID의 변화를 감지하여 자동으로 정보를 가져오는 기능을 구현한다.

<img src="/observer-before-1-uml.png" alt="UML">

`UserTable` 클래스에서 특정 유저를 선택하는 `selectUserId` 메서드가 실행되면 `userId` 값을 저장하고, `userInfoView.update()` 메서드를 통해 값이 변경됨을 통보함으로써 변화를 감지하도록 한다.

```java
public class UserTable {
  private String userId;
  private UserInfoView userInfoView;

  public void setUserInfoView(UserInfoView userInfoView) {
    this.userInfoView = userInfoView;
  }

  public String getUserId() {
    return userId;
  }

  public void selectUserId(String userId) {
    this.userId = userId;
    userInfoView.update();
  }
}
```

```java
public class UserInfoView {
  private UserTable userTable;

  public UserInfoView(UserTable userTable) {
    this.userTable = userTable;
  }

  public void display(String userId) {
    System.out.println("================ " + userId + " User Info ===============");
    System.out.println("User ID: ...");
    System.out.println("User Name: ...");
    System.out.println("===============================================");
    System.out.println("");
  }

  public void update() {
    String userId = userTable.getUserId();
    display(userId);
  }
}
```

```java
public static void main(String[] args) {
  UserTable userTable = new UserTable();
  UserInfoView userInfoView = new UserInfoView(userTable);

  userTable.setUserInfoView(userInfoView);
  userTable.selectUserId("U001");
}
```


```console
================ U001 User Info ===============
User ID: ...
User Name: ...
===============================================
```

의도했던 대로 유저를 선택하면 변화된 `userId` 값을 감지하고 해당 유저의 기본 정보를 출력하게 되었다.


## 💣 문제점

하지만 만약 우리가 유저 목록 테이블에서 선택한 유저 ID가 변화할 때, 해당 유저의 기본 정보뿐만 아니라, 결제 기록, 활동 로그 등 다양한 데이터를 가져오게 하고 싶다면 `UserTable` 클래스를 직접 수정해야 할 것이다.

```java
public class UserTable {
  private String userId;
  private UserInfoView userInfoView;
  private UserLogView userLogView;

  public void setUserInfoView(UserInfoView userInfoView) {
    this.userInfoView = userInfoView;
  }

  public void setUserLogView(UserLogView userLogView) {
    this.userLogView = userLogView;
  }

  public String getUserId() {
    return userId;
  }

  public void selectUserId(String userId) {
    this.userId = userId;
    userInfoView.update();
    userLogView.update();
  }
}
```

```java
public class UserLogView {
  private UserTable userTable;

  public UserLogView(UserTable userTable) {
    this.userTable = userTable;
  }

  public void display(String userId) {
    System.out.println("================ " + userId + " User Log ===============");
    System.out.println("2022-08-21 12:45:50: ...");
    System.out.println("2022-08-19 09:21:00: ...");
    System.out.println("2022-08-17 06:01:21: ...");
    System.out.println("2022-08-16 14:58:44: ...");
    System.out.println("2022-08-15 16:22:07: ...");
    System.out.println("2022-08-14 20:11:54: ...");
    System.out.println("===============================================");
    System.out.println("");
  }

  public void update() {
    String userId = userTable.getUserId();
    display(userId);
  }
}
```

```java
public static void main(String[] args) {
  UserTable userTable = new UserTable();
  UserInfoView userInfoView = new UserInfoView(userTable);
  UserLogView userLogView = new UserLogView(userTable);

  userTable.setUserInfoView(userInfoView);
  userTable.setUserLogView(userLogView);
  
  userTable.selectUserId("U001");
}
```


```console
================ U001 User Info ===============
User ID: ...
User Name: ...
===============================================

================ U001 User Log ===============
2022-08-21 12:45:50: ...
2022-08-19 09:21:00: ...
2022-08-17 06:01:21: ...
2022-08-16 14:58:44: ...
2022-08-15 16:22:07: ...
2022-08-14 20:11:54: ...
===============================================
```

새로운 유저 정보 View를 추가하여 적용하기 위해 `UserTable` 클래스를 직접적으로 수정하게 됐다. 이렇게 작성하게 된다면 해당 유저의 다른 정보 조회의 기능이 추가 될 때마다 `UserTable` 클래스를 반복적으로 수정해야할 것이다. 이는 곧 **개방 폐쇄 원칙(Open Close Principle)**에 위배하게 된다. 

## 💡 해결법
이렇게 정보 조회의 기능이 추가되거나 수정이 되어도, `UserTable` 클래스를 변경하지 않고 그대로 재사용 할 수 있어야 한다. **옵저버 패턴**을 적용하여 이 문제를 해결해보자.

<img src="/observer-uml.png" alt="Observer Pattern UML">

앞서 말했듯 옵저버 패턴은 상태가 변하는 특정 객체를 옵저버들이 구독하고, 특정 객체에서 상태의 변화가 나타날 때 자신을 구독하고 있는 옵저버들에게 상태가 변화 됐음을 통지하는 패턴이라 설명했다. `Subject` 클래스를 통해 변화하는 객체를 감시하는 옵저버들을 구독시키거나, 해제한다. 그리고 옵저버가 될 객체는 `observer` 인터페이스를 구현하도록 하고, 선택한 유저 ID가 변화할 때 즉,  `UserTable` 클래스의 `selectUserId` 메소드가 실행되면, `Subject` 클래스의 `notifyObserver` 메소드를 실행하여, 구독하고 있는 옵저버들에게 변화를 통지한다. 통지받은 옵저버들은 `Observer` 인터페이스를 통해 `update` 메소드를 실행하게 된다.


```java
public interface Observer {
  public void update();
}
```
통지 대상을 인터페이스로 추상화하고, 변화를 통지 받았을 때 처리하는 `update` 메서드를 추가한다.

<br>

```java
public abstract class Subject {
  private List<Observer> observers = new ArrayList<Observer>();

  public void subscribe(Observer observer) {
    observers.add(observer);
  };

  public void unsubscribe(Observer observer) {
    observers.remove(observer);
  };

  public void notifyObservers() {
    for (Observer o:observers) {
      o.update();
    }
  };
}

```
통지 대상인 옵저버를 구독하고, 해제하는 기능을 구현한다.

<br>

```java
public class UserTable extends Subject {
  private String userId;

  public String getUserId() {
    return userId;
  }

  public void selectUserId(String userId) {
    this.userId = userId;
    notifyObservers();
  }
}

```
`userId`가 변화하면 각 옵저버들에게 변화를 알린다.

<br>

```java
public class UserInfoView implements Observer {
  private UserTable userTable;

  public UserInfoView(UserTable userTable) {
    this.userTable = userTable;
  }

  public void display(String userId) {
    System.out.println("================ " + userId + " User Info ===============");
    System.out.println("User ID: ...");
    System.out.println("User Name: ...");
    System.out.println("===============================================");
    System.out.println("");
  }

  public void update() {
    String userId = userTable.getUserId();
    display(userId);
  }
}
```


```java
public class UserLogView implements Observer {
  private UserTable userTable;

  public UserLogView(UserTable userTable) {
    this.userTable = userTable;
  }

  public void display(String userId) {
    System.out.println("================ " + userId + " User Log ===============");
    System.out.println("2022-08-21 12:45:50: ...");
    System.out.println("2022-08-19 09:21:00: ...");
    System.out.println("2022-08-17 06:01:21: ...");
    System.out.println("2022-08-16 14:58:44: ...");
    System.out.println("2022-08-15 16:22:07: ...");
    System.out.println("2022-08-14 20:11:54: ...");
    System.out.println("===============================================");
    System.out.println("");
  }

  public void update() {
    String userId = userTable.getUserId();
    display(userId);
  }
}

```

```java
public static void main(String[] args) {
  UserTable userTable = new UserTable();
  UserInfoView userInfoView = new UserInfoView(userTable);
  UserLogView userLogView = new UserLogView(userTable);

  userTable.subscribe(userInfoView);
  userTable.subscribe(userLogView);
  
  userTable.selectUserId("U001");

  userTable.unsubscribe(userLogView);

  userTable.selectUserId("U002");
}
```
상속받은 `Observer` 인터페이스의 `update` 메소드를 구현함으로써 옵저버가 된다.

<br>

```console
================ U001 User Info ===============
User ID: ...
User Name: ...
===============================================

================ U001 User Log ===============
2022-08-21 12:45:50: ...
2022-08-19 09:21:00: ...
2022-08-17 06:01:21: ...
2022-08-16 14:58:44: ...
2022-08-15 16:22:07: ...
2022-08-14 20:11:54: ...
===============================================

================ U002 User Info ===============
User ID: ...
User Name: ...
===============================================
```

 이제 `UserTable` 클래스는 `Subject` 클래스를 상속받아 변화하는 객체를 구독하고 해지할 수 있도록 `Subject` 클래스를 통해 관리함으로써, 관심 객체를 직접 참조할 필요가 없게되었고, `UserTable` 클래스를 직접 수정하지 않고도 옵저버를 추가, 제거가 가능해져 객체가 느슨한 결합이 되었다고 볼 수 있을 것이다.