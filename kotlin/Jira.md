
<br/>
### JetBrains IDE(IntelliJ , Android Studio등) 사용시 JIRA 와 연동하는 방법에 대한 내용입니다.

### 장점 

1. 해당 이슈 자동 상태 변경 ( ToDo -> In Progress -> Review ) 

2. 커밋 메시지 및 브랜치 자동 생성 

3. 배포시 해당 이슈 복사-붙여넣기 가능


## 1. JIRA 플러그인 설치

( Jira Integration만 설치해도 가능 )

![img](https://blog.kakaocdn.net/dn/bp4bd7/btqMMh3p8EL/zenH0F1GUBtkSfhbpXcSdk/img.png)


## 2. API Token가져오기

기존 이메일, 비밀번호 로그인 방식은 막혔습니다. ( IntelliJ 버전이 낮은 경우엔 비밀번호로 가능 )

 Jira 사이트 -> 프로필 -> 계정 설정

![img](https://blog.kakaocdn.net/dn/sdEUG/btqK8jPqFv2/bRi15BbF3EfdARHQmLsRBK/img.png)


**보안 -> API 토큰 생성 및 관리**

![img](https://blog.kakaocdn.net/dn/UmyTZ/btqLf5WFe9Q/WNYNUgJNspsv4FCDk9GAUk/img.png)

<br/>
레이블 임의 지정

![img](https://blog.kakaocdn.net/dn/dsnHhs/btqLblziQka/NEYwwHwa5snMrKP0SvHjh0/img.png)
<br/><br/>
## 3. 서버 연결 및 설정 

설정 화면 [Cmd + ,] -> **위에서 생성한 API Token 입력**

![img](https://blog.kakaocdn.net/dn/bnSASn/btqMM3Q66At/xKaFSRrvydw37A5vBnCAK0/img.png)



하단 부분의 JIRA Interation에도 "**드라이버" 버튼 -> 계정 연결**

![img](https://blog.kakaocdn.net/dn/BpdQd/btqMMigV61n/uDe3lZJbSZP8VZElhzZ2Wk/img.png)

![img](https://blog.kakaocdn.net/dn/bdrRsf/btqMN2EeE8V/KXNpnWyJKhpcLlhIKXLrwK/img.png)

**왼쪽 리스트 버튼을 눌러 JQL 커스텀** 

\- [ 나에게 할당됨, 직구로운생활, 양재동코드랩 ] 

\- shared 기능을 이용해보려고 했지만 ( Alias를 적용하는 방법을 못찾아 JQL 전체 명시..) 

\- (JQL 참고자료 : [www.lesstif.com/jira/jql-jira-query-language-jira-issue-18220188.html](https://www.lesstif.com/jira/jql-jira-query-language-jira-issue-18220188.html) )



![img](https://blog.kakaocdn.net/dn/bPNTWU/btqMOx41x48/nmtg41dlWiusouR0c1K1iK/img.png)



![img](https://blog.kakaocdn.net/dn/5XFGI/btqMOyizlsg/hXNaVEKy761hmeVeiUdyuK/img.png)


![img](https://blog.kakaocdn.net/dn/RfvyG/btqMNwyNXvo/ApPCahvjZkUHNUxiEAUYFK/img.png)
<br/>

####  *** 이 부분은 배포시 복사 붙여넣기 가능**

![img](https://blog.kakaocdn.net/dn/bxZKmN/btqMNxLioKr/SavgJrZ4BmUIhY4YMTRq4k/img.png)

<br/>

## 4. 작업하기

**Open Task** (Shift + option + n)

![img](https://blog.kakaocdn.net/dn/tIEoF/btqMTg9kcpZ/QThFj4ZhmXjWDa2w6fGMck/img.png)



이슈 선택

![img](https://blog.kakaocdn.net/dn/pqGb1/btqMPh1Zy9j/KoILqcxiF6ZkmaQDdIvAr1/img.png)



**이슈 상태 : "진행중"** 

**create changelist : 커밋 메시지**

**create branch : 브랜치명 from 분기 브랜치**

![img](https://blog.kakaocdn.net/dn/bwoYGu/btqMNxdqHkn/aK4nkvYwRIJ9HSXOQj6k70/img.png)

#### **-> 지라의 해당 이슈가 진행중(In Progress)으로 변경됩니다**

####  

### ...중간 커밋...

커밋은 중간 중간 하면됩니다. 

<br/>

## 5. 마지막 작업 완료

Close Task ( shift + option + w )



![img](https://blog.kakaocdn.net/dn/PdDak/btqMMhPWE39/IJTw3UJwmkPIYdjEbHK8N0/img.png)



Merget Branceh : 자동으로 Merge후 삭제할지 결정 (Pull Request를 할 목적이면 Merge 하지 않습니다)

Update issue state : 리뷰 및 테스트로 변경

![img](https://blog.kakaocdn.net/dn/bRnMns/btqMNxLiLM7/K4KkfRP6JiAiYoirg6LgAK/img.png)

#### -> 해당 지라 이슈가 리뷰 및 테스트로 변경됩니다
