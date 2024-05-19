---
layout: post
date: 2024-01-26
title: "[Android] 비동기 처리 방법 with MVVM 패턴"
tags: [Android, MVVM, Kotlin, 비동기, Coroutine, LiveData, ]
categories: [Android, ]
---


---


![0](/assets/img/GBD22/0.png).png)



## 왜 필요한가?


---


정확하게 말하면 Activity에서 바로 Retrofit의 `enqueue`를 호출해서 비동기적으로 처리하면 다른 특별한 방법은 필요없다. 이게 바로 비동기 처리 방법이기 때문이다. (코드예시 : 수정하기 전 코드)


> 💡 Call.enqueue는 비동기적으로, Call.excute는 동기적으로 처리한다.


하지만 우리는 개발을 하면서 다양한 아키텍쳐 디자인 패턴을 사용하게 된다. 아키텍쳐 디자인 패턴을 적용하게 되면 코드가 분리되는데 데이터를 조회하는 코드도 분리가 된다. 이런 경우에는 데이터 조회하는 클래스에서 화면을 보여주는 클래스로 이 데이터를 전달하는 과정이 필요하다. 예를 들어, Repository에서 데이터를 조회한 후, ViewModel에 이 데이터를 저장하거나 거쳐서 Activity에 이 데이터를 화면에 보여주는 것이다.


이 경우에 ViewModel이나 Repository에서 어떻게 Activity까지 결과값을 전달해올지가 물음표다. 어떻게 넘겨야할지 감도 안잡힐수도 있다.


데이터를 가지고 와서 보여주기위해 Activity에서 ViewModel을 호출하고, ViewModel은 Repository의 데이터 호출 함수를 호출한다. 그럼 ViewModel과 Repository에서 데이터 호출을 실행하는 동안 비동기 처리를 안해주면 데이터를 조회해와도 Activity에 전달을 못하거나 Activity의 Thread가 멈추는 불상사가 발생한다.


그래서 이 글에선 Callback, Coroutine, Flow 등을 이용한 방법으로 데이터 조회하는 코드가 분리되는 경우 어떻게 비동기적으로 처리하는 지 기록한다.


깔끔하고 재사용성이 좋은 코드를 위해 MVVM 패턴으로 변경할 것이다. 이 글은 패턴에 대한게 중점이 아닌 비동기 처리 방법이 중점이므로 패턴 설명에 대한 것은 생략한다.


아래는 MVVM 패턴으로 적용하면서 다양한 비동기 처리 방법에 대한 내용이다.



## 수정하기 전 코드


---


참고로 아래 코드는 앱 만들다가 갑자기 정리를 해야겠다는 생각이 들어서 하게 된거라 전혀 상관없는 코드도 있다.


이 코드를 베이스로 각 처리방식으로 수정해 볼 것이다.


**PriceService.kt**

- `@GET()`안에 있는 건 url 뒤에 경로다. 사람마다 쓰는 API에 따라 다를 수 있다.
- `Call<데이터 클래스>`안에 있는 데이터 클래스는 내가 data class를 만들어서 사용하여 만든 data class다.


{% raw %}
```text

interface PriceService {
	@GET("ListNecessariesPricesService/1/5/")
    fun getAllItems(): Call
}
```
{% endraw %}



**RetrofitManager.kt**

- "url 주소"는 API url을 말한다.


{% raw %}
```text

object RetrofitManager {

    private const val BASE_URL = "url 주소"

// okHttpClient Settings
    private val okHttpClient = OkHttpClient.Builder()
        .connectTimeout(10, TimeUnit.SECONDS)
        .readTimeout(10, TimeUnit.SECONDS)
        .writeTimeout(10, TimeUnit.SECONDS)
        .build()

    private val gson = GsonBuilder().setLenient().create()

    private val retrofit = Retrofit.Builder()
        .baseUrl(BASE_URL)
        .client(okHttpClient)
        .addConverterFactory(GsonConverterFactory.create(gson))
        .build()

    val priceService: PriceService by lazy { retrofit.create(PriceService::class.java) }
}
```
{% endraw %}



