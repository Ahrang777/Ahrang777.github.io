---
layout: single
title:  "블로그 로컬에서 실행"
categories: etc
tag: [blog, jekyll, ruby]
toc: true
author_profile: false
sidebar:
    nav: "docs"
# search: false # 검색되길 원하지 않을경우 false    
---

# 로컬에서 실행

## 사용 이유

깃허브 블로그를 수정하거나 내용을 추가하는 등 작업을 할때 깃허브에 올린 내용이 반영되는데 어느정도 시간이 걸린다. 잘못된 내용을 작성했을 경우 다시 수정하고 제대로 반영되었는지 확인하며 수정해 나가는것은 많은 시간이 소요된다.

때문에 로컬에서 내가 작성한 블로그를 실행해서 바로바로 변경사항을 반영해보고 원하는대로 반영이 된것을 확인하면 그때 commit, push 를 하여 깃 블로그에 반영한다면 편할것이다.



## 로컬 개발환경 설정 방법

### 과정 1)


https://rubyinstaller.org/downloads/ 에서 WITH DEVKIT 부분에서 원하는 RubyInstaller를 선택하여 다운로드 하고 루비를 설치한다. 



### 과정 2)


설치를 완료하면 창이 나오는데 3번째 것을 선택하여 install 한다.



### 과정 3)

이후 cmd창을 열고 아래 명령어 입력

cd Documents (이 명령어를 입력하면 위와같은 모습이 된다.)

gem install jekyll

gem install bundler



### 과정 4)

깃허브에서 클론한 폴더(제 경우엔 Ahrang777.github.io 입니다.) 를 선택하고 shift + 우클릭 을 한 후 '여기에 Powershell창 열기' 선택하고 'bundle install' 명령 입력하여 설치한다.

'bundle exec jekyll serve' 명령을 입력하면 로컬 서버가 구동된다.

'bundle exec jekyll serve' 를 입력하였는데 에러가 발생하며 'cannot load such file 이라고 하면서 webrick 파일 없다' 이런식의 메시지가 나오면 'bundle add webrick' 입력하고 다시 'bundle exec jekyll serve' 를 입력해보면 정상적으로 로컬 서버가 구동된다.



### 과정 5)

http://localhost:4000/ 로 들어가면 내가 만든 깃블로그의 모습을 볼 수 있다. 이후 포스팅한 글이나 설정들을 수정한 후 로컬 서버를 구동하고 http://localhost:4000/ 로 들어가보면 바로 바로 변경사항이 반영됨을 알 수 있다.



