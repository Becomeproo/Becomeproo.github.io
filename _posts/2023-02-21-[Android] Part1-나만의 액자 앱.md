---
title:  "[Android] Part1-나만의 액자 앱"

categories:
  - Android
tags:
  - [Android, Kotlin, 안드로이드 강의]

toc: true
toc_sticky: true
 
date: 2023-02-21 14:51:21+0900
last_modified_at: 2023-02-21 14:51:25+0900
---

<br>
<br>
<br>

> 아래 내용은 패스트 캠퍼스 [35개 프로젝트로 배우는 Android 앱 개발 feat. Jetpack Compose]의 강의 내용을 기반으로 작성한 자습 기록입니다.

<br><br>

# 나만의 액자 앱

## ℹ️앱 설명

외부 앱을 사용할 수 있는 권한을 받아 권한을 허용하면 이미지 파일을 리스트 형식으로 가져온 뒤, 해당 이미지들을 ViewPager를 사용해 슬라이드 방식으로 넘기면서 볼 수 있는 액자 앱

<br>

## ✅구현 기능

* 이미지를 가져오기 위한 권한 설정
* 가져온 이미지 리스트를 RecyclerView에 업데이트
* 액자 UI
* 커스텀 툴바

<br>

## ✅사용되는 기능

* UI
  * RecyclerView, ListAdapter
  * ViewPager2
  * Toolbar

* Kotlin
  * data class, sealed class

* Android
  * Permission
  * SAF(Storage Access Framework)
  * registerForActivityResult

---

<br><br>

## 📔기능 구현 - 권한 설정

외부 앱에서 이미지를 가져오기 전에, 이에 대한 권한을 먼저 설정해야 한다. 이미지를 가져와야 하는 상황에 맞는 권한 요청을 해야 하므로 xml 파일에 버튼을 하나 만들어 두었다. 해당 버튼을 클릭했을 때, 권한을 요청한다.

권한을 요청하기 위해서는 세 가지 조건에 따라 다른 기능을 제시한다. 먼저 해당 기능에 대한 권한이 허용되어 있을 경우에는 이미지를 가져오는 기능을 실행한다. 두 번째로 권한 요청에 대해 1회 승인을 거부하였을 경우, 권한을 승인해야 하는 이유에 대한 교육용 UI를 제공해야 한다. 따라서 이를 설명하기 위한 팝업 알림을 제공하기 위해 다이얼로그를 만들어 설명하도록 한다. 마지막은 위 두 상황을 제외한 모든 경우이므로 그대로 권한 요청을 보내도록 한다.

이미지를 가져오는 기능은 뒤에서 구현할 것이므로 일단 Toast 메시지를 통해 이미지를 가져올 기능임을 명시해 두었다.

`requestPermission()` 메서드는 권한을 요청하는 기능을 구현한다. `requestPermissions()` 메서드를 사용하여 가져올 권한이 무엇인지, 그리고 이를 구분하기 위한 코드를 인자로 넣어 권한을 요청한다.

`showRequestPermissionDialog()` 메서드에서는 권한 허용이 필요한 이유에 대해서 설명하는 다이얼로그를 표시한다. 다이얼로그에서 사용자에게 권한이 필요한 이유에 대해 최대한 자세히 설명할 필요가 있는데, 사용자가 앱을 신뢰하도록 하는 데 도움이 되고 권한을 승인할 가능성을 높일 수 있기 때문이다. 그럼에도 사용자가 승인을 거부한다면 개발자는 앱에 대한 사용을 제한하는 것이 아닌 해당 기능을 제외하고 앱을 사용할 수 있도록 도움을 주어야 한다. 본론으로 돌아와 해당 앱에서는 사용자가 권한을 승인하면 이미지를 가져오는 기능을 수행하도록 설정해 주었다. 다이얼로그의 마지막에 `show()`를 붙이는 것을 잊지 말자.

