
# Week 13 - GitOps 및 모니터링 실습

## 1. 실습 목표

13주차에는 Argo CD를 활용하여 GitOps 방식의 자동 배포를 구현하고, Prometheus와 Grafana를 이용하여 Kubernetes 클러스터의 상태를 모니터링하였다.

주요 실습 내용은 다음과 같다.

- Argo CD를 통한 GitOps 자동 배포
- Git 변경에 따른 Kubernetes 리소스 자동 동기화
- SelfHeal 및 Prune 동작 확인
- Git Revert를 통한 배포 복구
- GitHub Actions, GHCR, Argo CD 연동
- Metrics Server를 이용한 CPU 및 메모리 확인
- Prometheus와 Grafana를 이용한 모니터링
- 부하 테스트 및 로그 분석

## 2. 실습 환경

| 항목 | 사용 환경 |
|---|---|
| OS | Ubuntu 24.04 |
| 컨테이너 | Docker |
| Kubernetes | Minikube |
| CI | GitHub Actions |
| 이미지 저장소 | GitHub Container Registry (GHCR) |
| CD | Argo CD |
| 모니터링 | Prometheus, Grafana |
| 패키지 관리 | Helm |
| 프로젝트 | Flask 기반 Guestbook |

GitHub 저장소:

https://github.com/yejun5380/devops

## 3. Argo CD 및 GitOps 실습

### 3.1 GitOps 개념

GitOps는 Git 저장소에 선언된 설정을 기준으로 실제 Kubernetes 클러스터의 상태를 관리하는 방식이다.

Argo CD는 Git 저장소와 클러스터 상태를 비교하고, 차이가 발생하면 설정에 따라 자동으로 동기화한다.

이번 실습에서는 GitHub 저장소의 `guestbook/k8s` 디렉터리를 배포 설정의 기준으로 사용하였다.

### 3.2 Argo CD Application 설정

`argocd-application.yaml` 파일을 작성하여 Guestbook 애플리케이션을 등록하였다.

```yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: guestbook
  namespace: argocd
spec:
  project: default
  source:
    repoURL: https://github.com/yejun5380/devops.git
    targetRevision: main
    path: guestbook/k8s
  destination:
    server: https://kubernetes.default.svc
    namespace: default
  syncPolicy:
    automated:
      prune: true
      selfHeal: true
```

주요 설정:

- `automated`: Git 변경 사항 자동 반영
- `prune`: Git에서 삭제한 리소스를 클러스터에서도 삭제
- `selfHeal`: 클러스터를 직접 수정하면 Git에 선언된 상태로 복구

Argo CD 웹페이지는 포트 포워딩을 통해 접속하였다.

```bash
kubectl port-forward svc/argocd-server -n argocd 8443:443
```

접속 주소: `https://localhost:8443`

Application 등록 후 GitOps 동기화 상태를 확인하였다.

### 3.3 Git 변경에 따른 자동 배포

`deployment.yaml`의 Replica 수를 2개에서 4개로 변경하였다.

```yaml
replicas: 4
```

변경 사항을 Pull Request로 생성하고 `main` 브랜치에 병합하였다.

Argo CD가 Git 변경을 감지한 후 Kubernetes Deployment를 자동으로 변경하는 것을 확인하였다.

결과:

- 기존 Pod 2개에서 4개로 증가
- Argo CD 자동 동기화 정상 동작
- 별도의 수동 배포 없이 Git 변경으로 배포 완료

### 3.4 SelfHeal 실습

Git에는 Replica 수가 4개로 선언된 상태에서 Kubernetes의 Replica 수를 직접 변경하였다.

```bash
kubectl scale deployment guestbook --replicas=10
```

이후 Argo CD가 클러스터 상태와 Git 설정의 차이를 감지하였다.

`selfHeal: true` 설정에 따라 Replica 수가 다시 4개로 복구되는 것을 확인하였다.

이를 통해 GitOps 환경에서는 Git 저장소의 선언이 실제 운영 상태의 기준이 된다는 것을 확인하였다.

### 3.5 Git Revert 실습

Git 변경을 되돌리는 방식으로 Replica 수를 기존 2개로 복구하였다.

Git Revert로 생성한 변경 사항을 Pull Request를 통해 `main`에 병합하였다.

