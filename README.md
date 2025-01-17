# Basic-Docker-Container-and-Kubernetes
Docker와 쿠버네티스 학습 과정과 실습 내용을 정리한 저장소 | Learning and practicing Docker and Kubernetes from basics to advanced concepts

---

## Day 1: Docker 기초 개념

### 1. 도커(Docker)란?
- **컨테이너 런타임(Container Runtime)** 또는 **컨테이너 엔진(Container Engine)**을 의미
- 격리된 환경(컨테이너)에서 애플리케이션을 실행할 수 있도록 도와주는 소프트웨어

### 2. 도커를 왜 쓰는가?
1. **하드웨어 효율적 사용**  
   - 물리적 하드웨어를 논리적으로 분할해 사용 (예: 디스크 파티션, 가상화, 컨테이너 등)  
   - 하이퍼바이저(VM)보다 경량화된 방식으로 시스템 자원을 최대한 활용

2. **의존성 문제 해결**  
   - OS 버전, 라이브러리 충돌 등을 컨테이너 단위로 격리  
   - 애플리케이션을 “밀키트”처럼 묶어서 어디서든지 쉽게 실행

### 3. 컨테이너(밀키트)와 격리(Isolation)
- 컨테이너: “격리된 환경의 애플리케이션”. OS/하드웨어에 독립적으로 동작할 수 있음  
- 격리: 서로 다른 컨테이너가 충돌 없이 독립적으로 실행되도록 하는 핵심 개념

### 4. 컨테이너 엔진(런타임)
- 컨테이너를 실제로 실행하고 멈추는 주체  
- **Docker**가 대표적인 엔진(컨테이너를 조리하는 ‘주방’)  

### 5. 컨테이너 이미지(Container Image)
- 프로그램(애플리케이션)이 실행될 수 있도록 필요한 파일을 모두 모은 패키지  
- 예: `docker.hub` 등에서 다운로드 가능  
- 여러 ‘레이어(Layer)’로 이루어져, 공통 레이어를 공유함으로써 용량 절약

#### 5.1 이미지 네이밍 규칙
- 보통 4가지 요소로 구성됨:  
  **레지스트리/레포지토리/이름:태그**  
- 예) `docker.io/library/nginx:1.12`  
  - `docker.io` (레지스트리)  
  - `library` (레포지토리, 생략 시 보통 공식 오피셜 이미지)  
  - `nginx` (이미지 이름)  
  - `1.12` (태그, 버전)

### 6. 도커 명령어 구조
```
docker [대상] [세부명령어] [옵션]
```
- **대상**: `image`, `container`, `network`, `volume` 등  
- **세부명령어**: `run`, `ls`, `create`, `rm`, `inspect` 등  
- **run** 은 “실행”을 의미하므로, 주로 `container` 대상에 사용

| 대상        | 주요 명령어                               |
|-------------|-------------------------------------------|
| container   | run, start, stop, rm, ls, inspect 등      |
| image       | pull, push, rm, ls, build 등              |
| volume      | create, rm, ls                            |
| network     | create, rm, ls                            |

> `docker run`은 사실상 **`docker container create` + `docker container start`** 역할을 한 번에 수행  

### 7. Docker의 단점 → Kubernetes 필요성
- 물리적(또는 가상) 머신 1대에서 실행 가능한 컨테이너 수는 한계가 있음  
- 여러 대의 서버에 컨테이너가 분산되면 **관리**가 복잡해짐 → 오케스트레이션 툴(예: **쿠버네틱스**) 필요

### 8. 도커 설치 & 실습 준비
- **원격 접속** (예: PuTTY) 등을 사용해 서버에 접속 → 도커 설치  
- 컨테이너 실행 후 명령어로 확인/중지/삭제 실습

### 9. 컨테이너 실행 예시

1. 실행 중인 컨테이너 확인  
   ```bash
   docker container ps  # or 'docker ps'
   ```
   - 기본적으로 **실행 중(Running)** 상태인 컨테이너만 표시  
   - `-a` 옵션을 사용하면 **중지된 컨테이너**까지 모두 표시

2. 컨테이너 실행: `docker run`  
   ```bash
   docker run -d --name web2 -p 8282:80 nginx:1.12
   ```
   - `-d`: 백그라운드(Detached) 모드  
   - `--name web2`: 컨테이너 이름 지정  
   - `-p 8282:80`: 호스트(8282) ↔ 컨테이너(80) 포트 연결 (포트포워딩)  

3. 컨테이너 중지/재시작  
   ```bash
   docker stop web2
   docker start web2
   ```

---

## Day 1 핵심 요약

- **도커** = 컨테이너 런타임 (가벼운 가상화)  
- **장점**: 하드웨어 자원 효율 ↑, 의존성 문제 ↓  
- **컨테이너**는 격리된 환경, **이미지**는 실행에 필요한 패키지  
- **명령어**: `docker run`, `docker ps`, `docker stop`, `docker start` 등  
- **단점**: 여러 대 서버로 확장 시 관리 어려움 → 쿠버네틱스(K8s)

---
---

# Day 2: Intermediate Docker Concepts

이번 시간에는 컨테이너의 **실행 상태**, **환경 변수**, **스토리지(볼륨/바인드 마운트/tmpfs)**, 그리고 **네트워크**에 대해 학습했다. 마지막에 **WordPress + MySQL** 예제로 실습을 진행하였다.

<br/>

## 1. 컨테이너 실행 상태 (Lifecycle)

1. **컨테이너 종료 상태**  
   - 가상 머신이 종료되거나, 컨테이너를 중지(`docker stop`)하면  
   - `docker container ps -a` 명령어에서 해당 컨테이너 상태가 `Exited`로 표시됨

2. **재시작**  
   - 종료된 컨테이너를 다시 실행:  
     ```bash
     docker start <컨테이너이름>
     ```
   - 완전히 제거(`rm`) 전까지 이미 만들어진 컨테이너는 재활용 가능

3. **컨테이너 제거**  
   - 기본적으로 **`Exited`**(또는 `Created`) 상태여야 `docker rm`으로 제거 가능  
   - 실행 중인 컨테이너를 강제로 지우려면:  
     ```bash
     docker rm -f <컨테이너이름>
     ```
     (`-f` = `--force`)

<br/>

## 2. 환경 변수 & MySQL 컨테이너

1. **환경 변수(ENV) 설정**  
   - 컨테이너 실행 시 `-e` 또는 `--env-file` 옵션으로 환경 변수를 주입  
   - 예) MySQL 이미지는 `MYSQL_ROOT_PASSWORD` 등이 필수로 설정되어야 종료되지 않음
   ```bash
   docker run -d --name mydb3 \
     -e MYSQL_ROOT_PASSWORD=123 \
     mysql
   ```
   - 환경 변수가 여러 개라면 **텍스트 파일**에 정리 후 `--env-file`로 전달할 수 있음