추가적으로 `onRequestPermissionResult()` 메서드를 사용한다. 해당 메서드는 시스템 권한 다이얼로그의 응답에 대한 처리를 설정할 수 있다. 먼저 사용된 권한을 구분하기 위해 앞서 코드를 등록해 둔 것이 매개변수 등록되어 있다. 여러 권한을 사용할 수 있기 때문에 when 조건문을 사용하여 코드를 통해 사용된 권한의 종류를 구분해 준다. 이후 응답한 값을 통해 권한이 허용되었는지에 대해 확인하고 승인 여부에 따라 설정해 줄 값을 지정해 준다. 해당 앱에서는 이미지를 가져오는 기능을 실행시킨다.

```kotlin
class MainActivity : AppCompatActivity() {
    private lateinit var binding: ActivityMainBinding
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        binding = ActivityMainBinding.inflate(layoutInflater)
        setContentView(binding.root)

        binding.loadImageButton.setOnClickListener {
            checkPermission()
        }
    }

    private fun checkPermission() {
        when {
            // 권한이 허용되어 있는 경우
            ContextCompat.checkSelfPermission(
                this,
                android.Manifest.permission.READ_MEDIA_IMAGES
            ) == PackageManager.PERMISSION_GRANTED -> {
                loadImage()
            }
            // 권한에 대해 1회 승인 거부했을 경우
            shouldShowRequestPermissionRationale(android.Manifest.permission.READ_MEDIA_IMAGES) -> {
                showRequestPermissionDialog()
            }
            else -> {
                requestPermission()
            }
        }
    }
    private fun requestPermission() {
        requestPermissions(arrayOf(android.Manifest.permission.READ_MEDIA_IMAGES), REQUEST_READ_MEDIA_IMAGES)
    }

    private fun showRequestPermissionDialog() {
        AlertDialog.Builder(this).apply {
            setMessage("이미지를 가져오기 위한 권한이 필요합니다.")
            setNegativeButton("취소", null)
            setPositiveButton("허용") { _, _ ->
                requestPermission()
            }
        }.show()
    }

    private fun loadImage() {
        Toast.makeText(this, "이미지를 가져올 예정", Toast.LENGTH_SHORT).show()
    }

    override fun onRequestPermissionsResult(
        requestCode: Int,
        permissions: Array<out String>,
        grantResults: IntArray
    ) {
        super.onRequestPermissionsResult(requestCode, permissions, grantResults)

        when (requestCode) {
            REQUEST_READ_MEDIA_IMAGES -> {
                if (grantResults.isNotEmpty() && grantResults[0] == PackageManager.PERMISSION_GRANTED) {
                    loadImages()
                }
            }
        }
    }

    companion object {
        private const val REQUEST_READ_MEDIA_IMAGES = 100
    }
}
```

결과를 보면 처음 '이미지 가져오기' 버튼을 눌렀을 때 안드로이드 시스템 권한 요청이 생성되고 여기서 '허용 안함'을 누르고 다시 권한을 요청하면 위에서 생성했던 다이얼로그가 표시된다. 여기서 '허용'을 누르면 이전의 시스템 권한 다이얼로그가 생성된다. 여기서도 권한을 허용하고 버튼을 누르면 이미지를 가져오는 기능이 실행되는 것을 볼 수 있다.

![](/assets/images/fastcampus/part1/gallery/gallery1.jpg)
![](/assets/images/fastcampus/part1/gallery/gallery2.jpg)
![](/assets/images/fastcampus/part1/gallery/gallery3.jpg)

<br><br>

## 📔기능 구현 - 이미지 가져오기

권한 승인이 완료된 후에는 외부 앱에서 이미지 Uri를 가져와 recyclerview에 띄워주도록 한다. Adapter를 생성하기 앞서 이를 위한 준비를 하기 위해 `activity_main.xml` 파일에 recyclerview를 생성해 주었다.

<b>activity_main.xml</b>
```kotlin
<?xml version="1.0" encoding="utf-8"?>
<androidx.constraintlayout.widget.ConstraintLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    tools:context=".MainActivity">

    <androidx.recyclerview.widget.RecyclerView
        android:id="@+id/imageRecyclerview"
        android:layout_width="match_parent"
        android:layout_height="0dp"
        android:layout_marginBottom="16dp"
        app:layout_constraintBottom_toTopOf="@id/loadImageButton"
        app:layout_constraintTop_toTopOf="parent" />

    <Button
        android:id="@+id/loadImageButton"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="이미지 가져오기"
        app:layout_constraintBottom_toBottomOf="parent"
        app:layout_constraintEnd_toEndOf="parent"
        app:layout_constraintStart_toStartOf="parent"
        app:layout_constraintTop_toTopOf="parent"
        app:layout_constraintVertical_bias="0.9" />

</androidx.constraintlayout.widget.ConstraintLayout>
```

