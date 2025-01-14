# Docker-Study
Docker 학습 과정과 실습 내용을 정리한 저장소 | Learning and practicing Docker from basics to advanced concepts

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