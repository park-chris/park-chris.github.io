---
layout: post
date: 2024-04-30
title: "[Android] RecyclerView와 SeekBar 스크롤링 중첩"
tags: [에러, Android, Kotlin, RecyclerView, SeekBar, ]
categories: [Android, ]
---



## 스크롤링 중첩


---


혹시 개발 중 스크롤링이 중첩되는 경험을 한 적 있는가?


 RecyclerView의 아이템이 가지고 있는 SeekBar를 조작할 때 RecyclerView의 가로 스크롤이 작동한다.


내가 원하는 건 SeekBar의 Progress를 조절하는 것인데 이게 RecyclerView에서 가로채서 스크롤링을 하는 것이다. 이렇게 되면 내가 원하는 그림이 나오지 않기 때문에 SeekBar에서 터치 이벤트가 일어나면 부모뷰(RecyclerView)에 이벤트를 전달하지 않는 방법으로 해결해보겠다.



## 터치 이벤트 중첩 해결


---


방법은 간단하다.


SeekBar를 상속받아서 이벤트 처리를 할때 `requestDisallowInterceptTouchEvent()`를 호출하여 부모뷰로 이벤트를 전달하지 않는것이다.



{% raw %}
```kotlin
class CustomSeekBar(context: Context, attributeSet: AttributeSet? = null): AppCompatSeekBar(context, attributeSet) {
    override fun onTouchEvent(event: MotionEvent?): Boolean {
        when (event?.action) {
            MotionEvent.ACTION_DOWN -> {
                parent.requestDisallowInterceptTouchEvent(true)
            }
            MotionEvent.ACTION_MOVE -> {
                parent.requestDisallowInterceptTouchEvent(true)
            }
            MotionEvent.ACTION_UP -> {
                parent.requestDisallowInterceptTouchEvent(true)
            }
        }
        return super.onTouchEvent(event)
    }
}
```
{% endraw %}



이 클래스를 대신 사용하면 스크롤링이 각각 잘 작동하는 것을 볼 수 있다.



## 여담


---


원래는 RecyclerView에서 SeekBar의 범위를 감지하려고 했는데 SeekBar에서 조절하는 게 더 심플해서 이 방법으로 했다.

