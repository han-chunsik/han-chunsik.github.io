---
layout: post
title:  "배포 파이프라인 구축 - helm/kustomize"
date: 2025-06-22
categories: [kubernetes, container, devops]
description: "[Inflearn Warm-Up Club - DevOps] 매일 1% 성장하기"
comments: true
bird_image: "canary.webp"
bird_name: "카나리아 (Canary)"
bird_scientific_name: "Serinus canaria"
bird_description: "카나리아는 작고 밝은 노란색을 띠는 참새목 되새과의 새로, 원래 카나리아 제도, 아조레스 제도, 마데이라 제도에 서식한다. 아름다운 울음소리와 온순한 성격 덕분에 오랜 시간 애완조로 사랑받아 왔다. 특히 수컷은 소리가 청아해 휘파람과 노래를 잘 흉내낸다."
---

> [**쿠버네티스 어나더 클래스 (지상편) - Sprint 1, 2**](https://www.inflearn.com/course/%EC%BF%A0%EB%B2%84%EB%84%A4%ED%8B%B0%EC%8A%A4-%EC%96%B4%EB%82%98%EB%8D%94-%ED%81%B4%EB%9E%98%EC%8A%A4-%EC%A7%80%EC%83%81%ED%8E%B8-sprint1)

> 출처: [큐브옵스 - Helm과 Kustomize 비교하며 사용하기 (1/2)](https://cafe.naver.com/f-e/cafes/30725715/articles/98?boardtype=L&menuid=13&referrerAllArticles=false&page=2), [큐브옵스 - Helm과 Kustomize 비교하며 사용하기 (2/2)](https://cafe.naver.com/f-e/cafes/30725715/articles/104?boardtype=L&menuid=13&referrerAllArticles=false&page=1)    
> 본 글은 출처를 참고하여 테스트를 진행한 결과를 바탕으로 작성하였습니다. 설치 및 실습 과정은 원문을 참고하시고, 이 글에서는 핵심 개념 정리와 추가 설명에 집중하였습니다.

# 배포 파이프라인 구축 실습 - helm/kustomize
쿠버네티스에서 자원을 생성할 때 kubectl 명령어를 많이 사용하지만 CLI 도구일 뿐이다. 주로 리소스 조회나 간단한 수정을 위해 사용되며, 실제 배포 파이프라인에 사용하기에는 적합하지 않다.  
실제 운영에서는 Helm이나 Kustomize 같은 패키지 매니저를 사용하여 선언형으로 작성된 YAML 파일들을 명령어 한 번으로 배포할 수 있는 구조를 만든다. 

## Helm과 Kustomize의 공통점과 차이점

<img src="{{ '/assets/images/20250622_helm_kustomize_compare.png' | prepend: site.baseurl }}" alt="helm kustomize compare">

### 공통점
#### 사용 목적
- 배포에 사용되는 YAML 파일의 중복을 최소화하고 유지보수를 쉽게 하기 위해 사용한다.
- 마이크로서비스 아키텍처로 애플리케이션 종류가 많아지면, 환경에 따라 설정이 달라져야 하며, 다양한 배포 도구에서도 이를 지원하기 위해 패키지 매니저를 사용한다.

### 차이점
#### Helm
- Kustomize에 비해 배포 편의 기능이 많다.
- 마이크로서비스 구성과 다양한 환경별 설정을 한 패키지 안에 담기 쉽다.
- 오픈소스 제품 다수가 Helm Chart를 공식적으로 제공하므로, 운영 환경에 맞춰 쉽게 커스터마이징 가능하다.
- 대규모 프로젝트 (앱 종류가 5개 이상)에서 추천된다.
- helm은 프로젝트 단위보다는 제품 또는 배포 단위의 패키지 관리에 적합하다.

#### Kustomize
- Helm에 비해 배포 편의 기능은 적다.
- 마이크로서비스 구성 또는 환경별 배포 중 하나에 집중하는 것이 좋다.
- 프로젝트 수준에서 간단한 커스터마이징이 필요할 때 적합하다.
- 앱 종류가 5개 미만인 소규모 프로젝트에 적합하다.

### 결론
대부분의 오픈소스는 Helm Chart를 공식 제공하므로, 초반에 학습이 어렵더라도 Helm을 먼저 익히는 것이 유리하다.  
Kustomize는 다음과 같은 경우에 추천된다.
- 처음부터 끝까지 소규모 프로젝트인 경우
- 빠르게 배포 파이프라인을 구축하고, 추후 Helm으로 넘어갈 계획인 경우
- 이미 Kustomize 기반 배포 구조가 존재하고, 이를 분석하거나 확장해야 하는 경우


## 설치 및 구성 방식 비교
### Kustomize
- Kubernetes 1.14 이상부터는 기본 통합되어 별도 설치가 필요 없다.
- **Overlay 방식**으로 패키징한다.
- 공통 설정(base)을 정의하고, 환경별 설정(overlay)을 덮어쓰는 구조다.
- 환경이 많아질수록 파일 수가 증가하고, 구성 관리가 복잡해질 수 있다.

### Helm
- Template + values 파일 구조로 패키징한다.
- 템플릿 파일에 외부 파라미터를 주입하여 리소스를 생성한다.
- **함수형** 처리 방식으로 중복 제거와 조건 분기가 용이하다.
- 복잡한 로직, 조건 처리, 반복 작업에 적합하다.

## Helm 배포 시 체크리스트와 패키지 구조 이해
배포는 작은 실수 하나로 서비스 장애로 이어질 수 있기 때문에 사전에 점검하는 습관이 중요하다. Helm으로 배포하기 전에는 반드시 템플릿이 예상대로 렌더링되는지 확인해야 한다. 배포 과정 중간중간 확인 단계를 두는 것이 실수를 줄이는 핵심 포인트다.

### Helm package 구조
Helm 차트는 다음과 같은 구조로 구성된다.

```yaml
MainChart/                      # 메인 차트 폴더 (APP 단위)
├── Chart.yaml                  # 차트 메타데이터 정의
├── values.yaml                 # 템플릿 변수의 기본값 선언
├── .helmignore                 # 렌더링 제외할 파일 목록
├── templates/                  # 쿠버네티스 리소스 템플릿들
│   ├── deployment.yaml         # 배포 정의
│   ├── service.yaml            # 서비스 정의 (필요 시)
│   ├── NOTES.txt               # 배포 후 출력할 안내 메시지
│   ├── _helpers.tpl            # 전역 템플릿 함수, 변수 정의
│   └── tests/                  # 테스트 리소스 (예: 앱 헬스체크)
├── charts/                     # 서브차트 위치 (의존 차트)
```

```yaml
# Chart.yaml
name: my-chart                  # 차트 이름
description: 간단한 설명
version: 0.1.0                  # 차트 버전
appVersion: "1.0.0"             # 실제 앱 버전
```

Helm을 사용할 때, Chart.yaml, values.yaml, 그리고 Helm CLI에서 넘겨주는 값들은 템플릿 디렉터리(templates/) 내 모든 YAML 파일에서 참조할 수 있다. 이와 함께 _helpers.tpl에 정의된 템플릿 함수들은 templates 내부의 YAML 파일에서 재사용 가능하다.

#### 1. Helm CLI에서 넘긴 값 사용
helm install 명령어로 배포할 때 입력한 값은 템플릿 내에서 .Release 객체로 접근할 수 있다.

```shell
helm install api-tester-2221 ./api-tester -n anotherclass-222
```

이 명령어를 실행하면 deployment.yaml 등에서 다음과 같이 변수가 렌더링된다.

{% raw %}
```yaml
name: {{ .Release.Name }}
namespace: {{ .Release.Namespace }}
```

#### 2. _helpers.tpl 활용
_helpers.tpl은 재사용 가능한 템플릿 조각을 정의하는 파일이다. 주로 변수 조합이나 이름 생성 로직 등을 포함한다.

```
{{- define "api-tester.fullname" -}}  
{{- default .Chart.Name .Values.fullnameOverride | trunc 63 }}  
{{- end }}
```

위와 같이 정의된 함수는 deployment.yaml 등에서 아래처럼 호출하여 사용할 수 있다.

```yaml
metadata:
  name: {{ include "api-tester.fullname" . }}
```

#### 3. Chart.yaml 참조
Chart.yaml에 정의된 메타데이터는 .Chart 객체로 접근할 수 있다.

```yaml
app.kubernetes.io/name: {{ .Chart.Name }}
```

#### 4. values.yaml 참조
values.yaml의 값들은 .Values 객체로 접근할 수 있다.

```yaml
replicas: {{ .Values.replicaCount }}
```

> Helm의 .tpl 파일에서 `{{-` 구문은 템플릿 렌더링 시 불필요한 들여쓰기나 줄바꿈을 제거하라는 의미이다. 특히 앞의 `-`는 해당 블록 이전의 공백이나 줄바꿈을 제거한다. 반대로, 템플릿 블록 끝에 `| nindent <숫자>`를 붙이면 렌더링 결과에서 해당 줄 앞에 공백을 얼마나 넣을지 지정할 수 있다. 이때 숫자 값은 YAML 형식에서 들여쓰기를 맞추기 위한 공백 수를 의미한다.


### Helm 명령어 vs Kubectl 명령어 비교

|Helm 명령어|대응되는 Kubectl 명령어|설명|
|---|---|---|
|helm install|kubectl create|리소스가 없을 때 새로 생성|
|helm upgrade|kubectl patch|기존 리소스가 있을 때 변경 사항만 적용|
|helm upgrade --install|kubectl apply|리소스가 있으면 업데이트, 없으면 생성|

### Helm vs Kustomize 패키지 비교
Helm은 helm create 명령어를 통해 기본 디렉토리 구조와 예제 파일들을 자동으로 생성할 수 있는 반면, Kustomize는 디렉토리 구조와 배포할 YAML 파일들을 직접 구성해야 한다.


#### Kustomize 구조

```yaml
메인 폴더/
├── base/                    # 기본 리소스 정의
│   ├── *.yaml               # 공통 리소스(YAML)
│   └── kustomization.yaml   # 공통 설정 및 리소스 목록
└── overlays/                # 환경별 오버레이
    ├── dev/
    ├── qa/
    └── prod/
```

Kustomize의 주요 특징은 YAML 파일 내부에 변수를 직접 작성하지 않고, kustomization.yaml 파일을 통해 공통 설정이나 배포할 리소스를 지정하고 조합한다는 점이다. Helm은 템플릿 언어를 활용해 변수 치환 방식으로 처리하는 반면, Kustomize는 오버레이 방식으로 YAML을 조합한다.

## Helm 환경별 배포
Helm을 사용해 환경별로 애플리케이션을 배포할 때는, 각 환경에 맞는 values 파일을 따로 작성하여 사용한다.

```
values-dev.yaml   # 개발 환경 설정
values-qa.yaml    # QA 환경 설정
values-prod.yaml  # 운영 환경 설정
```

배포 시에는 -f 옵션으로 해당 환경의 값을 지정한다.

```shell
helm install my-app -f values-dev.yaml .
```

# 배포 파이프라인 구축 후 마주하게 되는 고민
쿠버네티스 환경에 애플리케이션을 배포하다 보면 몇 가지 실무적인 고민과 마주하게 된다. 특히 CI/CD 환경에서 인증서, 이미지 버전 관리, Helm 옵션 활용 등에 대해 명확한 기준이 없으면 장애나 운영 부담으로 이어질 수 있다.

## 1. 인증서와 이미지 인증 정보의 보안 관리
쿠버네티스 배포 시, Docker 이미지를 가져오기 위해 config.json을 사용하는데, 이 과정에서 쿠버네티스 인증서와 config.json의 보안이 중요해진다. 인증서가 탈취되면 클러스터 전체가 위협받을 수 있으므로, Jenkins 같은 CI 도구를 사용할 경우 인증서는 credentials로 등록해 암호화된 방식으로 사용하는 것이 바람직하다. config.json의 경우에도 docker-credential-helpers 같은 도구를 활용해 암호화하는 것이 일반적이다.

## 2. CI/CD 서버 디스크 관리
컨테이너 이미지 빌드가 반복되다 보면 CI/CD 서버의 디스크가 부족해져 죽는 경우가 발생한다. 이를 방지하려면 빌드 후 생성된 불필요한 이미지를 주기적으로 정리하는 스크립트를 운영하거나, 작업 후 자동으로 이미지를 삭제하는 정책을 설정하는 것이 좋다.

## 3. Helm 배포 시 네임스페이스 분리 전략
Helm 차트에서 네임스페이스까지 생성하고 관리할 수 있지만, 실무에서는 네임스페이스를 미리 만들어두고 Helm은 사용만 하는 방식이 더 안전하다.
- 네임스페이스는 팀 또는 서비스 단위로 나뉘기 때문에 관리 단위가 다르다.
- Helm이 네임스페이스를 만들게 하면 권한 문제나 충돌이 발생할 수 있다.
- 특히 CI/CD 환경에서는 보안상 네임스페이스 생성 권한을 주는 것이 위험하다.
따라서 kubectl create namespace로 미리 만들고, Helm은 -n 옵션으로 해당 네임스페이스에 배포하는 것이 좋다.

## 4. --wait 옵션의 활용
Helm의 --wait 옵션은 배포 시 Pod가 정상적으로 기동되어 서비스 가능 상태인지 확인한 후 배포를 종료한다. 초기 기동 검증을 자동화하고 싶은 경우 이 옵션을 활용하는 것이 유용하다.

## 5. 배포했는데도 업그레이드가 되지 않는 문제
Helm은 실제 템플릿에 변경사항이 있어야 업그레이드를 수행한다. 이 때문에 값은 바뀌지 않았지만 재배포가 필요한 경우 metadata.annotations 값을 랜덤하게 변경해 쿠버네티스가 리소스가 변경된 것으로 인식하게 하여 업데이트를 유도할 수 있다.

## 6. 이미지 버전과 Pull 정책 전략
개발 환경과 운영 환경의 성격이 다르기 때문에 이미지 태그와 Pull 정책을 다르게 가져가는 것이 좋다.

### 개발 환경
```
이미지 태그: latest
imagePullPolicy: Always
```

→ 잦은 배포와 빠른 반영이 중요하므로 항상 최신 이미지를 가져오게 한다.

### 운영 및 검증 환경

```
이미지 태그: 명시적 버전(예: v1.2.3, 커밋 해시, 배포 일시 등)
imagePullPolicy: IfNotPresent
```

→ 이미지가 노드에 이미 존재할 경우 재다운로드 없이 사용할 수 있고, 네트워크 문제 시에도 안전하게 Pod를 띄울 수 있다.

이때 중요한 점은, 개발 환경에서도 무작정 latest를 사용하는 것이 아니라, 개발 전용 태그 전략을 도입하는 것이다. 예를 들어 dev-20250622와 같이 날짜 기반 태그를 사용하면 롤백 시점 파악도 가능하고, 운영팀과의 커뮤니케이션도 수월해진다.

## 7. 이미지 정리와 쿠버네티스 가비지 컬렉션
컨테이너 이미지를 자주 빌드하게 되면 사용되지 않는 이미지들이 디스크 공간을 많이 차지하게 된다. 쿠버네티스는 일정 시간이 지난 후 사용되지 않는 이미지를 자동으로 [GC(Garbage Collection)](https://kubernetes.io/ko/docs/concepts/architecture/garbage-collection/) 처리한다. 하지만 수동으로 이미지 정리 스크립트를 운영하면 보다 빠르게 디스크 여유를 확보할 수 있다.

{% endraw %}