---
title:  "[Android] Part2-웹뷰 앱"

categories:
  - Android
tags:
  - [Android, Kotlin, 안드로이드 강의]

toc: true
toc_sticky: true
 
date: 2023-03-05 20:15:11+0900
last_modified_at: 2023-03-05 20:15:15+0900
---

<br>
<br>
<br>

> 아래 내용은 패스트 캠퍼스 [35개 프로젝트로 배우는 Android 앱 개발 feat. Jetpack Compose]의 강의 내용을 기반으로 작성한 자습 기록입니다.

<br><br>

# 웹뷰 앱

## ℹ️앱 설명

웹 뷰를 활용하여 네이버 웹툰을 보고 최근 웹툰을 저장할 수 있는 앱

<br>

## ✅구현 기능

* ViewPager와 Fragment를 사용해 3개의 페이지 구현
* SharedPreference를 활용해 최근 페이지 등록
* Dialog를 활용해 Tab 이름 설정

<br>

## ✅사용되는 기능

* Android
    * ViewPager2
    * TabLayoutMediator
    * WebView
    * Fragment
    * SharedPreference
    * Dialog


---

<br><br>

## 📔WebView 구현

웹툰 페이지를 표시하기 앞서 WebView를 먼저 구현해 보자. 이번에는 Activity가 아닌 Fragment 위에 뷰를 구현한다. 이를 위해 `WebViewFragment.kt` 파일과 `fragment_webview.xml` 파일을 만들어 주었다. Fragment는 activity에 종속되어 표시되기 때문에 fragment를 담을 activity에서 컨테이너를 만들어 주어야 한다. 따라서 activity_main 파일에서 FrameLayout으로 컨테이너를 만들어 주었다.

<b>activity_main.xml</b>

```kotlin
<?xml version="1.0" encoding="utf-8"?>
<androidx.constraintlayout.widget.ConstraintLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    tools:context=".MainActivity">

    <FrameLayout
        android:id="@+id/container"
        android:layout_width="match_parent"
        android:layout_height="match_parent" />

</androidx.constraintlayout.widget.ConstraintLayout>
```

`fragment_webview.xml` 파일에서는 화면에 보여줄 뷰 위젯을 지정하면 되므로 WebView 위젯을 넣어주도록 한다. 

<b>fragment_webview.xml</b>

```kotlin
<?xml version="1.0" encoding="utf-8"?>
<androidx.constraintlayout.widget.ConstraintLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    android:layout_width="match_parent"
    android:layout_height="match_parent">

    <WebView
        android:id="@+id/webView"
        android:layout_width="0dp"
        android:layout_height="0dp"
        app:layout_constraintBottom_toTopOf="@id/backToLastButton"
        app:layout_constraintEnd_toEndOf="parent"
        app:layout_constraintStart_toStartOf="parent"
        app:layout_constraintTop_toTopOf="parent" />

</androidx.constraintlayout.widget.ConstraintLayout>
```

Fragment를 상속받은 WebViewFragment의 `onViewCreated()` 함수 내부에 WebView를 연결시킨 뒤 몇 가지 설정들을 정의해 준다. webViewClient는 페이지 로드가 시작될 때 또는 완료되었을 때 등 페이지에 로드 중 발생할 수 있는 다양한 이벤트들을 다루는 데 사용된다. `javaScriptEnabled` 속성은 웹에서 사용되는 js 언어를 활용하여 동작하는 기능들을 허용할지 에 대해 설정하는 속성이다. 기본적으로 값은 false로 되어 있어 단순히 화면을 볼 수만 있는 반면, true로 값을 지정했을 때에는 웹의 기능들을 사용할 수 있다. 따라서 앱에서 웹툰 페이지의 기능들을 사용하기 때문에 true로 값을 지정해 주었다. `loadUrl()` 메서드는 감이 오듯이 말 그대로 페이지를 로드하는 메서드이다. 우선적으로 구글 홈페이지 url을 넣어 주었다.

<b>WebViewFragment.kt</b>

