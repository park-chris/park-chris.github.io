---
layout: post
date: 2022-01-20
title: "Anti-virus 오류 ... Your anti-virus program might be impacting your build performance. Android Studio checked the following directories"
tags: [Android, Kotlin, 에러 해결, ]
categories: [Android 에러 모음, ]
---


---



## **ERROR**

- AVD(Android Virtual Device) 실행 시 발생

![0](/assets/img/GBD34/0.png)


> ⚠️ Your anti-virus program might be impacting your build performance. Android Studio checked the following directories: C:\Users\timagate\AppData\Local\Android\Sdk C:\Users\timagate\AppData\Local\Google\AndroidStudio2020.3 D:\AndroidApp\firstTest C:\Users\timagate\.gradle



## **CAUSE**


> ❓ 



## **SOLUTION**


> 🛠 Windows 보안 > 바이러스 및 위협 방지 > 바이러스 및 위협 방지 설정 [설정관리] > 제어 [제외 추가 또는 제거] > [+ 제외 사항 추가] > 안내문구로 뜬 폴더 추가


![1](/assets/img/GBD34/1.png)