결과:

- Git에 선언된 Replica 수가 2개로 복구
- Argo CD가 변경을 자동 반영
- Kubernetes Pod 2개 정상 실행

### 3.6 Prune 실습

`guestbook/k8s/hello.yaml` 파일을 추가하여 ConfigMap을 생성하였다.

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: hello
data:
  message: hello
```

파일을 Git에 반영한 후 ConfigMap이 Kubernetes 클러스터에 자동 생성되는 것을 확인하였다.

이후 Git에서 `hello.yaml` 파일을 삭제하고 변경 사항을 병합하였다.

```bash
kubectl get cm hello
```

최종적으로 `NotFound`가 출력되어 ConfigMap이 삭제된 것을 확인하였다.

이는 Argo CD의 `prune: true` 설정이 정상적으로 작동한 결과이다.

## 4. GitHub Actions 및 GHCR 연동

### 4.1 애플리케이션 코드 변경

Guestbook의 `templates/index.html` 파일에서 제목을 수정하였다.

기존:

```html
<h1>{{ title }}</h1>
```

변경:

```html
<h1>{{ title }} CI/CD</h1>
```

변경 사항을 Pull Request로 병합한 후 GitHub Actions가 실행되었다.

### 4.2 컨테이너 이미지 자동 빌드

GitHub Actions를 통해 컨테이너 이미지가 빌드되고 GHCR에 게시되는 것을 확인하였다.

실습에서 사용한 이미지 태그:

```text
ghcr.io/yejun5380/guestbook:2f6ad5b
```

### 4.3 Kubernetes 배포 이미지 변경

`guestbook/k8s/deployment.yaml`의 이미지 태그를 변경하였다.

기존:

```yaml
image: ghcr.io/yejun5380/guestbook:v3
```

변경:

```yaml
image: ghcr.io/yejun5380/guestbook:2f6ad5b
```

변경 사항을 `main` 브랜치에 병합하자 Argo CD가 새로운 이미지를 자동으로 배포하였다.

다음 명령어로 결과를 확인하였다.

```bash
kubectl get deployment guestbook
```

```bash
kubectl get deployment guestbook -o jsonpath='{.spec.template.spec.containers[0].image}{"\n"}'
```

결과:

- 새로운 이미지 태그 적용
- Rolling Update 정상 진행
- Guestbook Pod 2개 정상 실행

전체 배포 흐름:

```text
코드 수정
   ↓
GitHub Pull Request
   ↓
main 브랜치 병합
   ↓
GitHub Actions
   ↓
GHCR 이미지 게시
   ↓
deployment.yaml 이미지 태그 변경
   ↓
Pull Request 병합
   ↓
Argo CD 자동 동기화
   ↓
