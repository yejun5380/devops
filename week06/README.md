# 6주차 - Dockerfile 이미지 빌드 및 실행

## 1. 실습 목표

- Dockerfile의 주요 명령어 이해
- Dockerfile을 이용한 Flask 방명록 이미지 빌드
- 멀티스테이지 빌드 적용 및 이미지 크기 비교
- Docker 컨테이너 실행과 웹 서버 정상 동작 확인

## 2. 실습 환경

- Ubuntu (WSL)
- Docker 29.1.3
- Python 3.12 기반 Flask 방명록
- 실습 프로젝트: `guestbook/`

## 3. Dockerfile 주요 명령어

| 명령어 | 설명 |
|---|---|
| FROM | 이미지의 기반이 되는 베이스 이미지 지정 |
| WORKDIR | 컨테이너 내부 작업 디렉터리 설정 |
| COPY | 로컬 파일을 이미지 내부로 복사 |
| RUN | 이미지 빌드 중 명령어 실행 |
| USER | 컨테이너 실행 사용자 지정 |
| ENV | 환경변수 설정 |
| EXPOSE | 컨테이너에서 사용할 포트 명시 |
| CMD | 컨테이너 시작 시 실행할 기본 명령어 지정 |

## 4. 일반 Docker 이미지 빌드

기존 `guestbook/Dockerfile`을 사용하여 이미지를 빌드하였다.

```bash
cd guestbook
docker build -t guestbook:fat .
docker images guestbook:fat
```

빌드 결과:

- 이미지: `guestbook:fat`
- 이미지 ID: `04fc71e82468`
- Disk Usage: 201MB
- Content Size: 49MB
- 빌드 성공

## 5. 멀티스테이지 이미지 빌드

기존 `guestbook/Dockerfile.multistage`를 이용하였다.

멀티스테이지 빌드는 빌드 단계와 최종 실행 단계를 분리하여 최종 이미지에 필요한 파일만 포함하도록 구성하는 방식이다.

```bash
docker build -f Dockerfile.multistage -t guestbook:slim .
docker images guestbook:slim
```

빌드 결과:

- 이미지: `guestbook:slim`
- 이미지 ID: `ac754f462087`
- Disk Usage: 199MB
- Content Size: 47.9MB
- 빌드 성공

### 이미지 크기 비교

| 구분 | 일반 빌드 | 멀티스테이지 빌드 |
|---|---|---|
| 이미지 | guestbook:fat | guestbook:slim |
| Disk Usage | 201MB | 199MB |
| Content Size | 49MB | 47.9MB |

멀티스테이지 이미지가 일반 이미지보다 약 2MB 작았다. 두 이미지 모두 `python:3.12-slim`을 사용하므로 크기 차이는 크지 않았다.

실제 측정 결과: [image-comparison.txt](image-comparison.txt)

## 6. 컨테이너 실행 및 웹페이지 확인

멀티스테이지 이미지로 컨테이너를 실행하였다.

```bash
docker run -d -p 8081:5000 --name week06-gb guestbook:slim
docker ps
curl -I http://localhost:8081
```

실행 결과:

- 컨테이너 이름: `week06-gb`
- 상태: `Up`
- 포트 연결: `8081:5000`
- HTTP 응답: `200 OK`
- 브라우저에서 방명록 페이지 정상 출력 확인

실습 기록:

- [container-status.txt](container-status.txt)
- [http-result.txt](http-result.txt)

## 7. 실습 컨테이너 정리

```bash
docker stop week06-gb
docker rm week06-gb
docker ps -a
```

6주차 실습용 컨테이너만 정리하였으며, 기존 컨테이너와 이미지는 유지하였다.

## 8. 실습 결과 및 느낀 점

Dockerfile을 통해 애플리케이션 실행 환경을 이미지로 만들고, 동일한 환경을 컨테이너로 실행할 수 있음을 확인하였다.

또한 멀티스테이지 빌드를 적용하여 이미지 크기를 비교하였다. 이번 실습에서는 약 2MB의 차이가 발생했으며, 이미지 최적화 효과는 베이스 이미지와 빌드 과정에 따라 달라질 수 있다는 점을 알게 되었다.
