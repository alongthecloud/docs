---
title: Github 에서 git 을 사용할 때 여러 SSH Key를 사용하기
---
### SSH Key 생성 및 저장

ssh-keygen 를 사용한다면 다음 처럼 하면 된다.
~~`ssh-keygen -t rsa -b 4096 -C "youremail@domain.com"`~~
rsa 형식보다 ed25519 또는 rsa-sha2-512 형식을 추천. 따라서,
`ssh-keygen -t ed25519 -C "youremail@domain.com"`

파일명을 따로 입력하지 않으면 ssh 폴더( 윈도우의 경우 %HOMEPATH%\.ssh ) 에 id_rsa, id_rsa.pub 가 생성된다. 어쨌든 .pub 가 붙은 파일이 공개키이며 이것을 Github에 등록하면 된다.

### Config 파일을 만들어서 계정별 사용

Github 에서는 하나의 ssh key를 하나의 계정에만 등록시킬 수 있다. 이미 등록되어 있는 ssh key를 다른 계정에도 등록시키려고 하면 "사용하고 있는 key"라는 이유로 등록이 되지 않는다.

방법을 찾아보았는데 config 파일을 만드는 방법이 깔끔하고 잘 되었다.

.ssh 폴더 아래에 config 파일을 만든다. 형식은 윈도우, 리눅스 둘 다 동일.

```
Host github.com-a
HostName github.com
User git
IdentityFile ~/.ssh/a_rsa

Host github.com-b
HostName github.com
User git
IdentityFile ~/.ssh/b_rsa
```
.ssh 폴더 내용은 다음 처럼 되어 있을 것이다. 리눅스의 경우 authorized_keys, known_hosts 등의 파일이 있을 수 있다.

```
.ssh
├── config
├── a_rsa
└── a.pub
```

### 사용할 때

git 으로 clone 할 때  [github.com](http://github.com) 의 부분을 위 config 에서 정한 host 이름으로 바꾸어주면 된다.

예를 들어, github 에서 가져온 repository 의 github 주소가 git@github.com:gitaccount/repository.git 이라면

`git clone git@github.com-a:gitaccount/repository.git`
