---
layout: post
title:  "GIT 변경사항 되돌리기"
categories: [git]
tags: [git]
---
# git 수정전으로 되돌리기

- repository 안의 모든 내용 수정 되될리기
    ~~~
    git checkout .
    ~~~

- 특정폴더 아래의 모든 내용 수정 되돌리기
    ~~~
    git checkout <dir>
    ~~~

- 특정 파일 수정 되돌리기
    ~~~
    git checkout <filename>
    ~~~

- stage에 올린경우 되돌리기
    ~~~
    git reset
    ~~~