2. **MySQL 컨테이너 예시**  
   - 환경 변수를 지정하지 않으면 컨테이너가 실행 즉시 종료될 수 있음 (필요한 설정값이 없으므로)

<br/>

## 3. `docker exec`: 컨테이너 내부 명령 실행

1. **단일 명령 실행**  
   ```bash
   docker exec mydb3 date
   ```
   - `mydb3` 컨테이너 안에서 `date` 명령을 실행

2. **인터랙티브 쉘 접속**  
   ```bash
   docker exec -it mydb3 bash
   ```
   - 컨테이너 내부의 쉘로 진입 (`-i` 인터랙티브, `-t` TTY)  
   - `env` 명령어로 환경 변수 확인 가능  
   - `exit`로 빠져나오기

3. **Nginx 컨테이너에 웹페이지 수정**  
   ```bash
   docker run -d --name www -p 8383:80 nginx:1.12
   docker exec -it www bash
   cd /usr/share/nginx/html
   ```
   - Nginx 기본 이미지에는 `vi`나 `nano` 등의 편집기가 없음  
   - 임시로 `echo "so cold" > index.html` 같이 `echo` 명령으로 파일에 내용을 넣을 수 있음

4. **컨테이너 외부에서 파일 복사**  
   ```bash
   docker cp index.html www:/usr/share/nginx/html/
   ```
   - 로컬에서 만든 파일을 컨테이너 내부로 넣을 수 있음 (`cp` = copy)

<br/>

## 4. Docker 스토리지 (Volume / Bind Mount / tmpfs)

### 4.1 컨테이너 삭제 시 데이터 손실 문제
- 컨테이너는 **1회용** 개념이 강함
- 컨테이너를 `docker rm -f`로 삭제하면 내부 데이터도 함께 사라짐
- **DB 데이터**, **사용자 업로드 파일** 등은 별도 스토리지에 보관해야 함

### 4.2 스토리지 종류
1. **바인드 마운트(Bind Mount)**  
   - 호스트(물리/원격) 디렉터리를 컨테이너에 연결  
   - 예)  
     ```bash
     mkdir /lunch
     docker run -d --name myweb1 -p 9090:80 \
       -v /lunch:/usr/share/nginx/html \
       nginx:1.12
     ```
   - 호스트 `/lunch` 디렉터리 ↔ 컨테이너 `/usr/share/nginx/html`를 연결  
   - 컨테이너 삭제 후 다시 생성해도, **호스트 디렉터리**에 남아있는 데이터는 재활용 가능  
   - **단점**: 디렉터리가 많아지면 복잡해져 관리가 어려울 수 있음

2. **볼륨(Volume)**  
   - 도커가 직접 관리하는 스토리지  
   - **`docker volume create`** 로 생성  
   - 예)  
     ```bash
     docker volume create --name webpage
     docker run -d --name myweb2 -p 9292:80 \
       -v webpage:/usr/share/nginx/html \
       nginx:1.12
     ```
   - 데이터는 `/var/lib/docker/volumes/` 아래 자동 관리됨  
   - 컨테이너 삭제(`rm -f`)해도 볼륨 데이터 유지  
   - 필요 없을 때 `docker volume rm` 또는 `docker volume prune`로 정리

3. **tmpfs**  
   - **메모리(RAM)** 기반 임시 스토리지  
   - 재부팅이나 컨테이너 종료 시 데이터 날아감(휘발성)  
   - 예)  
     ```bash
     docker run -it --name voltest --tmpfs /abc123 centos
     ```
   - `/abc123` 디렉터리는 RAM에만 존재 (속도는 빠르지만 영구성 없음)

<br/>

### 4.3 스토리지 연습 요약

- **바인드 마운트**: 관리 편의성 낮음(폴더 직접 지정), 그러나 설정이 간단  
- **볼륨**: 도커 관리 스토리지, 많이 사용되는 방식  
- **tmpfs**: 초고속/임시 용도

<br/>

## 5. Docker 네트워크

1. **docker0 브리지**  
   - 도커를 설치하면 자동으로 **가상 브리지**(`docker0`)가 생성됨  
   - 컨테이너를 실행할 때마다 내부적으로 `veth`(가상 랜카드)를 만들어서  
     `docker0` 브리지에 연결 → 컨테이너에 IP 할당
2. **문제점**  
   - 기본 브리지에 연결된 컨테이너 IP는 유동적임 (생성/삭제 시 IP가 바뀔 수 있음)
   - 컨테이너 이름으로 통신하고 싶다면 **사용자 정의 브리지** 사용 필요

### 5.1 사용자 정의 브리지

1. **네트워크 생성**  
   ```bash
   docker network create mynet
   ```
   - 필요 시 `--subnet 10.100.1.0/24` 같이 서브넷도 지정 가능

2. **컨테이너 네트워크 지정**  
   ```bash
   docker run -d --name web --network mynet -p 8080:80 nginx
   docker run -d --name db  --network mynet -e MYSQL_ROOT_PASSWORD=123 mysql
   ```
   - 동일 네트워크 `mynet` 안에서 **컨테이너 이름(web, db)**으로 서로 통신 가능  
   - `ping web`, `ping db` 등이 가능해짐

3. **네트워크 확인**  
   - `docker network ls`, `docker network inspect mynet`  
   - `brctl show` (Linux) → 실제 브리지 이름이 랜덤 문자열로 표시

<br/>

## 6. WordPress + MySQL 실습 (Quiz)

**목표**:  
- Volume(`dbfile`) + 사용자 정의 네트워크(`wordpress_net`) 사용  
- MySQL 컨테이너와 WordPress 컨테이너를 연결하여 웹 페이지 완성

### 6.1 실습 요구사항

1. **볼륨**: `dbfile`  
2. **네트워크**: `wordpress_net`  
3. **MySQL 컨테이너**: `mysqldb`  
   - 이미지: `mysql`  
   - 볼륨: `dbfile:/var/lib/mysql`  
   - 환경 변수  
     ```
     MYSQL_ROOT_PASSWORD=rootpass
     MYSQL_DATABASE=wpdb
     MYSQL_USER=wordpress
     MYSQL_PASSWORD=wordpress
     ```
4. **WordPress 컨테이너**: `web`
   - 이미지: `wordpress`
   - 포트: 8080 → 80  
   - 환경 변수  
     ```
     WORDPRESS_DB_HOST=mysqldb
     WORDPRESS_DB_NAME=wpdb
     WORDPRESS_DB_USER=wordpress
     WORDPRESS_DB_PASSWORD=wordpress
     ```

### 6.2 정답 예시

```bash
# 1. 볼륨 & 네트워크 생성
docker volume create dbfile
docker network create wordpress_net

# 2. MySQL 컨테이너 실행
docker run -d \
  --name mysqldb \
  --network wordpress_net \
  -v dbfile:/var/lib/mysql \
  -e MYSQL_ROOT_PASSWORD=rootpass \
  -e MYSQL_DATABASE=wpdb \
  -e MYSQL_USER=wordpress \
  -e MYSQL_PASSWORD=wordpress \
  mysql

# 3. WordPress 컨테이너 실행
docker run -d \
  --name web \
  --network wordpress_net \
  -p 8080:80 \
  -e WORDPRESS_DB_HOST=mysqldb \
  -e WORDPRESS_DB_NAME=wpdb \
  -e WORDPRESS_DB_USER=wordpress \
  -e WORDPRESS_DB_PASSWORD=wordpress \
  wordpress
```

