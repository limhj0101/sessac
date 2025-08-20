# DOCKER
## vagrant
```
# -*- mode: ruby -*-
# vi: set ft=ruby :

# VM 설정
VM_IMAGE = "nobreak-labs/ubuntu-noble"
VM_CONFIG = {
  name: "docker",
  image: VM_IMAGE,
  cpus: 2,
  memory: 4096,
  ip: "192.168.56.101"
}

# 프로비저닝 스크립트
CHANGE_APT_REPO = <<-SCRIPT
  ARCH=$(uname -m)
  RELEASE=$(lsb_release -cs)
  if [ "$RELEASE" = "jammy" ]; then
    # Ubuntu 22.04
    if [ "$ARCH" = "x86_64" ]; then
      sed -i 's/archive.ubuntu.com\\|kr.archive.ubuntu.com\\|security.ubuntu.com/ftp.kaist.ac.kr/g' /etc/apt/sources.list
    elif [ "$ARCH" = "aarch64" ]; then
      sed -i 's/ports.ubuntu.com/ftp.kaist.ac.kr/g' /etc/apt/sources.list
    fi
  elif [ "$RELEASE" = "noble" ]; then
    # Ubuntu 24.04
    if [ "$ARCH" = "x86_64" ]; then
      sed -i 's|^URIs:.*|URIs: http://ftp.kaist.ac.kr/ubuntu/|g' /etc/apt/sources.list.d/ubuntu.sources
    elif [ "$ARCH" = "aarch64" ]; then
      sed -i 's|^URIs:.*|URIs: http://ftp.kaist.ac.kr/ubuntu-ports/|g' /etc/apt/sources.list.d/ubuntu.sources
    fi
  fi
SCRIPT

INSTALL_DOCKER = <<-SCRIPT
  # Add Docker's official GPG key:
  apt-get update
  apt-get -y install ca-certificates curl
  install -m 0755 -d /etc/apt/keyrings
  curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc
  chmod a+r /etc/apt/keyrings/docker.asc

  # Add the repository to Apt sources:
  echo \
    "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/ubuntu \
    $(. /etc/os-release && echo "${UBUNTU_CODENAME:-$VERSION_CODENAME}") stable" | \
    tee /etc/apt/sources.list.d/docker.list > /dev/null
  apt-get update
  apt-get -y install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
  usermod -aG docker vagrant
SCRIPT

# VM 템플릿
Vagrant.configure("2") do |config|
  config.vm.define VM_CONFIG[:name] do |node|
    node.vm.box = VM_CONFIG[:image]
    node.vm.provider "virtualbox" do |vb|
      vb.name = VM_CONFIG[:name]
      vb.cpus = VM_CONFIG[:cpus]
      vb.memory = VM_CONFIG[:memory]
    end
    node.vm.hostname = VM_CONFIG[:name]
    node.vm.network "private_network", ip: VM_CONFIG[:ip], nic_type: "virtio"
    node.vm.synced_folder ".", "/vagrant", disabled: true
    
    if node.vm.box.include?("ubuntu")
      node.vm.provision "shell", inline: CHANGE_APT_REPO
      node.vm.provision "shell", inline: INSTALL_DOCKER
    end
  end
end
```

https://docs.docker.com/engine/install/ubuntu/\
1,2 레포지토리 리눅스에서 실행

![alt text](image.png)

docker container prune 
\
중지되어있는 컨테이너 일괄 삭제

## 1번
```
docker image inspect ubuntu:latest | grep -A2 Cmd
    "Cmd": [
        "/bin/bash"
    ],
#1
docker container run -itd --name shell ubuntu:latest

#2
docker container run -d --name web httpd:latest

#3
docker container run --help | grep -A1 restart 
docker container run --help | grep -A1 rm

docker container run --rm hello-world:latest

#4
docker container run -itd --name shell2 httpd:latest /bin/bash

#5
docker container run --name web_always -d --restart=always httpd:latest

docker ps
sudo systemctl restart docker
docker ps
```