다음으로 recycler 내부에 들어갈 아이템 뷰를 생성해 준다. 선택된 이미지들이 표시되고 그 뒤에 같은 형식의 이미지 불러오기 기능을 가진 아이템 뷰를 하나 더 만들어 주었다.

<b>item_image.xml</b>
```kotlin
<?xml version="1.0" encoding="utf-8"?>
<androidx.constraintlayout.widget.ConstraintLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="wrap_content"
    xmlns:app="http://schemas.android.com/apk/res-auto">

    <ImageView
        android:id="@+id/imageItemImageView"
        android:layout_width="match_parent"
        android:layout_height="0dp"
        app:layout_constraintTop_toTopOf="parent"
        app:layout_constraintDimensionRatio="1.0" />

</androidx.constraintlayout.widget.ConstraintLayout>
```
<b>item_load_more_image.xml</b>
```kotlin
<?xml version="1.0" encoding="utf-8"?>
<androidx.constraintlayout.widget.ConstraintLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    android:layout_width="match_parent"
    android:layout_height="wrap_content">

    <TextView
        android:id="@+id/loadMoreImageTextView"
        android:layout_width="match_parent"
        android:layout_height="0dp"
        android:gravity="center"
        android:textSize="18sp"
        android:textColor="@color/black"
        android:text="사진 불러오기.."
        app:layout_constraintDimensionRatio="1.0"
        app:layout_constraintTop_toTopOf="parent" />

</androidx.constraintlayout.widget.ConstraintLayout>
```

이제 adapter를 만들어보자. 이번에는 RecyclerView.Adapter가 아닌 ListAdapter를 사용해 보도록 한다. ListAdapter는 RecyclerView.Adapter보다 더욱 추가된 요소들에 대한 관리가 편리하다. RecyclerView.Adapter에서 각각의 추가된 사항들마다 해당하는 메서드들을 작성해 업데이트를 해주었지만 ListAdapter에서는 DiffUtil 클래스가 있어 이러한 변경 사항에 대해 좀 더 편리하게 처리할 수 있다.

ImageAdapter라는 이름의 ListAdapter를 상속받는 클래스를 생성해 주었다. 생성될 아이템 뷰의 종류는 두 가지로 나뉜다. 가져온 이미지를 출력하는 아이템 뷰와 가장 뒤에 위치해 이미지를 추가하는 기능을 가진 아이템 뷰가 존재한다. 따라서 두 가지 요소는 반드시 들어가야 할 요소들이므로 sealed class를 만들어 주었다. sealed class는 반드시 필요한 요소들에 대해 컴파일러 상에서 해당 요소들에 대한 올바른 관리를 할 수 있도록 도와준다. 주로 when 조건문에서 사용되는데 sealed class 요소 중 하나가 조건문에서 입력되지 않았다면 컴파일러가 이를 파악하고 경고를 표시한다. 

<b>ImageAdapter.kt</b>
```kotlin
sealed class ImageItems {
    data class Image(
        val uri: Uri,
    ) : ImageItems()

    object LoadMoreImage : ImageItems()
}
```

sealed class가 포함된 adapter 클래스의 생성은 다음과 같다. ListAdapter의 인자에 sealed class인 ImageItems와 RecyclerView.ViewHolder가 인자로 들어가고, 그 뒤에 DiffUtil 클래스를 정의해서 넣어주었다.

DiffUtil의 ItemCallback은 두 가지 메서드를 사용한다. `areContentsTheSame()`과 `areItemsTheSame()`으로 아이템이 추가할 때마다 기존 아이템과 새로운 아이템에 대해 각각 두 항목의 데이터가 동일한지의 여부와 두 객체가 동일한 항목을 나타내는지를 확인하기 위해 호출된다. DiffUtil이 동작하는 방법에 대해서 나중에 자세히 다루어보도록 한다.

