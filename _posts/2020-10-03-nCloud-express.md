---
layout: post
title:  "nCloud로 express server 배포하기"
date:   2020-10-03T22:00:00
author: JunYoung Jang
categories: Nodejs
---

# Backend 지식 공부

## Ncloud를 이용하여 Express server 배포하기

naver cloud platform을 이용하여 express generator로 생성한 express server를 pm2를 활용하여 배포해보았습니다. 나중에 스스로 잊지 않기 위해 과정을 간략히 적어놓으려고 합니다..ㅎㅎㅎ

### 1. naver cloud platform console에서 서버 생성

서버 생성에 관한 전체적인 흐름은

https://docs.ncloud.com/ko/compute/compute-1-1-v2.html

naver cloud platform에서 제공하는 서버 생성 설명서를 참고하면 좋습니다!

#### ACG(Access Control Group) 설정

저는 mysql과 node.js express를 활용하는 server가 필요하기 때문에 다음과 같이 ACG 설정을 진행하였습니다.

![image](https://user-images.githubusercontent.com/61405355/93170642-8ea11b00-f762-11ea-9f11-d3823fcc5e97.png)

- 22 : ssh 접속을 위한 port 번호
- 80 : http 기본 포트
- 3000 : nodejs의 기본 개방 서버 포트
- 3306 : mysql 접속을 위한 포트

### 2. 서버에 접속 & 사용자 설정

이번에는

https://docs.ncloud.com/ko/nodejs/nodejs_console.html

이 설명을 참고하면서 보시면 좋습니다.

<br>
<br>

위 과정에 맞게 서버를 생성하면 console창 server에서 성공적으로 생성된 server를 확인할 수 있습니다. 이 생성된 서버에 접속하는 방법은 2가지가 있습니다.

1. 포트포워딩을 이용하는 접속
2. 공인 IP를 이용하는 접속

node.js 상품을 사용하기 위해는 공인 IP 주소가 필요하기 때문에, 저는 2번 방법을 이용했습니다(공인 IP 주소 사용은 요금이 별도로 부과됩니다!!)

**공인 IP 방법을 사용하시면 포트포워딩을 설정해주시면 충돌이 일어나 접속에 문제가 생길 수 있습니다**

공인 IP를 생성한 후에는 서버에 접속을 시도해보면 됩니다. 저는 windows 운영체제를 사용하고 있어 putty를 이용하여 서버에 접속하였습니다.(git bash를 이용하여 `ssh root@공인아이피` 명령어를 통해 접속할 수도 있습니다)  
최초에 login은

```
login as : root
root@'공인IP's password : 관리자 비밀번호(관리자 비밀번호 확인 방법은 위 링크 확인)
```

다음과 같이 진행하면 됩니다.

그 다음 이어지는 단계는 서버에서의 작업을 용이하게 하기 위한 작업들입니다.

1. root 이외의 사용자를 생성합니다.
   : `adduser 사용자 이름`을 통해 사용자를 추가합니다.

2. sudoers 파일을 수정함으로써 권한을 수정해 줍니다.
   : 우리가 추가한 사용자는 sudo를 사용할 수 있는 권한이 아직 없습니다. root 권한으로 서버에 접속한 뒤에,  
   `sudo vi /etc/sudoers` 명령어를 입력하면 sudoers 파일이 뜹니다. 여기서

   ```
    root    ALL=(ALL:ALL) ALL
   ```

   이 적힌 부분이 있을 텐데 이 부분에

   ```
    사용자이름   ALL=(ALL) NOPASSWD:ALL
   ```

   을 입력해주면 해당 사용자는 sudo 를 사용할 수 있는 권한을 가지고, sudo를 사용할 때 password를 물어보지 않게 할 수 있습니다.

   저는 이부분에서 입력을 잘못해서 sudo를 입력하면

   ```
   >>> /etc/sudoers: syntax error near line ## <<<

   sudo: parse error in /etc/sudoers near line ##

   sudo: no valid sudoers sources found, quitting

   sudo: unable to initialize policy plugin

   ```

   다음과 같은 오류가 발생했습니다. 다음 오류는 sudoers파일을 잘못 건드렸을 때 발생하는 오류입니다 ㅠㅠ  
    혹시 sudoers 파일을 잘못 건드리신 분들은 다음 페이지를 참고하시길 바랍니다.

   https://brownbears.tistory.com/228

3. ssh key로 접속하도록 설정합니다.
   : 공개키와 비밀키를 이용해서 접속을 쉽게 할 수 있으며, 패스워드 방법보다 훨씬 안전한 방법입니다.

   git bash에서 `ssh-keygen` 명령어를 이용하여 공개키, 비밀키를 생성합니다. 그 후에 `~/ .ssh`폴더에 생성된 id_rsa.pub 파일을
   `pcsp id_rsa.pub 사용자이름@공인IP:/home/사용자이름` 명령어를 이용해서 서버로 복사해줍니다.  
   그 다음 서버에 새로 생성한 사용자로 접속하여 .ssh 폴더를 만들어줍니다. 그 이후 아까 복사된 id_rsa.pub 파일의 내용을 .ssh/authorized_keys 경로로 복사해줍니다.  
   이제 로그아웃 후에 다시 접속하면 password없이 다시 접속할 수 있게 됩니다.

4. 패스워드를 삭제시킵니다.
   : 사용자 id로 접속한 후에  
   `sudo passwd -d 사용자이름` 명령어를 사용하면 사용자의 password를 삭제할 수 있습니다. 이렇게 하면 해당 사용자는 password로는 접속이 불가능해지므로 보안을 강화시킬 수 있습니다.

## pm2로 배포해보기

이제 서버에서 제가 로컬에서 개발한 git project를 배포하는 일만 남았습니다..!!

우선 서버에 접속한 뒤에 `apt-get update`를 이용하여 update를 실행해줍니다.

그 이후에 서버에 git, node, npm, pm2를 각각 설치하여 줍니다.

```
app-get install git
sudo apt-get install curl
curl -sL https://deb.nodesource.com/setup_버전(ex> 12).x | sudo -E bash -
apt-get install nodejs
npm install pm2@latest -g
```

이후에 git clone 명령어를 이용하여 배포하고자 하는 프로젝트를 clone합니다.

이후, package-json이 있는 폴더에서 pm2 start ./bin/www를 이용하여 서버를 켜주면..!!(express generator로 생성한 프로젝트는 서버를 실행시키는 실질적인 파일이 /bin/www입니다)

로컬에서 공인아이피:3000 으로 접속했을 때 제가 배포하고자 하는 node.js 웹 어플리케이션이 배포되었음을 확인할 수 있습니다!

기본 start 명령어를 이용하면 fork 모드로 실행시켜 단일 스레드에서 동작하게 됩니다. 따라서 클러스터 모드로 실행시켜 코어를 최대한 활용할 수 있도록 해야 합니다.

아래 링크를 참고하시면 좋을 것 같습니다.

https://engineering.linecorp.com/ko/blog/pm2-nodejs/
https://m.blog.naver.com/sssang97/221981962293
