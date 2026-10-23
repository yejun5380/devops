# 5주차 - Docker 및 GitHub Issue 실습

## 1. 학습 목표

- 가상 머신(VM)과 컨테이너의 차이를 이해한다.
- Docker 이미지와 컨테이너의 관계를 이해한다.
- Docker 명령어를 이용하여 컨테이너를 생성하고 관리한다.
- GitHub Issue, Branch, Pull Request, Merge를 활용하여 실습 과정을 관리한다.

## 2. Docker 기본 개념

### Docker란?

Docker는 애플리케이션과 실행에 필요한 환경을 이미지로 구성하고, 이를 컨테이너 형태로 실행하는 플랫폼이다.

가상 머신은 별도의 게스트 운영체제를 사용하지만, 컨테이너는 호스트의 커널을 공유하므로 상대적으로 가볍고 빠르게 실행할 수 있다.

### 이미지와 컨테이너

- **이미지(Image)**: 애플리케이션 실행에 필요한 파일과 설정을 포함한 템플릿
- **컨테이너(Container)**: 이미지를 기반으로 생성된 실행 환경

하나의 이미지로 여러 컨테이너를 생성할 수 있다.

## 3. Docker 실행 환경 확인

Docker 버전:

```bash
docker --version
```

실행 결과:

```text
Docker version 29.1.3, build 29.1.3-0ubuntu4.1
```

Docker 엔진 정보:

```bash
docker info
```

Docker 클라이언트와 서버가 정상적으로 연결되는 것을 확인하였다.

## 4. hello-world 실행

```bash
docker run --rm hello-world
```

실행 결과:

```text
Hello from Docker!
This message shows that your installation appears to be working correctly.
```

Docker Hub에서 이미지를 가져와 컨테이너를 생성하고 실행하는 과정을 확인하였다.

`--rm` 옵션은 컨테이너 실행이 종료되면 해당 컨테이너를 자동으로 삭제한다.

## 5. nginx 웹 서버 실행

```bash
docker run -d -p 8080:80 --name web nginx
```

주요 옵션:

- `-d`: 백그라운드 실행
- `-p 8080:80`: 호스트 8080 포트를 컨테이너 80 포트에 연결
- `--name web`: 컨테이너 이름 지정

### 웹 서버 응답 확인

```bash
curl -I http://localhost:8080
```

실행 결과:

```text
HTTP/1.1 200 OK
Server: nginx/1.31.8
```

HTTP 200 OK 응답을 통해 nginx 웹 서버가 정상적으로 실행되는 것을 확인하였다.

### 컨테이너 상태 확인

```bash
docker ps
```

실행 결과에서 `web` 컨테이너가 `Up` 상태로 표시되는 것을 확인하였다.

## 6. 컨테이너 정리

```bash
docker stop web
docker rm web
docker ps -a
```

실습에 사용한 nginx 컨테이너를 중지하고 삭제하였다.

기존에 생성한 다른 컨테이너는 유지하였다.

## 7. 주요 Docker 명령어

| 명령어 | 기능 |
|---|---|
| docker run | 컨테이너 생성 및 실행 |
| docker ps | 실행 중인 컨테이너 확인 |
| docker ps -a | 전체 컨테이너 확인 |
| docker images | 이미지 목록 확인 |
| docker logs | 컨테이너 로그 확인 |
| docker exec | 실행 중인 컨테이너 내부 명령 실행 |
| docker stop | 컨테이너 중지 |
| docker rm | 컨테이너 삭제 |

## 8. GitHub Issue 기반 실습 관리

이번 실습은 GitHub Issue #3을 생성하여 진행하였다.

작업 순서:

1. Issue #3 생성
2. `docs/issue-3-docker-start` 브랜치 생성
3. Docker 실행 환경 확인
4. hello-world 및 nginx 실습
5. 실행 결과 기록
6. README.md 작성
7. Commit 및 Push
8. Pull Request 생성 및 Merge

Issue를 통해 작업 목표와 완료 조건을 정리하고, 별도의 브랜치에서 실습 내용을 관리하였다.

## 9. 실습 결과 파일

| 파일 | 내용 |
|---|---|
| docker-version.txt | Docker 버전 |
| docker-info.txt | Docker 실행 환경 |
| docker-run-hello-world.txt | hello-world 실행 결과 |
| curl-result.txt | nginx HTTP 응답 |
| docker-ps.txt | nginx 컨테이너 실행 상태 |

## 10. 실습 소감

이번 실습을 통해 Docker 이미지와 컨테이너의 관계를 이해할 수 있었다. 특히 nginx 이미지를 이용해 별도의 웹 서버 설치 과정 없이 컨테이너를 실행하고, 포트를 연결하여 웹 서버에 접근하는 과정을 확인하였다.

또한 GitHub Issue와 브랜치를 활용하여 작업을 분리하고 실습 결과를 기록하는 방법을 익혔다.
