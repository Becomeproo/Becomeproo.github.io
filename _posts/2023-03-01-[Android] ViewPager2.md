---
title:  "[Android] ViewPager2"

categories:
  - Android
tags:
  - [Android, Android 정리, ViewPager2]

toc: true
toc_sticky: true
 
date: 2023-03-01 12:24:27+0900
last_modified_at: 2023-03-01 12:24:32+0900

---

<br>
<br>
<br>

> 아래 내용은 [geesksforgeeks-viewpager2 문서](https://www.geeksforgeeks.org/viewpager2-in-android-with-example/)의 내용을 참고하여 작성한 자습 기록입니다.

<br>
<br>

# 📚ViewPager2
ViewPager2는 기존 ViewPager 위젯을 보완하여 업데이트된 위젯으로, 기존 ViewPager는 PagerAapter 기반으로 구성되어 있어 스크롤 진행 시, instantiateItem()과 destroyItem() 메서드가 호출되기 때문에 버벅거리는 현상이 발생했다. ViewPager2는 이를 보완하여 RecyclerView를 기반으로 만들어져 이러한 문제를 해결할 수 있다.
또한 사용자가 화면을 수평으로만 슬라이드 할 수 있던 반면 ViewPager2는 수평 슬라이드뿐만 아니라 수직 슬라이드 또한 지원한다.

![](/assets/images/android/viewpager/viewpager1.png)

<br>

이외에도 ViewPager2에 대한 특징은 다음과 같다.

* Fragment 지원
* DiffUtil 클래스 사용 가능
* RTL(Right-to-Left) 지원

# 📚ViewPager2 구현

ViewPager2 구현을 위해 xml 파일에 viewPager를 작성해 준다. ViewPager2는 Android Studio에서 기본적으로 지원하기 때문에 의존성을 따로 추가하지 않아도 된다. 

<b>activity_main.xml</b>

```kotlin
<?xml version="1.0" encoding="utf-8"?>
<androidx.constraintlayout.widget.ConstraintLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    tools:context=".MainActivity">

    <androidx.viewpager2.widget.ViewPager2
        android:id="@+id/viewPager"
        android:layout_width="0dp"
        android:layout_height="0dp"
        app:layout_constraintStart_toStartOf="parent"
        app:layout_constraintEnd_toEndOf="parent"
        app:layout_constraintTop_toTopOf="parent"
        app:layout_constraintBottom_toBottomOf="parent" />

</androidx.constraintlayout.widget.ConstraintLayout>
```

이후 ViewPager의 아이템에 표시될 레이아웃을 추가한다. 구성할 내용으로는 간단히 이미지만 보이도록 하겠다.

<b>item_image.xml</b>

```kotlin
<?xml version="1.0" encoding="utf-8"?>
<androidx.constraintlayout.widget.ConstraintLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent">

    <ImageView
        android:id="@+id/itemImageView"
        android:layout_width="match_parent"
        android:layout_height="match_parent" />

</androidx.constraintlayout.widget.ConstraintLayout>
```

RecyclerView와 구현하는 과정이 동일하다. 다음으로 추가한 아이템을 표시하기 위해 Adapter 클래스를 만들어 준다. 연결하는 방법 또한 RecyclerView와 동일하다.

<b>ViewPagerAdapter.kt</b>

```kotlin
class ViewPagerAdapter(private val images: List<Image>) : RecyclerView.Adapter<ImageViewHolder>() {
    override fun onCreateViewHolder(parent: ViewGroup, viewType: Int): ImageViewHolder {
        val inflater = parent.context.getSystemService(Context.LAYOUT_INFLATER_SERVICE) as LayoutInflater
        val binding = ItemImageBinding.inflate(inflater, parent, false)
        return ImageViewHolder(binding)
    }

    override fun onBindViewHolder(holder: ImageViewHolder, position: Int) {
        holder.bind(images[position])
    }

    override fun getItemCount(): Int = images.size

}

data class Image(
    val image: Int
)

class ImageViewHolder(private val binding: ItemImageBinding) : RecyclerView.ViewHolder(binding.root) {
    fun bind(dataImage: Image) {
        binding.itemImageView.setImageResource(dataImage.image)
    }
}
```

ViewPager를 표시하기 위한 준비가 완료되었으니 MainActivity에 표시할 데이터를 추가하고 Adapter를 연결시켜주기만 하면 된다. 

<b>MainActivity.kt</b>

```kotlin
class MainActivity : AppCompatActivity() {
    private lateinit var binding: ActivityMainBinding

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        binding = ActivityMainBinding.inflate(layoutInflater)
        setContentView(binding.root)

        val dummyUri = listOf(
            Image(R.drawable.androidparty),
            Image(R.drawable.androidparty),
            Image(R.drawable.androidparty),
            Image(R.drawable.androidparty),
            Image(R.drawable.androidparty),
            Image(R.drawable.androidparty)
        )

        binding.viewPager.adapter = ViewPagerAdapter(dummyUri)

    }
}
```

# 📚ViewPager Indicator

추가적으로 표시된 ViewPager에 대한 indicator를 구현할 수 있다. 간단한 예시로 각각의 구현된 아이템의 위치에 대해서 점으로 표시하고 현재 위치해 있는 아이템에 indicator의 색을 다르게 하여 표시하는 indicator를 구현해 보자.

먼저 activity_main에서 TabLayout을 원하는 위치에 지정해 준다. 이후 indicator의 모양으로 직접 만들어 구현할 것이므로 drawable 파일을 생성하여 지정해 준다. 위치에 있을 때와 그렇지 않을 때 표시해야 할 색이 다르기 때문에 두 개의 파일을 만들어 준다.

<b>tab_layout_item_selected.xml</b>

```kotlin
<?xml version="1.0" encoding="utf-8"?>
<shape xmlns:android="http://schemas.android.com/apk/res/android"
    android:shape="ring"
    android:thickness="3dp"
    android:innerRadius="0dp"
    android:useLevel="false">
    <solid android:color="@color/white" />
</shape>
```

<b>tab_layout_item_unselected.xml</b>

```kotlin
<?xml version="1.0" encoding="utf-8"?>
<shape xmlns:android="http://schemas.android.com/apk/res/android"
    android:shape="ring"
    android:thickness="3dp"
    android:innerRadius="0dp"
    android:useLevel="false">
    <solid android:color="@color/gray" />
</shape>
```

이후 두 가지 유형을 구분하여 표시할 수 있는 selector 파일을 만들어 준다.

<b>tab_layout_selector.xml</b>

```kotlin
<?xml version="1.0" encoding="utf-8"?>
<selector xmlns:android="http://schemas.android.com/apk/res/android">
<item android:state_selected="true" android:drawable="@drawable/tab_layout_item_selected" />
<item android:state_selected="false" android:drawable="@drawable/tab_layout_item_unselected" />
</selector>
```

해당 내용을 적용하기 위해 다시 activity_main의 TabLayout으로 돌아와 `app:tabIndicator="@null"` 속성을 통해 기존 indicator에 적용되어 있던 내용을 초기화시켜 준다. 이후 `app:tabBackground="@drawable/tab_layout_selector"` 속성으로 적용시킬 indicator를 지정해 주면 된다.

이제 MainActivity에서 indicator를 viewpager와 연결시켜 준다. TabLayoutMediator를 생성하여 인자 값으로 연결시킬 TabLayout과 ViewPager를 넣어주고 람다 내부에는 표시할 방법을 넣어준다. 아래 내용을 작성하면 Indicator가 적용된 ViewPager를 볼 수 있다.

<b>MainActivity.kt</b>

```kotlin
TabLayoutMediator(
    binding.tabLayout,
    binding.viewPager
) { tab, position ->
    binding.viewPager.currentItem = tab.position
}.attach()
```
