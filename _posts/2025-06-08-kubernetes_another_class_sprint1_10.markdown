---
layout: post
title:  "배포를 시작할 때 고려해야할 요소"
date: 2025-06-14
categories: [kubernetes, container, devops]
description: "[Inflearn Warm-Up Club - DevOps] 매일 1% 성장하기"
comments: true
bird_image: "ostrich.webp"
bird_name: "타조 (Ostrich)"
bird_scientific_name: "Struthio camelus"
bird_description: "타조는 세계에서 가장 큰 새로, 날지는 못하지만 강한 다리로 시속 70km에 달하는 빠른 속도로 달릴 수 있다. 아프리카의 사막과 초원 지역에 서식하며, 긴 목과 다리, 커다란 눈이 특징이다. 알도 세계에서 가장 큰 크기를 자랑한다. 방어 시 강한 발차기를 하며, 초식성으로 식물의 씨앗, 잎, 열매 등을 먹는다."
---

> [**쿠버네티스 어나더 클래스 (지상편) - Sprint 1, 2**](https://www.inflearn.com/course/%EC%BF%A0%EB%B2%84%EB%84%A4%ED%8B%B0%EC%8A%A4-%EC%96%B4%EB%82%98%EB%8D%94-%ED%81%B4%EB%9E%98%EC%8A%A4-%EC%A7%80%EC%83%81%ED%8E%B8-sprint1)

# 배포를 시작할 때 고려해야할 요소
최신 솔루션을 많이알고, 신규 업데이트된걸 빨리써보는게 중요한게 아니다. 환경에 맞는 기술을 단계적으로, 알잘딱깔센하게 적용하는 것이 더 중요하다.

## CI/CD 파이프라인 구성 시 고려사항
### 1. 관리 책임: 통합 vs 분리

<img src="{{ '/assets/images/20250616_ci_cd_flow1.png' | prepend: site.baseurl }}" alt="ci cd flow 1">

CI/CD의 주요 흐름은 다음과 같다
소스 빌드 → 컨테이너 빌드 → 배포

예를 들어 Jenkins는 이 모든 단계를 하나의 파이프라인으로 통합해서 처리할 수 있다. 하지만 조직 규모가 커지고 각 단계의 담당자가 분리되면, 파이프라인을 기능별로 나누는 것이 관리 측면에서 유리하다. 즉, 자동화는 트리거로 처리되더라도, 관리/운영 주체를 고려해 단계 분리를 고민해야 한다.

### 2. 운영 정책(단일 VS 분리)

<img src="{{ '/assets/images/20250616_ci_cd_flow2.png' | prepend: site.baseurl }}" alt="ci cd flow 2">

ArgoCD 같은 툴을 사용할 때, 하나의 ArgoCD 인스턴스로 여러 클러스터를 관리(1:N)할 수도 있고, 클러스터마다 개별적으로 구성(1:1)할 수도 있다.

|구성 방식|장점|단점|
|---|---|---|
|1:N|운영 편리, 인프라 절감|장애 시 전체 영향|
|1:1|장애 영향 분산|이중 관리 부담|

운영 정책에 따라 장애 영향도와 관리 효율성을 비교해 선택해야 한다.

### 3. 제품 선정 기준
도구를 고를 때는 단순히 유명세가 아니라, 보안 정책, 온라인/오프라인 환경, 참고 자료(레퍼런스) 유무 등을 기준으로 선택하는 것이 중요하다.

|구분|온라인 도구|오프라인 도구|
|---|---|---|
|CI/CD 통합|GitHub Actions|Jenkins, JenkinsX|
|CI 전용|CircleCI|GitLab CI|
|CD 전용|-|ArgoCD, Spinnaker|

> #### 컨테이너 빌드 툴 선택 시 주의사항
> Docker는 데몬 기반이라 무겁다. CI/CD 서버에서 도커를 직접 실행하지 않도록, 데몬리스 빌더(예: buildah, kaniko) 도입이 고려되기도 한다. 다만 아직까지 현업에서는 Docker가 주류이므로, 전환에는 신중함이 필요하다.

## 배포 전략 고려사항
배포 전략은 크게 네 가지로 나눌 수 있다. 각 전략을 선택할 때는 서비스의 특성과 환경을 고려해, 어떤 유즈케이스에 적합한지 잘 파악하는 것이 중요하다.

<img src="{{ '/assets/images/20250616_deploy_img.png' | prepend: site.baseurl }}" alt="deploy image">

### Recreate
기존 버전을 완전히 종료한 후 새로운 버전을 배포하는 방식, 자동 배포가 가능하지만, 트래픽 제어가 불가능하고 다운타임이 발생한다.

### Rolling Update
기존 인스턴스를 점진적으로 교체하는 방식, 자동 배포가 가능하며 다운타임은 없지만, 트래픽을 세밀하게 제어하기는 어렵다.

### Blue/Green
새로운 버전을 별도의 환경(Green)에 배포하고, 충분히 검증한 뒤 기존 운영 환경(Blue)과 트래픽 스위칭하는 방식, 운영 환경에서만 테스트가 가능한 경우에 적합하고, 수동 배포 시 빠른 롤백이 가능하다. 스크립트를 활용해 자동화할 수도 있다.

### Canary
일부 사용자만 새로운 버전(v2)을 사용하게 하여 점진적으로 배포하는 방식, 헤더 값, IP, 사용자, 언어 등 조건에 따라 v2 트래픽 유입을 제한할 수 있다. 콜드 스타트 방지(초기 지연 완화), 두 버전의 비교, A/B 테스트 등에 적합하다.

> #### 콜드 스타트 방지
> 초기 배포 시 인스턴스가 준비되지 않아 응답이 느려지는 현상을 방지하기 위해, 일부 트래픽을 미리 보내 인스턴스를 예열하는 것

---

## 실습 미션

<details>
<summary>Docker, Containerd 커맨드 사용해보기</summary>
<div markdown="1">

### Docker

#### 0.실습 준비

```shell
# 도커 파일 및 App 소스 다운로드
curl -O https://raw.githubusercontent.com/k8s-1pro/install/main/ground/etc/docker/Dockerfile
curl -O https://raw.githubusercontent.com/k8s-1pro/install/main/ground/etc/docker/hello.js
```

#### 1. 빌드
```shell
docker build -t <docker hub ID>/hello:1.0.0 .
```

<img src="{{ '/assets/images/20250617_mission_01.png' | prepend: site.baseurl }}" alt="mission 01">

#### 2. 이미지 리스트 조회
```shell
docker image list
```

<img src="{{ '/assets/images/20250617_mission_02.png' | prepend: site.baseurl }}" alt="mission 02">

#### 3. 태그 변경
```shell
docker tag <docker hub ID>/hello:1.0.0 <docker hub ID>/hello:2.0.0

## 확인
docker image list
```

<img src="{{ '/assets/images/20250617_mission_03.png' | prepend: site.baseurl }}" alt="mission 03">

#### 4. 로그인 및 이미지 업로드
```shell
docker login -u <docker hub ID>
docker push <docker hub ID>/hello:1.0.0
```

<img src="{{ '/assets/images/20250617_mission_04.png' | prepend: site.baseurl }}" alt="mission 04">

#### 5. 이미지 삭제
```shell
docker rmi <docker hub ID>/hello:1.0.0
docker rmi <docker hub ID>/hello:2.0.0
```

<img src="{{ '/assets/images/20250617_mission_05.png' | prepend: site.baseurl }}" alt="mission 05">

#### 6. 이미지 다운로드
```shell
docker pull <docker hub ID>/hello:1.0.0
```

<img src="{{ '/assets/images/20250617_mission_06.png' | prepend: site.baseurl }}" alt="mission 06">

#### 7. 이미지 → 파일 변환
```shell
docker save -o file.tar <docker hub ID>/hello:1.0.0
ls file.tar
```

<img src="{{ '/assets/images/20250617_mission_07.png' | prepend: site.baseurl }}" alt="mission 07">

#### 8. 파일 → 이미지 변환
```shell
docker load -i file.tar
```

<img src="{{ '/assets/images/20250617_mission_08.png' | prepend: site.baseurl }}" alt="mission 08">


### Containerd

> 참고: containerd는 이미지·컨테이너 등을 "네임스페이스"라는 단위로 구분해서 관리한다. 쿠버네티스는 containerd 내부에서 오직 k8s.io 네임스페이스만 사용한다. 따라서 ctr 명령어로 default 같은 다른 네임스페이스에 이미지를 pull하거나 컨테이너를 만들면 쿠버네티스는 사용할 수 없다.

#### 1. 네임스페이스 조회
```shell
## default namespace로 조회됨
ctr ns list
```

<img src="{{ '/assets/images/20250617_mission_09.png' | prepend: site.baseurl }}" alt="mission 09">

#### 2. 특정 네임스페이스 내 이미지 조회
```shell
ctr -n k8s.io image list
```

<img src="{{ '/assets/images/20250617_mission_10.png' | prepend: site.baseurl }}" alt="mission 10">

#### 3. 다운로드 및 이미지 확인 (이미지는 default라는 네임스페이스에 다운 받아진다.)
```shell
ctr images pull docker.io/<docker hub ID>/hello:1.0.0
ctr ns list
crt image list
```

<img src="{{ '/assets/images/20250617_mission_11.png' | prepend: site.baseurl }}" alt="mission 11">

#### 4. 태그 변경
```shell
ctr images tag docker.io/<docker hub ID>/hello:1.0.0 docker.io/<docker hub ID>/hello:2.0.0
ctr images list
```

<img src="{{ '/assets/images/20250617_mission_12.png' | prepend: site.baseurl }}" alt="mission 12">

#### 5. 업로드
```shell
ctr image push docker.io/<docker hub ID>/hello:2.0.0 --user <docker hub ID>
```

<img src="{{ '/assets/images/20250617_mission_13.png' | prepend: site.baseurl }}" alt="mission 13">

#### 6. 이미지 (namespace : default) -> 파일로 변환 
```shell
ctr -n default image export file.tar docker.io/<docker hub ID>/hello:1.0.0
ls
```

<img src="{{ '/assets/images/20250617_mission_14.png' | prepend: site.baseurl }}" alt="mission 14">

#### 7. 파일 -> 이미지로 변환 (namespace : k8s.io)
```shell
ctr -n k8s.io image import file.tar
ctr -n k8s.io image list | grep hello
```

<img src="{{ '/assets/images/20250617_mission_15.png' | prepend: site.baseurl }}" alt="mission 15">

#### 8. 삭제 (namespace : k8s.io)
```shell
ctr -n k8s.io image remove docker.io/<docker hub ID>/hello:1.0.0
ctr -n k8s.io image list | grep hello
```

<img src="{{ '/assets/images/20250617_mission_16.png' | prepend: site.baseurl }}" alt="mission 16">

</div>
</details>


<details>
<summary>Docker, Containerd의 이미지 크기가 다른 이유</summary>
<div markdown="1">

### 이미지 크기 확인

#### Docker
```shell
docker pull 1pro/api-tester:latest
docker image list
```

<img src="{{ '/assets/images/20250617_mission_18.png' | prepend: site.baseurl }}" alt="mission 18">


#### Containerd
```shell
ctr image pull docker.io/1pro/api-tester:latest
ctr image list
```

<img src="{{ '/assets/images/20250617_mission_17.png' | prepend: site.baseurl }}" alt="mission 17">

### 가설 1: 플랫폼(amd64, arm64) 차이 때문에 이미지 크기가 달라졌을 것이다.
#### 검증
```shell
# Docker
docker image inspect 1pro/api-tester:latest |grep Arch

# Containerd
ctr image list
```

#### 결과

<img src="{{ '/assets/images/20250617_mission_19.png' | prepend: site.baseurl }}" alt="mission 19">
<img src="{{ '/assets/images/20250617_mission_20.png' | prepend: site.baseurl }}" alt="mission 20">

- Containerd의 PLATFORMS는 지원 플랫폼을 보여줄 뿐, 실제 받은 이미지가 여러 플랫폼이라는 의미는 아님. 이미지를 받아온 환경에 따른 이미지를 받아오므로 플랫폼 차이 없음.

### 가설 2: Layer 공유로 인해 기존에 있던 Layer는 다시 받지 않아서 크기가 줄었을 것이다.
#### 테스트 방법
1. Docker 이미지 → tar 저장 → containerd로 import
```shell
docker image list
docker save -o docker-image.tar 1pro/api-tester:latest
ls -lh docker-image.tar
ctr image import docker-image.tar
ctr image list
```

2. containerd 이미지 → tar 저장 → Docker로 import
```shell
ctr image pull docker.io/1pro/api-tester:latest
ctr image list
ctr image export containerd-image.tar docker.io/1pro/api-tester:latest
ls -lh containerd-image.tar
docker load -i containerd-image.tar
docker image list
```

3. 크기 비교

#### 결과
- Docker → containerd: 크기 거의 동일

<img src="{{ '/assets/images/20250617_mission_21.png' | prepend: site.baseurl }}" alt="mission 21">

- Containerd → Docker: 크기 증가

<img src="{{ '/assets/images/20250617_mission_22.png' | prepend: site.baseurl }}" alt="mission 22">


### 결론: Runtime이 달라서 이미지 크기가 달라질 수 있다.
Docker는 다른 많은 기능들을 지원해 주기 때문에 실제 이미지를 다운 받은 이후 자신의 매타데이터 규격에 맞게 데이터들을 더 추가하고 이미지를 재구성하기 때문에 크기가 달라질 수 있다. 즉, Containerd를 사용하는 Kubernetes 환경에서는 Docker로 빌드한 이미지를 그대로 사용하는 것이 이미지 용량 최적화에 도움이 되지 않을 수 있다.

</div>
</details>