---


**CategoryActivity.kt**

- `BaseActivity()`은 `AppCompatActivity`을 상속받는 Class고 Toolbar의 모양별 처리를 위해 만들어둔것이고 이번 포스팅과 상관이 없다.
- `binding`과 `baseBinding` 부분도 이번 포스팅과는 상관없다.
- `onResume`을 보면 이번에 수정할 코드가 있다.
	- object인 `RetrofitManager`의 priceService와 함수를 호출한다.
	- `enqueue`를 이용해서 비동기로 코드가 실행된다.


{% raw %}
```text

class CategoryActivity : BaseActivity(ToolbarType.BACK) {
    private lateinit var binding: ActivityCategoryBinding

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)

        binding = ActivityCategoryBinding.inflate(layoutInflater)
        baseBinding.contentLayout.addView(binding.root)
    }

    override fun onResume() {
        super.onResume()

        val itemRequest = RetrofitManager.priceService.getAllItems()
        itemRequest.enqueue(object : Callback {
            override fun onResponse(
                call: Call,
                response: Response
            ) {
                Result.Success( response.body())
                Log.e(TAG, "ee : ${response.body()}")
                Log.e(TAG, "ee : ${response}")
            }

            override fun onFailure(call: Call, t: Throwable) {
                Result.Failure(Exception("API request failed"))
                Log.e(TAG, "error : ${t}")
            }
        })
    }
}
```
{% endraw %}




## Callback으로 처리하기


---


Callback이라는 단어는 개발하면서 한번쯤은 들어봤을 것이다. 말 그대로 돌아온다는 의미다. 예를 들어, A Class에서 B Class의 함수를 실행할 때 이 Callback을 함수에 전달하면 B Class에서 함수를 실행한 후 이 Callback에 함수의 결과값을 담아 A Class에 돌려줄 수 있다.


이것을 사용해보자.



### 1. Callback 인터페이스 생성


제일 먼저 해야할 건 Callback을 만들어주는 것이다.


**ResultCallback.kt**

- `ResultCallback<T>`에서 T는 제네릭 타입임을 의미한다. 이 Callback을 객체화할 때 Type을 지정한다.
- `onSuccess`는 성공 시 이 함수를 통해 결과값을 전달한다.
- `onFailure`는 실패 시 에러를 전달한다.


{% raw %}
```text

interface ResultCallback {
    fun onSuccess(result: T)
    fun onFailure(error: Throwable)
}
```
{% endraw %}




### 2. Repository, RepositoryImpl 생성


Repository는 MVVM 패턴에서 API나 데이터베이스에서 값을 가지고 와 ViewModel에 전달해주는 역할을 담당한다.


`interface`로 Repository가 데이터를 어떻게 가져오고 저장하는지에 대한 코드를 감추고, 추상화된 형태로 사용하게 한다. 그리고 `RepositoryImpl`에서 세부 내용을 구현하는 것이다. `RepositoryImpl`은 Repository를 상속받고 추상화한 메소드들의 세부 구현을 한다.


**PriceRepository.kt**

- interface로 Repository를 추상화한다.
- 위에서 정리한 Callback을 사용할 때 전달할 데이터 타입을 지정한다.


{% raw %}
```kotlin

        interface PriceRepository {
            fun getAllItems(callback: ResultCallback)
        }
```
{% endraw %}



**PriceRepositoryImpl.kt**

- Repository를 상속한 후 멤버 함수를 구현한다.
- 수정하기 전 Activity에 있는 Retrofit 호출 부분을 복사한 후 `onResponse`와 `onFailure`에서 callback으로 값을 전달한다. 참고로 response.body가 null일수도 있는 경우에는 체크해주어야 한다.