- 브라우저로 `http://<서버 IP>:8080` 접속 → WordPress 초기 설정 화면  
- DB 연결 정보가 올바르면 정상 설치 가능

<br/>

## 7. 기타 명령어 정리

1. **모든 컨테이너 삭제**  
   ```bash
   docker rm -f $(docker ps -aq)
   ```
2. **볼륨 확인 / 삭제**  
   ```bash
   docker volume ls
   docker volume rm <볼륨이름>
   docker volume prune   # 사용되지 않는 모든 볼륨 제거
   ```
3. **이미지 삭제**  
   ```bash
   docker image ls
   docker image rm nginx:1.12
   docker image prune -a  # 사용되지 않는 모든 이미지 삭제
   ```

---

# Day 2 요약

- **컨테이너 상태**: `run` → `Exited` → `start`/`rm`  
- **환경 변수**: `-e`, `--env-file`로 설정 (MySQL 등 필수 ENV 필요)  
- **docker exec**: 컨테이너 내부 명령 실행/쉘 접속  
- **스토리지**:  
  - 바인드 마운트(`/호스트폴더:/컨테이너폴더`)  
  - 볼륨(`docker volume create`)  
  - tmpfs(RAM 기반 임시 스토리지)  
- **네트워크**:  
  - 기본 브리지(`docker0`) vs 사용자 정의 브리지  
  - 컨테이너 이름으로 통신하려면 사용자 정의 브리지 필요  
- **WordPress + MySQL** 실습으로 Volume + Network 연습

---
---

# Day 3: Kubernetes 설치 & 기본 개념

## 1. Kubernetes란?

- **쿠버네틱스(Kubernetes)**: 컨테이너(‘격리된 애플리케이션’)를 **대규모로 배포**하고 **자동 관리**(오케스트레이션)해 주는 오픈소스 소프트웨어
- 쿠버네틱스 = “이 많은 컨테이너들을 효율적으로 배포/업데이트/확장하자!”
- **오픈시프트(OpenShift)**:  
  - 레드햇(Red Hat)이 제공하는 **쿠버네틱스 기반 상용 솔루션**  
  - 오픈소스 버전보다 유지보수/버전관리 측면에서 편리해 **기업 환경**에서 많이 활용  

<br/>

## 2. 실습 환경 개요

- **4대의 서버**(VM)로 쿠버네틱스 클러스터 구성
  - 1대는 **마스터 노드**(master.test.com)
  - 3대는 **워커 노드**(node1, node2, node3)
- 마치 여러 컴퓨터를 하나의 커다란 시스템(클러스터)처럼 동작시킴

<br/>

## 3. 사전 준비 & 설정

### 3.1 호스트네임 설정
```bash
# 마스터 노드에서 예시
hostnamectl set-hostname master.test.com
hostname  # 변경 확인
```
- FQDN(예: `master.test.com`) → `컴퓨터명.도메인명`

### 3.2 Swap 비활성화
쿠버네틱스는 스왑을 끈 상태로 동작하는 것을 권장(예측 가능한 리소스 관리).
```bash
swapon -s         # 현재 스왑 확인
swapoff -a        # 스왑 비활성화
swapon -s         # 다시 확인(없어야 함)

vi /etc/fstab
# /swap.img none swap sw 0 0  ← 주석 처리(#)로 비활성
```

### 3.3 리눅스 커널 설정
쿠버네틱스 네트워크를 위해 커널 모듈 & 네트워크 패킷 포워딩 설정.

1. **커널 모듈 로드 설정**  
   ```bash
   # /etc/modules-load.d/k8s.conf (새 파일)
   br_netfilter
   ```

2. **IP 포워딩 & 방화벽 관련 설정**  
   ```bash
   # /etc/sysctl.d/k8s.conf (새 파일)
   net.bridge.bridge-nf-call-iptables = 1
   net.bridge.bridge-nf-call-ip6tables = 1
   net.ipv4.ip_forward = 1
   ```
   ```bash
   systemctl --system  # 설정 재적용 후 q로 종료
   ```

<br/>

## 4. 쿠버네티스 설치 (마스터 노드)

공식 문서 참고:  
[Install kubeadm, kubelet, kubectl](https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/install-kubeadm/)

### 4.1 패키지 설치
```bash
apt-get update
apt-get install -y apt-transport-https ca-certificates curl gpg

curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.31/deb/Release.key \
  | sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg

echo 'deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] \
https://pkgs.k8s.io/core:/stable:/v1.31/deb/ /' \
| sudo tee /etc/apt/sources.list.d/kubernetes.list

apt-get update
apt-get install -y kubelet kubeadm kubectl
apt-mark hold kubelet kubeadm kubectl

systemctl enable --now kubelet
```
- 버전 확인:
  ```bash
  kubectl version
  kubeadm version
  kubelet --version
  ```

### 4.2 Docker 설정 수정
쿠버네티스 권장 방식인 `systemd` cgroup 사용 및 `overlay2` 스토리지 드라이버 설정.

```bash
cat > /etc/docker/daemon.json <<EOF
{
  "exec-opts": ["native.cgroupdriver=systemd"],
  "log-driver": "json-file",
  "log-opts": {
    "max-size": "100m"
  },
  "storage-driver": "overlay2"
}
EOF

systemctl daemon-reload
systemctl restart docker
systemctl enable docker
```

### 4.3 /etc/containerd/config.toml 백업
```bash
ls /etc/containerd/config.toml
mv /etc/containerd/config.toml /tmp
```
- (필요 시) 컨테이너런타임 구성에 맞춰 추가 설정 가능

<br/>

## 5. 모든 노드 /etc/hosts 설정

```bash
vi /etc/hosts
192.168.11.10   master.test.com
192.168.11.11   node1.test.com
192.168.11.12   node2.test.com
192.168.11.13   node3.test.com
```
- 마스터/노드 간 **도메인명**으로 통신 가능하도록

<br/>

## 6. 노드 VM 클론 & 개별 설정

- **node1** VM 클론 생성 후 부팅
  - `hostnamectl set-hostname node1.test.com`
  - IP 수정 (`/etc/netplan/00-installer-config.yaml` 등)
  - `netplan apply`  
- **node2**, **node3**도 같은 방식으로 진행  
- 마스터에서 `ping node1.test.com` 등으로 통신 테스트

<br/>

## 7. 마스터 노드 초기화 (kubeadm init)

```bash
kubeadm init \
  --apiserver-advertise-address=192.168.11.10 \
  --pod-network-cidr=10.244.0.0/16
```
- 실행 후 출력되는 `kubeadm join ...` 명령어를 **복사**해둠 (나중에 노드에서 join 명령에 사용)