Kubernetes 배포
```

## 5. GitOps 관리 종료

실습 종료 후 Argo CD Application 등록을 해제하였다.

```bash
kubectl delete -f argocd-application.yaml
```

실행 결과:

```text
application.argoproj.io "guestbook" deleted from argocd namespace
```

Argo CD의 Guestbook Application 등록을 삭제하여 GitOps 관리를 종료하였다.

## 6. Metrics Server 실습

### 6.1 Metrics Server 활성화

```bash
minikube addons enable metrics-server
```

실행 결과:

```text
The 'metrics-server' addon is enabled
```

Pod 상태를 확인하였다.

```bash
kubectl get pods -n kube-system | grep metrics-server
```

초기에는 `0/1 Running`이었지만 약 1분 후 `1/1 Running`으로 변경되었다.

### 6.2 노드 CPU 및 메모리 확인

```bash
kubectl top nodes
```

실행 결과:

```text
NAME       CPU(cores)   CPU(%)   MEMORY(bytes)   MEMORY(%)
minikube   118m         0%       1974Mi          12%
```

### 6.3 Pod CPU 및 메모리 확인

```bash
kubectl top pods
```

실행 결과:

```text
NAME                        CPU(cores)   MEMORY(bytes)
guestbook-67d94b655-tn6fg   1m           27Mi
guestbook-67d94b655-xnl5d   1m           27Mi
redis-57d7f66958-ppnbq      3m           5Mi
```

Metrics Server를 이용하여 Kubernetes 리소스 사용량을 확인할 수 있었다.

## 7. Prometheus 및 Grafana 설치

### 7.1 Helm 설치

```bash
sudo snap install helm --classic
```

설치 결과:

```text
helm 4.3.2 from Snapcrafters installed
```

### 7.2 Helm 저장소 추가

```bash
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
```

### 7.3 모니터링 패키지 설치

```bash
helm install monitoring prometheus-community/kube-prometheus-stack -n monitoring --create-namespace
```

설치 결과:

```text
STATUS: deployed
REVISION: 1
DESCRIPTION: Install complete
```

### 7.4 모니터링 Pod 상태 확인

```bash
kubectl get pods -n monitoring -w
```

최종 확인 결과:

| Pod | 상태 |
|---|---|
| Grafana | 3/3 Running |
| Prometheus | 2/2 Running |
| Alertmanager | 2/2 Running |
| Prometheus Operator | 1/1 Running |
| Kube State Metrics | 1/1 Running |
| Node Exporter | 1/1 Running |

### 7.5 Grafana 접속

```bash
kubectl port-forward -n monitoring svc/monitoring-grafana 3000:80
```

접속 주소:

http://localhost:3000

관리자 비밀번호 확인:

```bash
kubectl get secret monitoring-grafana -n monitoring -o jsonpath='{.data.admin-password}' | base64 -d
```

사용자명 `admin`으로 로그인하였다.

Grafana에서 다음 대시보드를 확인하였다.

```text
Dashboards
→ Kubernetes / Compute Resources / Pod
```

설정:

- Namespace: default
- Pod: guestbook
- 시간 범위: Last 15 minutes

CPU 사용량 및 CPU Throttling 그래프가 정상적으로 표시되는 것을 확인하였다.

## 8. 부하 테스트

### 8.1 부하 발생

BusyBox Pod를 생성하여 Guestbook 서비스에 반복적인 HTTP 요청을 보냈다.

```bash
kubectl run load --image=busybox --restart=Never -- \
  sh -c "while true; do wget -q -O /dev/null http://guestbook/; done"
