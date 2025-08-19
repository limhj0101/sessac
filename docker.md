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
