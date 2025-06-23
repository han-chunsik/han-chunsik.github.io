---
layout: post
title:  "배포 파이프라인 구축 - Argo"
date: 2025-06-22
categories: [kubernetes, container, devops]
description: "[Inflearn Warm-Up Club - DevOps] 매일 1% 성장하기"
comments: true
bird_image: "pionus.webp"
bird_name: "피어니스 (Pionus)"
bird_scientific_name: "Pionus"
bird_description: "피오니스 앵무는 중남미에 서식하는 중형 앵무새로, 차분한 성격과 상대적으로 조용한 소리로 잘 알려져 있다. 대표적으로 청머리피오니스는 파란 머리와 초록빛 몸통이 특징이며, 사육 난이도가 낮고 애완조로도 인기가 많다."
---

> [**쿠버네티스 어나더 클래스 (지상편) - Sprint 1, 2**](https://www.inflearn.com/course/%EC%BF%A0%EB%B2%84%EB%84%A4%ED%8B%B0%EC%8A%A4-%EC%96%B4%EB%82%98%EB%8D%94-%ED%81%B4%EB%9E%98%EC%8A%A4-%EC%A7%80%EC%83%81%ED%8E%B8-sprint1)

> 출처: [큐브옵스 - ArgoCD 빠르게 레벨업 하기 (1/3)](https://cafe.naver.com/f-e/cafes/30725715/articles/118?menuid=13&referrerAllArticles=false), [큐브옵스 - ArgoCD 빠르게 레벨업 하기 (2/3)](https://cafe.naver.com/f-e/cafes/30725715/articles/121?menuid=13&referrerAllArticles=false) , [큐브옵스 - ArgoCD 빠르게 레벨업 하기 (3/3)](https://cafe.naver.com/f-e/cafes/30725715/articles/125?menuid=13&referrerAllArticles=false)   
> 본 글은 출처를 참고하여 테스트를 진행한 결과를 바탕으로 작성하였습니다. 설치 및 실습 과정은 원문을 참고하시고, 이 글에서는 핵심 개념 정리와 추가 설명에 집중하였습니다.

# 배포 파이프라인 구축 실습 - Argo

<img src="{{ '/assets/images/20250623_argo_flow.png' | prepend: site.baseurl }}" alt="argo flow">

안정적인 배포를 위해서는 다양한 환경을 고려한 배포 전략의 개선이 필요하다. 쿠버네티스 기본 기능 외에도 Argo와 같은 배포 도구를 활용하면 보다 신뢰성 있는 배포가 가능하다. 이러한 새로운 솔루션은 각 단계를 직접 경험하며 기능을 충분히 이해한 뒤, 실제로 필요하다고 판단될 때 도입하는 것이 바람직하다.

## Argo 프로젝트 구성 요소
### Argo CD
Kubernetes 전용 GitOps 배포 도구로 모든 애플리케이션 변경 관리는 Git 저장소를 기준으로 수행된다.

### Argo Image Updater
컨테이너 이미지의 태그 변경(예: 최신 버전 등)을 감지하여 자동으로 Git 리포지토리의 manifest를 업데이트한다.

### Argo Rollouts
Canary, Blue-Green 등 고급 배포 전략을 지원하는 컨트롤러로 기존 Deployment보다 세분화된 롤링 전략을 구현할 수 있다.

### Argo Events
Kafka 등 메시지 브로커와 유사한 이벤트 버스 아키텍처를 제공한다. 외부 시스템 혹은 내부 워크플로우 간의 이벤트 전달 및 트리거 역할을 한다.

### Argo Workflows
Airflow, Kubeflow와 유사한 워크플로우 엔진이다. 작업 실행 순서를 정의할 수 있으며, 이벤트 기반으로 트리거되기도 한다. 이를 통해 배포, 테스트, 데이터 처리 등 다양한 파이프라인을 구성할 수 있다.

## Argo CD 기본 개념
### 구성 요소
#### Application
- Argo CD에서 **배포 단위**
- Jenkins의 Job과 비슷한 개념
- 애플리케이션 하나당 Application 리소스가 생성됨

#### Project
- Application들을 논리적으로 묶는 단위
- 기본값은 `default`
- Kubernetes의 네임스페이스와 유사한 개념

#### Source
- Git 저장소 정보
- Git URL, 브랜치, 디렉토리 경로 등을 설정

#### Destination
- 배포 대상 클러스터 정보
- 클러스터 이름, 네임스페이스 등을 설정

### Argo CD UI 주요 기능
#### Refresh 버튼
- Git 변경사항을 **즉시 확인**
- Argo CD는 주기적으로 Git 변경사항을 확인하지만, 수동 갱신도 가능

#### Sync 버튼
- Git의 리소스를 **Kubernetes에 실제로 배포**
- 이때 `Desired Manifest` 기준으로 `Live Manifest`를 동기화

#### Diff 버튼
- Git과 클러스터 간 리소스 차이가 있을 경우에만 활성화
- 수동으로 클러스터에 추가된 리소스는 Git 기준이 없어 diff가 발생하지 않음


### Sync 정책 및 옵션
#### Sync Policy
- 변경 감지 시 **자동으로 Sync할지**, **수동으로 할지** 설정 가능
- Auto 옵션을 사용하면 Git 변경 즉시 자동 반영 가능

#### Sync Options
- 네임스페이스 자동 생성 여부
- Hook 관련 설정 등 세부 배포 제어 옵션 포함

#### General
- 이름, 레이블, 태그 등 **일반 정보** 설정

### 배포 지원 방식
Argo CD는 다음과 같은 디렉토리 구조 또는 도구를 자동 인식하여 배포를 지원한다.
- `Helm` chart 디렉토리
- `Kustomization.yaml`이 포함된 디렉토리
- 단순 `YAML` 파일 모음
Argo가 자동으로 어떤 방식인지 판단하고 배포를 수행한다.

### Manifest 개념
#### Desired Manifest
- Git 저장소에서 내려받은 리소스 정의
- **읽기 전용**, 수동 수정 불가
- Git의 버전 이력을 그대로 반영

#### Live Manifest
- Kubernetes 클러스터에 실제 존재하는 리소스 상태
- 수동으로 kubectl 등으로 수정 가능

#### 동작 관계
- `Sync` 수행 시 → Desired 상태 기준으로 Live를 맞춤
- `kubectl edit` 등 수동 변경 시 → Live만 변경됨
- 둘 간 차이 있을 경우 UI에서 Diff 표시됨

> ####  배포 파이프라인 레포지토리 분리 전략
> 애플리케이션 소스 코드, 애플리케이션 배포를 위한 릴리즈 파일, ArgoCD와 같은 Kubernetes Addon 설치용 파일은 각각 전용 레포지토리로 분리하여 관리하는 것이 좋다. 이러한 분리는 사용자별 접근 권한을 명확히 관리할 수 있으며, 불필요한 코드 다운로드를 방지할 수 있다는 장점이 있다.

> #### ArgoCD Sync Policy
> Prune resources: git에서 리소스 삭제시 실제 kubernetes에서도 자원 삭제  
> Self heal: Auto Sync 상태에서 항상 Git에 있는 내용이 적용됨(Argo, k8s에 직접 수정한 내용 삭제됨)


## ArgoCD Image Updater를 이용한 이미지 자동 배포

<img src="{{ '/assets/images/20250623_argo_image_updater_flow.png' | prepend: site.baseurl }}" alt="argo image updater flow">

### 배포가 필요한 상황
서비스 운영 중 다음과 같은 이유로 배포가 필요해진다.
- Deployment의 배포 전략을 수정해야 할 때
- 수동으로 replica 수를 조정할 때
- 애플리케이션 버전이 업그레이드되어 이미지 변경이 필요할 때

이 중 리소스 스펙 변경은 Git 매니페스트 수정 → 수동 배포라는 흐름이 필요하다. 반면, 이미지 버전 변경은 비교적 자동화하기 쉬운 작업이다.

### ArgoCD의 GitOps 배포의 한계
ArgoCD는 GitOps 방식으로 동작하므로, 실제 배포를 트리거하려면 Git 저장소의 변경이 반드시 필요하다. 컨테이너 이미지만 바뀌어도 Git에 그 변경이 반영되지 않으면 배포는 이루어지지 않는다. 결국 단순한 이미지 교체조차도 매번 Git을 수동으로 수정해야 하는 불편이 발생한다.

### Argo CD Image Updater
이 문제를 해결하기 위해 Argo CD Image Updater를 도입할 수 있다.
Image Updater는 다음과 같은 흐름으로 자동화를 가능하게 한다.
1. 레지스트리 모니터링: Docker Hub 또는 다른 이미지 레지스트리를 주기적으로 확인
2. 새 이미지 태그 감지 시: Git 저장소의 Deployment manifest 파일 내 이미지 태그를 자동으로 수정
3. ArgoCD가 변경 감지 후 배포 수행: 이 방식은 GitOps의 원칙을 유지하면서도, 이미지 업그레이드를 자동화할 수 있다는 장점이 있다.

> 이미지 업데이터는 업데이트할 이미지 태그 대상을 지정할 수 있으며, 기본 대상 설정 외에도 semver(가장 높은 버전 기준), latest(latest 태그 우선), name(알파벳 순 정렬의 마지막 태그), digest(SHA 해시 비교) 등의 update-strategy와 태그 정규식을 활용해 세부적인 업데이트 조건을 설정할 수 있다.

## Argo Rollouts
Argo Rollouts는 **블루그린(Blue-Green)**과 **카나리(Canary)** 두 가지 배포 전략을 제공한다. Argo CD 없이도 단독으로 사용할 수 있으며, 자체 대시보드도 제공한다. Argo CD에서 Rollouts를 연동해 사용할 수 있을 뿐, 두 서비스는 독립적으로 운영된다.

### 블루그린(Blue-Green) 배포

<img src="{{ '/assets/images/20250623_rollouts_blue_green.png' | prepend: site.baseurl }}" alt="rollout blue green">

Argo Rollouts의 블루그린 배포 방식은 두 개의 Kubernetes 서비스를 지정하는 방식으로 작동한다. 하나는 **액티브 서비스**로 실제 사용자 트래픽을 처리하고, 다른 하나는 **프리뷰 서비스**로 새 버전을 테스트하는 데 사용된다.
  
버전을 업데이트하면 새로운 ReplicaSet(v2)이 생성되고, 액티브 서비스는 기존 버전(v1)을 그대로 바라보며, 프리뷰 서비스는 새로운 버전(v2)을 바라본다. Rollouts는 이 과정에서 서비스 셀렉터를 자동으로 전환하여 수동 개입 없이 배포 전환을 가능하게 한다.

### 카나리(Canary) 배포

<img src="{{ '/assets/images/20250623_rollout_canary.png' | prepend: site.baseurl }}" alt="rollout canary">

Argo Rollouts는 NGINX, Istio 등과 연동해 정교한 트래픽 분산이 가능한 카나리 배포도 지원한다. Rollout의 카나리 방식은 하나의 서비스만 사용하며, 새로운 버전 배포 시 Rollout 전략을 `Canary`로 설정하고 트래픽 비율 등을 세분화해 조절한다.
  
예를 들어 `setWeight: 33`으로 설정하면 전체 트래픽의 33%가 v2로 전달된다. 이후 수동으로 `promote` 명령을 수행하면 다음 단계로 넘어가며, 설정된 `pause` 시간만큼 대기한 후 점진적으로 트래픽이 전환된다. 마지막에는 이전 버전(v1)이 삭제된다.
  
기본적인 롤링 업데이트와 유사하지만, Rollouts는 배포 중간에 수동 중단이 가능하여 더욱 세밀한 카나리 배포 전략 구성이 가능하다.

## 실습 미션

<details>
<summary>ArgoCD Github 업데이트</summary>
<div markdown="1">

</div>
</details>

# 회고
여러 가지 배포 파이프라인 구축 방법에 대해 개념을 정리하고, 실습을 통해 각 도구들의 필요성과 사용 목적을 체감할 수 있었다. 단순히 "이 도구를 쓰면 된다"가 아니라, 어떤 문제를 해결하기 위해 이 도구를 선택하게 되었는지, 각 단계에서 어떤 역할을 하는지를 이해하는 데 집중할 수 있었던 시간이었다.  
무엇보다도, 새로운 기술을 익힐 때 단순히 빠르게 구축해보는 것도 중요하지만, 직접 수동으로 구성해보면서 내부 동작을 이해하는 과정이 훨씬 큰 학습이 된다는 점을 다시금 느꼈다. 앞으로도 단순히 결과만 보지 않고, 그 과정과 맥락을 함께 공부하는 태도를 유지하고 싶다.