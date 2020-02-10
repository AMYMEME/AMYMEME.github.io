---
layout: post
title: 제 3회 서울혁신챌린지
excerpt: "python(vevn, Ubuntu), 1d CNN"
categories: [단체 프로젝트]
comments: true
share : false
---

이 글은 회상글..이다. 지금은 2020년이지만 2019년에 수행했던 프로젝트에 대해 회상해보려고 한다.  
처음으로 20명 정도의 인원과 함께해보는 프로젝트였고, 프로젝트를 하면서 느낀점들에 대해서 쓰려고 한다.
{: .notice}

## 시작
아는 분에게 머신 러닝을 이용해 보안관련 프로젝트 진행하려고 한다고 연락이 왔다. *(프젝 필수요건이 ML이었다)*  
3~4월쯤?에 같이 하자고 연락이 왔었는데, 이 당시 [퍼듀 K-SW 여름 프로그램]({{ site.url }}/articles/2019-08-16-purdue_summer_program)(*포스트 아직 완성못해서 링크 안먹힘*)
에 지원한 상태여서 확답을 못 드렸던 기억이 난다.  
여차저차해서 일단 이름은 올려두었는데, 결국 여름에 퍼듀에 가게 되어서 잠깐 프로젝트 진행을 못 했었지만
돌아와 2019년 12월 12일?까지 참여했었다..

## 과정
### 팀 선정
팀은
* **플랫폼** 개발팀
* 테스트 **데이터셋(`abnormal packets` or `normal packets`) 생성**팀
* 생성된 데이터를 **전처리**하는 팀
* 전처리된 데이터를 기반으로 **학습 모델을 생성**하는 팀
* **통계**팀  

이렇게 있었다.  
나는 이 중에서 모델 생성팀에 들어가고 싶다고 했었는데, 패킷 데이터 전처리는 내가 해본 경험이 없고, 수학적인 지식이
있어야 데이터가 치우치지 않을 것 같았기 때문이다.
플랫폼 개발은 저 당시 내가 서버에 관심이 없었고, 거의 실력자분이 내정자(?)로 되어있었다.
통계팀도 마찬가지로 전공자 두 분이 프로젝트에 참여하시게 되면서 따로 팀을 꾸린거라 나는 데이터 생성팀이나 모델 생성 팀 중 하나를 골라야 했는데,
모델 생성이나 학습이 나에게 더 익숙하고 재미있을 것 같아서 선택하게 되었다.  

### 환경 구축
> 사실 나는 밑의 3번 팀에 속해있으므로, 나머지 팀의 자세한 과정에 대해서는 알지 못하나, 처음에 플랫폼 제작 구성은 이런식이었다. 

테스트 환경은 **단일 서버**로 구성되었으며, 
1. 데이터 생성 시스템에서 크롤러나 퍼저등을 이용해 생성한 데이터를 `pcap` 형식으로 위협 판별 시스템에게 넘긴다.
2. 전처리 모듈에서는 파서나 필터등을 제작하여 학습 모듈에 넘긴다.
3. 학습 모듈에서는 `CNN` 모델을 이용해 학습 결과를 갱신하고, 모든 결과는 `Dashboard`에서 확인이 가능하도록 내부 `DB(Logstash)`에 저장한다.
4. Dashboard는 `ELK` 스택으로 이루어지며, 데이터 조회나 검색은 `Elasticsearch`로, 시스템 사용자는 `Kibana`로 결과를 확인할 수 있다.
5. 사용자는 자신이 가진 패킷이 normal한지 abnormal한지의 여부를 대시보드를 거쳐 학습 모듈로 판별하며, 결과를 확인하도록 한다.
6. 통계 모듈에서는 변환기나 `TF-IDF`, 다향 로직스틱 회귀 등을 이용해 전처리 모듈의 성능을 개선한다.

따라서 이 프로젝트에서는 **내가 따로 환경 구축을 하지 않았다.**  
일단은 예선 통과 이전인 2019년 여름까지는 사업계획서를 써서 제출하고 구현 계획을 세우는 과정의 연속들이었다.
예선 통과가 되면, 시제품 제작 단계에 들어가 서버 마련 비용이 들어왔었고, 그 이후에 플랫폼 팀에서 서버를 파주셨고, 서버 주소는 직접 알려주지 않으셨으나 깃으로 푸쉬를 하면
젠킨스로 자동 배포하는 식으로 진행하였다. 
하지만 어느날 젠킨스 접속이 안되면서 (*플랫폼 팀이 아니라 모르겠으나 아마 젠킨스 서버가 다운되었던 것 같다*)
그 이후에는 서버 주소를 통해 `xftp`로 로컬 작업 파일을 옮기거나 `x shell`, `pycharm pro` 버전으로 리모트에 붙어서 작업하였다.  

