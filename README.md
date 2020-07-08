# personal blog

### From Leonids Jekyll Themes (Thank You !)
[Leonids](http://renyuanz.github.io/leonids) is a clean Jekyll theme perfect for powering your GitHub hosted blog.

#### ~~2020\. 01. 29 로컬 테스트 수정~~ (Windows, Deprecated)
1. docker로 재빌드 시 파일이 깨지는 현상이 발생하여 직접 ruby devkit 깔아서 실행
    아마도 버전문제로 충돌이 일어나는 것 같음
2. gem install jekyll
3. gem install bundler
4. jekyll serve 시 오류 발생
5. `gem install tzinfo-data`
6. Gemfile에 `gem 'tzinfo-data', platforms: [:mingw, :mswin, :x64_mingw]` 추가
7. `bundle install`
8. `bundle exec jekyll serve`


#### 2020\. 07. 08 로컬 테스트 수정 (MacOS)
Mac OS 구매 후 [jekyll 설치](https://jekyllrb-ko.github.io/docs/installation/macos/)를 하면 된다.  
나는 위 페이지에서 rbenv가 필요없어 보여서 안했는데, 만약 rbenv를 설치 하지 않으면 gem install bundler 시
```bash
$ gem install bundler
ERROR:  While executing gem ... (Gem::FilePermissionError)
    You don't have write permissions for the /Library/Ruby/Gems/2.3.0 directory.
```
이런 오류가 뜬다.

시스템 ruby를 이용하고 있기 때문에 권한이 없어 gem 설치가 막힌다는 것..  
sudo를 통해 실행하면 설치가 가능하지만, 보안상 이유로 권장하지 않는 설치법이라고 한다.  
**brew로 rbenv까지 설치하자**

1. gemfile 있는 디렉토리로 이동
2. bundle install
3. bundle exec jekyll serve
