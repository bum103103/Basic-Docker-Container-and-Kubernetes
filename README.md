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