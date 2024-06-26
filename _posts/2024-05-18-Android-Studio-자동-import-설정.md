---
layout: post
date: 2024-05-18
title: "Android Studio 자동 import 설정"
tags: [설정, ]
categories: [Android, ]
---



## 프로젝트 자동 import 설정하기


---

- `Android Studio` > `File` > `Settings` > `Editor`> `General` > `Auto Import` 를 확장하면 아래 그림과 같이 자동 import를 설정할 수 있는 옵션이 나타남 → 안드로이드 스튜디오에서 작업하는 모든 프로젝트에 적용됨
- 파란색 박스의 옵션은 Java를 사용할때, Kotlin을 사용할때는 빨간색 박스의 옵션을 체크하면 됨
- _Add unambiguous imports on the fly_ : 체크하면 외부 패키지의 클래스나 인터페이스를 사용하는 코드를 작성하는 시점에서 안드로이드 스튜디오가 해당 클래스나 인터페이스의 패키지를 찾아 자동으로 import 문을 추가한다.
- _Optimize imports on the fly_ : 체크하면 프로젝트를 빌드할 때 사용하지 않는 패키지의 import 문이 소스 코드에서 자동 삭제된다.

![0](/assets/img/GBD5/0.png)

