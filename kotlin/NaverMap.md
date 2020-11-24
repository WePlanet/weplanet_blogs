<br>

### NaverMap 장점     #


1. 국내용 서비스로는 구글맵보다 유리

2. 비용 무료

3. 쉬운 설치 과정 ( 카카오맵은 설치과정이 까다로움 ( lib 파일 설치등) ) 

4. A/S 문의 대응이 빠르다 ( 보통 3~4시간 안에 답변을 받음 )

![img](https://blog.kakaocdn.net/dn/bDzPzK/btqMMiVFlD8/DKZJt6Qnlgnm9SbJoYpAFK/img.png)

![img](https://blog.kakaocdn.net/dn/c5iYMd/btqMOxKNTwh/OasVveq5AKjbeazgF8TvU0/img.png)

**Mobile Dynamic Map API**를 이용할 예정입니다  
<br>
## 1. 초기 설정   

네이버 공식 문서에 잘 되어있으므로 링크로 대신합니다

https://navermaps.github.io/android-map-sdk/guide-ko/1.html  

<br>

정리하자면 

네이버맵을 사용하기 위해 두가지 방법이 있습니다.

1) android.support.v4.app.Fragment 를 상속한 MapFragment

```xml
<fragment
    android:id="@+id/map"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:name="com.naver.maps.map.MapFragment" />
```

2) com.naver.maps.map.MapView 객체 사용 

```xml
<com.naver.maps.map.MapView
    android:id="@+id/map_view"
    android:layout_width="match_parent"
    android:layout_height="match_parent" />
```

2번 방법으로 사용하는 경우 Activity/Fragment의 라이프사이클과 싱크를 맞춰줘야하므로 (번거로움)

**네이버에서 권장하는 방법은 1번 방법입니다**  
<br>
## 2. 구현 예정 디자인

![img](https://blog.kakaocdn.net/dn/bPPKxN/btqMPifBsCR/hs6KurVXovYGgLVZgm5kzk/img.png)

TabLayout + ViewPager2 구조로 하면 되겠군요. 
<br><br>

#### **-> 하지만 이슈가 발생했습니다** 

### 이슈 1) 

 1) 맵 로딩이 되지 않고 (getMapAsync로의 콜백을 못받는다) 

 2) 다른 탭으로 이동한 후 다시 돌아가면 검은화면 튕김 (에러가 나타나지 않음)
 <br>
### 메일 문의 

#### ![img](https://blog.kakaocdn.net/dn/GnxqW/btqMM5uJ06b/iFKs9tZISUcgY1DkQi0Y9k/img.png)
<br>

### 답변 내용

![img](https://blog.kakaocdn.net/dn/chVvpL/btqMPifB6w2/RuD7iOgYE9g6XIzEPkeGNk/img.png)

> (정확하게 문제를 인식하고 있었음. 하지만 아직까지 해결되지 않았음..)

<br>

#### Fragment Adapter 시리즈 정리 

  1) Fragment**Pager**Apdater : **ViewPager**와 연동, **Fragment 수가 적은 경우** 적합

 2) Fragment**StatePager**Adapter : **ViewPager**와 연동, **Fragment 수가 많은 경우** 적합

 3) Fragment**State**Adapter : **ViewPager2**와 연동, RecyclerView.Adapter 상속

![img](https://blog.kakaocdn.net/dn/Vzovf/btqMVCEqQ2k/MgWxiA0xkOnqHSnIOmcoX1/img.png)

<br>
<br>

##### **Fix 1) FragmentPagerAdapter를 사용**

###### **-> 기존에 사용했던 FragmentPagerAdapter(FragmentManager fm)는 Deprecated** 되었기에 **Behavior Int값을 추가**

![img](https://blog.kakaocdn.net/dn/bnEz2Z/btqMRlpGKaY/39RRgw8mRmha314cmcuyS1/img.png)

*참고 : 이 현상은 카카오맵API에서도 동일하게 발생합니다 ( 콜백 못받고 튕기는 현상 ) 

<br>

###  이슈 2) 터치 스와이프 문제 

 : 맵을 드래그 한경우 ViewPager에 터치 이벤트가 걸려서 맵 이동을 할 수 없다.

 [맵 이동이 안되는 경우.GIF](https://imgur.com/IztocbY)

<br>

-> 네이버 맵 사용 방법 2번째인 "com.naver.maps.map.MapView" 이용해야함  

##### 1) MapView를 상속하여 TouchEvent Intercepter  

```kotlin
class ScrollAwareMapView : MapView {
    constructor(context: Context) : super(context)
    constructor(context: Context, attrs: AttributeSet?) : super(context, attrs)
    constructor(context: Context, attrs: AttributeSet?, defStyleAttr: Int) : super(
        context,
        attrs,
        defStyleAttr
    )

    override fun dispatchTouchEvent(event: MotionEvent?): Boolean {
        parent.requestDisallowInterceptTouchEvent(true)
        return super.dispatchTouchEvent(event)
    }
}
```  


##### 2) Custom된 MapView 적용   

```xml
<com.package.view.widget.ScrollAwareMapView
                android:id="@+id/map"
                android:layout_width="match_parent"
                android:layout_height="match_parent" />
```  


##### 3) LocationFragment   


```kotlin
 private val mapView : MapView by lazy{
        requireViewDataBinding().map
 }

```  

##### 4) Fragment 생명주기와 싱크 연동

```kotlin
override fun onViewCreated(view: View, savedInstanceState: Bundle?) {
    super.onViewCreated(view, savedInstanceState)
    mapView.getMapAsync(this)
    mapView.onCreate(savedInstanceState)
}

override fun onStart() {
    super.onStart()
    mapView.onStart()
}

override fun onResume() {
    super.onResume()
    mapView.onResume()
}

override fun onPause() {
    super.onPause()
    mapView.onPause()
}

override fun onSaveInstanceState(outState: Bundle) {
    super.onSaveInstanceState(outState)
    mapView.onSaveInstanceState(outState)
}

override fun onStop() {
    super.onStop()
    mapView.onStop()
}

override fun onDestroyView() {
    super.onDestroyView()
    mapView.onDestroy()
}

override fun onLowMemory() {
    super.onLowMemory()
    mapView.onLowMemory()
}  
```
```kotlin
@UiThread
override fun onMapReady(naverMap: NaverMap) {
		//TODO
}
```  
