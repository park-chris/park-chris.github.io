---
layout: post
date: 2024-03-11
title: "[Android] 빌드 에러 - NoSuchAlgorithmException: Algorithm HmaxPBESHA256 not avaliable"
tags: [Android, Kotlin, 에러, ]
categories: [Android, ]
---







## 에러 및 원인


---


AAB로 앱을 빌드하고 있는 데 갑자기 에러가 발생했다.


Android Studio 업데이트 전에는 빌드하는 데 문제가 없었다. 그래서 막연히 Android Studio 버전 문제라고 생각했다.



에러메세지는 위와 같이 떴다. 에러 메세지의 마지막 줄을 보면 `NoSuchAlgorithmException: Algorithm HmaxPBESHA256 not avaliable`을 볼 수 있다. 이거 때문에 KeyStore에서 Key를 읽는게 실패했다는 뜻이다.


![0](/assets/img/GBD15/0.png)


우선 HmaxPBESHA256가 그렇다면 뭔지부터 확인해봤다. HmaxPBESHA256은 해시 기반 메세지 인증 코드(HMAC)를 사용하여 SHA-256 해시 함수를 암호화하는 기술이라고 한다. 이 알고리즘은 JDK 8 이상에서 지원한다. 최근 JDK에서도 이 알고리즘을 지원하다고 한다.


나는 각 버전마다 알고리즘이 동일하지가 않을 수 있다고 생각이 들었고 첫번째로 한 것은 JDK 버전을 확인했다. 동일하게 JDK 11.0.8을 사용했지만 이 방법으로는 계속 에러가 났다.


만약 JDK 버전을 동일하게 세팅을 했는데도 에러가 생긴다면 아래 방법을 참조한다.



## 해결 방법


---


우선 JDK 버전을 동일하게 설정했지만 같은 에러가 난다면 직접 JDK 경로를 지정해서 key를 생성한다. windows라면 cmd를 켜고 Mac이면 터미널을 연다. 아니면 Android Studio에 있는 Terminal에서 실행해도 무관하다.



### 1. JDK 경로 지정해서 Keystore 및 Key 생성


**명령어**


```shell
[JDK 경로]\bin\keytool -genkey -v -keystore [Keystore 경로] -keyalg RSA -keysize 2048 -validity 9125 -alias [키 이름] -storetype JKS
```
  



**예시**



 
```shell
C:\Java\jdk-11\bin\keytool  -genkey -v -keystore "D:\androidApp\android_project_key_store/todayPrice.jks" -keyalg RSA -keysize 2048 -validity 9125 -alias tp_upload_key -storetype JKS
```
  



**<옵션 설명>**

- `genkey` : 새 키 쌍을 생성
- `v` : 상세한 출력 표시
- `keystore` : 생성할 keystore의 경로와 이름
- `keyalg` : 키 생성 알고리즘 지정
- `keysize` : 생성할 키의 비트 수
- `validity` : 생성할 키의 유효 기간 (단위 : 일, day)
- `alias` : keystore 내에서 사용할 key 별칭
- `storetype` : keystore 유형 지정


### 2. Keystore 형식 변경


1번 방법으로도 계속 에러가 뜬다면 storetype을 pkcs12로 바꿔본다.


**명령어**



  
```shell
keytool -importkeystore -srckeystore [Keystore 경로] -destkeystore [Keystore 경로] -deststoretype pkcs12
```
  



**예시**



  
```shell
keytool -importkeystore -srckeystore D:\androidApp\android_project_key_store\todayPrice.jks -destkeystore D:\androidApp\android_project_key_store\todayPrice.jks -deststoretype pkcs12
```
  



**<옵션 설명>**

- `importkeystore` : 다른 형식의 keystore로 keystore를 가져오는 옵션
- `srckeystore` : 원본 keystore의 경로와 이름
- `destkeystore` : 대상 keystore의 경로와 이름
- `deststoretype` : 대상 keystore 유형 지정

이렇게 해결 방법을 알아봤다. 나는 위 해결 방법까지 가서 해결 됐지만 혹시 나중에 다른 방법이 있다면 추가로 기록해야겠다.

