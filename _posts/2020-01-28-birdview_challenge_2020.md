---
layout: post
title: 서버 개발 챌린지
excerpt: "django(python), MySQL, jsonAPI"
categories: [개인 프로젝트]
comments: true
share : false
---

## 시작
2020 첫 개인 프로젝트로 서버개발을 공부해보고 싶었다.  
혼자 그냥 공부하면 안 할 걸 뻔히 알았기에 마침 프로그래머스에서 [버드뷰 서버 개발 챌린지](https://programmers.co.kr/competitions/130/2020-hwahae-blind-recruitment)를 하고 있어서 신청했다.
{: .notice}

## 과정
### 환경 구축
환경 구축은 언제나 힘들다... 1/24에 날잡고 환경구축을 했었는데 그 때 배운 점을 노션에 기록한 걸 옮겨보려고 한다. ~~*아마 또 까먹었을 테니까*~~
##### 스택 목록
- [x]  MySQL 8.0.17 (for windows)
- [ ]  Python 3.6.5 → Python 3.8로 작업 (finalenv라는 venv로 작업)
- [ ]  Ubuntu 16.04 → Windows 10 Education으로 작업
- [x]  dj-database-url==0.5.0
- [x]  Django==2.2.4
- [x]  gunicorn==19.9.0
- [ ]  mysqlclient==1.4.4 → 1.4.6
- [x]  pytz==2019.2
- [x]  sqlparse==0.3.0
- [x]  whitenoise==4.1.3  

* * *

### 1. 도커 탈락
#### docker install for Windows 10 "Home"
일단 이번에 도커를 제대로 해보려고 세팅~~하는 것 부터~~애를 먹었다. 일단 내 윈도우 10 이 Windows Home이라는게 문제였다..  
Windows에서는 도커를 사용하려면 `Hyper-V`(*MS에서 만든 가상화 SW*)가 필요한데 Windows 10 Home에는 Hyper-V가 없으므로
Home에서는 VirtualBox를 깔아야 했다. 이는 `docker toolbox`를 통해 까는 방식이다.
`docker toolbox`를 깔면 virtualbox를 어차피 까는데, 맨 처음에는 이렇게 설치해서 
`dockr run hello-world`나 `docker run -it ubuntu bash`까지 성공했었지만.....

#### docker install for Windows 10 "Education"
다른 도커 이미지들을 다운받고 컴포넌트를 생성하는 과정에서 뭔가 충돌이 생기고 더이상 컨테이너들이 아예 실행되지 않아서 빡쳐서 다 밀어버리고
이왕 이렇게 된거 학교 계정으로 윈10 에듀케이션을 설치했다. 연계 학교에 한해서 평생 무료로 준다니 땡큐!  
이렇게 받아도 내 노트북 사양은 램이 4기가라 ... 1기가로 할당해놔도 계속 메모리 없다고... 몇 번의 시도로 성공했다.
* * * 
### 2. Ubuntu 16.04 LTS
#### Ubuntu 16.04 와 Python 3.6
내 목표는 우분투 16.04를 도커로 실행시켜서 그 안에 파이썬 3.6.5와 mysqlclient 1.4.4와 mysql server 8.0을 까는 것이었는데, 일단 문제가 있었다.  
**우분투 16.04에 파이썬 3.6은 잘 안깔린다.**
* 공식 레포지토리에도 없어서 ppa(서드파티 레포)등 다른 곳으로 우회하여 깔아야 한다.
* 우분투 기본 터미널들 중 몇몇이 3.6으로는 작동이 안되는 경우가 있어서 충돌이 생기는 사례가 있다고 한다.
* 결국 설치를 하게 되면, 파이썬 `2.7`와 `3.6`, `3.5`가 모두 설치되는데, 위와 같은 이유로 python 3.6을 사용하려면 `venv`를 구성해야 함

결국 나는 ... **윈도우로 하기로 했다..** 사실 프로그래머스 측에서 헤로쿠로 빌드 환경을 제공하므로 계속 마이그레이션이나 코드 수정을 하지 않으면 환경 구축할 필요가 없었지만
한번에 코드를 짤 수 없기 때문에 코드 수정을 할 수 밖에 없고, 헤로쿠로 올릴 때마다 마스터 브랜치로 계속 푸쉬했어야 하므로 일부러 환경 구축을 한 것이었다.
이런 삽질 과정에서도 정말 많이 배운다는 걸 다시금 느꼈다...
{: .notice}
* * * 
### 3. MySQL Server 8.0.17
진짜 개..삽질을 한 것 같다. 실제로 '데이터베이스'라고 명명하는게 `MySQL Server`라면, 그 서버에 접속하기 위해 필요한 라이브러리 중 하나가 `MySQL Client`인데
테스팅을 위해선 둘 다 깔아야 했다. MySQL 서버는 일년 전에 써 본 적이 있었지만 기억이 가물가물해서 고생 좀 했다.
#### MySQL Server 8.0.17 for windows-32bit
그냥 사이트가서 MSI 파일로 다운받아서 깔았다.
참고로 이 떄 내 윈도우에는 파이썬 3.8이 깔려있었다. 

#### MySQL Connector
설치하다보면 몇 개는 자동으로 설치가 안되서 직접(manually)깔아야 하는데, 내 경우엔 C++이랑 Python이 그랬다. 
그래서 직접 Python 버전에 맞게 깔았고, C++도 직접 깔았는데, Python 버전을 최대한 요구한 환경에 맞추고 싶어서 3.6으로 깔았다가....
~~나중에 다시 깔게된다.~~

#### 만나 본 MySQL Server 8.0 실행 오류들
##### can connect mysql server 10061 
아래는 삽질 과정 순..
1. mysql server 실행을 안했음
2. cmd로 mysql명령어 인식이 안됨
3. 환경 변수를 설정함 (C:\Program Files\MySQL\MySQLServer8.0\bin)
4. 서비스에서 MySQL 찾기 및 실행 시작
5. MySQL가 서비스 목록에 없어 열려있는 파이참이나 워크벤치 등을 끄고 환경 변수 없애고 cmd(*관리자 권한으로 실행해야 함*)
를 통해 bin 경로로 들어가서 `mysql --initialize`하기
6. 서비스 목록에서 시작했을 때 `mysql 서비스가 로컬 컴퓨터에서 시작했다가 중지되었습니다.`라는 메시지가 뜬다면 cmd로 bin 경로로 들어가
`mysql --remove` 및 `mysql --install`, `mysql --initialize`등을 실행

##### Install/Remove of the Service Denied!
윗윗줄에 언급한 cmd를 관리자 권한으로 실행해야 한다는 이유임. 나는 ms store에서 preview를 쓰고 있는데 그건 안됨...^^!

##### ERROR 1045 (28000) : Access denied for user 'root'
이 [링크](https://dev.mysql.com/doc/refman/8.0/en/testting-permissions.html)을 따라하면 된다.  
MySQL Server의 루트 계정 패스워드를 바꾸는 건 버전 5인가를 기준으로 방식이 다른데, 
떠돌아다니는 5이상 버전의 방법을 써도 안되길래 공식 문서를 뒤져서 
따라해봤더니 해결했다. 아래는 그 과정
* 서비스 목록에서 MySQL 중지 후 밑의 내용을 `txt` 파일로 저장한다. 
    {% highlight sql%}
    ALTER USER 'root'@'localhost' IDENTIFIED BY 'MyNewPass';
    {% endhighlight %}
* cmd을 통해 bin 경로로 이동해서 앞서 만든 파일을 옵션으로 붙여서 실행
   {% highlight bash%}
   C:\> cd "C:\Program Files\MySQL\MySQL Server 8.0\bin"
   C:\> mysqld --init-file=C:\\mysql-init.txt
   {% endhighlight %}
* 서비스에서 MySQL 재시작  

* * *  
### 4. Django with Pycharm
#### Project Interpreter
아까 말했듯이 파이썬 3.6을 다운받아서 `venv`를 하고 `requirement.txt`에 맞게 설치하려는데...  
**Window에서 `python 3.6`이랑 `mysqlclient` 라이브러리는 호환이 잘 안된다.**  
결국 `.whl`을 다운받아야하는데, venv에서 whl을 을 실행하기에...는..좀 그래서 기본적으로 깔려있던 `python 3.8` 기본 인터프리터에
 whl을 통해 mysqlclient 1.4.6(~~1.4.4는 또 없다..젠장~~)을 깔았다.  
 이를 기반으로 venv를 만들어 나머지 requirement들을 만족시켰다.

#### Django와 MySql server 8.0 연결
원래 `Django`에서는 `settings.py`가 있고, 이를 기반으로 실행이 되는데, local테스트를 위하여 local과 production 설정을 분리해놨다./
(좋은 방법인 것 같아 나중에 써먹어야 겠다)  
아무튼 `local.py`로 가면,
{% highlight python%}
DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.mysql',
        'HOST': 127.0.0.1,
        'NAME': 'my_database',
        'USER': 'root',
        'PASSWORD': 'password'
    }
}
{% endhighlight %}
라고 되어있는데, 이를 위해 `root%localhost`에서 패스워드는 `password`로, DB 명은 `my_database`로 맞춰놔야 한다.  
이를 위해 워크벤치에서
{% highlight sql %}
CREATE DATABASE my_database; 
{% endhighlight %}
를 실행한다.