## 2번
```
#1 
docker container attach shell
ls
exit

#2
docker container exec -it shell2 /bin/bash
ls
exit

#3
docker container exec web ls htdocs
```

## 3번
```
docker container cp web:/usr/local/apache2/htdocs/index.html .

vi index.html
(수정 후)
docker container cp index.html web:;usr/local/apache2/htdocs/index.html
docker container exec web cat htdocs/index.html
docker container diff web
```

## 4번
```
docker container run --cpus 0.3 --memory 1G -it --name test ubuntu:latest
dd if=/dev/zero of=/dev/null &
ps -ef

(ctrl p q)
docker container stats --no-stream test
docker container inspect test | grep -i cpu
```

## 5번
```
docker container top test
docker container exec test ps -ef
--두개의 PID가 다른 이유는 docker top은 호스트 입장에서 관리, 컨테이너 내부에서 관리

docker container exec test kill -15 (내부 PID)
또는
sudo kill -15 (호스트 PID)

kill -15 옵션이 안되면 -9 로 강제종료
```


# 볼륨 실습 가이드
1. 호스트의 특정 디렉토리를 컨테이너에 연결해서 파일을 공유해보자
	ubuntu 이미지로 컨테이너 실행 시 호스트의 /etc/apt/sources.list.d/ 디렉토리를 컨테이너의 /mnt 에 연결
	컨테이너 실행 후 내부에서 git 패키지를 설치해보세요.								2시까지 다 풀고 다했으면 쉬세요~
2. 볼륨을 만들고 컨테이너 연결해서 데이터 영구저장과 공유를 해보자
	1) httpd 이미지로 컨테이너를 실행하면서 htdocs/ 디렉토리를 contents 볼륨에 읽기 전용으로 연결
	2) alpine 이미지로 컨테이너 실행 시 contents 볼륨을 /html 디렉토리에 연결 후 확인
	3) 2번 컨테이너에서 파일 내용을 편집하고 3번 컨테이너에 접속해서 확인해보기
	4) 3번 컨테이너에서 파일 편집 시도해보기
3. 컨테이너 삭제 시 데이터를 확인해보자
	1) 위 컨테이너들을 모두 삭제하고 해당 디렉토리와 컨테이너 안의 파일들을 확인
	2) httpd이미지로 new-web 이라는 새로운 컨테이너를 만들면서 contents 볼륨을 htdocs 디렉토리에 연결
네트워크 실습 가이드
1. 컨테이너 네트워크 확인 및 접속
	1) new-web 컨테이너의 IP주소를 확인하고 접속해보자.
2. 새로운 브릿지 네트워크 생성 및 테스트
	1) my-net 이라는 이름의 브릿지 네트워크를 생성하고 IP대역과 게이트웨이 주소를 확인
	2) rockylinux:9 이미지로 my-net에 연결하는 client라는 이름의 컨테이너 생성
	3) httpd이미지로 my-net에 연결하는 my-web라는 컨테이너와 bridge에 연결한 basic-web 컨테이너 생성
	4) client 컨테이너에서 curl 명령어로 각각의 IP주소로 접속해보기
	5) client 컨테이너에 bridge 네트워크를 추가로 연결하고 각각의 IP주소와 이름으로 접속해보기
3. 외부에서 접속 가능한 웹서버 구성
	1) my-net 네트워크에 연결하면서 호스트의 8080포트로 접속할 수 있도록 웹서버 컨테이너 생성
	2) host 네트워크 연결해서 웹서버 컨테이너 생성 및 각각 외부(윈도우)에서 접속해보기
종합문제? (환경변수 및 볼륨 연결에 대한 부분은 도커허브에서 문서 확인)
0. 실습했던 모든 컨테이너 및 네트워크/볼륨 삭제. 이미지는 백업 후 삭제
1. wordpress 이미지와 mysql 이미지를 다운로드
2. mysql 이미지를 다음 조건에 맞게 컨테이너로 실행하기
	1) 컨테이너 이름 : database , 네트워크 : internal (브릿지) , 볼륨 : mysql-vol , 항상 자동재시작
	2) root패스워드 : 123 , 사용자 : wp-user , 패스워드 : qwer , 데이터베이스 : wp-db