```kotlin
class WebViewFragment : Fragment() {
    private lateinit var binding: FragmentWebviewBinding

    override fun onCreateView(
        inflater: LayoutInflater,
        container: ViewGroup?,
        savedInstanceState: Bundle?
    ): View {
        binding = FragmentWebviewBinding.inflate(inflater)
        return binding.root
    }

    override fun onViewCreated(view: View, savedInstanceState: Bundle?) {
        super.onViewCreated(view, savedInstanceState)

        binding.webView.apply {
            webViewClient = WebViewClient()
            settings.javaScriptEnabled = true 
            loadUrl("https://www.google.com/")
        }
    }
}
```

WebView를 사용할 준비를 마쳤으니 MainActivity에서 WebViewFragment를 호출하도록 하자. 버튼을 눌렀을 때 우선 버튼을 눌렀을 때 화면에 보이도록 지정해 주었다. 이를 위해 `activity_main.xml` 파일에 버튼을 하나 추가해 주었다. 클릭 리스너 내부에서 Fragment를 호출하기 위해 supportFragmentManager를 사용해 beginTransaction() 메서드를 호출한다. supportFragmentManager는 액티비티 내의 Fragment를 관리하는 데 사용한다. 이후 beginTransaction() 메서드를 통해 fragment에 대한 작업을 수행하는 데 사용되는 FragmentTransaction 객체를 반환한다. apply 함수로 같이 초기화될 설정으로 `replace()`와 `commit()` 메서드를 넣어준다. replace() 함수의 매개변수로 Fragment를 표시할 컨테이너와 표시할 Fragment가 필요하다. 따라서 앞서 만들어 둔 FrameLayout의 컨테이너와 WebViewFragment를 인자로 넣어준다. 이후 `commit()` 메서드를 통해 설정한 트랜잭션을 실행하고 변경 사항을 액티비티 레이아웃에 적용한다.

<b>MainActivity.kt</b>

```kotlin
class MainActivity : AppCompatActivity() {
    private lateinit var binding: ActivityMainBinding

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        binding = ActivityMainBinding.inflate(layoutInflater)
        setContentView(binding.root)

        binding.button1.setOnClickListener {
            supportFragmentManager.beginTransaction().apply {
                replace(R.id.container, WebViewFragment())
                commit()
            }
        }
    }
}
```

마지막으로 WebView를 사용하기 위한 인터넷 권한을 설정해 주어야 한다. 매니페스트 파일에서 해당 권한을 허용하도록 하자.

```kotlin
<uses-permission android:name="android.permission.INTERNET" />
```

실행 결과를 보면, 버튼을 눌렀을 때 구글 홈페이지가 화면에 표시되는 것을 볼 수 있다.

![](/assets/images/fastcampus/part2/webview/webview1.jpg)
![](/assets/images/fastcampus/part2/webview/webview2.jpg)

<br><br>

## 📔ProgressBar 구현

보통 웹 페이지가 로드되기를 기다릴 때 페이지가 로딩되고 있는 것을 시각적으로 표시하기 위해 ProgressBar를 사용한다. 구현하는 방법은 어렵지 않다. `fragment_webview.xml` 파일에서 ProgressBar 위젯을 새롭게 추가해 준다. 

<b>fragment_webview.xml</b>

```kotlin
<?xml version="1.0" encoding="utf-8"?>
<androidx.constraintlayout.widget.ConstraintLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    android:layout_width="match_parent"
    android:layout_height="match_parent">

    <WebView
        android:id="@+id/webView"
        android:layout_width="0dp"
        android:layout_height="0dp"
        app:layout_constraintBottom_toBottomOf="@id/backToLastButton"
        app:layout_constraintEnd_toEndOf="parent"
        app:layout_constraintStart_toStartOf="parent"
        app:layout_constraintTop_toTopOf="parent" />

    <ProgressBar
        android:id="@+id/progressBar"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        app:layout_constraintBottom_toBottomOf="@id/webView"
        app:layout_constraintEnd_toEndOf="parent"
        app:layout_constraintStart_toStartOf="parent"
        app:layout_constraintTop_toTopOf="parent" />

</androidx.constraintlayout.widget.ConstraintLayout>
```