#### Pycharm과 Django 연결
나는 Pro를 쓰고 있어서 Community 버전은 잘 모르겠지만, `Run Configurations`에서 장고 서버를 만들어 주면된다.  
만들 때 빨간색으로 root 디렉토리랑 setting 디렉토리 설정하라고 하는데, 해주면 된다.  
그리고 아까 local에서는 settings.py 대신에 `local.py`를 사용한다고 했으므로, `Environment variables`에서 `DJANGO_SETTINGS_MODULE`을
`myapp.setting.local`로 세팅해준다.

#### django.db.utils.OperationsError: (2059, <NULL>)
런해주면 처음에 이런 에러가 뜨는데, 검색해보면 그냥 네트워크가 연결 안되서 생기는 일반적인 에러 문구인데, 생길 이유가 없어서 검색ㅎ 배ㅗ면
저 에러랑 조금 다르게 `sha2`어쩌고 하는 에러도 같이 뜬다. (~~고통받는 지나가는 보안학과~~) 아래는 해결 방법  
* 일단 아래 쿼리로 설정 DB를 사용한다고 얘기한다.
    {% highlight sql %}
    USE mysql;
    {% endhighlight %}
* 아래 쿼리로 user의 `plugin`을 보면, `caching_sha2_password`로 설정되어 있을 것이다.
    {% highlight sql %}
    SELECT user, plugin FROM user;
    {% endhighlight %}
