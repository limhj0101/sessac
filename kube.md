# Kubespray를 이용한 Production Ready Kubernetes 클러스터 배포

[Kubespray GitHub 저장소](https://github.com/kubernetes-sigs/kubespray)

> kubespray는 프로덕션 환경에 Kubernetes를 설치할 수 있는 배포방법

## 0. VM 시스템 구성

OS: Ubuntu 24.04 LTS(Noble)

| Control Plane | IP               | CPU | Memory |
|:-------------:|:----------------:|:---:|:------:|
| kube-control1 | 192.168.56.11/24 | 2   | 3072MB |

| Node          | IP               | CPU | Memory |
|:-------------:|:----------------:|:---:|:------:|
| kube-node1    | 192.168.56.21/24 | 2   | 2560MB |
| kube-node2    | 192.168.56.22/24 | 2   | 2560MB |
| kube-node3    | 192.168.56.23/24 | 2   | 2560MB |

## 1. Requirements

- Ansible, python-netaddr
- Jinja
- 인터넷 연결(도커 이미지 가져오기)
- IPv4 포워딩
- SSH 키 복사
- 배포 중 문제가 발생하지 않도록 방화벽 비활성
- 적절한 권한 상승(non-root 사용자인 경우, passwordless sudo 설정)

- Control Plane
  - Memory: 1500MB
- Node
  - Memory: 1024MB

### 1.1 기본 필수 패키지 설치

```bash
sudo apt update  
sudo apt install -y python3 python3-pip python3-venv git
```

### 1.2 SSH 키 생성

```bash
ssh-keygen -t ed25519 -f ~/.ssh/id_ed25519 -N ''
```

### 1.3 SSH 키 복사

```bash
ssh-copy-id -i ~/.ssh/id_ed25519.pub vagrant@192.168.56.11
ssh-copy-id -i ~/.ssh/id_ed25519.pub vagrant@192.168.56.21
ssh-copy-id -i ~/.ssh/id_ed25519.pub vagrant@192.168.56.22
ssh-copy-id -i ~/.ssh/id_ed25519.pub vagrant@192.168.56.23
```
> vagrant 이미지의 기본 사용자 vagrant, 기본 패스워드 vagrant

## 2. Kubespray 배포

### 2.1 kubespray Git 저장소 클론

```bash
cd ~
```

```bash
git clone --branch release-2.27 https://github.com/kubernetes-sigs/kubespray.git  
```

### 2.2 Python 가상환경 생성

```bash
python3 -m venv ~/kubespray
```

### 2.3 Python 가상환경 활성화

```bash
source ~/kubespray/bin/activate
```

### 2.4 디렉토리 변경

```bash
cd ~/kubespray
```

### 2.5 requirements.txt 파일에서 의존성 확인 및 설치

```bash
pip install -U -r requirements.txt  
```

### 2.6 Ansible 인벤토리 준비

```bash
cp -rfp inventory/sample inventory/mycluster
```

### 2.7 인벤토리 수정

`inventory/mycluster/inventory.ini`

```ini
[kube_control_plane]
kube-control1 ansible_host=192.168.56.11 ip=192.168.56.11

[etcd:children]
kube_control_plane

[kube_node]
kube-node1    ansible_host=192.168.56.21 ip=192.168.56.21
kube-node2    ansible_host=192.168.56.22 ip=192.168.56.22
kube-node3    ansible_host=192.168.56.23 ip=192.168.56.23
```

### 2.8 파라미터 확인 및 변경

`inventory/mycluster/group_vars/k8s_cluster/addons.yml`

```yaml
...
helm_enabled: true
...
metrics_server_enabled: true
...
ingress_nginx_enabled: true
...
metallb_enabled: true
...
metallb_config:
  address_pools:
    primary:
      ip_range:
        - 192.168.56.200-192.168.56.209
      auto_assign: true
  layer2:
    - primary
...
```

`inventory/mycluster/group_vars/k8s_cluster/k8s-cluster.yml`

```yaml
...
kube_proxy_strict_arp: true
...
kube_encrypt_secret_data: true
kube_encryption_algorithm: "aesgcm"
kube_encryption_resources: [ secrets ]
...
```

### 2.9 Ansible 통신 가능 확인

```bash
ansible all -i inventory/mycluster/inventory.ini -m ping
```

```
# 출력 결과 예시
kube-control1 | SUCCESS => {
    "changed": false,
    "ping": "pong"
}
kube-node1 | SUCCESS => {
    "changed": false,
    "ping": "pong"
}
kube-node3 | SUCCESS => {
    "changed": false,
    "ping": "pong"
}
kube-node2 | SUCCESS => {
    "changed": false,
    "ping": "pong"
}
```

### 2.10 플레이북 실행

```bash
ansible-playbook -i inventory/mycluster/inventory.ini --become cluster.yml 
```

> 호스트 시스템 사양, VM 개수, VM 리소스 및 네트워크 성능에 따라 15~60분 소요

### 2.11 자격증명 가져오기

```bash
mkdir ~/.kube
sudo cp /etc/kubernetes/admin.conf ~/.kube/config
sudo chown $USER:$USER ~/.kube/config
```

### 2.12 kubectl 명령 자동완성 확인

```bash
source /etc/bash_completion.d/kubectl.sh
```

```bash
kubectl [TAB][TAB]
kubectl get [TAB][TAB]
```

### 2.13 Kubernetes 클러스터 확인

```bash
kubectl get nodes
kubectl cluster-info
```

## (옵션) 3. 클러스터 관리

### 3.1 노드 추가

1. 인벤토리 파일에 노드 추가 설정
2. scale.yml 플레이북 실행

```
ansible-playbook -i inventory/mycluster/inventory.ini --become scale.yml
```

### 3.2 노드 제거

제거 하려는 노드를 node 변수에 지정하여, remove-node.yml 플레이북 실행

```
ansible-playbook -i inventory/mycluster/inventory.ini --become --extra-vars "node=kube-node4" remove-node.yml
```

> 제거하려는 노드에 SSH 연결이 안된다면, reset_nodes=false 변수를 추가하여, 노드 재설정 단계를 건너뛰어 제거한다.

### 3.3 버전 업그레이드

업그레이드하려는 버전을 kube_version 변수에 지정하여, upgrade-cluster.yml 플레이북 실행

```
ansible-playbook -i inventory/mycluster/inventory.ini --become --extra-vars "kube_version=v1.32.0" upgrade-cluster.yml 
```