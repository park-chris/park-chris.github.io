---
layout: post
date: 2022-01-20
title: "[Toast] setGravity 에러 ... setGravity() shouldn't be called on text toasts, the values won't be used"
tags: [Android, Kotlin, 에러 해결, ]
categories: [Android 에러 모음, ]
---


---



## CASE


---

- 에뮬레이터 API 30 버전에서 Toast.setGravity 기능 실행 시 적용이 안되는 현상이 발생
- setGravity로 Toast의 위치를 바꾸었지만 그대로 하단에서 출력되는 현상
- 로그캣에서는 아래와 같은 문구 출력

	![0](/assets/img/GBD32/0.png)



## CAUSE


---

- developers 사이트의 공식문서에서 setGravity 부분의 Warning에서 이유가 나옴
- R 버전 이상부터는 이 setGravity 메소드는 동작하지 않는다고 명시되어 있음
- 하위버전인 Pie(API 28) 버전에서 확인결과 소스가 잘 작동되는 것을 확인
- 결론적으로 R(API 30) 버전부터 setGravity 메소드는 사용못함

![1](/assets/img/GBD32/1.png)



## Reference


---

- [https://developer.android.com/reference/kotlin/android/widget/Toast#setGravity(kotlin.Int, kotlin.Int, kotlin.Int)](https://developer.android.com/reference/kotlin/android/widget/Toast#setGravity(kotlin.Int,%20kotlin.Int,%20kotlin.Int))