두 종류의 아이템을 연결시키기 때문에 두 가지 메서드를 재정의해 아이템을 구분하고 따로 추가할 수 있도록 해보자. `getItemCount()` 메서드를 통해 현재 리스트의 아이템의 개수가 없다면 그대로 0을 반환하고 아이템이 존재한다면 이미지 추가 기능을 수행할 아이템 뷰를 위해 개수를 하나 더 추가하여 반환하도록 하였다. `getItemViewType()` 메서드를 통해 viewType을 반환받을 수 있는데 이때 현재 아이템의 개수에서 1을 줄였을 때와 아이템 인덱스의 값이 일치한다면 마지막 아이템임을 의미하므로 해당 아이템의 viewType을 이미지 추가 기능을 수행할 ITEM_LOAD_MORE로 반환하도록 하였다.

위 두 과정을 토대로 `onCreateViewHolder()` 메서드에서 viewType을 기준으로 각각의 경우에 맞게 뷰 바인딩을 호출하여 ViewHolder로 반환해 준다. `onBindViewHolder()`에서도 마찬가지로 해당하는 viewType에 맞게 아이템을 넘겨준다. 여기서 이미지 추가 기능을 사용하는 LoadMoreImageViewHolder에는 ItemClickListener 인터페이스를 만들어 인자로 넣어주도록 한다. 

<b>ImageAdapter.kt</b>
```kotlin
class ImageAdapter(
    private val itemClickListener: ItemClickListener
) : ListAdapter<ImageItems, RecyclerView.ViewHolder>(diffUtil) {

    override fun getItemCount(): Int {
        val originSize = currentList.size
        return if(originSize == 0) 0 else originSize.inc()
    }

    override fun getItemViewType(position: Int): Int {
        return if (itemCount.dec() == position) ITEM_LOAD_MORE else ITEM_IMAGE
    }

    override fun onCreateViewHolder(parent: ViewGroup, viewType: Int): RecyclerView.ViewHolder {
        return when (viewType) {
            ITEM_IMAGE -> {
                val inflater = parent.context.getSystemService(Context.LAYOUT_INFLATER_SERVICE) as LayoutInflater
                val binding = ItemImageBinding.inflate(inflater, parent, false)
                ImageViewHolder(binding)
            }
            else -> {
                val inflater = parent.context.getSystemService(Context.LAYOUT_INFLATER_SERVICE) as LayoutInflater
                val binding = ItemLoadMoreImageBinding.inflate(inflater, parent, false)
                LoadMoreImageViewHolder(binding)
            }
        }
    }

    override fun onBindViewHolder(holder: RecyclerView.ViewHolder, position: Int) {
        when(holder) {
            is ImageViewHolder -> {
                holder.bind(currentList[position] as ImageItems.Image)
            }
            is LoadMoreImageViewHolder -> {
                holder.bind(itemClickListener)
            }
        }
    }

    interface ItemClickListener {
        fun onLoadMoreClick()
    }

    companion object {
        val diffUtil = object : DiffUtil.ItemCallback<ImageItems>() {
            override fun areItemsTheSame(oldItem: ImageItems, newItem: ImageItems): Boolean {
                return oldItem === newItem
            }

            override fun areContentsTheSame(oldItem: ImageItems, newItem: ImageItems): Boolean {
                return oldItem == newItem
            }
        }

        const val ITEM_IMAGE = 0
        const val ITEM_LOAD_MORE = 1
    }
}

class ImageViewHolder(private val binding: ItemImageBinding) : RecyclerView.ViewHolder(binding.root) {
    fun bind(image: ImageItems.Image) {
        binding.imageItemImageView.setImageURI(image.uri)
    }
}

class LoadMoreImageViewHolder(binding: ItemLoadMoreImageBinding) : RecyclerView.ViewHolder(binding.root) {
    fun bind(itemClickListener: ImageAdapter.ItemClickListener) {
        itemView.setOnClickListener { itemClickListener.onLoadMoreClick() }
    }
}
```

