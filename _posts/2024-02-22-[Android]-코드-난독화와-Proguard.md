---
layout: post
date: 2024-02-22
title: "[Android] 코드 난독화와 Proguard"
tags: [설정, Android, Kotlin, 보안, 코드 난독화, ]
categories: [Android, ]
---


---



## 코드 난독화와 Proguard


---


코드 난독화는 프로그래밍 언어로 작성된 코드에 대해 읽기 어렵게 만드는 작업이다. 코드의 가독성을 낮춰 리버스엔지니어링의 대비책으로 사용된다.


Proguard는 자바 기반 코드를 난독화해주는 도구로, 난독화(Renaming), 용량 축소, 코드 추소, 최적화 등의 기능을 제공한다. 여기서 Proguard의 난독화는 식별자 전환, 클래스/메소드의 이름을 a,b,c 등 의미없는 이름으로 대체해주는 기능이다.


난독화의 종류는 다양하고 Proguard가 제공하는 난독화 기능은 정말 보안에 있어서 최소한의 성의다.



## Proguard 설정 방법


---


설정은 매우 쉽다. App의 `build.gradle`에 proguard의 설정을 true로 변경해주면 된다.


**bulid.gradle (:app)**



{% raw %}
```kotlin
android {
    ...
    buildTypes {
        release {
            minifyEnabled true
            proguardFiles getDefaultProguardFile('proguard-android-optimize.txt'), 'proguard-rules.pro'
            proguardFiles('proguard-rules.pro')
        }
        debug {
            minifyEnabled true
            proguardFiles getDefaultProguardFile('proguard-android-optimize.txt'), 'proguard-rules.pro'
            proguardFiles('proguard-rules.pro')
        }
    }
}
```
{% endraw %}



위에 코드는 release와 debug 각각에 대한 proguard 설정이다. release와 debug에 대한 proguard 설정을 활성화한다는 의미다.


<옵션 설명>

- minifyEnabled : true면 Proguard 활성화 (값 : true, false)
- proguardFiles getDefaultProguardFile : 기본 Proguard 설정
- proguardFiles : proguard 파일 추가


## 자주 사용되는 라이브러리 Proguard rules


---


Proguard를 적용하면 이제 신경써줘야 하는 부분이 있다. 바로 라이브러리다. Proguard가 클래스나 메소드의 이름을 바꾸는 원리로 난독화를 하다보니 밖에서 끌어다쓰는 라이브러리에 대한 제외 규칙을 설정해줘야한다.


아래는 자주 사용하는 라이브러리에 대한 제외 규칙이다.


**proguard-rules.pro**



{% raw %}
```kotlin
# kakao sdk
-keep class com.kakao.sdk.**.model.* { <fields>; }
-keep class * extends com.google.gson.TypeAdapter
-dontwarn org.bouncycastle.jsse.**
-dontwarn org.conscrypt.*
-dontwarn org.openjsse.**

# kakao map
-keep class com.kakao.vectormap.** { *; }
-keep interface com.kakao.vectormap.**

# firebase
-keepattributes *Annotation*
-keepattributes Signature
-keep class com.google.android.gms.** { *; }
-keep class com.google.firebase.** { *; }
-keepclassmembers class com.crystal.todayprice.data.** {
  *;
}

# okHttp, retrofit
-dontwarn okhttp3.**
-dontwarn okio.**
-dontnote okhttp3.**
-dontnote retrofit2.Platform
-keepattributes Exceptions
```
{% endraw %}



<옵션 설명>

- dontwarn 패키지명.** : 지정해서 경고 무시
- keep class 패키지명.** : 난독화가 필요하지 않은 경우
- ignorewarnings : 경고 무시
- dontoptimize : 최적화 하지 않기
- dontshrink : 사용하지 않는 메소드 유지
- keepclassmembers : 특정 클래스 멤버 원상태 유지
- keepattributes : 내부 클래스 원상태 유지 적용

여기서 firebase의 real-time database나 firestore를 사용할 때 역직렬화를 사용하는 사람은 `-keepclassmembers class com.crystal.todayprice.data.** { *;}`을 사용해야 한다. 여기서 `com.crystal.todayprice.data`은 내 data class가 있는 패키지명이고 본인들이 설정한 패키지 명으로 바꿔줘야 한다.



## 난독화 확인 방법


---


이제 난독화가 제대로 됐는지 확인해야 한다. 난독화 확인 방법으로는 2가지가 있다. 리버스 엔지니어링을 하기 위해 Dex2Jar, JD-GUI 등 툴을 이용해서 apk 파일을 프로그래밍 코드까지 복원해서 확인하는 방법과 Android Studio에서 제공하는 방법이 있다.


모든 과정은 apk 파일 생성 후 진행된다.



### 방법 1 : 리버스 엔지니어링 Tool 사용 (Windows)


> 💡 Dex2Jar : apk/dex 파일을 jar 파일로 변환  
> JD-GUI : jar 파일을 UI로 확인


1. Dex2Jar 파일 다운로드 후 압축해제 (최신버전으로 다운받기를 추천한다. 옛날 버전은 최신 JDK 버전과 맞지 않아서 Jar 파일로 변환이 안된다)


- Dex2Jar Github : [https://github.com/pxb1988/dex2jar](https://github.com/pxb1988/dex2jar)


2. Dex2Jar를 이용한 apk 파일을 Jar 파일로 변환


압축해제한 폴더에서 cmd를 열고 아래 명령어 입력하면 해당 폴더에 jar 파일을 하나 생성된다. 나는 압축해제한 폴더에 apk 파일을 옮겨놓고 실행해서 파일명만 사용하면 됐다.



{% raw %}
```text
dex-tools-v2.4>d2j-dex2jar.bat [APK 파일 위치]
```
{% endraw %}



![0](/assets/img/GBD18/0.png)_명령어 결과_


3. JD-GUI 다운로드


- JD-JUI Github : [https://java-decompiler.github.io/](https://java-decompiler.github.io/)


4. 다운로드한 파일을 압축해제하면 아래와 같이 파일들이 있다. jd-gui.exe를 클릭하자.


![1](/assets/img/GBD18/1.png)


5. JD-GUI 프로그램이 켜지면서 빈 화면이 나오는 데 거기에 jar 파일을 드래그해서 옮기면 jar 파일을 확인할 수 있다.


<난독화 활성화>


![2](/assets/img/GBD18/2.png)


<난독화 비활성화>


![3](/assets/img/GBD18/3.png)


이렇게 확인하면 된다.



## **방법 2 : Android Studio 기능 활용**


Android Studio에서 제공하는 "Analyze APK" 기능이 있다. 이 기능을 활용하면 방법 1보다 훨씬 쉽게 난독화 확인이 가능하다.


1. Android Studio > Build > Analyze Apk...


![4](/assets/img/GBD18/4.png)


2. 분석할 apk 파일 선택


![5](/assets/img/GBD18/5.png)


3. 분석이 끝나면 classes.dex 파일을 찾는다. classes.dex을 누르면 아래와 같이 프로젝트 파일을 볼 수 있다. 아래 사진을 보면 왼쪽이 난독화 설정을 안한 apk 파일이고, 오른쪽이 난독화 설정을 한 apk 파일이다. 난독화 설정을 한 apk  파일을 분석했을 때 사이즈가 줄어든것을 확인할 수 있다.


![6](/assets/img/GBD18/6.png)


이렇게 Android 프로젝트의 Proguard 설정을 알아봤다.

