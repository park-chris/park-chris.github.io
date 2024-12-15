---
layout: post
date: 2022-04-26
title: "KaKaoTalk SDK 설정 및 연동"
tags: [Android Studio, Kotlin, Android, Kakao SDK, ]
categories: [Android, ]
---


---



## 카카오톡 로그인 연동


![0](/assets/img/GBD45/0.png)



#### Android Studio 


1.settings.gradle 


	
{% raw %}
```kotlin
	dependencyResolutionManagement {
	    repositoriesMode.set(RepositoriesMode.FAIL_ON_PROJECT_REPOS)
	    repositories {
	        ...
	        maven { url 'https://devrepo.kakao.com/nexus/content/groups/public/' }
	    }
	}
```
{% endraw %}



2.build.gradle(:app)


	
{% raw %}
```kotlin
	dependencies {
	    ...
	    implementation "com.kakao.sdk:v2-user:2.12.1"
	}
```
{% endraw %}



3.AndroidManifest.xml 에서 인터넷 퍼미션 허용


	
{% raw %}
```kotlin
	<manifest xmlns:android="http://schemas.android.com/apk/res/android"
	    package="com.example.sample">
	 
	    <!-- 인터넷 사용 권한 설정-->
	    <uses-permission android:name="android.permission.INTERNET" />
	
	    <application
	    ...
```
{% endraw %}



4.키 해시 값 출력

	- release

	
{% raw %}
```kotlin
	keytool -exportcert -alias [저장소 별칭] -keystore [저장소 path 확장자 jks] | openssl sha1  -binary | openssl base64
```
{% endraw %}



	![1](/assets/img/GBD45/1.png)

	- debug

	
{% raw %}
```kotlin
	keytool -exportcert -alias androiddebugkey -keystore [저장소 path 확장자 jks] -storepass anbdroid -keypass android |  openssl sha1 -binary | openssl base64
```
{% endraw %}



	![2](/assets/img/GBD45/2.png)



#### KaKao Developers 애플리케이션 콘솔


1.내 애플리케이션 > 플랫폼 > Android에 위에서 출력한 해시 값 등록


	![3](/assets/img/GBD45/3.png)


2.카카오 로그인 활성화를 위해 `내 애플리케이션 > 카카오 로그인` 에서 활성화 설정의 상태를 ON으로 설정


	![4](/assets/img/GBD45/4.png)


3.정보 수집을 위해 `내 애플리케이션 > 카카오 로그인 > 동의항목` 에서 필요한 개인정보 체크


	![5](/assets/img/GBD45/5.png)



#### Android Studio


1.AnbdroidManifest.xml에 activity 추가

	- 네이티브 앱 키는 카카오 디벨로퍼스 > 내 애플리케이션 > 요약 정보에서 확인 가능
	- `scheme` 속성의 값은 "kakao${NATIVE_APP_KEY}" 형식으로 입력한다. 예를 들어 네이티브 앱 키가 "123456789"라면 "kakao123456789"를 입력한다

	
{% raw %}
```kotlin
	<activity
	            android:name="com.kakao.sdk.auth.AuthCodeHandlerActivity"
	            android:exported="true">
	
	            <intent-filter>
	                <action android:name="android.intent.action.VIEW" />
	                <category android:name="android.intent.category.DEFAULT" />
	                <category android:name="android.intent.category.BROWSABLE" />
	
	                <data android:host="oauth"
	                    android:scheme="kakao[네이티브 앱 키]" />
	            </intent-filter>
	        </activity>
```
{% endraw %}



2.카카오톡 SDK 초기화를 하기위해 초기 실행하는 activity에 초기화 코드를 적고, AndroidManifest.xml에 초기화를 수행하는 클래스 명 기재

	- LaunchApplication.kt

		
{% raw %}
```kotlin
		class LaunchApplication: Application() {
		
		    override fun onCreate() {
		        super.onCreate()
		        UserRepository.initialize(this)
		        SoundRepository.initialize(this)
		
		        // KaKao SDK initialized
		        KakaoSdk.init(this, "4e3343f68ffe29a64b911140ebabb567")
		    }
		
		}
```
{% endraw %}


	- AndroidManifest.xml

		
{% raw %}
```kotlin
		<?xml version="1.0" encoding="utf-8"?>
		<manifest xmlns:android="http://schemas.android.com/apk/res/android"
		    xmlns:tools="http://schemas.android.com/tools"
		    package="com.dn.digitalnutrition">
		
		    <application
		        android:name=".LaunchApplication"
		        android:allowBackup="true"
		        ...
		    </application>
		
		</manifest>
```
{% endraw %}



	3.카카오톡으로 로그인하기


		
{% raw %}
```kotlin
		private fun kakaoLogin() {
		
		        if (UserApiClient.instance.isKakaoTalkLoginAvailable(this)) {
		            UserApiClient.instance.loginWithKakaoTalk(this) { token, error ->
		
		                if (error != null) {
		                    Log.e(TAG, "로그인 실패", error)
		                    if (error is ClientError && error.reason == ClientErrorCause.Cancelled) {
		                        return@loginWithKakaoTalk
		                    }
		                    loginWithKaKaoAccount(this)
		                }
		                else if (token != null) {
		                    Log.i(TAG, "로그인 성공 ${token.accessToken}")
		                }
		
		            }
		        } else {
		            loginWithKaKaoAccount(this)
		        }
		
		    }
		
		private fun loginWithKaKaoAccount(context: Context) {
		        UserApiClient.instance.loginWithKakaoAccount(context) { token: OAuthToken?, error: Throwable? ->
		            if (error != null) {
		                Log.e(TAG, "로그인 실패", error)
		            }
		            else if (token != null) {
		                Log.i(TAG, "로그인 성공 ${token.idToken}")
		
		                Log.d(TAG, "token: $token")
		
		            }
		        }
		    }
```
{% endraw %}