{% raw %}
```kotlin

class PriceRepositoryImpl : PriceRepository {

    override fun getAllItems(callback: ResultCallback) {
        val itemRequest = RetrofitManager.priceService.getAllItems()

        itemRequest.enqueue(object : Callback {
            override fun onResponse(
                call: Call,
                response: Response
            ) {
                if (response.isSuccessful) {
                    callback.onSuccess(response.body()!!)
                } else {
                    callback.onFailure(Throwable(message = "response body 없음"))
                }
            }

            override fun onFailure(call: Call, t: Throwable) {
                callback.onFailure(t)
            }
        })
    }
}
```
{% endraw %}




### 3. ViewModel 생성


ViewModel은 Repository에서 전달받은 값을 Activity에 전달한다. 하지만 여기선 Activity에 전달만 할 뿐 데이터를 보존해야하는 ViewModel의 중요한 기능을 상실했다. <Callback과 LiveData로 처리하기> 에서 이 중요한 기능을 사용한다.


**PriceViewModel.kt**



{% raw %}
```kotlin

class PriceViewModel(private val priceRepository: PriceRepository) : ViewModel() {

    fun getAllItems(callback: ResultCallback) {
        priceRepository.getAllItems(callback)
    }

    class PriceViewModelFactory(private val priceRepository: PriceRepository) :
        ViewModelProvider.Factory {
        override fun  create(modelClass: Class): T {
            return PriceViewModel(priceRepository) as T
        }
    }

}
```
{% endraw %}




### 4. Activity에서 호출


위 1, 2, 3에서 만든것을 확인해보자.


**CategoryActivity.kt**

- ViewModel을 생성한다. 이때 ViewModelFactory에 `RepositoryImpl`을 넣는다.
- 이제 viewModel의 메소드를 호출한다. 호출할 때 인수로 위에서 생성한 Callback을 넣고 Callback의 각 메소드를 구현해준다.


{% raw %}
```kotlin

class CategoryActivity : BaseActivity(ToolbarType.BACK) {
    private lateinit var binding: ActivityCategoryBinding

    private val priceViewModel: PriceViewModel by viewModels {
        PriceViewModel.PriceViewModelFactory(PriceRepositoryImpl())
    }

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)

        binding = ActivityCategoryBinding.inflate(layoutInflater)
        baseBinding.contentLayout.addView(binding.root)

    }

    override fun onResume() {
        super.onResume()

        priceViewModel.getAllItems(object : ResultCallback {
            override fun onSuccess(result: ListNecessariesPricesResponse) {
                Log.e(TAG, "result: $result")
            }
            override fun onFailure(error: Throwable) {
                Log.e(TAG, "error: ${error.message}")
            }
        })
    }

}
```
{% endraw %}




## Callback과 LiveData로 처리하기


---


Callback으로만 처리할 때는 Callback으로만 구현하기엔 ViewModel의 장점을 버려야하는 단점이 있다. 그런 단점을 보완하기 위해 LiveData를 같이 사용하여 비동기식 처리를 할 수 있다.



### 1. Repository, RepositoryImpl 생성


Repository는 MVVM 패턴에서 API나 데이터베이스에서 값을 가지고 와 ViewModel에 전달해주는 역할을 담당한다.


`interface`로 Repository가 데이터를 어떻게 가져오고 저장하는지에 대한 코드를 감추고, 추상화된 형태로 사용하게 한다. 그리고 `RepositoryImpl`에서 세부 내용을 구현하는 것이다. `RepositoryImpl`은 Repository를 상속받고 추상화한 메소드들의 세부 구현을 한다.


**PriceRepository.kt**

- interface로 Repository를 추상화한다.
- 인수로 결과를 받으면 Unit으로 돌려주는 callback을 인자로 넣는다.


{% raw %}
```kotlin
interface PriceRepository {
	fun getAllItems(callback: (ListNecessariesPricesResponse?) -> Unit)
}
```
{% endraw %}



**PriceRepositoryImpl.kt**