이제 MainActivity에서 recyclerView에 adapter를 등록해 주도록 한다. ImageAdapter의 인자로 클릭 리스너에 대해 이미지 추가 기능을 수행하도록 checkPermission()을 넣어 권한 요청이 되어있는지 먼저 확인한 뒤 기능을 수행할 수 있도록 해주었다. layoutManager의 인자로 격자 모양으로 이미지들을 출력할 것이기 때문에 GridLayoutManager를 사용하여 한 행에 2개의 아이템이 들어갈 수 있도록 설정해 주었다.

<b>MainActivity.kt</b>

```kotlin
private fun initRecyclerView() {
    imageAdapter = ImageAdapter(object : ImageAdapter.ItemClickListener {
        override fun onLoadMoreClick() {
            checkPermission()
        }
    })

    binding.imageRecyclerview.apply {
        adapter = imageAdapter
        layoutManager = GridLayoutManager(context, 2)
    }
}
```

이제 생성된 adapter에 이미지를 넣어주기만 하면 된다. registerForActivityRersult를 사용하여 선언한 `updateImages()` 메서드에서 받아온 uri 리스트를 데이터 클래스인 ImageItems.Image에 넣어주고 images 변수에 할당해 주었다. 여기서 이미지를 누적해서 추가할 수 있도록 구현하기 위해 imageAdapter의 currentList를 호출하면 누적되어 있는 이미지의 리스트를 불러올 수 있고 이를 변경 가능한 리스트로 변환하여 만들어둔 이미지를 넣어준 뒤, `submitList()`를 통해 업데이트해주면 된다.

```kotlin
private fun updateImages(uriList: List<Uri>) {
    val images = uriList.map { ImageItems.Image(it) }
    val updatedImages = imageAdapter.currentList.toMutableList().apply { addAll(images) }
    imageAdapter.submitList(updatedImages)
}
```

결과를 확인하면 선택된 이미지 모두 외부 앱에서 잘 가져온 것을 볼 수 있고, 여기에 다시 이미지를 추가하면 누적된 이미지 또한 잘 추가된 것을 볼 수 있다.

![](/assets/images/fastcampus/part1/gallery/gallery4.jpg)
![](/assets/images/fastcampus/part1/gallery/gallery5.jpg)

<br><br>

## 📔기능 구현 - 앨범에 이미지 불러오기

앨범 화면으로 이동하여 가져온 이미지들을 출력시켜보도록 하자. 출력하는 방법으로는 ViewPager2를 사용하였다. ViewPager2를 사용하면 각각의 이미지들을 슬라이드 하여 한 장씩 보이게 할 수 있다. ViewPager를 구현하는 방법은 RecyclerView와 크게 다르지 않다. 이를 위해 먼저 각각의 이미지를 보여줄 아이템 뷰를 생성한다.
 
앨범을 표시할 FrameActivity를 새로 생성하고 ViewPager를 만들어 준다. 

<b>activity_fram.xml</b>
```kotlin
<?xml version="1.0" encoding="utf-8"?>
<androidx.constraintlayout.widget.ConstraintLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    tools:context=".FrameActivity">

    <androidx.viewpager2.widget.ViewPager2
        android:id="@+id/viewPager"
        android:layout_width="match_parent"
        android:layout_height="match_parent" />
    
</androidx.constraintlayout.widget.ConstraintLayout>
```

ViewPager에 표시할 아이템 뷰를 생성한다. 아이템 뷰의 이름은 item_frame으로 지정해 주었다.

<b>item_frame.xml</b>
```kotlin
<?xml version="1.0" encoding="utf-8"?>
<androidx.constraintlayout.widget.ConstraintLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent">

    <ImageView
        android:id="@+id/frameImageView"
        android:layout_width="match_parent"
        android:layout_height="match_parent" />

</androidx.constraintlayout.widget.ConstraintLayout>
```

RecyclerView를 생성할 때와 마찬가지로 ViewPager 또한 Adapter를 필요로 한다. 따라서 RecyclerView.Adapter를 상속받는 Adapter 클래스를 생성해 준다. 구성하는 것은 RecyclerView와 동일하다. 

<b>FrameAdapter.kt</b>

