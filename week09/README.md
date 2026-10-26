# Week 09 - Kubernetes 실습

## 1. 실습 환경
- Ubuntu 24.04 (WSL2)
- Docker 29.1.3
- kubectl v1.37.1
- Minikube v1.39.0
- Kubernetes v1.37.0

## 2. Minikube 클러스터 구성
Docker 드라이버를 사용하여 Minikube 클러스터를 실행하였다.

실행 명령어:
- minikube start --driver=docker
- minikube status
- kubectl get nodes
- kubectl get pods -A

노드가 Ready 상태인 것을 확인하였다.

## 3. Pod 생성 및 관리
Nginx 이미지를 이용해 mynginx Pod를 생성하였다.

실행 명령어:
- kubectl run mynginx --image=nginx
- kubectl get pods
- kubectl describe pod mynginx
- kubectl logs mynginx
- kubectl exec -it mynginx -- bash

## 4. Deployment 자가 치유 실습
Nginx Deployment의 Pod 3개 중 하나를 삭제하였다.

실행 명령어:
- kubectl delete pod mynginx-668c4f948b-cncwv
- kubectl get pods -w

삭제된 Pod 대신 새로운 Pod가 자동 생성되었으며, 총 3개가 유지되는 것을 확인하였다.

## 5. Guestbook 배포
GHCR에 등록한 방명록 이미지를 Kubernetes에 배포하였다.

사용 이미지:
ghcr.io/yejun5380/guestbook:v2

실행 명령어:
- kubectl delete deployment mynginx
- kubectl create deployment guestbook --image=ghcr.io/yejun5380/guestbook:v2
- kubectl get pods -w
- kubectl get deployments
- kubectl logs -l app=guestbook

Deployment가 READY 1/1 상태로 정상 실행되었다.

## 6. 브라우저 접속
실행 명령어:
- kubectl port-forward deployment/guestbook 8080:5000

http://localhost:8080 에 접속하여 방명록 화면이 정상 출력되는 것을 확인하였다.

## 7. Context 확인
실행 명령어:
- kubectl config get-contexts
- kubectl config use-context minikube
- kubectl config current-context

현재 Context가 minikube임을 확인하였다.

## 8. 실습 결과
이번 실습을 통해 Kubernetes의 Pod, Deployment, 자가 치유 기능을 이해하였다.
기존에 제작한 Guestbook 이미지를 Kubernetes에 배포하고 브라우저에서 정상 동작하는 것도 확인하였다.