### 모델 생성까지의 고찰
> 결론적으로 말하면 `1d(dimension) CNN(Convolution Neural Network)`를 이용하였다.  

#### 왜 CNN이었나?
#### 1. 생성 데이터 - 패킷의 구조
우리가 처리해야 할 데이터는 **전처리가 된** 패킷이었다. 데이터 생성팀에게 여쭤보니 *정상* 패킷 데이터를 수집하는 방법은 **단순한 검색 서치나
일반적인 웹 서핑, 동영상 재생 등의 활동**을 하면서 와이어샤크로 패킷을 수집하는 것이라고 하셨고,
*비정상* 패킷은 `DVWA`나 웹곳같은 곳에서 인젝션이나 `xss`, `csrf` 등의 공격을 수행하거나 치트 시트등을 활용하여 직접 모의 해킹 테스트 사이트에서 
**침입에 많이 쓰일 법한 패킷을 생성**하는 방식이었다. 여러가지 툴도 이용했고.. 그치만 내가 속한 팀도 아니고 자세한 설명은 하지 않으려고 한다..  
중요한건 생성하는 데이터 형식이 `pcap`이며, 패킷 유형은 `HTTP`나 `HTTPS`라고 하셨다는 점이다.  

#### 2. 데이터 전처리 - 방식
전처리 팀에서는 당시 특허 준비 중이었던 sql 필터 기술을 이용하셨다. 사실 백신 프로그램, IDS, IPS 등의 장비들은 **제조사마다 공격 유형이 통일되어
있지 않다**는 문제를 해결하기 위하여 **언어 기반 분류**를 하셨다.  
공격 유형에 따라 공격 형태가 어느 정도 비슷하며, 공격에 이용되는 언어의 범위도 한정되어 있다는 점을 이용하셨고, 따라서 언어와 공격 
유형 범위를 특정하여 언어 유형에 따라 자주 쓰이는 함수 키워드를 이용해 필터를 구성하셨다.
정리하면, 
1. `pcap` **헤더와 바디를 파싱**해 `JSON`으로 변환
2. JSON으로 변환된 패킷의 내용이 나타내는 **언어를 판단**
3. 미리 **언어별로 제작한 필터**를 적용해 **`numpy` 배열로 `feature` 산출**  

이런 식이었다. 참고로 `.npy` 파일의 numpy 배열(`ndarray`)의 첫번째 인덱스[0]는 `flag`로 쓰여졌다.
`normal`이면 `0`, `abnormal`이면 `1`로 표시하였고, 따라서 feature는 `labeled feature`이다. 

#### 3. Input & Output
이번 프젝을 진행하면서 가장 헷갈렸던 부분이었다. 
앞에서 전처리 팀에서 feature을 .npy 파일 형태로 주시므로, 딥러닝 모델을 구축하기만 하면되었다. 
* **Input : .npy파일**
* **Output : 모델 실행 결과**  

가 *시제품 단게*에서의 `I/O`였고, 
만약 *본선 진출시*에는 **실시간 패킷에 대한 처리**도 해야하므로, 모델의 실행 결과 뿐만아니라, 하나의 패킷이 normal인지 abnormal인지도
판단하여 넘겨줘야 했다.  
추가로 **파라미터(하이퍼 파라미터)를 자동으로 조정해주는 라이브러리**를 만들어 달라고 하셨다.
학습 시킬 때, `batch size`나 `filter size`, `learning rate` 에 따라 결과가 달라지기도 하는데, 이를 수작업하지 말고 몇 개를 후보군에
넣어놓고 자동화 할 수 있도록 해달라고 하셨다.  

#### 그래서 왜 CNN인가?
1. 앞서 전처리 과정에서 언어 별로 공격 페이로드에 많이 등장하는 필터를 적용하므로, `BoW`과 비슷한 구성의 `feature` 였다.
    * 패킷도 통신을 하기 위하여 쓰이고, 더군다나 HTTP 패킷이므로 패킷을 컴퓨터 간 소통에 쓰이는 문장으로 접근하고 싶었다. 
2. Input으로 들어오는 feature들이 `RNN`이나 `LSTM`같은 모델을 적용하기엔 적합하다고 생각되지 않았다.
    * feature들 간에 앞뒤 연관관계가 있는 것도 아니었고, `sequential` 하지도 않았기 때문이다.
