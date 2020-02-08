---
layout: post
title: 단백질 이름 추출하기
excerpt: "python(anaconda, jupyter notebook), NLP, Naive Baysian, Bag of words"
categories: [개인 프로젝트]
comments: true
share : false:q
---

이 글은 회상글..이다. 지금은 2020년이지만 2019년에 수행했던 프로젝트에 대해 회상해보려고 한다.   
이 개인 프로젝트는 기계학습과 인공지능 강의를 수강하고 NLP 토이 개인 프로젝트로 단백질 NER을 주제로 잡아
진행한 경험이다
{: .notice}

# 목표
> GNI 코퍼스에에 단백질 Named Entity를 태깅하자

### 고찰
단백질을 분류하기 위해 단백질 이름들이 어떻게 쓰이는지 알 필요가 있었다.  
논문에 따라 단백질 이름을 축약하기도 하였으므로 하나의 단백질은 각기 다른 논문에서 다른 이름으로 불리기도 한다.  

### 과정
내가 진행한 과정은 이렇다.
1. 사람의 단백질 이름과 유전자 이름을 labeled set으로 구성하여, 유전자와 단백질의 feature를 추출하여 유전자와 단백질 중에 단백질을 분류하는 모델을 구성한다.  
2. GNI 코퍼스 중 전문용어(`terminology`)를 추출한다.  
3. 2에서 추출된 `terminology`가 단백질인지의 여부를 1을 이용하여 결정  
(*사실 2에서 추출한 terminology는 단백질이나 유전자 외 다른 단어가 많을 것 같다.*)
4. `PoS Tag`에 단백질 `Named Entity`를 태깅함

# 세부과정
## 환경 구성
1. `Python 3`
2. `Anaconda`
3. `Jupyter Notebook`
4. ~~SublimeText(데이터 전처리만 함..)~~
5. 따라서 `windows 10 home`

