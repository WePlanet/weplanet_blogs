회원가입시 아래와 같은 화면을 구현해보겠습니다.

조건

1) 입력후 하단 버튼을 눌러 다음 단계로 이동

2) 이전에 입력한 필드를 선택하여 해당 단계로 이동 ( 값 초기화 )

![](https://blog.kakaocdn.net/dn/MXYav/btqN02bhmaZ/s3OSQs70L4r620FHBjaoUK/img.png)
 
<br/>
분석

1) 위쪽 인디케이터가 있지만 슬라이드 기능은 없음 -> ViewPager로 하지 않아도 됨

2) 입력하게 되면 아래쪽에 쌓이는 구조 ->  RecyclerView로 한 경우 뷰타입을 각각 분리해야하고 텍스트도 각각 1:1로 지정해줘야함 ( RadioButton 타입도 섞여있음)

3) 각 항목을 선택하면 해당 단계로 돌아가야 함 -> 각 항목의 클릭 리스너도 각각 1:1로 지정해줘야함

-> **결국 모든 화면을 Fragment로 나누는것으로 결정**
<br/>

(1) 화면 및 클래스 생성

-   fragment_nickname.xml, fragment_password.xml, fragment_password_confirm.xml, fragment_target.xml
-   NickNameFragment, PasswordFragment, PasswordConfirmFragment, TargetFragment
<br/>

(2) 데이터 공유는 부모 Activity의 ViewModel을 이용 (sharedViewModel)

```
class NicknameFragment {

	override val viewModel: RegisterViewModel by activityViewModels()

}
```
<br/>

#### nav_register

```
<navigation xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:id="@+id/nav_register"
    app:startDestination="@id/nickname">

    <fragment
        android:id="@+id/nickanme">
        <action
            android:id="@+id/goPassword"
            app:destination="@id/password"
            app:launchSingleTop="true" />
    </fragment>

    <fragment
        android:id="@+id/password">
        <action
            android:id="@+id/goPasswordConfirm"
            app:destination="@id/passwordConfirm"
            app:launchSingleTop="true" />
    </fragment>

    <fragment
        android:id="@+id/passwordConfirm">
        <action
            android:id="@+id/goTarget"
            app:destination="@id/target"
            app:launchSingleTop="true" />
    </fragment>

    <fragment
        android:id="@+id/target">
        <action
            android:id="@+id/target"
            app:launchSingleTop="true" />
    </fragment>

</navigation>
```

startDestination : 시작 프래그먼트 : nickname

action [ goPassword ] : nickname 에서 password 이동

action [ goPasswordConfirm ] : password 에서 passwordConfirm 이동

action [ goTarget ] : passwordConfirm 에서 target 이동
<br/>
2-way Binding 연결

```
class RegisterViewModel(){
  private val nickname: MutableLiveData<String> = MutableLiveData()
  private val password: MutableLiveData<String> = MutableLiveData()
  private val passwordConfirm: MutableLiveData<String> = MutableLiveData()
  private val target: MutableLiveData<String> = MutableLiveData()
  
  private val goPassword : MutableLiveData<Unit> = SingleLiveData()
  private val goPasswordConfirm : MutableLiveData<Unit> = SingleLiveData()
  private val goTarget : MutableLiveData<Unit> = SingleLiveData()
  private val goMain : MutableLiveData<Unit> = SingleLiveData()
  
  fun goPassword(){
  	goPassword.value = Unit
  }
  
  fun goPasswordConfirm(){
  	goPasswordConfirm.value = Unit
  }
  
  fun goTarget(){
    goTarget.value = Unit
  }
  
  fun goMain(){
  	goMain.value = Unit
  }
}
```

```
<EditText
    android:id="@+id/et_nickname"
    android:layout_width="match_parent"
    android:layout_height="wrap_content"
    android:hint="닉네임 입력"
    android:inputType="textEmailAddress|textAutoComplete"
    android:text="@={viewModel.nickname}"/>
```

하단 "다음" 버튼을 누른 경우 이벤트 연결

```
 <Button
   android:id="@+id/bt_password"
   android:layout_width="match_parent"
   android:layout_height="wrap_content"
   android:onClick="@{()->viewModel.goPassword()}"
   android:text="다음" />
```

action ID를 지정하여 이동

```
class RegiserActivity {
    goPassword.observe(this, Observer{
        viewDataBinding.flContainer
        .findNavController()
        .navigate(goPassword)
    }
}
```

뒤로 돌아가기 **( 기본적으로 Navigation Component는 BackStack에 쌓입니다 )**

**전역 action 추가 : 어떤 화면에서도 닉네임화면, 비밀번호화면 등으로 이동할 수 있으므로 fragment밖에 지정합니다**

```
<navigation xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:id="@+id/nav_register"
    app:startDestination="@id/nickname">

    <fragment
        android:id="@+id/nickanme">
        <action
            android:id="@+id/goPassword"
            app:destination="@id/password"
            app:launchSingleTop="true" />
    </fragment>

    <fragment
        android:id="@+id/password">
        <action
            android:id="@+id/goPasswordConfirm"
            app:destination="@id/passwordConfirm"
            app:launchSingleTop="true" />
    </fragment>

    <fragment
        android:id="@+id/passwordConfirm">
        <action
            android:id="@+id/goTarget"
            app:destination="@id/target"
            app:launchSingleTop="true" />
    </fragment>

    <fragment
        android:id="@+id/target">
        <action
            android:id="@+id/target"
            app:launchSingleTop="true" />
    </fragment>
    
    <action
        android:id="@+id/backNickname"
        app:destination="@id/nickname"
        app:popUpTo="@id/nickname"
        app:popUpToInclusive="true" />

    <action
        android:id="@+id/backPassword"
        app:destination="@id/password"
        app:popUpTo="@id/password"
        app:popUpToInclusive="true" />

</navigation>   
   
```

**popUpToInclusive="true" 로 지정하면 되돌아가면서 중간 화면들도 같이 pop 됩니다 (backStack에서 제거)**

뒤로 돌아가기 위해 해당 TextView화면(패스워드) onClick 추가

```
<TextView
    android:id="@+id/tv_password"
    android:layout_width="match_parent"
    android:layout_height="wrap_content"
    android:text="@{output.password}"
    android:onClick="@{()->viewModel.backToNicknameDialog()"
/>
```

타겟화면에서 비밀번호로 이동한 경우 팝업창 추가

```
class RegisterViewModel(){
    private val backNicknameDialog : MutableLiveData<Unit> = SingleLiveData()
    private val backPasswordDialog : MutableLiveData<Unit> = SingleLiveData()


    override fun backToNicknameDialog() {
       backNicknameDialog.value = Unit
    }

    override fun backToPasswordDialog() {  
       backTargetDialog.value = Unit
    }


    override fun backToNickname() {
       backToPassword()
       nickname.value = ""
    }

    override fun backToPassword() {
      password.value = ""
      target.value = ""
    }
}
```

확인을 선택한 경우 닉네임, 비밀번호, 대상 초기화

```
class RegisterActivity{

  backToNicknameDialog.observe(this,Observer{
      //팝업창 (닉네임 입력창으로 이동하시겠습니까?) 
      (확인) ->	viewModel.backToNickname()
  }
}
```

**Activity 화면에서 Navigation 연결 -> 자동으로 처음 NicknameFragment 부터 시작됩니다**

```
<androidx.fragment.app.FragmentContainerView
    android:id="@+id/fl_container"
    android:name="androidx.navigation.fragment.NavHostFragment"
    app:navGraph="@navigation/nav_register"
    app:defaultNavHost="true" />
```