```kotlin

class FrameAdapter(private val images : List<Image>) : RecyclerView.Adapter<FrameViewHolder>(){
    override fun onCreateViewHolder(parent: ViewGroup, viewType: Int): FrameViewHolder {
        val inflater = parent.context.getSystemService(Context.LAYOUT_INFLATER_SERVICE) as LayoutInflater
        val binding = ItemFrameBinding.inflate(inflater, parent, false)
        return FrameViewHolder(binding)
    }

    override fun onBindViewHolder(holder: FrameViewHolder, position: Int) {
        holder.bind(images[position])
    }

    override fun getItemCount(): Int = images.size
}

data class Image(
    val uri: Uri,
)

class FrameViewHolder(private val binding: ItemFrameBinding) : RecyclerView.ViewHolder(binding.root) {
    fun bind(image: Image) {
        binding.frameImageView.setImageURI(image.uri)
    }
}
```

이제 FrameActivity에서 ViewPager를 연결해 주면 된다. `intent.getStringArrayExtra()`를 사용해 앞에서 보낸 문자열 배열 자료형의 이미지를 가져온다. 이후 해당 값들을 map을 사용해 Uri 자료형으로 변환하여 ViewPager를 위한 데이터 클래스에 넣어준다. 해당 값을 변수로 지정해 Adapter에 넣어준 뒤, viewPager의 adapter에 연결해 주면 된다.

<b>FrameActivity.kt</b>

```kotlin
class FrameActivity : AppCompatActivity() {
    private lateinit var binding: ActivityFrameBinding

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        binding = ActivityFrameBinding.inflate(layoutInflater)
        setContentView(binding.root)


        val images = (intent.getStringArrayExtra("images") ?: emptyArray())
            .map { uriString -> Image(Uri.parse(uriString)) }
        val frameAdapter = FrameAdapter(images)

        binding.viewPager.adapter = frameAdapter

    }
}
```

ViewPager에 가져온 이미지들이 잘 보이는 것을 볼 수 있다.

![](/assets/images/fastcampus/part1/gallery/gallery6.jpg)
![](/assets/images/fastcampus/part1/gallery/gallery7.jpg)

<br><br>

## 📔기능 구현 - Indicator 구현

viewPager의 하단에 현재 페이지의 위치를 보여주는 Indicator를 구현해 보도록 하자. 이를 위해서는 TabLayout을 만들어줘야 한다. viewPager를 담고 있는 activity_frame 파일에 tabLayout을 생성해 준다. 일반적인 tabLayout을 사용하면 자주 보이는 Indicator가 생성되지 않으므로 몇 가지 파일을 만들어 추가해 주도록 한다.

 먼저 선택된 페이지를 보여주기 위한 모양을 만들기 위해 하나는 선택된 페이지의 모양을, 다른 하나는 그 외 페이지를 표시하기 위한 새로운 리소스 파일을 두 개 만들어 주었다.

<b>ic_item_selected.xml</b>
 ```kotlin
<?xml version="1.0" encoding="utf-8"?>
<shape xmlns:android="http://schemas.android.com/apk/res/android"
    android:shape="ring"
    android:innerRadius="0dp"
    android:thickness="3dp"
    android:useLevel="false">
    <solid android:color="@color/black" />
</shape>
```

<b>ic_item_unselected.xml</b>
```kotlin
<?xml version="1.0" encoding="utf-8"?>
<shape xmlns:android="http://schemas.android.com/apk/res/android"
    android:shape="ring"
    android:innerRadius="0dp"
    android:thickness="3dp"
    android:useLevel="false">
    <solid android:color="#ABABAB" />
</shape>
````

위 두 리소스 파일을 활용한 selector 파일을 만들어 준다. 해당 파일에서 선택되었을 때와 그렇지 않을 때의 모양을 각각 지정해 준다.

<b>frame_selector.xml</b>
```kotlin
<?xml version="1.0" encoding="utf-8"?>
<selector xmlns:android="http://schemas.android.com/apk/res/android">
    <item android:drawable="@drawable/ic_item_selected" android:state_selected="true" />
    <item android:drawable="@drawable/ic_item_unselected" android:state_selected="false" />
