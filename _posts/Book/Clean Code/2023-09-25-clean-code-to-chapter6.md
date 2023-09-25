---
title: Clean Code ~Ch.6
author: ryan park
date: 2023-09-25 +0800
categories: [Book, Clean Code]
tags: [clean_code, java]
image:
  path: https://github.com/kiryanchi/kiryanchi.github.io/assets/50714602/d21330ba-9c9e-49cd-aa73-bf900c7cc7ba
  alt: clean code text image
---

Spring과 Kotlin을 공부하면서 경험적으로 느꼈던 생각들을 텍스트로 읽은 챕터였다.
개인적으로 Java로 SpringApplication을 작성할 때, Lombok을 이용해 Getter/Setter를 습관적으로 다는 것이 싫다.
내 주언어는 Python이고 Spring을 공부가 아닌 첫 프로젝트로 Kotlin을 채택했던터라 Getter/Setter를 사용하는 것이 싫었다. (사실 핑계같지만..)
Setter 는 값을 설정할 때 조건을 추가한다는 관점에서 이해는 갔지만 Getter는 Setter를 사용하기위해 어쩔 수 없이 사용되는 느낌...?

이 챕터는 어느정도 이것에 대한 두루뭉실한 생각을 지우며 객체와 자료구조에 대해 고찰해볼 수 있었다.

# Chapter.6

---

> _사용자가 구현을 모른 채 자료의 해심을 조작할 수 있어야 진정한 의미의 클래스다._
> _개발자는 객체가 포함하는 자료를 표현할 가장 좋은 방법을 **심각하게** 고민해야 한다._

Getter/ Setter를 간접적으로 비판하고 있다. 책에서는 **변수 사이에 함수라는 게층을 넣는다고 구현이 저절로 감춰지지 않는다.** 라 한다.

함수를 만들 때 이름도 중요하다고 언급하고 있다.

```java
public interface Vehicle {
  // bad
  double getFuelTankCapacityInGallons();
  double getGallonsOfGasoline();

  // good
  double getPercentFuelRemaining();
}
```

bad 예시는 연료의 양을 구체적으로 어디서 가져오는 지 이름에 드러나있다. 이것또한 내부의 자료를 외부로 공개하는 일이다.
good 예시처럼 우리는 그저 자동차 연료 상태를 알기만 하면 된다.

<br>

> _분별 있는 프로그래머는 모든 것이 객체라는 생각이 **미신**임을 잘 안다._

객체와 자료구조의 차이점을 설명하며 객체를 객체지향 코드, 자료구조를 절차적인 코드로 설명한다.

```java
public double area(Object shape) throws NoSuchShapeException
{
  if (shape instanceof Square) {
    ...
  }
  else if (shape instanceof Rectangle) {
    ...
  }
  else if (shape instanceof Circle) {
    ...
  }
  throw new NoSuchShapeException();
}
```

이 코드를 보고나니 XPA 코드가 생각이 났다.

```python
def _switchWidget(button):
    buttonIndex = {
        "Home": 0,
        "Work": 1,
        "Setting": 2,
    }

    Log.info(self, f"{button.text()} 전환")
    self.mainStackedWidget.setCurrentIndex(buttonIndex[button.text()])
```

button에 적혀있는 text를 읽어서 index에 맞는 창으로 widget을 변경하는 코드다.
저 코드를 작성할 당시에 절차니 객체니 생각하지도 않았고 그저 이렇게 하면 깔끔하겠다고 생각해 작성했다.

나름 오랜 시간 고민한 뒤에 작성한 로직이었다. 그만큼 기억에 남아서 아마 이 예제를 읽고 바로 생각이 난 것 같다.
혼자서 이런 코드를 작성했다는 사실에 기쁜 한편 정말 오랜 시간 고민했었던 문제인데 책으로 읽게돼 허무하기도 했다.
그래도 허무할지라도 뒤에도 이런 내용이 더 많이 보이면 좋겠다.

---

짧게 정리가 가능한 챕터였다. 객체와 자료구조에 대해 더 고찰하고 코드를 작성할 때, 두 개를 확실히 분리하라는 저자의 의도가 분명히 보였다.
