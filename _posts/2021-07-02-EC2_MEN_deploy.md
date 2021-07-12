---
layout: post
title : AWS EC2에서 Express.js 서버 배포 및 MongoDB 설정
category: Web
tags: [AWS, EC2, ExpressJS, MongoDB, NodeJS, nginx, pm2]
complete: true
---

##### 참고 사이트
* [Deploying NodeJS, Express, MongoDB application on AWS EC2](https://chinmaypatil.medium.com/deploying-nodejs-express-mongodb-application-on-aws-ec2-e32a8e3ea8a1)


## Express.js, MongoDB 환경설정

```
sudo apt-get update
sudo apt-get install git
```

#### node, npm 설치

```
sudo apt-get install nodejs
sudo apt-get install npm
```

원하는 npm version이 있다면

```
sudo apt-get install npm@[version] -g
```

#### MongoDB 설치
다음 <a href="https://docs.mongodb.com/manual/tutorial/install-mongodb-on-ubuntu/">docs</a>를 따라하면 어렵지 않게 설치후 실행가능하다.

```
sudo systemctl status mongod
```

위 명령어를 통해 MongoDB가 잘 실행되는 것을 확인했으면 DataBase 보안을 위해 authorization을 추가해야한다.


## DataBase 설정


#### admin database 실행 후, root 관리자 생성
```
mongo
```

```
> use admin
> db.createUser({ user: '이름', pwd: '비밀번호', roles: ['root']})
```

#### mongod.conf에서 authorization을 enabled로 설정
```
# mongod.conf

net:
  port: 27017
  bindIp: 0.0.0.0

# ...

security:
  authorization: 'enabled'
```
<span class="warning">이때 들여쓰기 2칸 제대로 안했다가 conf error 난...</span>


#### mongod service '재실행'

```
sudo service mongod restart
sudo systemctl status mongod
```

#### 서버의 MONGO URI에 해당하는 database 및 user, pwd 생성

<span class="warning">MONGO URI는 `.env` 파일에 설정되어 있다.</span>

나의 경우 admin database로 가 server database owner에 대한 role도 root user에 추가해주었다.

```
> db.updateUser("username", {roles:[{role: 'dbOwner', db: 'database name'}]})
```

##### 생성한 user 정보로 server database 접속해보기

```
mongo --port 27017 -u 사용자계정 -p '비밀번호' --authenticationDatabase '디비이름'
```


이렇게 database 설정까지 완료했으면 이제 Express.js 서버를 가져와 실행해준다. (database를 미리 한 이유는 안하면 서버를 먼저 가져와서 실행해봤자 database 접속 못 할거라)


## Express.js 실행

#### package.json 설치
github에 존재하는 Express.js repository를 clone 해온 후 해당 폴더로 이동해 package.json에 해당하는 패키지들을 설치한다.

```
npm install
```

여기까지 마치고 나면 해당 폴더의 하위에 `node_modules/` 폴더가 생성된 것을 확인할 수 있다.
여기까지 마치면 서버를 실행해 서버가 잘 실행되고 데이터베이스에 잘 연결되었는지 확인한다.


**포트 방화벽 해제**
외부에서 서버의 특정 포트에 접근하려면 해당 포트의 방화벽을 해제해야 한다. <br>
<span class="warning text-highlight">어떤 포트를 왜 열어야 되는지 정확히 모르겠다</span>
{: .comment-box}


## NGINX 설정

#### nginx 설치

```
sudo apt-get install nginx
```

##### nginx

1. HTTP 프로토콜을 준수하여 HTML, CSS, JavaScript, 이미지와 같은 정적 파일을 웹 브라우저(Chrome, Opera ...)에 전송하는 역할을 한다.
2. 리버스 프록시(Reverse Proxy)로서의 역할
  * proxy server는 가상 서버이고 reverse server가 실제 응용프로그램 서버를 의미한다.
  * client가 proxy server로 request를 보내면 proxy server는 reverse server로부터 데이터를 가져오는 역할을 한다.
  * 웹 응용 프로그램 서버에 reverse proxy가 필요한 이유는 request에 대한 버퍼링 때문이다. client가 직접 app server에 요청을 보낼 경우, 2개 이상의 요청이 들어올 경우, 처리 중인 request 외에는 응답 대기 상태가 되기 때문에 proxy server를 두어 request를 배분하는 역할을 한다.


#### nginx configuration 파일 설정

http/https로 보낸 요청을 실제 서버 port로 pass 해주어야 한다.

`/etc/nginx/sites-available` 에 존재하는 기존의 default 파일을 삭제 후 http/https로 넘어오는 request를 처리할 server block을 생성한다.

```
# default

server {
	listen 443;
	server_name `domain name`; # 여러 개의 도메인을 설정할 경우 space로 구분하면 됨
	location / {
		proxy_pass http://127.0.0.1:`web server port`;
	}
}
```

<span class="text-highlight warning"> MongoDB compass로 내 데이터베이스를 좀 더 쉽게 관리하고 싶은데 nginx 설정 후, DATABASE로 접근이 안된다... configuration 설정을 어떻게 해야하는걸까...</span>

#### nginx 실행하기
먼저 configuration file이 제대로 작성되었는지 확인한다.

```
sudo nginx -t
```
제대로 작성되었다면 nginx를 실행한다.

```
sudo systemctl enable nginx
sduo systemctl status nginx
```

## PM2

원래는 구매한 도메인 SSL certificate 완료를 해서 해당 도메인을 서버에 씌우고 web server의 port를 443으로 변경하여 실행하였다. 근데 PM2에서 해당 서버를 실행하면 <span class="text-highlight warning">Error: listen EACCES 0.0.0.0:443</span> 다음과 같은 error가 떠서 서치해본 결과 pm2에서 http, https 포트로의 실행은 불가능했고 따라서 다시 기존처럼 서버 포트를 설정하고 nginx를 통해 http/https 요청이 서버 port로 넘어갈 수 있게 하고 pm2를 실행하였다.
{: .comment-box}

#### ecosystem file 생성

#### ecosystem file 실행

## cerbot
원래는 pm2에서 포트 443을 이용해 https 통신을 할 경우 cerbot을 통해 도메인에 대한 ssl certificate key를 받아 이를 설정하는 것 같은데 key 발급을 위해 http로 도메인에 request를 보내 인증을 진행하는 것 같았다.

하지만 나의 경우 http로 할 경우 이미 SSL 인증을 받아서 그런지 502 error가 떴고 오히려 443으로 열었을 경우 진행이 잘 되었는데 이렇게 해도 되는지 모르겠으며 왜 되는지도 모르겠다.