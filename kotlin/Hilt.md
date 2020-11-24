
<br/>
Hilt를 적용하면서 **실 사용법**과 Module 주입시 **Binds 와 Provides** 방법중 어떤것을 사용해야할지에 대해 정리하였습니다.

 
### 1. Hilt 사용법

build.gradle

```
implementation("com.google.dagger:hilt-android:${Versions.daggerHiltAndroidVersion}")
implementation("androidx.hilt:hilt-lifecycle-viewmodel:${Versions.androidHiltVersion}")

kapt("com.google.dagger:hilt-android-compiler:${Versions.daggerHiltAndroidVersion}")
kapt("androidx.hilt:hilt-compiler:${Versions.androidHiltVersion}")
```

 기본 사용법

- BaseApplication 

```
@HiltAndroidApp
class BaseApplication : Application() {}
```

 

- Module **( schedulerProvider는 Interface, SchedulerProviderImpl 은 구현체)** 

```
@Module
@InstallIn(ApplicationComponent::class)
class AppModule {

    @Provides
    fun provideSchedulerProvider(): SchedulerProvider = SchedulerProviderImpl()
    
}
```

 

- Activity 

```
@AndroidEntryPoint
class MainActivity : AppCompatActivity {
    @Inject lateinit var schedulerProvider: SchedulerProvider
}
```
**-> 사용법 끝.. 참 쉽죠?**
<br/>

활용 1) ViewModel 추가

```
class MainViewModel @ViewModelInject constructor(
    private val schedulerProvider: SchedulerProvider
)
```

viewModel은 KTX의 ComponentActivity.viewModels 를 이용합니다

```
class MainActivity : AppCompatActivity {
 	private val viewModel: MainViewModel by viewModels<MainViewModel>()
}
```

 

**@ViewModelInject** 어노테이션을 지정해주면 아래와 같이 파일 3개가 생성됩니다



![img](https://blog.kakaocdn.net/dn/wAj9x/btqNpGn4bwL/pHQFysPZlmlhWhKRgNxVi1/img.png)



MainViewModel_HiltModule 클래스를 확인해보면

자동으로 **모듈화** 되고 **ActivityRetainedComponent**로 지정됩니다 ( Dagger와 달리 자동 생성 )



![img](https://blog.kakaocdn.net/dn/bShrvv/btqNwwKJlPJ/viIyIVxiEqYsoOcufT2Xsk/img.png)

<br/>

**활용 2) SavedStateHandle도 @Assisted 어노테이션만 추가하면 바로 사용할 수 있습니다**

```
class MainViewModel @ViewModelInject constructor(
    @Assisted stateHandle: SavedStateHandle,
    private val schedulerProvider: SchedulerProvider
)
```
---
<br/>

### 2. Binds vs Provides

논란의 시작 : [www.youtube.com/watch?v=KI3L6d6Sm3Q&t=915s](https://www.youtube.com/watch?v=KI3L6d6Sm3Q&t=915s) 

해당 영상에서 Binds 와 Provides 를 설명하면서 **Binds 는 복잡한(complicated) 방법**이라고 계속 표현합니다. 

그리고 **Provides는 직관적 (clear,simple)** 이라고 provides만 사용할것이라고 합니다.

<img  src="https://blog.kakaocdn.net/dn/b5DnOm/btqNqCSYrmi/FKoaAPRak6NmSCHuNOnnqk/img.png"  width="300">

<img  src="https://blog.kakaocdn.net/dn/bPt8og/btqNuQbFI0T/MEiqnYCwi7vR882SJ3v9l1/img.png"  width="300">


**-> 저도 그런줄 알고 Provides 만 사용했습니다. 확실히 Provides의 방법이 더 직관적입니다.**

**(Binds는 이해가 잘 안됩니다..)**

해당 댓글 내용을 보겠습니다 (위 영상의 댓글입니다)

![img](https://blog.kakaocdn.net/dn/0NBte/btqNwwqsKLM/BMwvk7EQyqcil4gEUQBkv1/img.png)

- Coding in Flow : @Binds가 더 효과적이다.
- 유튜버 (글쓴이) : ??? 얼마나 더 효과적인데 ? ( 쓰기 불편한 @Binds 왜 쓰냐고...)  
- Coding In Flow : 자바에서 Binds는 Provides 보다 읽기 쉽고 파싱도 쉬워, 코틀린 문법에선 더 이상 아닐지라도
<br/>

그래서 제가 테스트를 직접 해보았습니다.

위의 @Provides SchedulerProvider을 아래와 같이 변경하였습니다

```
class SchedulerProviderImpl @Inject constructor() : SchedulerProvider {}
@Binds
abstract fun bindSchedulerProvider(schedulerProvider: SchedulerProviderImpl) : SchedulerProvider
```

( @Provides와 달리 **구현체에 @Inject constructor() 는 추가**해주어야합니다 )

 

 

빌드해보니 아래파일이 사라졌습니다.

 



![img](https://blog.kakaocdn.net/dn/YDuaO/btqNqex14HX/iLZUMpUX55RuYXqB7UZBB0/img.png)

![img](https://blog.kakaocdn.net/dn/cwwW6p/btqNtiztdGp/AK01omSenaDLxVo8hMKYk1/img.png)

그리고 사용법이 @Provides 보다 더 쉽다는걸 느꼈습니다.

Service를 Provides 하는 경우를 보겠습니다 

```
 @Provides
 fun provideAuthService(
     authRepository: AuthRepository,
     mainRepository: MainRepository
 ): AuthService 
    = AuthServiceImpl(authRepository, mainRepository)
```

-> 만약 **Service에서 사용하는 Repository가 더욱 많아 진다면?? 각각 파라미터를 확인하면서 명시**해줘야 합니다.

 
<br/>
Binds 를 이용한 경우엔?

```
 @Binds
 abstract fun bindAuthService(authServiceImpl: AuthServiceImpl): AuthService
```

이렇게만 명시해주면 됩니다.

 

오히려 더 간단합니다.

 

그럼 Binds는 Provides 보다 파일도 덜 생성되고 (MultiDex대처) 사용법도 간결해지므로 Binds만 사용하면 될까요?

 

하지만 Binds 는 제약조건이 있습니다. (그래서 Provides가 있나봅니다. )

구현체에 @Inject constructor()를 명시해주어야 합니다.

 그렇지 않은 경우 아래의 에러가 발생합니다

![img](https://blog.kakaocdn.net/dn/bGofSE/btqNti7ioAU/44UI1wN623UTkSUsvQnQk0/img.png)

**-> 결국 Retrofit같은 생성해서 사용하는 객체는 Binds 를 적용하기가 어렵습니다.**


---


### 3. 후기

Dagger2와 Koin의 단점을 보완하여 사용하기 쉬운 라이브러리라고 느꼈습니다.

정식 버전이 출시되어 안정화 되면 많이 애용할것 같습니다.

현재는 alpha 버전이라 버그가 좀 있습니다. ( retrofit2:adapter의 RxJava2CallAdapterFactory를 Provides하는 과정에서 Module클래스를 internal로 지정해놓으니 에러가 발생합니다 : implementation 대신 api 로 사용 )