앞서 언급했듯이 페이지가 로드되는 동안 ProgressBar가 표시되어야 한다. 따라서 이를 관리할 때 사용되는 WebViewClient에서 기능을 정의해 주어야 한다. WebViewClient를 상속받는 WebtoonWebViewClient라는 이름의 클래스를 새로 생성해 준다. 해당 클래스의 메서드에서 ProgressBar의 기능을 정의해야 하기 때문에 속성으로 받아온다.

클래스 내부에서는 3개의 메서드를 재정의해 준다. `onPageStarted()`와 `onPageFinished()` 메서드는 말 그대로 페이지 로드가 시작되었을 때와 완료되었을 때 호출되는 메서드이다. 로드가 시작되었을 때에는 progressBar가 보이도록, 로드가 완료되었을 때에는 gone 처리가 되도록 설정해 준다. 

`shouldOverrideUrlLoading()` 메서드는 WebView가 URL 요청을 처리해야 할 때마다 호출되는 메서드이다. 메서드의 인자로 요청을 수행하는 View와 요청 중인 URL에 대한 정보가 포함된 request 두 개가 사용된다. 메서드는 Boolean 값을 반환하는데, true를 반환할 시 앱이 요청을 처리했으므로 웹 뷰에서 Url을 로드하지 않아도 됨을 의미한다. 반대로 false를 반환하면 WebView는 일반적으로 Url을 로드한다. 해당 메서드는 나중에 sharedPreference에 저장된 페이지의 처리를 위해 우선 false를 반환한다. 실행 결과를 확인하면 progressbar가 잘 구현된 것을 볼 수 있다.

<b>WebtoonWebViewClient.kt</b>

```kotlin
class WebtoonWebViewClient(
    val progressBar: ProgressBar
) : WebViewClient() {
    override fun onPageStarted(view: WebView?, url: String?, favicon: Bitmap?) {
        super.onPageStarted(view, url, favicon)

        progressBar.visibility = View.VISIBLE
    }

    override fun onPageFinished(view: WebView?, url: String?) {
        super.onPageFinished(view, url)

        progressBar.visibility = View.GONE
    }

    override fun shouldOverrideUrlLoading(view: WebView?, request: WebResourceRequest?): Boolean {
        return false
    }
}
```

 ![](/assets/images/fastcampus/part2/webview/webview3.jpg)

<br><br>

## 📔 뒤로 가기 구현

웹 뷰 페이지에서 몇 번의 페이지 이동을 거친 뒤 뒤로 가기를 실행했을 때 앱이 바로 종료된다. 몇 가지 구현을 통해 이전 페이지로 돌아가도록 지정해 주도록 하자.

먼저 WebViewFragment에서 WebView를 다루기 때문에 해당 파일에서 뒤로 가기를 실행했을 때 이전 페이지로 이동하는 것에 대해 설정할 수 있다. `canGoBack()`이라는 이름의 함수를 새로 만들어 Boolean 값을 반환하도록 한다. WebView에서는 `canGoBack()`이라는 메서드를 갖고 있는데 WebView에 이전 페이지가 존재하는지에 대해 Boolean 값으로 반환해 주는 메서드이다. 물론 존재할 경우 true를 반환한다.

<b>WebViewFragment.kt</b>

```kotlin
fun canGoBack(): Boolean {
    return binding.webView.canGoBack()
}
```

위 메서드를 통해 이전 페이지 존재 여부를 확인하고 존재한다면 뒤로 가기를 실행하는 메서드를 `goBack()` 메서드를 만들어 작성해 준다. 그리고 뒤로 가기를 수행하는 메서드인 `webView.goBack()` 작성해 준다.

```kotlin
fun goBack() {
    binding.webView.goBack()
}
```