## 데이터 수집
### 사람의 단백질에 대한 정보
사람의 단백질에 대한 정보를 알기 위하여 `goa_human.gaf`를 [다운](http://current.geneontology.org/annotations/)받았다.
gaf는 `go-annotation-file`로 포맷에 대한 정의는 [여기](http://geneontology.org/docs/go-annotation-file-gaf-format-2.1/)에 나와있다. 
초기 goa_human.gaf 파일 앞부분에는 파일 관련 정보로 인해 제대로 raw text slicing이 되지 않으므로 직접 전처리해준다. *(그냥 지워주면된다)*
이를 이용하여 파일 포맷에 맞게 3번째 인덱스[2]를 symbol로 하여 `protein_object_symbol`라는 중복 제거 리스트로 저장한다. 

### 사람의 유전자에 대한 정보
사람 유전자 정보는 `MTB`라는 데이터베이스에서 얻을 수 있다. 이 데이터베이스에서 얻은 gaf에서는 2번째 인덱스[1]가 symbol이므로 단백질 정보와
마찬가지로 이를 따로 `gene_object_symbol` 중복 제거 리스트로 저장한다.

~~~ python
import re

f1 = open('goa_human.gaf')
lines = f1.readlines()
protein_object_symbol = []
protein_object_symbol.append(re.split('\t', line)[2]) for line in lines
protein_object_symbol = list(set(protein_object_symbol))

f2 = open('goa_gene.gaf')
lines = f2.readlines()
gene_object_symbol = []
gene_object_symbol.append(re.split('\t', line)[2]) for line in lines
gene_object_symbol = list(set(gene_object_symbol))
~~~

## bag of words
protein_object_symbol에 있는 symbol들을 위키피디아에 검색한 결과의 일부분을 저장하였는데, 원래는 모든 단어를 저장하려고 했지만
중복 제거된 protein만 **19만**개여서 실제로는 일부분만 저장하였다.

한 단백질에 대해서 나오는 단어들을 `labeled_protein` 튜플로 저장하고, 유전자 대해서도 마찬가지로 진행한다.
이를 모두 `bag`에 넣고 위키피디아에 검색된 내용이 없거나 모호한 결과가 나오면 에러 처리를 한다.

~~~python
# ...생략...
import wikipedia as wiki
from nltk.corpus import stopwords

labeled_protein = []
for protein in protein_object_symbol[:200]:
    try:
        raw = wiki.page(protein).cocntent
        bag += set( [w for w in raw.split() if not w.lower() in stop_words] )
    except wiki.exceptions.PageError as e1:
        print(e1)
    except wiki.exceptions.DisambiguationError as e2:
        print(e2.options)
# gene도 같은 방법으로 진행
~~~


## feature 추출과 학습
앞서 만들어 놓은 bag을 이용하여 bag의 단어들이 `terminology`를 위키피디아에서 검색한 결과의 내용에도 있으면 True를 반환하도록 한다.  
미리 만들어 놓은 labeled 리스트를 이용하여 `labeled_protein`과 `labeled_gene`을 합친 전체 리스트를 랜덤하게 섞고 이 들을 `trainset`과 `testset`으로 나눈다.  
`NaiveBaysian`을 이용하였는데, 이유는 기초 수준에서 가장 손쉽게 모델을 구성해볼 수 있었고, 
지금 내가 구성한 `feature set`이 단백질이나 유전자와 관련있는(자주 등장하는) 단어이기 때문이다.  
따라서 이런 `BoW`에선 나이브 베이지안이 꽤 적합한 모델이고, 데이터셋이 문장형태가 아니므로 RNN 등을 적용하지 않기로 했다.
결론적으로 정확도는 약 `80%`가 나왔다.

~~~python
import nltk
import random

def features(terminology):
    features = {}
    for word in bag:
        features['contains({})'.format(word)] = (word in terminology)
    return features

labeled=labeled_protein+labeled_gene
random.shuffle(labeled)
featuresets=[(features(w), p) for (w, p) in labeled]

size=int(len(featuresets)*0.1)
train_set=featuresets[:size]
test_set=featuresets[size:]
classifier=nltk.NaiveBayesClassifier.train(train_set)
print(nltk.classify.accuracy(classifier, test_set))
# classifier.show_most_informative_features(5)
~~~



## GNI Corpus 적용
### 사전 작업 1
1. `PoS tagging`을 한 후에
2. 명사류가 2개 이상인 것들에 대해서 `NP chunk` 처리하고
3. 연속된 단어 형태를 `NNlist`에 넣어준다.
4. 또한 나중에 `PROTEIN`으로 `Named Entity` 하기 위해 태깅된 단어를 모아 named_entity 리스트에 넣어준다.

~~~python
import nltk 
from nltk.corpus import *
import os
path='코퍼스 경로'
GNICorpus=PlaintextCorpusReader(path, 'gni-9-4-197.txt', encoding='utf-8')
sents=nltk.sent_tokenize(GNICorpus.raw())
NNlist=[]
named_entity=[]
for sent in sents:
    words=nltk.word_tokenize(sent)
    tagged_word=nltk.pos_tag(words)
    named_entity.extend(tagged_word)
    chunk=nltk.RegexpParser("NP: {<N.*>{2,}}").parse(tagged_word)
    for subtree in chunk.subtrees():
         if subtree.label() == 'NP': NNlist.append(' '.join([w for (w,tag) in subtree.leaves()]))
~~~

### terminology
앞에 labeled에서 학습시킨 모델을 통해 GNI 코퍼스에서 terminology 중 단백질을 태깅하는 것이 이번 프로젝트의 주 목적이었다.  
{: .notice}
1. `stop_words`와 `wikipedia`에 없는 단어들만 추출하는 `word_filter` 함수를 정의한다. 
2. 앞의 `NNlist`에 대해서 GNI 에만 등장하는 기관명, 도메인, 특수문자 등을 nltk의 `Regexp tagger`을 이용해 `etc`로 태깅한다.
3. 나머지는 모두 `NN`으로 태깅한다.
4. 앞서 정의한 word_filter을 적용해 전문 용어 리스트인 `terminology`를 만든다.

~~~python
def word_filter(list):
    from nltk.corpus import stopwords
    from nltk.stem import WordNetLemmatizer
    
    stop_words=set(stopwords.words('english'))
    temp=[w for w in list if not w.lower() in stop_words]
    lemmatizer=WordNetLemmatizer()
    dictionary=nltk.corpus.words.words('en')
    final=[]
    for w in temp:
        for word in w.split():
            if lemmatizer.lemmatize(word.lower(), pos='n') not in dictionary: final.append(word)
    return final

patterns=[
    (r'\d+', 'etc'),
    (r'[,./;\'\[\]<>?:\"{}`!@#$%^&*()_+-=\\\|~]+', 'etc'),
    (r'org|orcid|ORCID|http|et|Tel|Fax|kr|ac|co|com', 'etc'),
    (r'.*', 'NN')
]

NNlist=nltk.RegexpTagger(patterns).tag(NNlist)
NNlist=[a for (a,b) in NNlist if b=='NN']
terminology=set(word_filter(NNlist))
~~~

### terminology에 features를 적용
생성해놨던 모델(`classifier`)을 통해 protein이라는 판단을 내리면 `PROTEIN`으로 태깅한다.
~~~python
NER=dict(named_entity)
for term in set(terminology):
    try:
        raw=wiki.page(term).content
        terminology_words=raw.split()[:10]
        if classifier.classify(features(terminology_words))=='protein':
            NER[term]='PROTEIN'
    except wiki.exceptions.PageError as e1:
        print(term)
    except wiki.exceptions.DisambiguationError as e2:
        print(term)
~~~

# 한계점
#### 단백질 여부 판단
bag of words 에 terminology의 위키피디아 단어들이 포함되는가를 feature으로 구현했더니 관련 없는 내용이 우연히 겹쳐도 단백질로 판단함
feature 정의할 때 부터 발생한 문제이므로 feature을 정의하는 방식을 word2vec 등을 이용하여 추출해야 할 것 같음  
#### 데이터 부족
상대적으로 단백질 이름에 비해 gen의 이름을 모아놓은 DB 크기가 작아 labeled에 불균혀이 생김  
다른 gene DB를 찾아보아야 할 것 같음  

# 느낀점
> 2020년에 이 개인프로젝트를 회상하면서 느낀점은.. 글 초기 부분의 진행과정에서 언급한 것처럼
> GNI 코퍼스에서 추출한 terminology에는 단백질이나 유전자 외 다른 단어가 많을 것이므로 
> 애초에 feature 추출 할 때 단백질과 유전자만 이용하지 말고 더 다양한 단어 pool을 이용하는 게
> 좋지 않았을까...생각이 든다. 
> 지금이라면 word2vec으로 다양한 분야의 단어들, 단백질이나 유전자를 포함한 bio 전문 단어들 중에서 단백질 단어의 feature만을 추출할 것 같다.   
> 그리고 주피터에서 작업해서 코드들이 시퀀셜하게 작성되어있는데, 조금 리팩토링이 필요한 것 같다..
> 나중에 모듈식으로 고쳐서 깃헙에 올려봐야겠다.  
> 지금 보니까 여러모로 부족한 점이 많은 프로젝트다..