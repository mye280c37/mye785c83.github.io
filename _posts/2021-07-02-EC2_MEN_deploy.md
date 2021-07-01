---
layout: post
title : AWS EC2에서 Express.js 서버 배포 및 MongoDB 설정
category: Web
tags: [AWS, EC2, Express.js, MongoDB, Node.js, nginx, pm2]
---

## Express.js, MongoDB 환경설정
<br>

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

<br><br>

## DataBase 설정
<br>

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
*이때 들여쓰기 2칸 제대로 안했다가 conf error 난...*

<br>

#### mongod service '재실행'

```
sudo service mongod restart
sudo systemctl status mongod
```

#### 서버의 MONGO URI에 해당하는 database 및 user, pwd 생성

*MONGO URI는 `.env` 파일에 설정되어 있다.*

나의 경우 admin database로 가 server database owner에 대한 role도 root user에 추가해주었다.

```
> db.updateUser("username", {roles:[{role: 'dbOwner', db: 'database name'}]})
```

##### 생성한 user 정보로 server database 접속해보기

```
mongo --port 27017 -u 사용자계정 -p '비밀번호' --authenticationDatabase '디비이름'
```

<br><br>

이렇게 database 설정까지 완료했으면 이제 Express.js 서버를 가져와 실행해준다. (database를 미리 한 이유는 안하면 서버를 먼저 가져와서 실행해봤자 database 접속 못 할거라)

<br><br>

## Express.js 실행
<br>

#### package.json 설치
github에 존재하는 Express.js repository를 clone 해온 후 해당 폴더로 이동해 package.json에 해당하는 패키지들을 설치한다.

```
npm install
```

여기까지 마치고 나면 해당 폴더의 하위에 `node_modules/` 폴더가 생성된 것을 확인할 수 있다.
여기까지 마치면 서버를 실행해 서버가 잘 실행되고 데이터베이스에 잘 연결되었는지 확인한다.