이후 MainActivity로 와서 `onBackPressed()` 메서드를 재정의하여 뒤로 가기를 수행했을 때의 동작을 지정한다. "currentFragment"라는 변수를 만들어 supportFragmentManager를 통해 Fragment들 중 첫 번째 Fragment에 대한 정보를 가져와 넣어준다. 이후 "currentFragment"가 WebViewFragment 인지에 대해 확인한 후 일치한다면 앞서 작성해 두었던 `canGoBack()`와 `goBack()` 메서드를 가져와 이전 페이지의 유무를 파악하고 뒤로 가기를 수행할 수 있도록 지정해 준다. 그렇지 않다면 앱을 종료하도록 한다.

구현한 앱을 실행시켜보면 이전 페이지가 존재할 경우 뒤로 가기를 수행하고 그렇지 않다면 앱을 종료시키는 것을 볼 수 있다.

<b>MainActivity.kt</b>

```kotlin
override fun onBackPressed() {
    val currentFragment = supportFragmentManager.fragments[0]
    if (currentFragment is WebViewFragment) {
        if (currentFragment.canGoBack()) {
            currentFragment.goBack()
        } else {
            super.onBackPressed()
        }
    } else {
        super.onBackPressed()
    }
}
```

## 📔 TabLayout 구현

ViewPager를 사용하여 3개의 Fragment를 슬라이드 또는 탭으로 이동하는 등의 기능을 추가하여 보자.

먼저 activity_main.xml에서 기존의 버튼 2개를 지워주고 ViewPager와 TabLayout을 추가시키도록 한다. 기존의 버튼 2개는 Fragment의 동작을 먼저 구현했던 것이고 ViewPager를 사용하여 다시 만들도록 한다. 

<b>activity_main.xml</b>

```kotlin
<?xml version="1.0" encoding="utf-8"?>
<androidx.constraintlayout.widget.ConstraintLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    tools:context=".MainActivity">

    <com.google.android.material.tabs.TabLayout
        android:id="@+id/tabLayout"
        android:layout_width="0dp"
        android:layout_height="wrap_content"
        app:layout_constraintEnd_toEndOf="parent"
        app:layout_constraintStart_toStartOf="parent"
        app:layout_constraintTop_toTopOf="parent" />

    <androidx.viewpager2.widget.ViewPager2
        android:id="@+id/viewPager"
        android:layout_width="match_parent"
        android:layout_height="0dp"
        app:layout_constraintBottom_toBottomOf="parent"
        app:layout_constraintTop_toBottomOf="@id/tabLayout" />

</androidx.constraintlayout.widget.ConstraintLayout>
```

이후 Adapter가 필요하기 때문에 ViewPagerAdapter라는 이름의 클래스를 새로 만들어 준다. 해당 클래스는 FragmentStateAdapter를 상속받고 FragmentActivity를 생성자 인자로 받기 때문에 mainActivity를 인자로 받아주는 클래스를 작성하도록 한다. 이후 두 개의 메서드를 재정의 하는데, `getItemCount()` 메서드는 ViewPager에 보일 fragment의 수를 반환한다. 

`createFragment()` 메서드는 매개변수로 받는 position에 따라 fragment 객체를 생성하고 반환하는 메서드이다. 따라서 해당 메서드에서 when 분기문으로 나누어 position에 따라 다른 Fragment를 생성하도록 한다. 해당 앱에서는 모두 WebViewFragment를 반환한다. 하지만 각각의 WebViewFragment는 서로 다른 Fragment이다.

<b>ViewPagerAdapter.kt</b>

```kotlin
class ViewPagerAdapter(mainActivity: MainActivity) : FragmentStateAdapter(mainActivity) {

    override fun getItemCount(): Int = 3

    override fun createFragment(position: Int): Fragment {
        return when (position) {
            0 -> {
                WebViewFragment()
            }
            1 -> {
                WebViewFragment()
            }
            else -> {
                WebViewFragment()
            }
        }
    }

}
```

Adapter 클래스에 대한 정의가 완료되었다면 MainActivity에서 Adapter를 연결해 준다. 다음으로 Tab 기능을 위해 TabLayoutMediator를 생성해 주고 인자로 xml 파일에서 만들어 둔 tabLayout과 viewPager, 그리고 tab의 이름을 지정하는 함수를 인자로 넣어준다. 마지막에 attach()를 붙이는 것을 잊지 않도록 하자.