### 7.1 Kubeconfig 설정
```bash
mkdir -p $HOME/.kube
cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
chown $(id -u):$(id -g) $HOME/.kube/config

# 혹은
export KUBECONFIG=/etc/kubernetes/admin.conf
vi ~/.bashrc
# 아래 두 줄 추가
export KUBECONFIG=/etc/kubernetes/admin.conf
source <(kubectl completion bash)
```
- 이 설정으로 마스터 노드에서 `kubectl` 명령 사용 가능

### 7.2 네트워크 플러그인 설치 (Flannel 예시)
```bash
kubectl apply -f https://github.com/flannel-io/flannel/releases/latest/download/kube-flannel.yml
```
- `kubectl get pod --namespace kube-system` → 플래널(Flannel) 관련 파드가 정상 실행되는지 확인

<br/>

## 8. 워커 노드 Join

1. **마스터에서 ssh-keygen** & **ssh-copy-id**로 노드에 무비밀번호 접속 설정(옵션)
   ```bash
   ssh-copy-id node1.test.com
   ssh-copy-id node2.test.com
   ssh-copy-id node3.test.com
   ```
2. **각 노드에서 Join**  
   ```bash
   ssh node1.test.com
   kubeadm join 192.168.11.10:6443 --token <토큰> \
     --discovery-token-ca-cert-hash sha256:<해시값>
   exit
   ```
   - node2, node3도 동일 진행
3. **노드 확인**  
   ```bash
   kubectl get nodes
   ```
   - READY 상태가 되면 클러스터 가입 완료

### 8.1 토큰 만료 시 재발행
```bash
kubeadm token list
kubeadm token delete <기존토큰>
kubeadm token create --print-join-command
```
- 새 토큰으로 join 명령어 다시 실행

<br/>

## 9. 스냅샷 & 클러스터 확인

- 모두 설정이 끝났다면 **VM 스냅샷** 촬영(복구용)
- `kubectl cluster-info`, `kubectl get nodes -o wide` 등으로 상태 확인

<br/>

## 10. 기본 쿠버네틱스 리소스 실습

### 10.1 Pod 생성
```bash
kubectl run web --image=nginx:1.12
kubectl get pod
kubectl get pod -o wide
```
- `curl http://<Pod IP>` 방식으로 HTTP 응답 테스트 (같은 네트워크에서 접근 가능)

### 10.2 Pod 수정 (예: nginx → httpd)
```bash
kubectl edit pod web
# 이미지 부분을 httpd로 변경
```
- YAML 문법 주의(들여쓰기/스페이스 등)

### 10.3 YAML 매니페스트 파일 작성
```yaml
# 1.yaml
apiVersion: v1
kind: Pod
metadata:
  name: web2
spec:
  containers:
  - name: web2
    image: httpd
```
```bash
kubectl apply -f 1.yaml
kubectl get pod
```
- Pod 상태(Pending, ContainerCreating, Running 등)를 확인

### 10.4 Pod 삭제
```bash
kubectl delete -f 1.yaml
kubectl delete pod web
```

<br/>

## 11. 쿠버네틱스 구조 개념

1. **마스터 노드**  
   - **API 서버**: 클라이언트(kubectl 등) 명령을 받아들이는 핵심  
   - **etcd**: 클러스터 상태/메타데이터 저장  
   - **스케쥴러**: 새 파드를 어느 노드에 배치할지 결정  
   - **컨트롤러 매니저**: 노드·파드 상태 모니터링 & 자동 복구(아프면 다른 노드로 재할당)

2. **워커 노드**  
   - **kubelet**: 파드 상태를 모니터링 & 실행 관리  
   - **kube-proxy**: 파드 접근(네트워크) 설정, 로드밸런싱 역할  
   - **컨테이너 런타임**: 실제 파드를 실행하는 소프트웨어(도커 등)  
   - **Pod**: 컨테이너를 담는 최소 배포 단위(‘화분’에 식물을 심듯)

3. **추가 개념**  
   - **ReplicaSet**, **Deployment**: 파드를 여러 개 복제, 버전 업데이트 등(파드의 상위 개념)  
   - **Service**: 파드에 안정적인 접근(클러스터 내부/외부) 제공  
   - **Ingress**: 외부 트래픽을 파드로 라우팅하는 진입점  
   - **Volume/Storage**: 파드가 재시작되거나 재배포돼도 데이터 보존을 위해 외부 스토리지 연결  
   - **Namespace**: 클러스터 리소스를 논리적으로 구분/격리하는 단위 (예: `kube-system`, `default` 등)

<br/>

## Day 3 요약

1. **쿠버네틱스**: 대규모 컨테이너 관리/오케스트레이션 도구  
2. **환경 설정**: 스왑 비활성화, 커널 모듈 설정, Docker cgroup 등 필요  
3. **kubeadm init & join**으로 마스터/워커 노드 클러스터 구성  
4. **Flannel 등 네트워크 플러그인 설치** 필수적임임  
5. **kubectl** 명령으로 파드(Pod) 생성·확인·삭제  
   - YAML 매니페스트 파일(`apiVersion`, `kind`, `metadata`, `spec`) 중요  
6. **쿠버네틱스 아키텍처**: 마스터(API 서버, etcd, 스케쥴러 등) / 워커(kubelet, kube-proxy, 컨테이너 런타임)  

---
아래는 **Day 4** 학습 내용을 **가능한 자세하게** 정리한 자료입니다.  
메모에 있는 모든 내용(매니페스트 파일, 라벨, 파드 재시작 정책, 파일 복사, ReplicaSet, Deployment 개념 등)을 충분히 담아 재구성했습니다.  
(필요에 따라 더욱 간략히 줄이거나, 항목별로 세분화할 수 있습니다.)

---

# Day 4: 매니페스트 파일·라벨·파드 재시작 정책·ReplicaSet·Deployment

## 1. 매니페스트 파일(YAML)

1. **개념**  
   - 쿠버네티스의 자원(파드, ReplicaSet, Deployment 등)을 정의하는 **설정 파일**  
   - YAML 형식으로 작성, `kubectl apply -f <파일명>`으로 적용  
   - 작성된 매니페스트는 API 서버 → etcd에 저장되어, **관리**되고 **상태**를 유지

2. **매니페스트 예시 & 문법 주의**  
   - 들여쓰기는 탭이 아닌 **스페이스**  
   - **키: 값** 형태  
   - 여러 값이 필요한 경우 `-` (하이픈)으로 리스트 작성  
   - **잘못된 예시** (메모 내용 중 ‘종속 관계가 어긋난’ YAML):
     ```yaml
     taxonomy:
       kingdom: 생물
         phylum: 동물
           class: 포유류
             species: 소나무    # X (식물이 포유류 아래에 들어갈 수 없음)
             species: 고양이    # O (동물/포유류에 해당)
     ```
     이런 식으로 로직이 잘못되면 YAML 구문 에러 또는 의도치 않은 설정이 되므로 주의