- Repository를 상속한 후 멤버 함수를 구현한다.
- 수정하기 전 Activity에 있는 Retrofit 호출 부분을 복사한 후 `onResponse`와 `onFailure`에서 callback으로 값을 전달한다.


{% raw %}
```kotlin

class PriceRepositoryImpl : PriceRepository {

    override fun getAllItems(callback: (ListNecessariesPricesResponse?) -> Unit) {

        val itemRequest = RetrofitManager.priceService.getAllItems()

        itemRequest.enqueue(object : Callback {
            override fun onResponse(
                call: Call,
                response: Response
            ) {
                callback(response.body())
            }

            override fun onFailure(call: Call, t: Throwable) {
                callback(null)
            }
        })
    }
}
```
{% endraw %}




### 2. ViewModel 생성


ViewModel은 Repository에서 전달받은 값을 LiveData로 저장하고 Activity는 이 LiveData를 관찰한다.


**PriceViewModel.kt**

- `_items`를 MutableData 형태로 선언한다.
- `items`는 변수를 미리 선언하고 외부에서 호출이 되는 순간마다 `_items`의 값을 제공한다.
- `getAllItems()`를 실행시키면 Repository의 getAllItems 메소드 뒤에 후행 람다를 통해 callback으로 받는 값을 `result`로 받을 수 있다.
- `postValue()`는 `MutableLiveData`의 value 속성을 메인 스레드에서만 접근할 수 있기 때문에, `postValue()`를 통해 백그라운드에서 비동기적으로 데이터를 `MutableLiveData`의 `Value`에 업데이트한다.


{% raw %}
```kotlin

class PriceViewModel(private val priceRepository: PriceRepository) : ViewModel() {

    private val _items = MutableLiveData()
    val items: LiveData by lazy { _items }

    fun getAllItems() {
        priceRepository.getAllItems {result ->
            if (result != null) {
                _items.postValue(result)
            }
        }
    }
    class PriceViewModelFactory(private val priceRepository: PriceRepository) :
        ViewModelProvider.Factory {
        override fun  create(modelClass: Class): T {
            return PriceViewModel(priceRepository) as T
        }
    }

}
```
{% endraw %}




### 3. Activity에서 호출


만든 것을 확인해보자.


**CategoryActivity.kt**

- ViewModel의 items을 관찰하고 값이 변할 때마다 로그를 찍는다.
- ViewModel의 `getAllItems()`을 호출한다.


{% raw %}
```kotlin

class CategoryActivity : BaseActivity(ToolbarType.BACK) {
    private lateinit var binding: ActivityCategoryBinding

    private val priceViewModel: PriceViewModel by viewModels {
        PriceViewModel.PriceViewModelFactory(PriceRepositoryImpl())
    }

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)

        binding = ActivityCategoryBinding.inflate(layoutInflater)
        baseBinding.contentLayout.addView(binding.root)

    }

    override fun onResume() {
        super.onResume()

        priceViewModel.items.observe(this, Observer { result ->
            Log.e(TAG, "result: $result")
        })

        priceViewModel.getAllItems()
    }
}
```
{% endraw %}




## Coroutine으로 처리하기


---


Coroutine을 Retrofit에 접목시키거나 다른 비동기처리를 할 때 사용하면 매우 편리하며, 코드 가독성도 좋아진다. 물론 위에서 소개한 Callback보다도 편리하다.



### 1. Retrofit API를 정의한 Interface 수정


본격적으로 Coroutine을 사용하기 위해 API를 정의한 Interface를 수정한다. Interface안의 추상 메소드 앞에 `suspend`를 붙여준다.


**PriceService.kt**

- `suspend` 키워드는 이 함수가 일시 중단이 가능함을 의미하고, 비동기 처리를 할 때 많이 사용한다.


{% raw %}
```kotlin

interface PriceService {
    @GET("ListNecessariesPricesService/1/5/")
    suspend fun getAllItems(): ListNecessariesPricesResponse
}
```
{% endraw %}




### 2. Repository, RepositoryImpl 생성


