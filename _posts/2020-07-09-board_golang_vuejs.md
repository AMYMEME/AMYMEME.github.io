---
layout: post
title: 사이드 프로젝트 - 게시판 만들
excerpt: "golang, MySQL, Vue.js, REST, OAuth2"
categories: [개인 프로젝트]
comments: true
share : false
---

# 시작
인턴 끝나고 ~~다들 그냥 쉬라고 했었지만~~ 방학동안 할 수 있는 일이 있지 않을까 했었는데, `golang`으로만 구성된 백엔드와
새롭게 프론트를 도해보고 싶었다.  
{: .notice}

* * *


# 과정
> 결론만 말하자면, 클라이언트 쪽은 정말 안맞는 것 같다.. 아직 뭘 제대로 해보지도 않고 그런 소리를 하면 안된다고들 하셨는데, 
> 백엔드보다 배우는 속도도 느리고, 설정 파일들도 너무 많고... 너무 헷갈린다ㅜ  
> 지금 이 사이드 프로젝트는 20일?정도 걸렸다. 이 중에 백엔드는 4일정도 걸렸다면, 나머지는 FE에 썼던 것 같다.
> 이후에 맥북을 산 기념으로 스위프트로 간단한 캘린더를 만들려고 했었지만, 
> 정말 모르겠어서 지금은 레포도 삭제하고(내 깃헙 잔디ㅠㅠ) 백엔드를 더 열심히 해보려고 한다. (~~일단 졸프부터...~~)

## DB
#### controller - repository
golang과 MySQL 연동에서 놀랐던 게 있었는데, golang 라이브러리에서는 JPA같이 ORM을 많이 쓰는 분위기가 아니라는 것이다.  
아예 없는 것은 아니었는데, 예제도 찾기 힘들고, 많이들 안쓰길래 하고 그냥 `go-sql-driver/mysql`을 사용했다.  

그리고 웹 프레임워크로는 `gin`을 쓰는데 해당 url로 요청이 들어오면 DB를 참고해야하니까 `gin context`를 파라미터로 갖는 핸들러들이 DB 커넥션이 가능해야 했다.  
전통적인 컨트롤러의 역할을 핸들러들이 하는데, 이들이 서비스, 레포지토리를 호출하고, 서비스, 레포지토리 쪽에서 데이터를 호출하거나 직접 가져와야 한다.
그런데, golang에서는 같은 패키지내 대문자로 시작하면 `protected`, 소문자로 시작하면 `private` 클래스 개념이라 외부 패키지 폴더에 있는 함수들이나 클래스들을
참고할 수가 없다는 문제가 있었다. 
개인적으로 해결한 방법은, `repository` 패키지 내에는 `*sql.DB` 객체를 래핑해서(`DBConfig`), 각각의 쿼리문 결과를 리턴해주는 메소드를 만들었다. 
그리고 컨트롤러(패키지 이름은 `api`라고 지음..)에서는 실질적으로 DB정보를 통해 커넥팅하는 부분과, 커넥션 체크를 하는 부분을 `common.go`내에 위치시키고,
나머지 게시글이나 멤버, 댓글의 추가/수정/삭제를 수행하는 `??api.go`의 모든 함수 초기부분에는 `common`부분의 DB체크 부분이 들어가게 하였다.  

컨트롤러가 `DBConfig`의 메소드가 되는 것은 `gin-gonic`에서 핸들러로 호출할 때 문제가 되므로, 레포지토리 쪽을 `DBConfig` 메소드로 작동하도록 했는데  
아예 `main.go`에서 `DBConfig`를 정의하는 것도 괜찮을 것 같다.  

뭐.. 그래서 지금도 좋은 방법인지는 모르겠지만, 아무튼 이런 식으로 해결했다. 만약 트랜잭션이 필요한 부분에는 어떤식으로 해야할지 고민을 더 해야할 것 같다.  
또한 로그남기는 방식도 이와 비슷하게 래핑해서 사용하였다. 

