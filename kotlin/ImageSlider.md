  
ViewPager2와 TabLayout 을 이용하여 아래와 같은 이미지 슬라이더를 만들어 보겠습니다.

  

![](https://blog.kakaocdn.net/dn/dakmIK/btqNLPDDCcW/Dk6uxiDQkCvrla98WWYFDk/img.png)
  <br/>
  
### Resource 계산

점의 크기는 5dp

![](https://blog.kakaocdn.net/dn/rwees/btqNMg8P88G/iHIencxjKA6g1dNhD3Tjh0/img.png)

점의 간격은 10dp

![](https://blog.kakaocdn.net/dn/4TYgb/btqNHqkLL7B/EpLxwYfx1Hu0C58grwc6f1/img.png)

<br>

*dot_size (점의 크기) : 제플린에 보이는 사이즈에서 1/2로 줄입니다
*dot_size : 3dp (5dp / 2 )*
  <br/>
*dot_padding (간격) : 아래의 표를 보고 적용하겠습니다.

*dot_padding : 8dp ( 10dp -> 8dp )*

![](https://blog.kakaocdn.net/dn/ch7jY0/btqNMNL88XM/1CFqPcGYk9c6sT4RQkhIcK/img.png)
  
( 절대적인 계산법은 아닙니다. 상황에 따라 달라질 수 있습니다. 수정해가며 맞추는걸 추천드립니다. )
<br>

@values/dimens

```
<dimen name="dot_size">3dp</dimen>
<dimen name="dot_padding">8dp</dimen>
```
<br>
@drawable/selector_tab_white

```
<selector xmlns:android="http://schemas.android.com/apk/res/android">
    <item android:drawable="@drawable/ic_dot_white_selected" android:state_selected="true" />
    <item android:drawable="@drawable/ic_dot_white_unselected" />
</selector>
```
<br>
@drawable/ic_dot_white_unselected

```
<layer-list xmlns:android="http://schemas.android.com/apk/res/android">
    <item>
        <shape
            android:innerRadius="0dp"
            android:shape="ring"
            android:thickness="@dimen/dot_size"
            android:useLevel="false">
            <solid android:color="#80ffffff" />
        </shape>
    </item>
</layer-list>
```
<br>
@drawable/ic_dot/white_selected

```
<layer-list xmlns:android="http://schemas.android.com/apk/res/android">
    <item>
        <shape
            android:innerRadius="0dp"
            android:shape="ring"
            android:thickness="@dimen/dot_size"
            android:useLevel="false">
            <solid android:color="@color/white" />
        </shape>
    </item>
</layer-list>
```
<br>
@layout/activity_main

```
 <androidx.viewpager2.widget.ViewPager2
     android:id="@+id/vp_main_banner"
     android:layout_width="0dp"
     android:layout_height="0dp"
     android:orientation="horizontal"
     app:layout_constraintDimensionRatio="1:1"
     app:layout_constraintEnd_toEndOf="parent"
     app:layout_constraintStart_toStartOf="parent"
     app:layout_constraintTop_toBottomOf="@id/img_home_logo" />

<com.google.android.material.tabs.TabLayout
    android:id="@+id/tab_main_banner"
    android:layout_width="0dp"
    android:layout_height="wrap_content"
    android:scrollIndicators="none"
    app:layout_constraintBottom_toBottomOf="@id/vp_main_banner"
    app:layout_constraintEnd_toEndOf="parent"
    app:layout_constraintStart_toStartOf="parent"
    app:tabBackground="@drawable/selector_tab_white"
    app:tabGravity="center"
    app:tabIndicator="@null"
    app:tabPaddingEnd="@dimen/dot_padding"
    app:tabPaddingStart="@dimen/dot_padding" />
```

<br>
@MainActivity

```
 vpMainBanner.adapter = RecyclerViewAdapter()
 TabLayoutMediator(
   tabMainBanner,
   vpMainBanner
 ) 
 { tab, position ->
     vpMainBanner.setCurrentItem(tab.position) 
 }.attach()
```
<br>
(Adapter는 기본적인  RecyclerView.Adapter를 사용하면 됩니다)

-끝-