3. wordpress 이미지로 컨테이너 실행하기
	1) 컨테이너 이름 : webserver , 네트워크 : external (브릿지) , 볼륨 : wp-vol , 포트포워딩 : 80 -> 80
	2) 환경변수를 사용해서 위 데이터베이스 컨테이너와 연결설정
	3) 컨테이너의 cpu 사용량 50% , 메모리 2G 로 제한설정.
	3) 외부에서 접속확인 후 internal 네트워크도 추가로 연결하고 다시 접속해보기
4. 확인작업
	1) 각 컨테이너들의 세부정보 확인 (실행하는 프로그램, 환경변수, 볼륨, 네트워크, 리소스)
	2) webserver의 리소스 사용량 및 프로세스 목록 확인
	3) 각 컨테이너의 로그 확인

```
docker run -it --name test1 /etc/apt/sources.list.d/:/mnt --rm ubuntu

docker run -v contents:/usr/local/apache2/htdocs:ro --name web -d httpd
(확인법)
docker exec -it web bash

docker run -v contents:/html --name os -it alpine

docker rm web os 

docker run -v contents:/usr/local/apach2/htdocs --name new-web -d httpd

docker inspect new-web



  421  docker network create my-net
  422  docker network inspect
  423  docker network inspect my-net
  424  docker network inspect ip
  425  docker run -it --name client rockylinux:9
  426  docker rm client
  427  docker run -itd --name client --network my-net rockylinux:9
  428  docker ps -a
  429  docker run -d --network my-net --name my-web httpd:latest
  430  docker run -d --name basic-web httpd:latest
  431  docker attach client
  432  docker inspect basic-web | prep IPAdd
  433  docker inspect basic-web | grep IPAdd
  434  docker inspect my-web | grep IPAdd
  435  docker attach client
  436  docker network connect bridge client
  437  docker attach client

   451  docker rm -f $(docker ps -aq)
  452  docker network prune
  453  docker volume prune -a
  454  docker image prune
  455  docker image pull wordpress
  456  docker image ls
  457  docker image pull mysql
  458  docker image inspect
  459  docker image inspect -a
  460  docker image inspect mysql:latest
  461  docker network create internal
  462  docker volume create mysql-vol
  463  docker run -v mysql-vol:/var/lib/mysql --network interneal --name database --restart always -e MYSQL_ROOT_PASSWORD=123 -e MYSQL_DATABASE=wp-db -e MYSQL_USER=wp-user -e MYSQL_PASSWORD=qwer -d mysql:latest
  464  docker run -v mysql-vol:/var/lib/mysql --network internal --name database --restart always -e MYSQL_ROOT_PASSWORD=123 -e MYSQL_DATABASE=wp-db -e MYSQL_USER=wp-user -e MYSQL_PASSWORD=qwer -d mysql:latest
  465  docker ps -a
  466  docker rm database
  467  docker run -v mysql-vol:/var/lib/mysql --network internal --name database --restart always -e MYSQL_ROOT_PASSWORD=123 -e MYSQL_DATABASE=wp-db -e MYSQL_USER=wp-user -e MYSQL_PASSWORD=qwer -d mysql:latest
  468  docker image inspect wordpress:latest
  469  docker network create external
  470  docker volume create wp-vol
  471  docker run -d --name webserver --network external -v wp-vol:/var/www/html -p 80:80 -e WORDPRESS_DB_HOST=database -e WORDPRESS_DB_USER=wp-user -e WORDPRESS_DB_PASSWORD=qwer -e WORDPRESS_DB_NAME=wp-db --cpus 0.5 --memory 2g wordpress:latest
  472  docker network connect internal webserver
  473  docker inspect webserver
  474  docker stats --no-stream webserver
  475  docker top webserver
  476  docker log webserver
  477  docker logs webserver