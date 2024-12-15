---
layout: post
date: 2023-03-07
title: "Styles & Themes"
tags: [설정, Android, Kotlin, ]
categories: [Android, ]
---


---



## Style 설정

1. res 마우스 우클릭 > New > Android Resource File 클릭 후 파일명은 styles로 생성

![0](/assets/img/GBD38/0.png)

1. styles.xml에 Attribute 등록
	- name : 아무거나 지정 가능
	- parent : 내가 지정하고 싶은 위젯 (TextView를 꾸미고 싶으면 TextView로)

	
{% raw %}
```javascript
	<?xml version="1.0" encoding="utf-8"?>
	<resources>
	    <style name="Title" parent="Widget.AppCompat.TextView">
	        <item name="android:textColor">@color/brown</item>
	        <item name="android:textSize">24sp</item>
	        <item name="android:layout_marginTop">36dp</item>
	        <item name="textInputStyle">bold</item>
	    </style>
	</resources>
```
{% endraw %}


1. styles을 적용해주고 싶은 곳에 `style=”@style/이름”` 로 저장해주기

	
{% raw %}
```javascript
	<TextView
	        android:id="@+id/nameTextView"
	        android:layout_width="wrap_content"
	        android:layout_height="wrap_content"
	        android:layout_marginStart="36dp"
	        android:text="이름"
	        <u>style="@style/Title"</u>
	        app:layout_constraintStart_toStartOf="parent"
	        app:layout_constraintTop_toTopOf="parent" />
```
{% endraw %}



---



## Colors와 테마


colors에 다크 테마를 따로 두기 위해 night용 colors.xml 파일을 만든다.

1. values 디렉토리 마우스 우클릭 > New > Values Resource File
2. 파일 이름은 `colors` 디렉토리 이름은 `values-night`으로 생성

	![1](/assets/img/GBD38/1.png)

1. colors와 colors(night)를 공통된 resources 이름 사용하여 테마에 이용