**PriceRepository.kt**

- 추상 메소드에는 `suspend` 키워드를 붙여서 만들어준다.


{% raw %}
```kotlin
interface PriceRepository {
    suspend fun getAllItems(): ListNecessariesPricesResponse
}
```
{% endraw %}



**PriceRepositoryImpl.kt**

- `withContext()`로 main 스레드가 아닌 IO 스레드에서 작업 후 결과값을 반환한다.
- priceService.getAllItems()의 반환값은 추상 메소드를 정의할 때 데이터 클래스로 설정하면 Retrofit이 매핑된 데이터 클래스로 반환해준다.
- 사실 Coroutine으로 비동기 작업하는 건 main 스레드에서도 가능하지만 **Coroutine 또한 main 스레드가 담당하는 작업에 영향을 미치기** 때문에 네트워크 작업이나 긴 작업에 용의한 IO 스레드에서 실행하는 걸 추천한다.


{% raw %}
```kotlin

interface PriceRepository {
	suspend fun getAllItems(): ListNecessariesPricesResponse
}
```
{% endraw %}




### 3. ViewModel 생성


Repository 함수를 CoroutineScope 내에서 호출한 후 `_items`의 value에 저장한다. `postValue`로 저장하지 않은 이유는 `CoroutineScope`를 생성할 때 특정 스레드를 지정하지 않으면 기본적으로 main 스레드에서 지정하기 때문에 그냥 value에다가 바로 대입한다. 이렇게 `_items`에 저장된 값은 다른 클래스에서 `items`를 통해 확인할 수 있다.


**PriceViewModel.kt**

- `suspend` 키워드가 선언된 함수는 `CoroutineScope`에서만 호출할 수 있다. 그래서 `viewModelScope.launch`로 `CoroutineScope`를 생성하고 scope 내에서 Repository의 메소드를 호출해준다.


{% raw %}
```kotlin

class PriceViewModel(private val priceRepository: PriceRepository) : ViewModel() {

    private val _items = MutableLiveData()
    val items: LiveData by lazy { _items }

    fun getAllItems() {
        viewModelScope.launch {
            try {
                val result = priceRepository.getAllItems()
                _items.value = result
            } catch (e: Exception) {
                Log.e(TAG, "error: $e")
            }
        }
    }
    class PriceViewModelFactory(private val priceRepository: PriceRepository) :
        ViewModelProvider.Factory {
        override fun  create(modelClass: Class): T {
            return PriceViewModel(priceRepository) as T
        }
    }

}
```
{% endraw %}




### 4. Activity에서 호출


1, 2, 3에서 구현한 내용을 확인해보자


**Category.kt**

- ViewModel의 `items`를 관찰하고, `ViewModel.getAllItems()`를 호출하면 ViewModel -> Repository -> 외부 API로 요청 후 반환값을 ViewModel의 `_items`의 value에 저장할것이고, `_items`를 바라보고 있는 `items`의 값도 변한다. 그리고 Observer가 변경된 값을 확인하고 뒤에 실행문을 실행한다.


{% raw %}
```kotlin

class CategoryActivity : BaseActivity(ToolbarType.BACK) {
    private lateinit var binding: ActivityCategoryBinding

    private val priceViewModel: PriceViewModel by viewModels {
        PriceViewModel.PriceViewModelFactory(PriceRepositoryImpl())
    }

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)

        binding = ActivityCategoryBinding.inflate(layoutInflater)
        baseBinding.contentLayout.addView(binding.root)

    }

    override fun onResume() {
        super.onResume()

        priceViewModel.items.observe(this, Observer { result ->
            Log.e(TAG, "result: $result")
        })

        priceViewModel.getAllItems()
    }
}
```
{% endraw %}




## 마무리


---


이렇게 데이터를 어떻게 끌어와서 전달해주는 지를 알아봤다.


위 방법은 그저 내가 찾은 방법이고, 각자 본인 원하는 스타일대로 짜면 좋겠다.