#### 구성
아쉬운 점이 있다면, 멤버(회원) 별 고유 ID가 있는데, 게시판을 전체 조회할 때 멤버 별 고유 ID를 보여주고 멤버 아이디나 이름 등으로 보여주지 않는다는 점이 개인적으로 아쉬웠다.  
안 예뻐보인달까.. 쿼리문을 한 번 더하면 해결할 수 있겠지만, 게시판 전체 조회 시 DB성능도 고려해야 하니까 DB 설계부터 잘못된 것이 아닐까 싶어서 나중에 고유 ID를 부여하지 않고, 아이디나 이름에 인덱스를 넣는 방식로 변경하려 한다.  


## OAuth2.0
#### 이유
말 그대로 게시판을 만들어 보자가 내 목표였지만, 지금까지 `SSR`(타임리프 혹은 html+css+바닐라 js조금)형식의 웹을 만들다가, 
`SPA`형태를 뷰와 고언어로 구성해보려고 하니까 생각보다 어려웠다.  
한글 자료는 없을거라고 생각했지만, 생각보다 영어자료가 많아서(중국어 보단 해석이 되니까) 둘을 연동시키는 데에는 별로 문제되지 않았다.  
그저 JS 자체를 처음 해봐서 vue-cli뭐고 webpack을 그래서 어떻게 사용해야 하지..?하는 문제점들이 있었다.  

인턴십 때 oauth2.0을 깃헙으로 했었는데(정확히는 깃헙 엔터프라이즈),
그 때 얼핏 보니 스프링 시큐리티에서 깃헙, 구글, 페이스북은 oauth2.0 기본 클라이언트로 별다른 설정없이 할 수 있던 기억이 난다.
(~~물론 커스터마이징 하거나 하면 얘기가 복잡해지지만~~)  
그치만 카카오나 네이버는 일일이 다 구성했어야 했던 기억이 있는데, 
이왕 하는 거 우리나라에서 더 많이 쓰이는 카카오와 네이버로 `oauth2.0`을 `vue.js`를 이용해 구현해보고자 했다.

#### jwt
oauth2.0은 내부적으로 jwt가 베이스이다. 하지만 나는 내 웹사이트와 사용자의 브라우저 간에도 jwt를 적용하고자 했다.  

따라서 oauth2.0이 완료되면, 회원정보를 담고있는 DB에 해당 정보가 있으면 그 정보를 바탕으로 jwt를 생성하여 vue에 전달하고, 
vue 라우터 쪽에서는 브라우저를 렌더링할 때 헤더에 붙이고 로컬 스토리지에 저장하는 방식이다.
회원정보가 없다면(새로 로그인), 회원 정보를 새로 저장하고, 똑같이 jwt를 생성하여 vue쪽에 넘겨준다.