* 이 말은 password를 제대로 입력해도 sha2로 저장이 되어있어서 제대로 인증이 안되는 것이므로, `native-password`로 인증 방식을 바꿔야 한다.
    아래와 같은 쿼리를 통해 바꿔준다.
    {% highlight sql %}
    ALTER user 'root'@'localhost' IDENTIFIED WITH mysql_native_password;
    {% endhighlight %}  

## DB 설계
내용에 대해선 언급할 순 없지만 오랜만에 DB 설계하니까 재밌었다. 
DB 설계는 항상 맞는 것 같으면서 헷갈린다.. 정규화 과정이 꼭 필요하다는 걸 다시금 느꼈다. 
    
## 코드 작성 (로직)
코테를 위해 파이썬을 공부했는데 열흘 정도 안했다고 그새 다 까먹더라.. 다시 정리해야 겠다.
아무튼 코드를 짜긴했는데 분명 더 좋은 방법이 있을 텐데 말이다.. 이제 for i in range는 왠지 모를 자존심 때문에 쓰기가 싫다.  
꾸준히 하는 수 밖에....
시간이 없어서 그냥 짰지만, 더 좋은 방법이 있을 것 같은데 프로그래머스 측 피드백을 얼른 보고 싶다.

그리고 API만 짜는거 포스트맨이나 다른 툴로 json만 주고받으면 되는데, 멍청하게 프론트 간단하게 짜다가 아차싶었다.
**제발 생각하고 코딩하자..**
    
    
## 새로 사용해본 것들
### MySQL 8.0 Workbench
사실 MySQL Server은 항상 쉘로 써와서 워크벤치는 이름만 듣고 써본 적이 없었는데, GUI로 통합된 환경을 사용할 수 있는 툴이었다. 
>직관적이라 좋았다.