</selector>
```

위 설정을 마쳤다면 다시 activity_frame 파일로 돌아와 tabLayout에서 `app:tabIndicator` 속성을 null로 설정해 준다. null 값으로 지정해 줘야 기존 indicator의 모양이 제거된다. 다음으로 방금 구현한 indicator를 적용하기 위해 `app:tabBackground`속성의 값으로 `frame_selector`를 지정해 준다.

<b>activity_frame.xml</b>
```kotlin
<?xml version="1.0" encoding="utf-8"?>
<androidx.constraintlayout.widget.ConstraintLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    tools:context=".FrameActivity">

    <androidx.viewpager2.widget.ViewPager2
        android:id="@+id/viewPager"
        android:layout_width="match_parent"
        android:layout_height="match_parent" />

    <com.google.android.material.tabs.TabLayout
        android:id="@+id/tabLayout"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:background="@null"
        android:paddingHorizontal="12dp"
        app:layout_constraintBottom_toBottomOf="parent"
        app:layout_constraintStart_toStartOf="parent"
        app:layout_constraintEnd_toEndOf="parent"
        app:tabIndicator="@null"
        app:tabBackground="@drawable/frame_selector"
        />

</androidx.constraintlayout.widget.ConstraintLayout>
```

이제 Indicator를 클래스 파일에서 연결하면 된다. FrameActivity에서 `TabLayoutMediator()`를 사용해 인자로 tabLayout과 viewPager를 넣어준 뒤, 구성 방식으로 viewpager의 현재 아이템에 Indicator의 위치 값을 할당해 준다. 끝으로 `attach()`를 작성해 주면 Indicator가 적용된 것을 볼 수 있다.

![](/assets/images/fastcampus/part1/gallery/gallery10.jpg)
![](/assets/images/fastcampus/part1/gallery/gallery11.jpg)

<br><br>

## 📔기능 구현 - Toolbar 구현

Toolbar를 구현하기 위해 기존의 ActionBar의 값을 바꾸어 주어야 한다. `themes.xml` 파일과 `themes.xml(nitght).xml` 파일 모두 ActionBar 속성을 NoActionBar로 바꾸어 준다.

<b>themes.xml</b>
```kotlin
<resources xmlns:tools="http://schemas.android.com/tools">
    
    <style name="Theme.RepeatMyGallery" parent="Theme.MaterialComponents.DayNight.NoActionBar">

        ...
    
    </style>
</resources>
```

이후 activity_main.xml 파일과 activity_frame.xml 파일에 toolbar 위젯을 구현해 준다. 

<b>activity_main.xml</b>
```kotlin
<?xml version="1.0" encoding="utf-8"?>
<androidx.constraintlayout.widget.ConstraintLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    tools:context=".MainActivity">
    
    <androidx.appcompat.widget.Toolbar
        android:id="@+id/toolbar"
        android:layout_width="match_parent"
        android:background="#673AB7"
        android:layout_height="?attr/actionBarSize"
        app:layout_constraintTop_toTopOf="parent" />

    <androidx.recyclerview.widget.RecyclerView
        android:id="@+id/imageRecyclerview"
        android:layout_width="match_parent"
        android:layout_height="0dp"
        android:layout_marginBottom="16dp"
        app:layout_constraintBottom_toTopOf="@id/loadImageButton"
        app:layout_constraintTop_toBottomOf="@id/toolbar" />

    <Button
        android:id="@+id/loadImageButton"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="이미지 가져오기"
        app:layout_constraintBottom_toBottomOf="parent"
        app:layout_constraintEnd_toEndOf="parent"
        app:layout_constraintStart_toStartOf="parent"
        app:layout_constraintTop_toTopOf="parent"
        app:layout_constraintVertical_bias="0.9" />

    <Button
        android:id="@+id/navigateToFrameButton"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        app:layout_constraintBottom_toBottomOf="parent"
        android:text="나만의 앨범 만들기" />

