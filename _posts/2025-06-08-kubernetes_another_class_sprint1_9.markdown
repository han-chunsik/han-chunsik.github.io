---
layout: post
title:  "DevOps 데브옵스"
date: 2025-06-13
categories: [kubernetes, container, devops]
description: "[Inflearn Warm-Up Club - DevOps] 매일 1% 성장하기"
comments: true
bird_image: "eurasian_wren.webp"
bird_name: "굴뚝새 (Eurasian wren)"
bird_scientific_name: "Troglodytes troglodytes"
bird_description: "굴뚝새는 몸집이 작고 동그란 형태에, 짧은 꼬리를 위로 세우는 특징이 있는 새다. 주로 숲 속이나 덤불, 돌 틈, 나무 구멍 등에서 서식하며, 높은 음의 특유의 맑고 빠른 소리로 울어 존재를 알린다. ‘굴뚝’처럼 좁고 어두운 곳에 둥지를 트는 습성 때문에 이름이 붙었으며, 한국에서는 텃새 또는 겨울철새로 관찰된다."
---

> [**쿠버네티스 어나더 클래스 (지상편) - Sprint 1, 2**](https://www.inflearn.com/course/%EC%BF%A0%EB%B2%84%EB%84%A4%ED%8B%B0%EC%8A%A4-%EC%96%B4%EB%82%98%EB%8D%94-%ED%81%B4%EB%9E%98%EC%8A%A4-%EC%A7%80%EC%83%81%ED%8E%B8-sprint1)

데브옵스는 개발에서 운영까지 원활한 흐름을 만드는 것으로, 오늘날에는 안정기를 넘어, 각 산업군과 기술 스택에 따라 다양하게 확장되고 있으며, 조직의 특성과 개발 언어에 따라 파이프라인 구성 방식도 달라진다. 

# 데브옵스 핵심
데브옵스의 핵심은 개발 → 빌드 → 실행 파일 생성 → 배포의 흐름을 빠르고 안정적으로 이어주는 데 있다.

<img src="{{ '/assets/images/20250614_devops_pipeline.png' | prepend: site.baseurl }}" alt="devops pipeline">


## 개발 환경
- 개발자는 이 환경에서 코드를 작성하고 빌드한다.
- 로컬 또는 통합 개발 환경에서 기본적인 테스트도 함께 수행한다.

## CI/CD 환경
- 빌드된 실행 파일을 배포 가능한 형태로 인프라 환경에 전달하기 위한 절차가 진행된다.
- 코드 변경 사항을 자동으로 테스트하고 빌드하며, 최종적으로 배포까지 이어지도록 한다.
- Git, Jenkins 등의 도구를 주로 활용한다.

## 인프라 환경
데브옵스에서 애플리케이션이 실제로 동작하는 인프라는 목적에 따라 다음과 같이 구분된다.

### 개발 환경
- 여러 명의 개발자가 함께 사용하는 테스트 목적의 환경이다.
- 자주 변경되며 보안을 제거하거나, 스토리지를 연결하지 않는 등 비교적 자유로운 구성이 가능하다.

### QA 환경
- 운영 환경과 동일한 구성으로, QA 담당자가 테스트를 수행하는 환경이다.
- 실제 서비스 환경과 유사한 조건에서 검증을 진행한다.

### 운영 환경
- 실제 서비스가 제공되는 환경으로, 고가용성과 안정성이 요구된다.
- 이중화 구성을 반드시 포함해야 한다.

# 데브옵스 앱 배포 파이프라인
개발 후 커밋을 하는 순간 운영 환경에서 앱이 자동으로 배포되는 파이프라인을 나누면 다음과 같은 단계로 나누어진다.

## 개발 환경
### 계획(이슈 관리 및 일정 공유)
- 개발일정 공유 및 이슈 사항 공유
- 주요 도구: Jira, Notion, Slack, Redmine

### 개발(코드 작성 및 테스트)
- 코드 개발 및 코드 분석, 개발 서포트
- 주요 도구: Spring Boot, Junit, GitHub, ItelliJ

## CI(빌드, 테스트 자동화)
### 빌드(코드 → 실행 파일/이미지)
- 소스 코드를 빌드하고 도커 이미지로 패키징
- 주요 도구: Gradle, Maven, Docker

### 테스트 - (정상 동작 여부 확인)
- 코드 병합 후 빌드 단계에서 기능, 성능, 커버리지 테스트 수행
- 주요 도구: Junit, JMeter

## CD(배포 자동화)
### 릴리즈(배포 전 단계)
- 도커 이미지 및 매니페스트 생성
- 주요 도구: Docker, Kubernetes

### 배포(실제 서비스 반영)
- 릴리즈된 결과물을 실제 클러스터에 반영
- 주요 도구: Helm, ArgoCD

## 운영 환경
### 운영(서비스 운영, 네트워크 관리)
- 서비스 실행, 트래픽 제어, 인프라 관리
- 주요 도구: ETCD, Istio, Nginx, Containerd

### 모니터링(서비스 상태 및 로깅)
- 리소스 사용량, 앱 로그, 트래픽 흐름 확인
- 주요 도구: Prometheus, Grafana, fluentd

> #### DevOps 외 다른 Ops들    
> - GitOps: 깃(Git)을 단일 소스로 삼아 인프라와 애플리케이션 배포를 자동화하고 일관성 유지
> - DevSecOps:개발부터 배포까지의 모든 과정에 보안을 통합하여 보안 위협을 사전에 방지
> - MLOps: 머신러닝 모델의 개발부터 운영까지의 전체 과정을 자동화하고 안정적으로 관리
> - LLMOps: 대규모 언어 모델(LLM)의 학습, 배포, 관리 과정을 효율적으로 운영
> - FinOps: 클라우드 비용을 모니터링하고 최적화하여 재무적 책임과 효율성을 동시에 달성

# CI/CD 테스트 환경 구성

> 출처: [큐브옵스 - 손쉽게 데브옵스 환경을 구축하는 방법](https://cafe.naver.com/kubeops/111) 

## [1. VM 초기 세팅]({{ "/2025-05-31/kubernetes_another_class_sprint1_4/#%ED%99%98%EA%B2%BD-%EA%B5%AC%EC%84%B1" | prepend: site.baseurl }})

### 호스트 환경
- **운영체제**: macOS (Apple Silicon - M3 Pro)
- **가상화 도구**: [UTM](https://mac.getutm.app/)

### 가상 머신 환경
- **운영체제**: Rocky Linux 9.2
- **CPU**: 2Core
- **메모리**: 2GB
- **디스크**: 32GB

## 2. CI/CD 서버 설치

<details>
<summary>빠른 설치를 위한 스크립트</summary>
<div markdown="1">

```shell
echo '======== [1] Rocky Linux 기본 설정 ========'
echo '======== [1-1] 패키지 업데이트 ========'
# 강의와 동일한 실습 환경을 유지하기 위해 Linux Update는 하지 마세요!
# yum -y update # (x)

echo '======== [1-2] 타임존 설정 ========'
timedatectl set-timezone Asia/Seoul

echo '======== [1-3] 방화벽 해제 ========'
systemctl stop firewalld && systemctl disable firewalld


echo '======== [2] Kubectl 설치 ========'
echo '======== [2-1] repo 설정 ========'
cat <<EOF | sudo tee /etc/yum.repos.d/kubernetes.repo
[kubernetes]
name=Kubernetes
baseurl=https://pkgs.k8s.io/core:/stable:/v1.27/rpm/
enabled=1
gpgcheck=1
gpgkey=https://pkgs.k8s.io/core:/stable:/v1.27/rpm/repodata/repomd.xml.key
exclude=kubelet kubeadm kubectl cri-tools kubernetes-cni
EOF

echo '======== [2-2] Kubectl 설치 ========'
yum install -y kubectl-1.27.2-150500.1.1.aarch64 --disableexcludes=kubernetes


echo '======== [3] 도커 설치 ========'
# https://download.docker.com/linux/centos/8/x86_64/stable/Packages/ 저장소 경로
yum install -y yum-utils
yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo
yum install -y docker-ce-3:23.0.6-1.el9.aarch64 docker-ce-cli-1:23.0.6-1.el9.aarch64 containerd.io-1.6.21-3.1.el9.aarch64
systemctl daemon-reload
systemctl enable --now docker

echo '======== [4] OpenJDK 설치  ========'
yum install -y java-17-openjdk

echo '======== [5] Gradle 설치  ========'
yum -y install wget unzip
wget https://services.gradle.org/distributions/gradle-7.6.1-bin.zip -P ~/
unzip -d /opt/gradle ~/gradle-*.zip
cat <<EOF |tee /etc/profile.d/gradle.sh
export GRADLE_HOME=/opt/gradle/gradle-7.6.1
export PATH=/opt/gradle/gradle-7.6.1/bin:${PATH}
EOF
chmod +x /etc/profile.d/gradle.sh
source /etc/profile.d/gradle.sh

echo '======== [6] Git 설치  ========'
# 기존엔 git-2.43.0-1.el8 버전을 Fix하였으나 Repository에 최신 버전만 업로드 됨으로 수정
yum install -y git

echo '======== [7] Jenkins 설치  ========'
wget -O /etc/yum.repos.d/jenkins.repo https://pkg.jenkins.io/redhat-stable/jenkins.repo
rpm --import https://pkg.jenkins.io/redhat-stable/jenkins.io-2023.key
yum install -y jenkins-2.440.3-1.1
systemctl enable jenkins
systemctl start jenkins

# Jenkns 설치 후 OpenSSL 최신 버전으로 업데이트
yum update -y openssh-server openssh-clients openssl openssl-libs
systemctl restart sshd
```

</div>
</details>

## 3. CI/CD 환경 설정 및 실습
- 출처 [링크](https://cafe.naver.com/kubeops/111) 참고