#### issues
[여기](https://github.com/AMYMEME/board-golang/issues/10)에 자세히 적어놓았듯이, 많은 이슈들이 있었다.

* * *
#### 네이버 인증 이슈
###### 1. 외부 라이브러리 임포트
이건 뷰를 처음 써봐서 몰랐던 내용이다. 네이버에서 제공하는 js 라이브러리(외부 라이브러리)를 임포트하고 싶은데 어떻게 하는지 몰랐는데, 
~~~ javascript
mounted() {
      const script = document.createElement('script')
      script.src = {JS script URL}
      script.onload = () => {
        this.initiate()
      }
      document.body.appendChild(script)
    }, 
...
data() {
      return {
        initiate: (store) => {
          const naverLogin = new naver.LoginWithNaverId(
...
~~~
이런 식으로 해결하였다. 

###### 2. 콜백 적용
네이버는 js 라이브러리에서 콜백함수 기능을 지원하는데, 이걸 사용하려면 **콜백 페이지**(리다이렉트 페이지)에서 처리를 해야한다. 
따라서 위의 외부 js 스크립트를 적용하는 페이지가 (로그인 페이지, 콜백 페이지)로 **두개가 되어야 한다.**  
로그인 페이지에서는 init()만하고(버튼을 표시하기 위해서), **콜백 페이지에서는 init()이후에 수행할 콜백함수를 정의해야 한다.**
리다이렉트가 되면서 가져온 내용을 바탕으로 콜백함수가 실행되므로 로그인 페이지에서 콜백 설정해봤자 의미가 없다. 

###### 3. 인증 토큰을 기반으로 프로필 내용을 가져오기
원래는 로그인 이후 회원 정보를 가져오려면 인증 토큰을 기반으로 한번 더 API를 이용해 회원 정보를 가져와야 한다. (구글, 카카오 등 거의 모든 사이트에서 그럼) 
하지만 위에서 임포트한 라이브러리에서는 따로 API 한 번 더 실행하지 않아도, 회원 정보도 가져올 수 있기 때문에 백엔드 서버에서 회원 프로필 조회 API를 사용하지 않아도 된다.  
백엔드에서 만약 따로 회원 프로필 조회 API를 쓰면 약간 골치아픈게, `fe -> be -> 회원 프로필 조회 -> be로 리퀘스트와 리스펀스가 전달`되어야 하므로, 
be에서 fe에서 받는 리스펀스와 회원 정보 API로 받은 리스펀스, 총 2개의 리스펀스를 받으므로, url을 분리하는 등의 추가작업을 따로 해야함
(경험 상 url 분리가 안되면 빈 회원이 저장되거나...여러 문제가 생길수 있음.. 아니면 따른 처리가 필요)

###### 4. vue store
로그인이 되면, 상단 바에 로그아웃 버튼을 띄우고, 로그인이 안되어 있으면 로그인 버튼을 띄우기 위해서 통합적인 상태 관리와 헤더나 로컬 스토리지를 이용한 상태 관리가 필요했다.    
이는 `vue store`로 해결할 수 있었는데, 문제는 js 스크립트를 적용할 때에 `data.initiate`함수를 이용했는데, 이 함수를 이용했을 때 store에 접근하기가 쉽지 않았다.
`method`도 인식하지 못하고, 그렇다고 이걸 `mounted`나 `created`로 빼자니 **스크립트 인식을 못하는** 등의 문제가 발생
-> 여러 삽질 끝에 `this.initiate()`를 `this.initiate(this.$store)`로 해주고 `initiate`에서 `store.dispatch`로 백엔드 서버로 네이버에서 가져온 회원 정보를 전달하기로 함

###### 5. hashbang
네이버에서 제공한 js 스크립트상에서 로그인을 시도하면, `{리다이렉트 url}/auth/naver#access_token={access_token}&state={state}&token_type=bearer&expires_in=3600` 으로 응답을 뱉는다.
해시뱅은 프래그먼트로, 저기서 바로 백엔드로 연결하면 값이 전달되지 않아 백엔드에서 저 값들을 조회할수가 없었다.  
따라서, 리다이렉트 url(콜백 페이지)을 fe쪽으로 설정하고, 거기서 다시 be로 토큰값을 전달하였다. 

###### 정리
1. 로그인 창
2. `js script import` 및 네아로연동 시도
3. 처음 로그인시 이름 가져옴
4. 안 가져올 시 리다이렉트 페이지에서 콜백함수에 의해 다시 정보 가져오라고 함
5. 가져왔으면 가져온 사용자의 이름과 유니크 아이디를 `golang`으로 전달 
6. `golang`에서는 `provider`가 `naver`고, `provider_id`가 유니크 아이디인 열을 확인
7. 존재하지 않으면 디비에 새로 저장 이후 정보를 바탕으로 `jwt`값을 생성 후 `vue`에 전달
8. 존재하면 그냥 생성한 `jwt`값을 에 `vue`전달
9. `vue`에서는 받은 `jwt`값을 헤더에 붙이고(`axios` 렌더링) 로컬 스토리지에도 저장 
10. 메인페이지로 리다이렉트 및 `vue store`로 인해 로컬 스토리지에 값 저장, 헤더바 상태 변경

* * * 
#### 카카오 인증 이슈
카카오는 가이드 가독성이 좋은 편이어서 행복했다. 네이버 가이드는 계속 스크롤을 해야 했고, 카카오보다 구현되어 있는 것들이 많아서인지 헷갈린 부분이 많았다..
그 외에 그냥 내가 카카오가 두 번째여서 감이 좀 온 것이었을 수도 있다.  

###### REST API CORS
앞서 네이버에서 `js sdk`를 이용했으니, js sdk를 이용하지 않고, `REST API`를 이용했었다.
golang과의 연결 시 `CORS` 문제가 발생했고, 개발자 가이드를 참고하니 CORS문제는 JS SDK를 사용하는 것밖에 답이 없다고 하셔서 그냥 수긍하기로 했다.

###### JS SDK
네이버와 비슷하게 이런식으로 했다.  
~~~ javascript
mounted() {
      const script = document.createElement('script')
      script.src = {JS script URL}
      script.onload = (() => {
                this.kakaoLogin(this.$store)
            })
      document.body.appendChild(script)
    }, 
...
methods: {
            kakaoLogin(store) {
                Kakao.init('JS APP ID')
                Kakao.Auth.createLoginButton({
                    container: '#kakao-login-btn',
                    success: function() {
                        Kakao.API.request({
                            url: '/v2/user/me',
                            success: function (res) {
                                store.dispatch('loginWithKakao', res)
...
                            },
                            fail: function () {
                                alert('사용자 정보를 가져오는 데에 실패하였습니다. 다시 시도해 주세요.');
                                window.location.replace('/login');
                            },
                        })
                    }
                })
            },
...
~~~
다만 `data`가 `method`로 바뀌었는데, 이게 더 의미가 통해보여서 네이버도 추후 이렇게 바꾸었다. 

###### success function
네이버와 다르게 콜백함수를 리다이렉트 페이지에서 쓰지 않아도 success function을 따로 지정할 수 있어서 페이지를 따로 두거나 하지 않아도 되고, 편리했다.

* * *
#### 구글 인증 이슈
원래는 구글 인증은 라이브러리를 이용했었는데, 똑같이 컴포넌트로 분리하기 위해서 라이브러리를 지우고 비슷한 방식으로 다시 구성하였다.  
솔직히 API 명세 진짜 읽기 힘들었다.. 영어의 문제라기보단 가독성이 넘 떨어짐... 스크롤도...절레...


###### JS SDK
~~~ javascript
mounted() {
             const script = document.createElement('script')
             script.src = 'https://apis.google.com/js/api.js'
             script.onload = (() => {
                 this.googleLogin(this.$store)
             });
             document.body.appendChild(script)
 ...
 methods: {
             googleLogin(store) {
                 var GoogleAuth
                 gapi.load('client:auth2', function () {
                     gapi.client.init({
                         'apiKey': 'app-key',
                         'clientId': 'client-id.googleusercontent.com',
                         'scope': 'https://www.googleapis.com/auth/userinfo.profile',
                         'discoveryDocs': ["https://people.googleapis.com/$discovery/rest?version=v1"]
                     }).then(function () {
                         GoogleAuth = gapi.auth2.getAuthInstance();
                         var googleLoginBtn = document.getElementById('google-login-btn');
                         googleLoginBtn.addEventListener('click', function () {
                             if (GoogleAuth.isSignedIn.get()) {
                                 gapi.client.people.people.get({
                                     'resourceName': 'people/me',
                                     'requestMask.includeField': 'person.names'
                                 }).then((res) => {
                                     store.dispatch('loginWithGoogle', res.result.names[0])
                                         .then(() =>  store.commit('backRedirect'))
                                 })
                             } else {
                                 GoogleAuth.signIn();
                             }
                         })
                     })
                 })
             },
         },
 ...
~~~
이런 식으로 하였다. 잘 보면, `scope`와 `discoveryDocs` 부분이 회원 정보를 가져오는 api를 규정하는 부분이라는 것을 알 수 있다. 


###### gapi.client
네이버나 카카오와 살짝 다르게 구글은 모든 API를 하나의 도메인(`gapi`)으로 규정하고, 그 안에서 지도를 가져오는 api를 쓰던지, drive API를 쓸 것인지 등등 명명을 하는 식으로 작동한다.  
따라서 우리는 회원 정보를 가져올 것이므로 `gapi.client`를 사용해야 하고, gapi.client는 시작시에 API에 해당하는 `scope`와 `discoveryDocs`를 선택해야한다.
이걸 헷갈려서 (예제 코드에는 drive API를 기준으로 설명하기 때문) 조금 해멨다.

###### store vs GoogleAuth.isSignedIn.get
이미 크롬브라우저 상에서 되어있는 경우가 아닌 새로 구글에 로그인을 해야하는 경우, 버튼을 한 번더 눌러야 하는 문제가 발생하였다.  
맨 처음에 GoogleAuth.isSignedIn쪽에서 문제가 생기는 것 같았는데, 며칠을 시도해도 별다른 해결 방법이 없어서, 

~~~ javascript
methods: {
            googleLogin(store) {
                var GoogleAuth
                gapi.load('client:auth2', function () {
                    gapi.client.init({
                        'apiKey': 'app-key',
                        'clientId': 'client-id.googleusercontent.com',
                        'scope': 'https://www.googleapis.com/auth/userinfo.profile',
                        'discoveryDocs': ["https://people.googleapis.com/$discovery/rest?version=v1"]
                    }).then(function () {
                        GoogleAuth = gapi.auth2.getAuthInstance();
                        var googleLoginBtn = document.getElementById('google-login-btn');
                        googleLoginBtn.addEventListener('click', function () {
                            GoogleAuth.isSignedIn.listen(() => {
                                gapi.client.people.people.get({
                                    'resourceName': 'people/me',
                                    'requestMask.includeField': 'person.names'
                                }).then((res) => {
                                    store.dispatch('loginWithGoogle', res.result.names[0])
                                        .then(() =>  store.commit('backRedirect'))
                                })
                            });

                            if (!GoogleAuth.isSignedIn.get()) {
                                GoogleAuth.signIn();
                            } else {
                                gapi.client.people.people.get({
                                    'resourceName': 'people/me',
                                    'requestMask.includeField': 'person.names'
                                }).then((res) => {
                                    store.dispatch('loginWithGoogle', res.result.names[0])
                                        .then(() =>  store.commit('backRedirect'))
                                })
                            }
                        })
                    })
                })
            },
        },
...
~~~
이런 식으로 해결했다. 

중복 코드가 발생한다는 문제점이 있지만, 따로 함수로 빼기에는 store 처리를 어떻게 해야할 지 모르겠어서.. 일단 이렇게 처리했다..

* * *
# 느낀점과 근황
도중에 예상치 못하게 아빠 회사에서 아르바이트를 하게 되어서 많이 신경을 못쓴 것 같았는데, 이렇게 보니까 꽤 열심히 한 것 같기도 하다.  
하면서 '와..JS는 진짜 예측할 수가 없는 언어다..'라고 생각했는데, 이후 `swift`를 하면서 '**?????**'가 되었던걸 생각하면 JS는 할만한 것 같다. 
앱 어떤식으로 작동하는지 전혀 몰라서 그런 것일 수도 있지만 말이다ㅠㅠ...
그리고 풀스택을 하려면 생각보다 아~주 많은 것을 고려해야하는 구나하는 생각이 들었다. DB설계, API설계, 로깅, 테스트 등에 퍼블리싱, 이벤트 핸들링, 백엔드 연결 등..등..  
JS...의 벽이 높다기보단, FE가 어떻게 작동하고 빌드되는지, 빌드 선택지는 왜그렇게 많고, 대체 뭐가 그렇게 다른건지 Webpack관리 도구는 뭘 해주는 건지에 대해서 
부족한 점이 많다는 생각이 들었다.  
{: .notice}

글을 쓰고 있는 현재는 스위프트 하다가 접고, 복학해서 학교를 다니고(?) 있다.
물론 코로나때문에 신촌은 못가지만, 아마 졸프 때문에라도 빠른 시간 내 학교에 갈 것 같다.  
수강신청이..4년 중 최악으로 망해서 15학점 듣느라 널널해서 토익을 준비하고 있다. 
900점 넘으면 2학점을 준다는데, 한 달안에 900 넘는 것이 목표다..!   
졸프는 주제를 정하고 있는데, 원래 내가 컴퓨터공학과가 아니만큼 보안이나 ML와 관련된 주제로 해야할 것 같다.  
또 오랜만에 디지털포렌식을 하고 있는데, 오랜만에해서 그런지 재밌다ㅎㅎ 역시 보안은 툴쓰는 맛이 있다.. (~~눈알빠질거같다~~)  
음 그리고 얼마전에 링크드인에서 예전 티스토리 글 잘 봤다고 연락이 왔었는데, 다시 들어가보니 그 땐 참 똑똑했구나.. 싶다..  
지금은 늙어서 머리가 잘 안돌아가는 것 같다...ㅜ..
다음엔 스프링(부트)로 프로젝트 하려고 생각중이다..! 앱 쪽은 아마 몇 년은 안하지 않을까...?
{: .notice}