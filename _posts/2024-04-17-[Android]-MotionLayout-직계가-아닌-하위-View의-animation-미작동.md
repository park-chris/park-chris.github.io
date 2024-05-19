---
layout: post
date: 2024-04-17
title: "[Android] MotionLayout 직계가 아닌 하위 View의 animation 미작동"
tags: [에러, Android, Kotlin, MotionLayout, Animation, ]
categories: [Android, ]
---



## MotionLayout 하위 View가 작동을 안한다.


---


어느 날과 같이 앱을 개발 중이었다. 나는 MotionLayout을 이용해서 유튜브의 미디어바와 같은 걸 만들려고 했다. 아래는 MotionLayout 계층 구조다.


![0](/assets/img/GBD14/0.png)_계층 구초 출처 : Crystal_


이런 구조다. 


열심히 만들어서 이제 애니메이션을 작동해보는데 ConstaintLayout은 작동을 하는데 View1, View2, View3은 내가 작성한 motion_scene처럼 작동을 안하는 것이다.


정확히 말하면 애니메이션은 작동을 하는데 View1, View2, View3의 사이즈와 위치 등과 같은 것이 적용이 안되었다.


그리고 2시간을 내다버리며 찾은 결과를 아래에 글로 쓴다. 사실 MotionLayout 적용할때마다 이러는거 같아서 글로 남긴다.



## MotionLayout은 직계 하위 View만 적용이 가능하다.


---


우선 View1, View2, View3이 Motion Scene에 작성한대로 작동안하는 이유는 MotionLayout의 직계 View가 아니라서다. 그렇다!


<u>**MotionLayout은 직계 하위 View에만 영향을 미친다!!**</u>


그래서 ConstraintLayout는 MotionLayout 바로 하위에 위치해서 잘 작동했지만 View1, View2, View3은 MotionLayout의 직계 하위 View가 아니다. 그래서 작동을 안한거다.


그러면 어떻게 수정해줘야 할까?


보통 ConstraintLayout이 MotionLayout에 필요한 경우는 그 사이즈대로 하위 View의 위치를 제어하고 배경을 원하는 것으로 고정하기 위해서다. 그래서 나는 View1, View2, View3을 MotionLayout 하위로 이동시키고 ConstraintLayout은 프레임으로만 쓰기로 했다.


우선 아래는 수정 전 코드다.



{% raw %}
```xml
<?xml version="1.0" encoding="utf-8"?>
<layout>

    <data>
        ...
    </data>

    <com.crystal.restsound.component.CustomMotionLayout xmlns:android="http://schemas.android.com/apk/res/android"
        xmlns:app="http://schemas.android.com/apk/res-auto"
        xmlns:tools="http://schemas.android.com/tools"
        android:id="@+id/motionLayout"
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        app:layoutDescription="@xml/fragment_player_scene"
        app:layout_collapseMode="parallax"
        tools:context=".ui.PlayerFragment">

        <androidx.constraintlayout.widget.ConstraintLayout
            android:id="@+id/containerLayout"
            android:layout_width="0dp"
            android:layout_height="0dp"
            android:background="@color/music_bar"
            app:layout_constraintBottom_toBottomOf="parent"
            app:layout_constraintEnd_toEndOf="parent"
            app:layout_constraintStart_toStartOf="parent"
            app:layout_constraintTop_toTopOf="parent" >

            <ImageView
                android:id="@+id/albumImageView"
                android:layout_width="50dp"
                android:layout_height="50dp"
                android:layout_marginStart="10dp"
                android:background="@color/highlight"
                android:scaleType="centerCrop"
                android:src="@drawable/bg_premium"
                app:animatorImage="@{item.imagePath}"
                app:layout_constraintBottom_toBottomOf="@id/containerLayout"
                app:layout_constraintStart_toStartOf="@id/containerLayout"
                app:layout_constraintTop_toTopOf="@id/containerLayout" />

            <androidx.appcompat.widget.AppCompatImageButton
                android:id="@+id/playPauseButton"
                android:layout_width="40dp"
                android:layout_height="40dp"
                android:background="@drawable/bg_sound_button"
                app:layout_constraintBottom_toBottomOf="@id/containerLayout"
                app:layout_constraintEnd_toEndOf="@id/containerLayout"
                app:layout_constraintTop_toTopOf="@id/containerLayout"
                app:playPause="@{mediaState.playing}" />

        </androidx.constraintlayout.widget.ConstraintLayout>

    </com.crystal.restsound.component.CustomMotionLayout>
</layout>
```
{% endraw %}



그리고 ConstaintLayout 하위 View를 밖으로 빼낸것이다.



{% raw %}
```xml
<com.crystal.restsound.component.CustomMotionLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:id="@+id/motionLayout"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    app:layoutDescription="@xml/fragment_player_scene"
    app:layout_collapseMode="parallax"
    tools:context=".ui.PlayerFragment">

    <androidx.constraintlayout.widget.ConstraintLayout
        android:id="@+id/containerLayout"
        android:layout_width="0dp"
        android:layout_height="0dp"
        android:background="@color/music_bar"
        app:layout_constraintBottom_toBottomOf="parent"
        app:layout_constraintEnd_toEndOf="parent"
        app:layout_constraintStart_toStartOf="parent"
        app:layout_constraintTop_toTopOf="parent" />

    <ImageView
        android:id="@+id/albumImageView"
        android:layout_width="50dp"
        android:layout_height="50dp"
        android:layout_marginStart="10dp"
        android:background="@color/highlight"
        android:scaleType="centerCrop"
        android:src="@drawable/bg_premium"
        app:animatorImage="@{item.imagePath}"
        app:layout_constraintBottom_toBottomOf="@id/containerLayout"
        app:layout_constraintStart_toStartOf="@id/containerLayout"
        app:layout_constraintTop_toTopOf="@id/containerLayout" />


    <androidx.appcompat.widget.AppCompatImageButton
        android:id="@+id/playPauseButton"
        android:layout_width="40dp"
        android:layout_height="40dp"
        android:background="@drawable/bg_sound_button"
        app:layout_constraintBottom_toBottomOf="@id/containerLayout"
        app:layout_constraintEnd_toEndOf="@id/containerLayout"
        app:layout_constraintTop_toTopOf="@id/containerLayout"
        app:playPause="@{mediaState.playing}" />

</com.crystal.restsound.component.CustomMotionLayout>
```
{% endraw %}



결론은 MotionLayout 직계 하위 View로 빼주면 잘 작동하는 것을 볼 수 있다.


늘 MotionLayout을 사용할 때마다 직계 하위에만 작동하는 것을 까먹는다.

