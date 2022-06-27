---
title: Ubuntu 20.04 LTS와 Github 연동하기
description: ubuntu를 쓰며 불편했던 github 로그인을 처리하자.
categories:
- ubuntu
- git
tags: 
- 우분투
---

# Ubuntu 20.04 LTS에서 push하기
ubuntu에서 내가 원하는 레포에 push를 하려니 username과 깃헙 토큰을 계속 입력해야했다. <br>
처음에는 별 대수롭지 않게 여겼는데 push를 할 때마다 깃헙 토큰을 복사해서 붙여넣는게 여간 귀찮은 일이 아니다. <br>
그래서 구글링을 해보니 아주 간편하게 해결하는 방법이 있었다..! (아주 감사합니다ㅠㅜ) <br>

```
git config credential.helper store
```
를 입력 후에
```
git push 자동 로그인 해야하는 레포 주소
# ex) git push https://github.com/DoIn-Sin/DoIn-Sin.github.io
```
를 입력해주면 그 후 push 할 때마다 입력했던 username과 깃헙 토큰을 입력해주면 된다!! <br>

아주 편리하게 마음껏 push를 날릴 수 있다ㅎㅎ

# Reference
[1] [참조한 블로그](https://daechu.tistory.com/33).