</androidx.constraintlayout.widget.ConstraintLayout>
```

이미지를 가져오는 화면에서는 우측 상단의 추가 아이콘을 누르면 이미지를 가져오는 기능을 추가로 구현할 것이므로 menu 이름의 리소스 폴더를 만들고 menu 레이아웃 파일을 만들어 준다.

title과 icon의 속성 값을 설정해 주고, showAsAction 속성으로 아이콘이 항상 보이도록 설정해 주었다.

<b>menu_main.xml</b>
```kotlin
<?xml version="1.0" encoding="utf-8"?>
<menu xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto">
    <item
        android:id="@+id/actionAdd"
        android:title="사진 가져오기"
        android:icon="@drawable/ic_baseline_add_24"
        app:showAsAction="always" />
</menu>
```

이제 클래스 파일에 각각의 toolbar를 연결시켜주면 된다. MainActivity.kt 파일에서 toolbar에 연결한 뒤 title 값으로 toolbar에 명시될 제목을 설정해 주고, `setSupportActionBar()` 메서드에 해당 toolbar를 연결시켜준다.

<b>MainActivity.kt</b>

```kotlin
override fun onCreate(savedInstanceState: Bundle?) {
    super.onCreate(savedInstanceState)
    binding = ActivityMainBinding.inflate(layoutInflater)
    setContentView(binding.root)

    binding.toolbar.apply {
        title = "사진 가져오기"
        setSupportActionBar(this)
    }

    ...

}
```

추가 기능을 구현하였으므로 이 또한 명시해 주어야 한다. `onCreateOptionsMenu()` 메서드를 재정의하여 menuInflater를 통해 기능을 연결해 준다. 또한 `onOptionsItemSelected()` 메서드에서 해당 menu가 클릭되었을 때의 구현할 기능도 명시해 준다.

```kotlin
override fun onCreateOptionsMenu(menu: Menu?): Boolean {
    menuInflater.inflate(R.menu.menu_main, menu)
    return true
}

override fun onOptionsItemSelected(item: MenuItem): Boolean {
    return when (item.itemId) {
        R.id.actionAdd -> {
            checkPermission()
            true
        }
        else -> {
            super.onOptionsItemSelected(item)
        }
    }
}
```

FrameActivity에서 또한 마찬가지로 toolbar를 등록해 주고 뒤로 가기 버튼을 구현해 준다. 뒤로 가기 버튼은 안드로이드에서 기본적으로 제공하기 때문에 클래스 파일 상에서 처리한다.

`setDisplayHomeAsUpEnabled()` 메서드의 인자로 true를 넣어주면 뒤로 가기 기능을 구현할 것임을 의미한다. 

<b>FrameActivity.kt</b>

```kotlin
override fun onCreate(savedInstanceState: Bundle?) {
    super.onCreate(savedInstanceState)
    binding = ActivityFrameBinding.inflate(layoutInflater)
    setContentView(binding.root)

    binding.toolbar.apply {
        title = "나만의 앨범"
        setSupportActionBar(this)
    }
    supportActionBar?.setDisplayHomeAsUpEnabled(true)

}
```

이후 `onOptionsItemSelected()`를 사용하여 뒤로 가기 버튼이 눌렸을 때의 기능을 구현해 준다.

```kotlin
override fun onOptionsItemSelected(item: MenuItem): Boolean {
    return when (item.itemId) {
        android.R.id.home -> {
            finish()
            true
        }
        else -> {
            super.onOptionsItemSelected(item)
        }
    }
}
```

![](/assets/images/fastcampus/part1/gallery/gallery8.jpg)
![](/assets/images/fastcampus/part1/gallery/gallery9.jpg)

<br>

## 📔전체 코드
<https://github.com/Becomeproo/my_gallery>

<br>

## 📔마무리
RecyclerView.Adapter와 ListAdapter를 비교했을 때 변경 사항이 많은 항목을 처리할 때에는 ListAdapter가 더욱 효율적이고 비용을 절감할 수 있다는 점을 느꼈다. ViewPager를 만드는 데 있어서 역시 제대로 해보지 않아 두려워했을 뿐 RecyclerView Adapter와 구현되는 과정이 동일해 정말 공부를 제대로 하고 있지 않았다라는 생각을 하였고 반성하게 되었다. 배울게 많이 남아있다는 점에서 더욱 의지를 갖고 부지런히 해야겠다는 생각을 굳건히 하게 되었다.