3. 여러 논문을 찾던 중에 악성 패킷 데이터의 feature을 CNN으로 추출한 내용을 발견하고 이를 적용해 보기로 하였다.
    * 이 논문의 feature도 우리 input의 feature 처럼 숫자 배열이었다.
4. CNN이 NLP에서도 쓰이나? 싶었지만 1dCNN은 NLP와 더불어 로보틱스나 음성 통신 판독 등에서도 많이 쓰이고 있었다.  

**사실 다소 아쉬운 부분이 많은 모델 선택 과정이었다. 지금이라면 feature을 다르게 추출하거나 그냥 나이브 베이지안만을 적용해도 되지 않았을까
하는 생각이 든다.**


### 모델 생성 과정
> 앞서 결국 CNN을 선택했지만, 많은 이슈들이 있었는데 그에 대해 써보려고 한다.


#### 모델 구성 개요
![모델 구성도]({{ site.url }}/img/shc.io.png)
사실 그 동안 `CNN`은 이미지 처리에서만 쓰인다고만 생각했었으나, 조사해보니 생각보다 `NLP`에서도 **문장의 로컬 특징을 추출**할 때 많이 쓰이는 것을 알 수 있었다.  
가장 힘들었던 건 input size를 맞추는 거였다.

#### 코드
~~~python
# ...생략
def load_dataset():
    trainX = train_data[:, 1:]
    trainX = trainX.reshape(trainX.shape[0], 19, 1)
    trainy = train_data[:, 0]
    testX = test_data[:, 1:]
    testX = testX.reshape(testX.shape[0], 19, 1)
    testy = test_data[:, 0]
    trainy = to_categorical(trainy, 2)
    testy = to_categorical(testy, 2)
    return testX, testy, trainX, trainy

def create_model(trainX, trainy, checkpoint_path, batch_size, filter):
    n_timesteps, n_features, n_outputs = trainX.shape[0], trainX.shape[1], trainy.shape[1]
    verbose, epochs = 0, 10
    checkpoint = ModelCheckpoint(filepath=checkpoint_path, monitor="val_accuracy", verbose=1, save_best_only=True, mode='max')
    model = Sequential()
    model.add(Conv1D(filters=filter, kernel_size=2, activation='relu', padding='SAME', input_shape=(n_features, 1)))  # (none, 19, 16)
    model.add(Conv1D(filters=filter*2, kernel_size=2, activation='relu', padding='SAME'))  # (none, 19, 32)
    model.add(Flatten())
    model.add(Dense(200, activation='relu'))
    model.add(Dense(100, activation='relu'))
    model.add(Dense(50, activation='relu'))
    model.add(Dense(n_outputs, activation='softmax'))
    model.compile(loss='categorical_crossentropy', optimizer='adam', metrics=['accuracy'])
    model.fit(trainX, trainy, epochs=epochs, batch_size=batch_size, verbose=verbose, callbacks=[checkpoint])
    return model


def get_score(model, testX, testy, batch_size):
    _, accuracy = model.evaluate(testX, testy, batch_size=batch_size, verbose=0)
    return accuracy


def summarize_results(scores):
    print(scores)
    m, s = np.mean(scores), np.std(scores)
    final.append((m,s))
    print('Accuracy: %.3f%% (+/-%.3f)' % (m, s))

# ...생략
~~~

### 이슈
#### virtualenv
`virtualenv`은 새로운 환경에 자신이 설치하고자 하는 파이썬 패키지를 일일히 구성해주는 일이 번거롭고,
그렇다고 계속 같은 환경을 쓰다보면 해당 프젝과 관련없는 패키지들을 모두 로딩해야 하므로.. 이럴 때 쓴다.  
이 때 같은 라이브러리들을 일일이 설치할 수 없으므로 하나의 가상환경에서 쓰여진 라이브러리들을 버전과 함께
명시해둔 파일이 `requirement.txt`이다.
**requirement.txt 생성**
자신이 구성한 환경을 `requirements.txt` 로 만들기 위한 명령어는 `pip freeze > requirements.txt` 이다.
![이 프젝에서 실제로 만든 과정](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fk.kakaocdn.net%2Fdn%2FceeaDO%2Fbtqz9tNqgyl%2Fk5qKVFKVKKVpdhYkE1jlwK%2Fimg.png)
이번 우분투 서버에서는 python2와 python3이 모두 설치되어있어서 버전 명시를 해줬어야 했다..  
**requirement.txt대로 설치**
`source (만들어준 virtualenv이름)/bin/activate`하면 실행되고, 
pip list를 확인해보니까 저거밖에 없어서 requirement.txt를 통해 한꺼번에 실행하였다. 
![이 프젝에서 실제로 설치한 과정](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fk.kakaocdn.net%2Fdn%2FTZ9a4%2Fbtqz8YNDH3E%2Fu6ofdKMcfmSoVpslJs7IFK%2Fimg.png)  