<b>MainActivity.kt</b>

```kotlin
binding.viewPager.adapter = ViewPagerAdapter(this)

TabLayoutMediator(
    binding.tabLayout,
    binding.viewPager
) { tab, position ->
    tab.text = "fragment $position"
}.attach()
```

또한 Fragment를 표시하는 방식에 ViewPager가 추가되었으므로 앞서 작성했던 뒤로 가기 기능을 처리하는 `onBackPressed()` 메서드에서 "currentFragment" 변수를 정의할 때, 0번째 fragment가 아닌 viewPager의 현재 자리의 fragment로 코드를 바꾸어 준다.

```kotlin
override fun onBackPressed() {
    val currentFragment = supportFragmentManager.fragments[binding.viewPager.currentItem] // 5. fragments[0]에서 변경
    if (currentFragment is WebViewFragment) {
        if (currentFragment.canGoBack()) {
            currentFragment.goBack()
        } else {
            super.onBackPressed()
        }
    } else {
        super.onBackPressed()
    }
}
```

 ![](/assets/images/fastcampus/part2/webview/webview4.jpg)
 ![](/assets/images/fastcampus/part2/webview/webview5.jpg)

<br><br>

## 📔 마지막 시점 돌아가기 기능 구현

이번에는 "마지막 시점 돌아가기" 기능으로, 특정 웹툰 페이지의 url이 기록되어 있다면 sharedPreference를 통해 해당 url을 저장해 두었다가 다른 페이지에서 "마지막 시점 돌아가기" 버튼을 클릭했을 때 저장되었던 페이지로 돌아가는 기능을 구현해 보도록 하자.

웹툰 페이지의 Url이 저장되는 시점은 해당하는 페이지가 로드될 때이므로 초반에 언급했던 WebtoonWebViewClient의 `shouldOverrideLoading()` 메서드에서 url을 저장한다. 그리고 해당 클래스에서는 액티비티가 없으므로 getSharedPreferences 메서드를 호출할 수 없다. 따라서 클래스의 메서드를 함수로 등록하고 외부에서 클래스를 생성 시, getSharedPreferences 메서드를 활용하는 함수를 인자로 가져오도록 지정한다. 이를 위해 클래스 생성자 부분에 함수형 인자를 등록한다. 

`shouldOverrideUrlLoading()` 메서드에서 request를 널 체크하고 request의 url이 특정 웹툰 페이지의 url을 담고 있는 경우 해당 페이지를 저장하도록 구현한다.

<b>WebtoonWebViewClient.kt</b>

```kotlin
class WebtoonWebViewClient(
    private val progressBar: ProgressBar,
    private val saveData: (String) -> Unit
) : WebViewClient() {
    
    ...

    override fun shouldOverrideUrlLoading(view: WebView?, request: WebResourceRequest?): Boolean {
        if (request != null && request.url.toString().contains("comic.naver.com/webtoon/detail")) {
            saveData.invoke(request.url.toString())
        }
        return super.shouldOverrideUrlLoading(view, request)
    }
}
```

WebViewFragment로 돌아와 WebView를 생성하는 구문에서 url을 저장할 함수를 정의한다. `getSharedPreferences()` 메서드는 액티비티 상에서의 호출을 요구한다. Fragment는 자신을 호출하는 액티비티가 무엇인지 알기 때문에 `activity` 키워드를 통해 종속하고 있는 액티비티를 호출할 수 있다. 따라서 해당 키워드로 `getSharedPreferences()` 메서드를 호출하고 키값과 공개 설정을 지정해 준 뒤, edit을 통해 `putString()`으로 키값과 웹툰 페이지의 url을 넣어준다.

이후 `putString()`에 사용될 키값으로 각각의 탭을 구분해서 지정해 줘야 하는데, 현재로서는 각각의 탭을 구분할 수 있는 방법이 없다. 이를 위해 Fragment 생성 시 postion 값을 속성 값으로 넣어주어 생성되는 각각의 Fragment를 구분하고 이를 `putString()`의 키값으로 활용할 수 있다. 따라서 Fragment가 생성되는 ViewPagerAdapter 클래스에서 position을 인자로 넣어준다.