```

실행 결과:

```text
pod/load created
```

Grafana에서 CPU 사용량이 증가하는 것을 확인하였다.

### 8.2 부하 테스트 결과

```bash
kubectl top pods
```

실행 결과:

```text
NAME                        CPU(cores)   MEMORY(bytes)
guestbook-67d94b655-tn6fg   163m         27Mi
guestbook-67d94b655-xnl5d   161m         33Mi
load                        534m         12Mi
redis-57d7f66958-ppnbq      37m          7Mi
```

부하 테스트 전후 CPU 비교:

| Pod | 테스트 전 | 테스트 중 |
|---|---:|---:|
| Guestbook 1 | 1m | 163m |
| Guestbook 2 | 1m | 161m |
| Redis | 3m | 37m |
| Load | 없음 | 534m |

반복 요청에 따라 CPU 사용량이 증가하는 것을 확인하였다.

### 8.3 부하 테스트 종료

```bash
kubectl delete pod load
```

실행 결과:

```text
pod "load" deleted from default namespace
```

불필요한 부하가 계속 발생하지 않도록 테스트 Pod를 삭제하였다.

## 9. Kubernetes 로그 확인

### 9.1 최근 로그 조회

```bash
kubectl logs -l app=guestbook --tail=20
```

Guestbook Pod들의 최근 로그를 확인하였다.

### 9.2 실시간 로그 확인

```bash
kubectl logs -l app=guestbook -f --prefix
```

각 Pod의 이름과 함께 실시간 로그가 출력되는 것을 확인하였다.

`Ctrl + C`로 로그 조회를 종료하였다.

### 9.3 최근 5분 로그 조회

```bash
kubectl logs deploy/guestbook --since=5m
```

부하 테스트로 발생한 HTTP 요청 로그와 Kubernetes 상태 검사 로그를 확인하였다.

### 9.4 오류 로그 필터링

```bash
kubectl logs -l app=guestbook | grep -i error
```

실행 결과 출력이 없었다.

조회된 로그에서 `error` 문자열이 발견되지 않았으며, HTTP 요청과 상태 검사에서는 `200` 응답을 확인하였다.

## 10. 실습 중 발생한 문제 및 해결

### 10.1 Argo CD 자동 동기화 지연

**증상**

GitHub에서 변경 사항을 병합했지만 Kubernetes 클러스터에 바로 반영되지 않는 경우가 있었다.

**원인 및 확인**

Argo CD가 Git 변경 사항을 감지하는 데 시간이 필요했으며, 동기화 상태가 이전 Git 리비전을 기준으로 표시될 수 있었다.

**해결**

잠시 기다린 후 상태를 다시 확인하였다.

필요할 때 다음 명령어로 Application 새로고침을 요청하였다.

```bash
kubectl annotate application guestbook -n argocd argocd.argoproj.io/refresh=normal --overwrite
```

**결과**

변경 사항이 반영되어 자동 동기화 및 Prune 동작을 확인하였다.

### 10.2 Metrics Server 준비 지연

**증상**

Metrics Server를 활성화한 직후 Pod 상태가 다음과 같이 표시되었다.

```text
0/1 Running
```

**해결**

일정 시간 기다린 후 Pod 상태를 다시 확인하였다.

```bash
kubectl get pods -n kube-system | grep metrics-server
```

**결과**

약 1분 후 다음과 같이 변경되었다.

```text
1/1 Running
```

이후 `kubectl top nodes`, `kubectl top pods`가 정상 실행되었다.

### 10.3 Grafana 대시보드 선택 문제

**증상**

Grafana 로그인 후 Dashboards 화면에서 `namespace`, `pod` 선택 항목이 보이지 않았다.

**원인**

대시보드 상세 화면이 아니라 목록 화면에 있었으며, 대시보드 이름 대신 체크박스가 선택된 상태였다.

**해결**

`Kubernetes / Compute Resources / Pod` 대시보드 이름을 클릭하였다.

**결과**

대시보드 상세 화면에서 Namespace, Pod, 시간 범위를 설정하고 CPU 그래프를 정상적으로 확인하였다.

## 11. 주요 개념 정리

### GitOps

Git 저장소를 기준으로 Kubernetes 클러스터 상태를 관리하는 방식이다.

### Argo CD

Git 저장소의 배포 설정과 실제 클러스터 상태를 비교하고 동기화하는 CD 도구이다.

### SelfHeal

클러스터가 Git에 선언된 상태와 달라지면 자동으로 원래 설정으로 복구하는 기능이다.

### Prune

Git에서 삭제한 리소스를 Kubernetes 클러스터에서도 삭제하는 기능이다.

### Rolling Update

기존 Pod를 유지하면서 새 Pod로 점진적으로 교체하는 배포 전략이다.

### Prometheus

CPU, 메모리 등 시스템 메트릭을 수집하고 저장하는 모니터링 도구이다.

### Grafana

수집된 메트릭을 그래프와 대시보드로 시각화하는 도구이다.

### Alertmanager

설정된 조건에 따라 모니터링 알림을 관리하는 도구이다.

## 12. 실습 결과 및 느낀 점

이번 실습에서는 GitHub의 변경 사항이 Argo CD를 통해 Kubernetes 클러스터에 자동으로 반영되는 GitOps 배포 과정을 직접 확인하였다.

특히 Replica 수를 수동으로 변경했을 때 SelfHeal 기능으로 복구되는 과정과 Git에서 삭제한 ConfigMap이 Prune 기능으로 자동 삭제되는 과정을 통해 GitOps의 동작 원리를 이해할 수 있었다.

또한 GitHub Actions로 컨테이너 이미지를 빌드하고 GHCR에 게시한 뒤, 이미지 태그를 변경하여 Argo CD가 새로운 버전을 배포하는 전체 흐름을 확인하였다.

모니터링 실습에서는 Metrics Server로 CPU와 메모리 사용량을 확인하고, Prometheus와 Grafana를 설치하여 시각적으로 관찰하였다.

BusyBox를 이용한 부하 테스트에서는 실제 CPU 사용량이 증가하는 것을 확인하였으며, 로그 조회 명령어를 통해 애플리케이션의 요청 처리 상태도 확인하였다.

이번 실습을 통해 CI/CD 자동화뿐 아니라 배포 이후 시스템 상태를 확인하고 관리하는 과정도 중요하다는 것을 배웠다.

