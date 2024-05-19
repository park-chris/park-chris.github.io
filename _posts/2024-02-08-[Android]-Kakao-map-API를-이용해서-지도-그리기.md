---
layout: post
date: 2024-02-08
title: "[Android] Kakao map API를 이용해서 지도 그리기"
tags: [Android, Kotlin, Kakao API, Kakao Map API, ]
categories: [Android, ]
---


---


카카오 지도 API를 사용해서 앱에 지도를 그려볼것이다.


대표적인 지도 API에는 구글, 네이버, 카카오가 있다. 이번에 카카오 인증을 써야해서 지도 API도 카카오 지도를 사용하기로 결정했다.


카카오 지도 API는 구글이나 네이버에서 제공하는 지도 API와 비슷한데 다른 점은 애뮬레이터에서는 작동이 안되고 실물폰에서만 된다는 것이다. 이 부분도 설정을 추가하여 애뮬레이터에서도 작동할 수 있도록 하겠다.



## Android Studio 설정


---

1. `settings.gradle` 파일에 kakao SDK 추가 후 `Sync now`


{% raw %}
```kotlin
plugins {
    ...
    id 'org.jetbrains.kotlin.android' version '1.7.10' apply false
    id 'com.google.gms.google-services' version '4.3.13' apply false
}
```
{% endraw %}


1. 앱 단의 build.gradle(`build.gradle (:app)`)에 kakao SDK 추가 후 `Sync now`


{% raw %}
```text
android {
    ...

    defaultConfig {
        ...
// 애뮬레이터에서 카카오 지도 맵을 사용하기 위한 설정
        ndk {
            abiFilters 'arm64-v8a', 'armeabi-v7a'
        }

    }
}

dependencies {
  	...
	implementation "com.kakao.sdk:v2-user:2.12.1"// 카카오 로그인 모듈 (keyHash값 때문에 설정)
	implementation 'com.kakao.maps.open:android:2.6.0'// 카카오 맵 API
}
```
{% endraw %}


1. `res/values` 폴더 마우스 우클릭 > New > Values Resource File 클릭하고 파일 이름은 api_key로 한다.

	카카오나 기타 API에 필요한 Key 값을 모아놓고 관리하기 위해 만들었다. `.gitignore`에 이 파일을 추가해서 저장소에 안올라가도록 설정한다.


**res/values/api_key.xml**



{% raw %}
```xml
<resources>
    <string name="kakao_api_key">api 키 값 자리</string>
</resources>
```
{% endraw %}


1. AndroidManifest.xml에 Internet 허용을 하고 <application 안에 meta-data를 추가한다.

**AndroidManifest.xml**



{% raw %}
```xml
<?xml version="1.0" encoding="utf-8"?>
<manifest xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools"
    package="com.crystal.todayprice">

    <uses-permission android:name="android.permission.INTERNET" />

    <application
        android:usesCleartextTraffic="true">
        ...

        <meta-data
            android:name="com.kakao.vectormap.APP_KEY"
            android:value="@string/kakao_api_key"/>
    </application>
</manifest>
```
{% endraw %}




## Kakao Developer 앱 등록


---


1.Kakao Developer([https://developers.kakao.com/](https://developers.kakao.com/))에서 로그인 후 내 애플리케이션 클릭


![0](/assets/img/GBD20/0.png)


2.애플리케이션 추가하기


**설정 항목**


- 앱 아이콘 (선택) : Kakao Developer에서만 쓰이는 앱 아이콘이다. 없어도 된다.


- 앱 이름 (필수) : Kakao Developer에서 쓰이는 앱 이름


- 사업자명 (필수) : 사업자명 또는 내 이름


- 카테고리 (필수) : 해당하는 카테고리 선택


![1](/assets/img/GBD20/1.png)


3.전체 애플리케이션 > 만든 애플리케이션 > 플랫폼 > Android 플랫폼 등록 클릭


![2](/assets/img/GBD20/2.png)


4.Android 플랫폼 등록 시 정보 입력 후 저장


**입력 정보**


- 패키지명 : 프로젝트의 패키지를 그대로 복사해오면 된다. (예시 : com.test.TestApp)


- 마켓 URL : 자동 생성이니 넘어간다.


- 키 해시 : 이미지 바로 아래 방법 참고


![3](/assets/img/GBD20/3.png)


**키 해시 구하기**


- 아무 Activity에서나 아래 코드 실행해서 로그로 찍히는 KeyHash값을 얻으면 된다.



{% raw %}
```text
import com.kakao.sdk.common.util.Utility

var keyHash = Utility.getKeyHash(this)
Log.d("My Key Hash: $keyHash")
```
{% endraw %}




## 안드로이드에서 지도 그리기


---


activity_market.xml



{% raw %}
```xml
<com.kakao.vectormap.MapView
    android:id="@+id/mapView"
    android:layout_width="match_parent"
    app:layout_constraintTop_toBottomOf="@id/descriptionTextView"
    android:layout_marginHorizontal="10dp"
    android:background="@color/hint"
    android:layout_marginTop="20dp"
    android:layout_marginBottom="100dp"
    app:layout_constraintBottom_toBottomOf="parent"
    android:layout_height="300dp" />
```
{% endraw %}



MarketActivity.kt



{% raw %}
```kotlin
override fun onResume() {
    binding.mapView.start(object : MapLifeCycleCallback() {
        override fun onMapDestroy() {
        	Log.e(TAG, "onMapDestroy")
        }

        override fun onMapError(error: Exception?) {
            Log.e(TAG, "onMApError", error)

        }

    }, object : KakaoMapReadyCallback() {
        override fun onMapReady(kakaoMap: KakaoMap) {
            Log.e(TAG, "onMapReady")
        }

        override fun getPosition(): LatLng {
// market이라는 변수가 없다면 실행
            market ?: return super.getPosition()

// market 변수가 있다면 market data의 경도와 위도를 넣어서 띄워주면 된다.
            return LatLng.from(market!!.latitude, market!!.longitude)
        }
    })
}
```
{% endraw %}




## 지도에서 마커 찍기


---


지도에 내가 원하는 drawable을 이용해서 마커를 찍어보자.



{% raw %}
```kotlin
binding.mapView.start(object : KakaoMapReadyCallback() {
            override fun onMapReady(kakaoMap: KakaoMap) {
// 스타일 지정. LabelStyle.from()안에 원하는 이미지 넣기
                val style = kakaoMap.labelManager?.addLabelStyles(LabelStyles.from(LabelStyle.from(R.drawable.ic_mark)))
// 라벨 옵션 지정. 위경도와 스타일 넣기
                val options = LabelOptions.from(LatLng.from(market!!.latitude, market!!.longitude)).setStyles(style)
// 레이어 가져오기
                val layer = kakaoMap.labelManager?.layer
// 레이어에 라벨 추가
                layer?.addLabel(options)
            }
            override fun getPosition(): LatLng {
                market ?: return super.getPosition()

// 카메라 위치 지정
                return LatLng.from(market!!.latitude, market!!.longitude)
            }
        })
```
{% endraw %}




## 결과물


---


아주 잘 지도가 뜨고 있다.


![4](/assets/img/GBD20/4.png)_라벨 X_


![5](/assets/img/GBD20/5.png)_라벨 O_



## 참고 문헌


---

- Kakao Developer : [https://developers.kakao.com/](https://developers.kakao.com/)
- Kakao Map Document : [https://apis.map.kakao.com/android_v2/](https://apis.map.kakao.com/android_v2/)