<b>ViewPagerAdapter.kt</b>

```kotlin
class ViewPagerAdapter(mainActivity: MainActivity) : FragmentStateAdapter(mainActivity) {

    ...

    override fun createFragment(position: Int): Fragment {
        return when (position) {
            0 -> {
                WebViewFragment(position)
            }
            1 -> {
                WebViewFragment(position)
            }
            else -> {
                WebViewFragment(position)
            }
        }
    }

}
```

이제 `putString()`의 키 값으로 각각의 Fragment를 구분해 지정할 수 있다.

<b>WebViewFragment.kt</b>

```kotlin
override fun onViewCreated(view: View, savedInstanceState: Bundle?) {
    super.onViewCreated(view, savedInstanceState)

    binding.webView.apply {
        webViewClient = WebtoonWebViewClient(binding.progressBar) { url ->
            activity?.getSharedPreferences(SHARED_PREFERENCE, Context.MODE_PRIVATE)?.edit {
                putString("tab$position", url)
            }
        }
        settings.javaScriptEnabled = true
        loadUrl("https://comic.naver.com/webtoon/detail?titleId=727188&no=202")
    }

    ...

    companion object {
        const val SHARED_PREFERENCE = "WEB_HISTORY"
    }
}
```

위 과정으로 특정 웹툰 페이지를 저장하였고 이제 버튼을 클릭했을 때 저장된 페이지를 불러오면 된다. "backToLastButton"의 클릭 리스너를 호출하고 블록 내부에서 sharedPreference를 불러온다. 이후 `getString()`을 통해 키값으로 해당 탭에 저장된 웹툰 페이지의 url을 가져오고 값이 존재한다면 `loadUrl()`을 통해 페이지 로드를, 없다면 Toast 메시지를 띄워주도록 한다.

```kotlin
override fun onViewCreated(view: View, savedInstanceState: Bundle?) {
    super.onViewCreated(view, savedInstanceState)

    binding.backToLastButton.setOnClickListener {
        val sharedPreference = activity?.getSharedPreferences(SHARED_PREFERENCE, Context.MODE_PRIVATE)
        val url = sharedPreference?.getString("tab$position", "")
        if (url.isNullOrEmpty()) {
            Toast.makeText(context, "마지막 저장 시점이 없습니다.", Toast.LENGTH_SHORT).show()
        } else {
            binding.webView.loadUrl(url)
        }
    }
}
```

<br><br>

## 📔 탭 이름 변경 기능 구현

하단의 "탭 이름 변경" 버튼을 클릭하면 상단 탭의 이름을 변경하도록 구현해 보자.

우선 버튼 클릭 시 동작하므로 "changeTabNameButton"에 클릭 리스너를 지정해 준다. 다이얼로그를 띄운 뒤 이름을 입력받고 변경을 수행할 것이므로 AlertDialog를 사용한다. EditText 위젯을 하나 생성한 뒤 `setPositiveButton()` 메서드에서 저장 버튼을 누르면 sharedPreference에 이름이 저장되도록 설정해 주었다. 

여기서 끝이 아니고 탭의 이름은 MainActivity에서 생성하기 때문에 MainActivity에 해당 값을 전달해 주어야 한다. 따라서 MainActivity에 변경된 값을 받을 리스너를 등록해 주어야 한다. 하단에 OnTabLayoutChanged라는 이름의 인터페이스를 생성한다. 인터페이스의 메서드로 `nameChanged(position: Int, name: String)`라는 이름의 메서드를 만들어 주었다. 인자로 위치와 이름을 받는다. 이 인터페이스를 listener라는 이름으로 속성 값을 추가해 준다.

<b>WebViewFragment.kt</b>