3. **명령어 vs 매니페스트 파일**  
   - 쿠버네티스 리소스를 만드는 방법은 명령어(`kubectl run`, `kubectl create`)도 있지만,  
   - **매니페스트(YAML) 작성**이 더욱 **체계적**이고 버전 관리에도 유리

<br/>

## 2. 라벨(Label)과 라벨 제거

1. **라벨**이란?  
   - 쿠버네티스 리소스(파드, 노드 등)를 **식별**하고 **그룹화**하기 위한 **Key=Value** 쌍  
   - 예: `state=red`, `version=dev`, `region=us-west`  
   - 매니페스트 파일의 `metadata.labels` 영역이나, `kubectl label` 명령어로 설정 가능

2. **라벨 추가/삭제**  
   - 예) 특정 파드에 `state=red` 라벨 추가:
     ```bash
     kubectl label pod web5 state=red
     ```
   - 라벨 제거(`state-`):
     ```bash
     kubectl label pod web5 state-
     kubectl get po --show-labels
     ```
   - 노드 라벨도 유사하게:
     ```bash
     kubectl label nodes node1.test.com region-
     kubectl get nodes --show-labels
     ```

3. **리소스 삭제**  
   - `kubectl delete pod --all` → 현재 네임스페이스의 모든 파드 제거  
   - 삭제 후 `kubectl get po`로 확인  
   - 테스트 시 `watch -n 1 kubectl get pod`(실시간 모니터링) 적극 활용

<br/>

## 3. 파드(Pod)의 재시작 정책 (restartPolicy)

1. **쿠버네티스 파드의 기본 정책**: `Always`  
   - 파드 내 컨테이너가 종료되면 **항상** 재시작  
   - 쿠버네티스는 **서비스 가용성** 유지 목적

2. **옵션**  
   - `OnFailure`: 비정상 종료(Exit code ≠ 0) 시에만 재시작. 정상 종료면 재시작 안 함 → 주로 배치 작업(성공 후 종료)  
   - `Never`: 어떤 상황에서도 재시작하지 않음 → 단 한 번만 실행해야 하는 작업

3. **Docker vs Kubernetes**  
   - Docker 기본 옵션: `--restart=no`(자동 재시작 안 함)  
   - Kubernetes 기본값: `restartPolicy=Always`  
   - → 쿠버네티스는 **장기 운영**(서비스 지속성)을 목표, 도커는 **개별 컨테이너 실행** 초점

4. **매니페스트 작성 예**  
   ```yaml
   apiVersion: v1
   kind: Pod
   metadata:
     name: web5
     labels:
       state: red
       version: devlope
   spec:
     # restartPolicy: OnFailure  # (기본값은 Always)
     containers:
     - name: www
       image: nginx:1.12
       command:
         - echo
         - "i am tired"
   ```
   - `kubectl apply -f 3.yaml` → 파드 생성  
   - `kubectl delete -f 3.yaml` → 삭제 후 재적용  
   - `restartPolicy`를 제거하거나 주석 처리하면 자동으로 `Always`가 적용됨

5. **kubectl explain**  
   - 명령어가 기억 안 날 때 유용  
   - 예) `kubectl explain pod.spec.restartPolicy`  
   - 파드 정의에서 각 필드의 설명/타입 등을 보여줌

<br/>

## 4. 파드 내부 명령 실행 & 파일 복사

1. **컨테이너 내부 명령 실행**: `kubectl exec`  
   - 예)  
     ```bash
     kubectl exec db -- ls /etc
     kubectl exec db -- env
     kubectl exec db -- ifconfig
     ```
   - `kubectl exec <pod명> -- <명령어>`

2. **파일 복사**: `kubectl cp`  
   - 구문: `kubectl cp <소스> <목적>`  
   - 예)  
     - **파드 → 로컬**: `kubectl cp db:/etc/xattr.conf x.conf`  
     - **로컬 → 파드**: `kubectl cp 4.yaml db:/etc`  
   - “(앞에 것)을 (뒤에 것)으로 복사”로 이해  
   - 파드 내부를 수정/확인할 때 간단히 활용 가능

3. **환경 변수 설정 예**  
   ```yaml
   apiVersion: v1
   kind: Pod
   metadata:
     name: db
   spec:
     containers:
     - name: db
       image: mysql
       env:
         - name: MYSQL_ROOT_PASSWORD
           value: donga
         - name: USERDB
           value: lunch
         - name: ConnectUser
           value: linuzle
   ```
   - `kubectl apply -f 4.yaml`  
   - `kubectl exec db -- env`로 환경 변수 확인 가능

<br/>

## 5. 간단 실습: 웹페이지 변경

1. **문제 시나리오**  
   - `www`라는 이름의 파드 생성 (이미지: `nginx:1.12`)  
   - 호스트(마스터 노드 등)에 `index.html` 파일 생성  
   - 해당 파일을 파드 내부(`/usr/share/nginx/html`)로 복사  
   - 파드 IP로 curl 확인

2. **실습 과정**  
   - ① 파드 생성  
     ```bash
     kubectl run www --image=nginx:1.12
     ```
   - ② `index.html` 작성  
     ```html
     <!DOCTYPE html>
     <html lang="en">
     <body>
       Hello k8s
     </body>
     </html>
     ```
   - ③ 파일 복사  
     ```bash
     kubectl cp index.html www:/usr/share/nginx/html
     ```
   - ④ 파드 IP 확인  
     ```bash
     kubectl get pod -o wide
     # EX) 10.244.x.y
     curl http://10.244.x.y
     # 결과: Hello k8s
     ```

3. **사이드카 패턴** (간단 개념)  
   - “일반적으로 파드 하나에는 컨테이너 하나”가 많지만,  
   - 서로 밀접하게 협력하는 “유사 컨테이너”가 있으면 한 파드 안에 여러 컨테이너를 두는 경우(사이드카)  
   - 예: 로그 수집 컨테이너와 실제 앱 컨테이너가 같은 파드에 들어가 협력

<br/>

## 6. ReplicaSet

### 6.1 문제 상황

- 만약 1000개의 파드를 생성하려면?  
  - `kubectl run pod1 --image=nginx` … `pod1000`처럼 명령어 1000번?  
  - 매니페스트 1000개 작성?  
  - **매우 비효율적** → 해결책 = **ReplicaSet**

### 6.2 ReplicaSet 개념

- “동일한 파드를 **N개** 유지”하는 쿠버네티스 리소스  
- `replicas:` 값 설정 → 필요한 파드 수 유지  
- **라벨 셀렉터(selector)**로 “어떤 파드를 관리할지” 지정

#### 예시 매니페스트
```yaml
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: myrs
spec:
  replicas: 3
  selector:
    matchLabels:
      aaa: web
  template:
    metadata:
      labels:
        aaa: web
    spec:
      containers:
      - name: www
        image: nginx:1.12
```
- `replicas: 3` → 3개 파드를 유지  
- `matchLabels: {aaa: web}` & `template`에 같은 라벨 설정 → 새로 생성되는 파드는 자동으로 `aaa=web` 라벨이 붙고, ReplicaSet이 이를 관리

