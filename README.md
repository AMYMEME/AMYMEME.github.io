# personal blog

### From Leonids Jekyll Themes (Thank You !)
[Leonids](http://renyuanz.github.io/leonids) is a clean Jekyll theme perfect for powering your GitHub hosted blog.

#### 2020\. 01. 29 로컬 테스트 수정과정
1. docker로 재빌드 시 파일이 깨지는 현상이 발생하여 직접 ruby devkit 깔아서 실행
    아마도 버전문제로 충돌이 일어나는 것 같음
2. gem install jekyll
3. gem install bundler
4. jekyll serve 시 오류 발생
5. `gem install tzinfo-data`
6. Gemfile에 `gem 'tzinfo-data', platforms: [:mingw, :mswin, :x64_mingw]` 추가
7. `bundle install`
8. `bundle exec jekyll serve`