```kotlin
class WebViewFragment(private val position: Int,
                      private val webViewUrl: String,
                      private var listener : OnTabLayoutNameChanged? = null
) : Fragment() {

   ...

    override fun onViewCreated(view: View, savedInstanceState: Bundle?) {
        super.onViewCreated(view, savedInstanceState)

        ...

        binding.changeTabNameButton.setOnClickListener {
            AlertDialog.Builder(requireContext()).apply {
                val editText = EditText(context)
                setView(editText)
                setPositiveButton("저장") { _, _ ->
                    activity?.getSharedPreferences(SHARED_PREFERENCE, Context.MODE_PRIVATE)?.edit {
                        putString("tab${position}_name", editText.text.toString())
                        listener?.nameChanged(position, editText.text.toString())
                    }
                }
                setNegativeButton("취소") { dialogInterface, _ ->
                    dialogInterface.cancel()
                }
            }.show()
        }
    }
}

interface OnTabLayoutNameChanged {
    fun nameChanged(position: Int, name: String)
}
```

MainActivity에서는 먼저 앱을 실행했을 때의 탭의 이름 또는 변경된 탭의 이름을 가져오기 위해 sharedPreference에서 탭의 이름을 가져와 각각 "tab0", "tab1", "tab2"라는 이름의 변수로 지정해 주었다.

<b>MainActivity.kt</b>

```kotlin
override fun onCreate(savedInstanceState: Bundle?) {
    super.onCreate(savedInstanceState)
    binding = ActivityMainBinding.inflate(layoutInflater)
    setContentView(binding.root)

    ...

    override fun nameChanged(position: Int, name: String) {
        val tab = binding.tabLayout.getTabAt(position)
        tab?.text = name
    }
}
```

마지막으로 ViewPagerAdapter 클래스에서 Fragment 생성 시 MainActivity를 리스너로 지정해 주면 된다. 

<b>ViewPagerAdapter.kt</b>

```kotlin
class ViewPagerAdapter(private val mainActivity: MainActivity) : FragmentStateAdapter(mainActivity) {

    override fun getItemCount(): Int = 3

    override fun createFragment(position: Int): Fragment {
        return when (position) {
            0 -> {
                WebViewFragment(position, "https://comic.naver.com/webtoon?tab=mon", mainActivity)
            }
            1 -> {
                WebViewFragment(position, "https://comic.naver.com/webtoon?tab=tue", mainActivity)
            }
            else -> {
                WebViewFragment(position, "https://comic.naver.com/webtoon?tab=wed", mainActivity)
            }
        }
    }

}
```

다시 WebViewFragment로 돌아와 `listener.nameChanged()`의 인자로 위치와 변경된 이름을 넣어주면 된다.

<b>WebViewFragment.kt</b>

```kotlin
binding.changeTabNameButton.setOnClickListener {
    AlertDialog.Builder(requireContext()).apply {
        
        ...

        setPositiveButton("저장") { _, _ ->
            activity?.getSharedPreferences(SHARED_PREFERENCE, Context.MODE_PRIVATE)?.edit {
                putString("tab${position}_name", editText.text.toString())
                listener?.nameChanged(position, editText.text.toString())
            }
        }
        
        ...

    }.show()
}
```

실행 결과를 보면 탭 이름을 변경하고 저장을 눌렀을 때 이름이 즉각 변경되어 적용되는 것을 볼 수 있고, 앱을 다시 실행했을 때에도 저장된 이름이 반영되어 있는 것을 볼 수 있다.

 ![](/assets/images/fastcampus/part2/webview/webview6.jpg)
 ![](/assets/images/fastcampus/part2/webview/webview7.jpg)

 <br><br>

## 📔전체 코드
<https://github.com/Becomeproo/webtoon>

<br>

## 📔마무리
앱을 만들어 보며 Fragment의 생명 주기를 계속해서 고려해 가면서 만들어 보았다. 아직까진 많은 부분이 헷갈리지만 Fragment는 액티비티에 종속되어 실행되며 이를 고려하며 개발을 진행해야 한다는 것을 알았고, 탭의 이름을 변경하는 부분에 있어서 인터페이스를 등록해 MainActivity에서 동작하도록 하는 흐름이 이해가 잘되지 않았다. 이 부분을 계속해서 공부해 꼭 이해해야겠다.