### Django 2.2
장고는 이론만 대충 알고 있었는데, 공부하다가 코테 준비하느라고 그 동안 조금 놓고 있다가 처음으로 실습을 해본 것이었다.  
이제 스프링에 손을 좀 대려고 하는데, 스프링 끝나면 장고를 조금 더 큰 프로젝트를 진행하며 제대로 배워보고 싶어졌다.  
가장 중요한 폴더 구조에 대해서 정리를 해보려고 한다.
* `/migrations` : DB 관련 작업이 있을 때 마이그레이션을 해야하는데(`python manage.py makemigrations` -> `python manage.py migrate`), 
    이러한 마이그레이션 작업이 성공할 때마다 폴더 안에 파일들이 쌓인다.
* `settings.py` : 각종 환경 설정(언어, 외부 DB 연결 등) 파일
* `admins.py` : 관리자 관련 파일
* `models.py` : DB 모델 설계서 작성 파일, Mysql의 테이블이 장고에선 클래스이다. 
* `urls.py` : url규칙을 작성함. 연결하는 건 `views.py`와 `URL`을 연결해서 해당 URL에서 어떤 함수를 수행할 것인지 매칭한다.
* `views.py` : 위에서 말한 것처럼 어떤 URL에서 수행할 함수들을 정의한다. 함수의 리턴값은 HttpResponse를 직접 넘겨줄 수도 있고, 
    render를 통해 HttpReosponse를 넘길 수도 있는 등 다양하다.
*(cf) render와 redirect 구분*
`render`는 템플릿을 불러오고, `redirect`는 그저 URL로 이동하므로 값을 html로 넘길 수 없다. 그저 또 다른 views가 다시 실행되는 것이다.
* `/templates` : urls.py에 있는 URL들에 대해 어떤 페이지를 보여줄 지를 정의한다. 따라서 이 폴더 안에는 html파일들이 많을 것임

>생각보다 재미있었는데, 너무 기본적인 알고리즘으로만 짠 것 같고, 장고를 정말 장고답게 이용해 보고 싶은 욕심이 생겼다.  
>그리고 테스트 코드도 작성해 보고 싶다.

### Trello 
>트렐로는 개인적으로 쓰기 정말 편했다. 너무 직관적인 UI였고, window 앱이나 아이패드 앱, 아이폰 앱 모두 써봤지만 모두 엄청난 안정성을 가지고 있었다.
>그리고 목표 설정이 뚜렷해서 좋았다. 아마 트렐로 없었으면 하루하루 남들이 하는 장고 코드만 구경하고 이론 공부만 하다 끝났을 수도 있었을 것이다.  

### Git Bash
>애증의 깃... 내가 깃을 여기에 넣은 이유는 깃 배시를 오랜만에 써보기도 했지만, 동시에 버전관리를 해가면서 커밋했기 때문이다.
>사실 지킬도 그렇고, 이번 과제도 마스터로 푸시해야 인식을 해서 모두 마스터로 ~~때려~~넣을 수 밖에 없었지만 생각하며 버전관리를 하는 습관을 들이는 이유를 알겠더라.  

### docker
도커! 일단 설치는 맨 앞에 쓴 것처럼 애를 좀 먹었지만, 이젠 vmware가 꼴도보기 싫을 정도..
사실 GUI를 써야하는 상황에선 vmware를 어쩔 수 없이 써야할 테지만, 그 외에는 아마 도커로 갈아 타지 않을까 싶다.  
>vmware랑 virtualbox만 썼던 나에겐 너무 가벼워서 좋았다. 나중에 기회가 되면 도커 컴포즈 yml파일도 작성해 보고 싶다.  

### Postman
포스트맨은 사실 언젠지 기억은 안나는데 저학년 때 프로젝트 때 REST API 테스트한다고 써봤던 기억이 있다. 하지만 이미 잊은지 오래...
이번에 이거 하면서 웹 공부를 꽤 많이 했었는데 그래서인지 옛날에 썼을 때보다 더 익숙했다.
>그리고 트렐로랑 마찬가지로 직관적인 UI가 너무 좋았다.  


## 느낀점
생각보다 서버 구축이 재밌다는 걸 느꼈다. 짧은 시간동안 많은 걸 해볼 수 있어서 좋았다.  
혼자가 아니라 애들이랑도 해보고 싶어졌다. 다음엔 간단한 앱이라도 만들어 봐야겠다.
{: .notice}