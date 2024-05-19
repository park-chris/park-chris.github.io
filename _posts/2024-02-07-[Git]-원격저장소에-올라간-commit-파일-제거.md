---
layout: post
date: 2024-02-07
title: "[Git] 원격저장소에 올라간 commit 파일 제거"
tags: [Git, ]
categories: [Git, ]
---


---


Android Studio를 사용하다가 git에 .gitignore에 추가해놓은 파일이 올라간 것을 확인했다.


API key 값처럼 보안이 필요한 파일들은 저장소에 올려놓으면 안되기에 당황스러웠지만 다행스럽게도 저장소가 비공개 설정이었다.


무튼 파일이 올라갔으니 제거를 해야되는데 나는 꾸준히 commit을 올렸기에 commit history를 보면 파일들이 다 남아있는 상황이었다. 그래서 준비했다.


원격저장소에 올라간 파일들을 깔끔히 삭제하는 방법을 말이다.



## Github Commit History 파일 삭제


---


1. 프로젝트 최상위 루트 디렉토리(폴더)에서 git bash 창을 연다.


<예시>


프로젝트 최상위 루트 : `D:\androidApp\TodayPrice`


2. 원격저장소 history 파일 제거 명령어



 
```shell
$ git filter-branch --force --index-filter 'git rm -r --cached --ignore-unmatch파일경로/파일명' --prune-empty -- --all
```
  



<명령어 옵션 설명>


- `--index-filter` : 인덱스에 직접 수행


- `git rm -r --cached --ignore-unmatch 경로` : 경로를 모든 브랜치 및 커밋에서 삭제


- `--prune-empty` : 빈 커밋 제거


- `-- --all` : 모든 브랜치와 태그에서 수행


<예시>



 
```shell
$ git filter-branch --force --index-filter 'git rm -r --cached --ignore-unmatch app/src/main/res/values/api_key.xml' --prune-empty -- --all
```
  



명령어가 진행중인 상태다. 많으면 많을수록 오래 걸리니 명령어가 끝날때까지 기다리자.


![0](/assets/img/GBD21/0.png)


3.  원격저장소에 강제 푸시



 
```shell
$ git push --force
```
  



강제 푸시 이유 : `git filter-branch` 를 사용해서 저장소의 히스토리를 변경하면 기존 커밋이 정되어서 강제로 푸시한다.