### 6.3 주요 동작

1. `kubectl apply -f rs.yaml` → `kubectl get rs` & `kubectl get po -o wide`  
2. 파드 개수 조정  
   - `kubectl scale rs myrs --replicas=7`  
   - 파드가 자동으로 7개 생성, 스케줄러가 노드에 배치  
   - `kubectl scale rs myrs --replicas=2` → 2개만 남기고 나머지는 삭제
3. 자가치유(자동 복구)  
   - 특정 파드를 `kubectl delete po <파드이름>`로 지워도, ReplicaSet이 “파드가 모자람”을 감지  
   - 새 파드를 생성해 다시 2개(또는 설정값) 상태를 유지

### 6.4 참고: 실무에서는 Deployment

- **ReplicaSet**은 파드 “숫자 유지”라는 기능만 존재  
- 서비스 운영 시 **버전 업데이트**, **롤백** 등 고급 기능 필요 → **Deployment** 사용  
- Deployment는 내부적으로 ReplicaSet을 생성·관리하는 **상위 컨트롤러**

<br/>

## 7. Deployment (기초 개념 복습)

1. **정의**  
   - “ReplicaSet” + “버전관리(업데이트/롤백)” 등을 통합  
   - `kind: Deployment`, 내부에서 자동으로 **ReplicaSet** 만들어 파드를 관리

2. **strategy (업데이트 전략)**  
   - **RollingUpdate(기본값)**: 파드를 하나씩 교체, 서비스 중단 없는 배포  
   - **Recreate**: 모든 파드를 한 번에 종료 후 새 버전 생성 → 다운타임 발생

3. **매니페스트 예시**  
   ```yaml
   apiVersion: apps/v1
   kind: Deployment
   metadata:
     name: mydp
   spec:
     replicas: 3
     selector:
       matchLabels:
         app: webapp
     template:
       metadata:
         labels:
           app: webapp
       spec:
         containers:
         - name: last
           image: nginx:1.12
   ```
   - `kubectl apply -f dp.yaml` → `kubectl get deploy,rs,po`  
   - 실제로는 **디플로이먼트**(mydp) → **레플리카셋**(mydp-xxxx) → **파드**(3개) 순으로 생성되며, 모든 과정 자동화

4. **스케일링 & 이미지 업데이트**  
   - 스케일링:
     ```bash
     kubectl scale deploy mydp --replicas=5
     ```
   - 이미지 업데이트:
     ```bash
     kubectl set image deploy mydp last=nginx:1.14
     ```
5. **롤아웃(Rollout) & 롤백(Rollback)**  
   - **롤아웃**: 새로운 버전으로 컨테이너 이미지를 교체  
   - **롤백**: 이전 버전(리비전)으로 되돌리기
   ```bash
   kubectl rollout history deploy mydp
   kubectl rollout undo deploy mydp --to-revision=1
   ```
   - 언제나 “**어느 시점으로**” 롤백할지(`--to-revision`)가 핵심

---

## Day 4 요약

1. **매니페스트(YAML) 파일**  
   - 쿠버네티스 자원(파드, ReplicaSet 등)을 정의하는 설정  
   - 라벨/셀렉터, 재시작 정책 등 다양한 옵션을 문서화  
2. **라벨(Label) 추가·제거**  
   - `kubectl label`, `metadata.labels`  
   - 핵심: 리소스를 그룹핑하고, ReplicaSet/Service 등에서 **선택(selector)** 시 활용  
3. **파드 재시작 정책**  
   - **Always**(기본), **OnFailure**, **Never**  
   - 쿠버네티스는 서비스 안정성 위해 항상 재시작하는 게 일반적  
4. **파드 내부 명령(`kubectl exec`) & 파일 복사(`kubectl cp`)**  
   - 환경 변수 확인, 파일 업로드/다운로드 등 편리하게 활용  
5. **ReplicaSet**  
   - “N개의 동일 파드를 자동 유지”  
   - 자가치유(파드가 사라지면 새 파드 생성)  
   - 실무에서 버전업 등 고급 기능은 없음 → **Deployment** 사용  
6. **Deployment** (간단 소개)  
   - ReplicaSet 상위 개념  
   - **버전 관리**(롤링 업데이트, 롤백) 기능 포함  
   - `kubectl set image`, `kubectl rollout history/undo` 명령으로 컨테이너 이미지를 유연하게 업데이트


---

---

# Day 5: Deployment 업데이트 전략, 다양한 Service 유형, Ingress

## 1. Deployment 업데이트 전략

### 1.1 RollingUpdate vs Recreate

1. **RollingUpdate (기본값)**  
   - 새 파드를 하나씩 생성하면서 동시에 기존 파드를 하나씩 제거  
   - 점진적이고 단계적인 업데이트 → **무중단 배포**가 가능  
   - 배포 도중에 일부 새 버전과 일부 구 버전이 혼재하는 시점이 존재  
   - 서비스 사용자는 구 버전과 새 버전을 순차적으로 만나게 됨  
   - 안전하지만 배포 중에 여러 버전이 공존한다는 점이 특징

2. **Recreate**  
   - 기존 파드를 **모두 중단** 후 새 버전을 생성  
   - 업데이트 시점에 기존 파드는 완전히 사라지고, 새 파드를 생성하기 전까지 **다운타임**이 발생  
   - 배포 과정이 간단하며 “버전 혼합”이 일어나지 않음  
   - 금융권 등 일부 업종에서는 잠깐의 서비스 중단을 감수해도 버전을 확실히 구분하고 싶을 때 사용 가능

#### 예시 매니페스트
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: mydp
spec:
  replicas: 10
  selector:
    matchLabels:
      app: websvr
  strategy:
    type: Recreate   # RollingUpdate → Recreate 로 변경
  template:
    metadata:
      labels:
        app: websvr
    spec:
      containers:
      - name: last
        image: nginx:1.12
