
# 3주차 - Python 웹 서버와 Shell Script 실습

## 1. 학습 목표

- Python을 이용한 웹 서버 구조 이해
- Ollama API와 Qwen AI 모델 연동 구조 학습
- Shell Script를 이용한 프로그램 실행
- 환경변수를 활용한 실행 설정 변경

## 2. 실습 파일 구성

```text
week03/
├── chat.py
└── qwen-web/
    ├── chat.py
    ├── run_py.sh
    ├── start.sh
    ├── start_with_export.sh
    └── start_with_export_2.sh
```

## 3. Python 웹 채팅 프로그램

`chat.py`는 Python 표준 라이브러리인
`BaseHTTPRequestHandler`와 `HTTPServer`를 활용한
웹 채팅 프로그램이다.

HTML 입력 폼에서 사용자의 질문을 입력받고,
로컬 Ollama API와 통신하도록 구성되어 있다.

### 주요 설정

| 항목 | 설정값 |
|---|---|
| AI 모델 | `qwen3:0.6b` |
| 기본 웹 포트 | `8000` |
| 프로그램 이름 | `Qwen Web` |
| Ollama API 주소 | `http://127.0.0.1:11434/api/generate` |

`MODEL`, `WEB_PORT`, `APP_NAME` 환경변수를 통해
프로그램의 설정을 변경할 수 있다.

AI 응답 기능을 사용하려면 Ollama 서버와
해당 AI 모델이 준비되어 있어야 한다.

## 4. Shell Script 실행 실습

### start.sh

```bash
#!/bin/bash
cd "$(dirname "$0")" || exit 1
exec python3 chat.py
```

스크립트가 저장된 디렉터리로 이동한 후
Python 웹 채팅 프로그램을 실행한다.

### start_with_export.sh

```bash
#!/bin/bash
cd "$(dirname "$0")" || exit 1
export MODEL="qwen3:0.6b"
exec ./start.sh
```

`MODEL` 환경변수를 설정하여 사용할 AI 모델을 지정한다.

### start_with_export_2.sh

```bash
#!/bin/bash
cd "$(dirname "$0")" || exit 1
export WEB_PORT="8080"
exec ./start.sh
```

`WEB_PORT` 환경변수를 설정하여
웹 서버 실행 포트를 8080으로 변경한다.

## 5. Python 기초 실행 예제

`run_py.sh`에는 다음과 같은 Python 코드가 작성되어 있다.

```python
#!/usr/bin/python3
print('Hello, DevOps!')
```

확장자는 `.sh`이지만 첫 줄의 Shebang을 통해
Python 인터프리터를 지정하고 있다.

## 6. 주요 개념 정리

| 개념 | 설명 |
|---|---|
| HTTP 서버 | 웹 브라우저의 요청을 받아 응답하는 프로그램 |
| Ollama | 로컬 환경에서 AI 모델을 실행하는 도구 |
| Qwen | 웹 채팅 프로그램에 설정된 AI 모델 |
| Shell Script | 여러 명령어를 파일로 작성하여 실행하는 방법 |
| export | 자식 프로세스에 전달할 환경변수를 설정하는 명령어 |
| exec | 현재 프로세스를 지정한 프로그램으로 대체하는 명령어 |
| Shebang | 스크립트 실행에 사용할 인터프리터를 지정하는 첫 줄 |

## 7. 실습 정리

3주차에는 Python으로 작성된 웹 채팅 프로그램의
구조와 실행 방식을 살펴보고, Shell Script를 이용해
실행 명령어를 자동화하는 방법을 학습하였다.

또한 환경변수를 활용하면 Python 코드를 직접 수정하지 않고도
AI 모델과 웹 서버 포트 등의 설정을 변경할 수 있음을 확인하였다.