#### 라이브러리 버전 문제 충돌
사실 라이브러리 충돌이 꽤 많았는데, 자세히 기억나지 않는다..
`keras`와 `tensorflow`, `python`간에 충돌이 제일 많았던 것 같다..
일단 **python3 을 사용해야 했고, tensorflow 2가 베타여서 어떤 함수 하나가 안된다**는 에러가 떴었던 것 같다.
그래서 tensorflow 1.13.1가 아니면 안된다해서 **다운그레이드했더니 keras와 버전이 안맞아**서.. 수정했던 기억이 난다.

실제 정확한 requirement.txt 내용은 다음과 같다.
- numpy==1.17.3
- tensorflow==1.13.1
- keras==2.2.4


## 정리
### 배운 것들과 느낀 점
#### 팀원의 중요성
이 프젝을 진행하기 직전 [R&D 데이터챌린지]({{ site.url }}/articles/2019-11-22-K_cyber_security_challenge_2019)(*포스트 아직 완성못해서 링크 안먹힘*)도 같이 진행했었는데, 여기서 1dCNN을 구현해보았지만 정확도가
0에 가깝게 나와서 다른 방법을 사용했다.. 자세한 건 저 포스트에 써놨다.  
그래서 만약 내가 혼자서 구현하라고 했으면 못했을 것 같다.. 이 프젝에서 가장 팀원의 힘들 느낀 순간이 아니었을까 생각한다. 

#### 소통
큰 프로젝트를 진행하니까 확실히 소통의 중요성을 느꼈다. 디스코드로 회의 날짜에 맞추어 음성 회의를 진행하긴 했지만, 잔디나 트렐로 같은 걸
이용했으면 조금 더 낫지 않았을까 하는 생각이 든다.



### 아쉬운 점
#### 1. !! 성능 !!

실제 성능이 되게 왔다갔다 했다... 90까지 웃돌았다가 갑자기 60이나 40으로 떨어지거나 말이다..
오버피팅이라고하기에는 오버피팅 후 서서히 올라가는게 아니라 **90->60->90->60->...** 이렇게 반복되는 형태였다.
우리는 **feature가 한쪽에 치우쳐져 있어서 그런게 아닐까 생각했지만 정확한 원인은 모르겠다.**  
다른 모델을 적용했어야 하지 않을까...그러기 위해선 feature를 추출할 때 다른 방법을 써야 했다고 생각한다.
필더를 구성할 때 단어를 하나하나씩 토큰화 하지 않고, 두 어절 이상으로 토큰화하거나 하는 등의 방법을 이용할 것 같다.  
또한 공격은 한 패킷만으로 진행되지 않으므로, 두개의 페이로드 즉, 두개의 패킷을 sequential하게 구성할 것 같다. 
그리고 feature로 들어오는 flag를 제외한 ndarray의 길이가 19뿐었었는데, 이 길이도 증가시킬 것 같다.  
음.. 쓰고보니까 데이터 전처리 팀과 모델 생성팀을 따로 나누지 않고 같이 진행했으면 좋지 않았을까 하는 생각이 든다.

#### 2. 정확도 예측
`softmax`는 다중클래스 모델의 출력층에서 확률값이 가장 높은 클래스(category)를 반환해준다.  
프젝중에 한 분이 **항목별 확률을 같이 달라**고 하셨는데, 우리가 만든 모델은 마지막 층에서 activation을 softmax로 지정하고
그냥 compile 해버리므로, 결과값으로 가장 높은 확률의 클래스만 알고 클래스별 확률은 구하지 못해서 해결하지 못했다.  
당시 시간이 없어서 해결하지 못했는데, *아마 functional로 구성하면 할 수 있지 않을까...?*  

 
### 느낀점
> 이렇게 큰 프로젝트에 참여해봤다는데 가장 의의가 있었던 것 같다.. 본선엔 진출하지 못해서 아쉬웠지만 문서화의 중요성 등등.. 많은 것을 느꼈다.  