```
- `strategy.type`을 `Recreate`로 지정하면, 기존 파드를 전부 종료 후 새 파드를 만드는 **전체 교체** 방식으로 배포가 진행됨

### 1.2 실제 업데이트 과정 관찰

1. **디플로이먼트 생성**  
   ```bash
   kubectl apply -f mydp.yaml
   kubectl get deploy,rs,po
   watch -n 1 kubectl get pod
   ```
   - `watch` 명령어로 파드 상태 변화를 실시간 관찰 가능

2. **이미지 교체 (RollingUpdate/Recreate 모두 가능)**  
   ```bash
   kubectl set image deployment mydp last=nginx:1.14
   ```
   - `last=nginx:1.14` → `컨테이너 이름=새 이미지` 형태  
   - RollingUpdate일 경우 새 파드(1.14)가 하나씩 올라오고, 구 파드(1.12)가 하나씩 내려가는 과정을 볼 수 있음  
   - Recreate일 경우 구 파드가 완전히 사라졌다가 새 파드가 생성 → 잠시 다운타임 발생

3. **롤백(Rollback) 확인**  
   - 롤아웃(업데이트) 히스토리:
     ```bash
     kubectl rollout history deploy mydp
     ```
   - 특정 리비전 확인:
     ```bash
     kubectl rollout history deploy mydp --revision=2
     ```
   - 리비전 2로 되돌리기:
     ```bash
     kubectl rollout undo deploy mydp --to-revision=2
     ```
   - 쿠버네티스는 과거 배포 버전을 “리비전” 단위로 저장해 두어, 필요 시 쉽게 돌릴 수 있음

<br/>

## 2. Service 개념 및 타입별 특징

쿠버네틱스에서 **파드(Pod)**는 언제든 **생성/종료**될 수 있고, IP도 자주 바뀝니다.  
이런 환경에서 파드에 안정적으로 접속하기 위해 **Service**를 사용합니다.  
**Service** = “라벨로 묶인 파드”에 **하나의 대표 IP/포트**를 부여하고, 부하분산(Load Balancing) 및 장애 시 다른 파드로 트래픽을 넘기는 역할도 담당.

### 2.1 Service 주요 기능

1. **파드 IP 변경 문제 해결**: 클라이언트가 직접 파드 IP를 모를 때, 서비스 주소(“대표번호”)만 알면 됨  
2. **로드 밸런싱**: 레플리카 개수가 많은 파드 중 하나에만 트래픽이 몰리지 않도록 자동 분산  
3. **Failover**: 특정 파드가 다운되면, 정상 파드로 자연스럽게 요청을 라우팅  
4. **DNS 연동**: 쿠버네티스 내부 DNS(CoreDNS)가 서비스 이름을 IP로 매핑 → “`curl svc-name`” 방식 사용 가능

### 2.2 Service 타입 정리

1. **ClusterIP (기본)**  
   - 클러스터 내부에서만 사용 가능한 IP 할당  
   - 외부(인터넷)에서 접근할 수 없음(내부망 용도)  
   - 주로 **내부 마이크로서비스** 간 통신에 사용

2. **NodePort**  
   - 노드(서버) 자체의 포트(30000~32767)를 열어 외부 접근 허용  
   - 예) `nodePort: 30000` → “`http://<노드IP>:30000`”  
   - 간단하지만, 큰 단점:  
     - 외부 사용자는 **노드 IP**와 **노드 포트**를 알아야 함  
     - 포트 충돌, 스케일링 문제 등 불편 사항 존재

3. **LoadBalancer**  
   - 클라우드 환경(GCP, AWS 등) 또는 **MetalLB** 같은 도구를 통해 **외부 IP**를 자동 할당  
   - NodePort 대신, 로드밸런서가 모든 노드 앞단에 서서 트래픽을 받아 파드로 전달  
   - **무중단**과 **고가용성(HA)** 보장에 적합  
   - 온프레미스 환경에서는 별도 로드밸런서(예: MetalLB)를 설치해야 가능

4. **ExternalName**  
   - 클러스터 내부 DNS에서 도메인만 **외부 주소**로 매핑  
   - “`external-svc.default.svc.cluster.local`” → “`www.google.com`” 처럼 연결  
   - 주로 내부 앱이 외부 서비스를 **로컬 주소**처럼 사용해야 할 때 유용

<br/>

## 3. ClusterIP 실습 예시

1. **파드 준비**  
   ```bash
   kubectl run myweb --image=nginx:1.12 --labels "app=web-svr" --port 80
   kubectl get pod -o wide
   ```
   - 라벨(`app=web-svr`)을 붙여둬야 서비스에서 파드를 인식 가능

2. **서비스 매니페스트**  
   ```yaml
   apiVersion: v1
   kind: Service
   metadata:
     name: svc1
   spec:
     selector:
       app: web-svr   # 파드의 라벨과 매칭
     type: ClusterIP  # 기본값이므로 생략 가능
     ports:
     - port: 80       # 서비스(대표) 포트
       targetPort: 80 # 실제 파드 컨테이너 포트
       protocol: TCP
   ```
   ```bash
   kubectl apply -f svc1.yaml
   kubectl get svc
   ```
3. **서비스 동작 확인**  
   - 클러스터 내부의 **또 다른 파드**(예: `clientpod`)에서 `curl svc1` or `curl <클러스터IP>`로 접속 테스트  
   - `kubectl exec -it clientpod -- bash` → `curl svc1`

<br/>

## 4. NodePort 실습 예시

1. **파드 생성**  
   ```bash
   kubectl run myweb2 --image=nginx:1.12 --labels "app=web-svc2" --port 80
   ```
2. **NodePort 매니페스트**  
   ```yaml
   apiVersion: v1
   kind: Service
   metadata:
     name: svc2
   spec:
     selector:
       app: web-svc2
     type: NodePort
     ports:
     - port: 80
       targetPort: 80
       nodePort: 30000
       protocol: TCP
   ```
   ```bash
   kubectl apply -f svc2.yaml
   kubectl get svc
   ```
3. **외부 접속 테스트**  
   - `<노드 IP>:30000` 으로 직접 접속  
   - 장점: 쉽게 외부 공개  
   - 단점: 노드 IP + 포트를 알아야 하고, 부하분산 제한

<br/>

## 5. MetalLB로 LoadBalancer 서비스 구현

- 클라우드 환경(GCP, AWS 등) 없이 “Bare-metal” 서버에서 **LoadBalancer 타입** 서비스를 쓰려면 **MetalLB** 설치가 필요
- **설치 흐름** (간략):
  1. kube-proxy 설정에서 `strictARP: true`  
  2. MetalLB 매니페스트(네임스페이스, metallb.yaml) 적용  
  3. `ConfigMap` 설정(`Layer2` 모드)로 IP 풀 범위 지정 (예: 192.168.11.230~192.168.11.233)
  4. `kubectl apply -f` 로 LoadBalancer 타입 Service 생성 → 자동 할당된 EXTERNAL-IP 확인

```yaml
apiVersion: v1
kind: Service
metadata:
  name: svc3
spec:
  selector:
    app: web-svc3
  type: LoadBalancer
  ports:
    - port: 80
      targetPort: 80
```
- 이후 `kubectl get svc` 하면 **EXTERNAL-IP** 칸에 192.168.11.230 같은 IP가 표시  
- 브라우저에서 `http://192.168.11.230:80`로 접속 가능

<br/>

## 6. Ingress: HTTP/HTTPS 라우팅 규칙

### 6.1 Ingress 개념

- 쿠버네티스 **Service**는 L4(TCP/UDP) 레벨에서만 포트 기반 접근을 제어하는 반면,  
  **Ingress**는 HTTP/HTTPS **경로** 및 **호스트명**을 기반으로 라우팅 규칙을 설정  
- 즉, 하나의 진입점에서 “`/blue` → 블루 파드, `/green` → 그린 파드” 식으로 트래픽을 분배 가능  
- **Ingress Controller**(예: ingress-nginx, Traefik 등)를 반드시 설치해야 동작  
- 실무에서 **도메인**(예: `www.bjkim2000.com`)을 Ingress에 매핑하여 **외부 HTTP/HTTPS** 요청을 받음

### 6.2 Ingress-NGINX 설치 예시 (Bare-metal)

1. **ingress-nginx** 설치 매니페스트(예: `deploy.yaml`) 적용  
2. `type: LoadBalancer` 또는 `NodePort`로 Ingress Controller를 외부에 노출  
3. `kubectl get svc -n ingress-nginx` → EXTERNAL-IP 확인  
4. Windows `hosts` 파일에 `EXTERNAL-IP`와 도메인(`www.bjkim2000.com`) 매핑

### 6.3 Ingress 설정 파일 예시

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: myingress
spec:
  rules:
  - host: www.bjkim2000.com
    http:
      paths:
      - pathType: Prefix
        path: "/"
        backend:
          service:
            name: nginx-main-svc
            port:
              number: 80
      - pathType: Prefix
        path: "/blue"
        backend:
          service:
            name: nginx-blue-svc
            port:
              number: 80
      - pathType: Prefix
        path: "/green"
        backend:
          service:
            name: nginx-green-svc
            port:
              number: 80

  - host: www.guru2025.co.kr
    http:
      paths:
      - pathType: Prefix
        path: "/"
        backend:
          service:
            name: httpd-main-svc
            port:
              number: 80
```
- `kubectl apply -f in.yaml` → `kubectl get ing` 확인  
- 브라우저에서 `http://www.bjkim2000.com/blue` → nginx-blue 서비스, `http://www.guru2025.co.kr/` → httpd-main 서비스로 라우팅

### 6.4 실제 운영 시나리오

- **여러 웹서비스**(Nginx, Apache 등)를 각각 별도 디플로이먼트/서비스로 배포  
- **Ingress**에서 호스트나 경로 기반으로 “이 요청은 Nginx 파드로, 저 요청은 Apache 파드로” 라우팅  
- 버전 업 시에는 Deployment RollingUpdate를 통해 **무중단** 배포 가능  
- 만약 특정 호스트/경로 트래픽이 많아지면, **`replicas`** 값을 늘려 스케일 아웃

<br/>

## 7. 종합 예제: Quiz

### 7.1 조건

1. **Deployment**:  
   - 이름: `quiz-deploy`  
   - 파드 레플리카: 5  
   - 라벨: `app=nginx-testbed`  
   - 컨테이너 이름: `www`, 이미지: `nginx:1.14`, 포트 80  
2. **Service**:  
   - 이름: `quiz-svc`  
   - 타입: `ClusterIP`  
   - selector: `app=nginx-testbed`  
   - ports: port=80, targetPort=80

### 7.2 정답 예시 매니페스트

```yaml
# quiz-deploy.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: quiz-deploy
spec:
  replicas: 5
  selector:
    matchLabels:
      app: nginx-testbed
  template:
    metadata:
      labels:
        app: nginx-testbed
    spec:
      containers:
      - name: www
        image: nginx:1.14
        ports:
        - containerPort: 80
---
# quiz-svc.yaml
apiVersion: v1
kind: Service
metadata:
  name: quiz-svc
spec:
  selector:
    app: nginx-testbed
  type: ClusterIP
  ports:
  - port: 80
    targetPort: 80
    protocol: TCP
```
- `kubectl apply -f quiz-deploy.yaml` → `kubectl apply -f quiz-svc.yaml`  
- `kubectl get deploy,rs,po,svc` 로 전체 확인

### 7.3 접속 테스트

1. **클라이언트 파드 생성**  
   ```bash
   kubectl run client --image=nginx
   ```
2. **파드 내부 접속**  
   ```bash
   kubectl exec -it client -- bash
   # 클러스터 내부 DNS로 quiz-svc 접근
   curl quiz-svc
   exit
   ```
- 제대로 설정됐다면 “Welcome to nginx!” 등의 페이지 응답이 나옴

<br/>

## 8. 마무리 정리

1. **Deployment**  
   - ReplicaSet을 내부적으로 관리 → 자동 스케일링/자가치유  
   - RollingUpdate / Recreate 등 **업데이트 전략**으로 무중단 배포 또는 단절 배포 선택  
   - `kubectl set image`, `rollout history`, `rollout undo`를 통해 **버전 관리** 가능
2. **Service**  
   - 파드의 IP가 변동되는 문제를 해결하기 위한 “대표 IP/포트”  
   - ClusterIP, NodePort, LoadBalancer, ExternalName 등 다양한 접근 방식  
   - 라벨(selector)로 어떤 파드들을 묶을지 정의  
3. **LoadBalancer(MetalLB)**  
   - 온프레미스 환경에서도 외부 IP를 자동 할당받아 NodePort 없이도 접근 가능  
   - IP 풀 범위를 설정하여 네트워크 충돌 방지  
4. **Ingress**  
   - HTTP/HTTPS **경로/호스트** 기반 라우팅 (L7 레벨)  
   - Ingress Controller 설치 필요 (ingress-nginx 등)  
   - 도메인/호스트를 여러 개 설정 가능 → 마이크로서비스 구성 시 유용  
5. **실무 시나리오**  
   - **Deployment**로 컨테이너 오브젝트를 무중단 업데이트 & 스케일링  
   - **Service**(ClusterIP/LoadBalancer)로 내부 통신 및 외부 접근을 담당  
   - **Ingress**로 도메인 기반 라우팅(여러 앱/버전 통합), SSL/HTTPS 설정, 다양한 경로 분기

---

# Day 5 요약

- **Deployment**  
  - 업데이트 전략(무중단 RollingUpdate vs 다운타임 Recreate)  
  - 롤백(rollout undo) & 히스토리(rollout history)  
- **Service**  
  - **ClusterIP**: 내부 전용  
  - **NodePort**: 노드 포트 개방  
  - **LoadBalancer**: 클라우드/MetalLB 이용, 외부 IP 자동 할당  
  - **라벨/셀렉터**로 파드를 그룹핑, DNS(CoreDNS) 통해 내부 접근 용이  
- **MetalLB**  
  - Bare-metal 환경에서 LoadBalancer 서비스 사용 가능  
  - IP 풀 범위 설정 → EXTERNAL-IP로 트래픽 분산  
- **Ingress**  
  - L7 계층(HTTP/HTTPS) 라우팅  
  - Ingress Controller 필수(예: ingress-nginx)  
  - 도메인/경로별로 서로 다른 서비스 연결  
- **전체 흐름**  
  - **배포**(Deployment) → **무중단 업데이트**(RollingUpdate) → **서비스**로 안정적 접근 → 필요 시 **Ingress** 설정으로 복잡한 라우팅/도메인 연결 구현

---
