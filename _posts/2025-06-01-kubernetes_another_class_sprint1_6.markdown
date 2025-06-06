---
layout: post
title:  "쿠버네티스 주요 오브젝트와 오브젝트 간 연결"
date: 2025-05-31
categories: [kubernetes, container, devops]
description: "[Inflearn Warm-Up Club - DevOps] 매일 1% 성장하기"
comments: true
bird_image: "eurasian-jay.webp"
bird_name: "어치 (Eurasian Jay)"
bird_scientific_name: "Garrulus glandarius"
bird_description: "어치는 참새목 까마귀과에 속하는 새로, 지능이 높고 경계심이 강하다. 깃털은 화려하며, 날개에 파란색 줄무늬가 특징이다. 주로 숲이나 공원에 서식하며, 도토리를 저장해두는 습성을 가진다."
---

> [**쿠버네티스 어나더 클래스 (지상편) - Sprint 1, 2**](https://www.inflearn.com/course/%EC%BF%A0%EB%B2%84%EB%84%A4%ED%8B%B0%EC%8A%A4-%EC%96%B4%EB%82%98%EB%8D%94-%ED%81%B4%EB%9E%98%EC%8A%A4-%EC%A7%80%EC%83%81%ED%8E%B8-sprint1)

# 쿠버네티스 주요 오브젝트

<img src="{{ '/assets/images/20250606_kubernetes_obj_reference.png' | prepend: site.baseurl }}" alt="kubernetes obj reference">

쿠버네티스는 다양한 오브젝트들로 구성되며, 서로 연결되어 애플리케이션을 배포하고 운영하는 데 핵심적인 역할을 한다.

## Namespace
> [Kubernetes Docs - Namespace](https://kubernetes.io/docs/concepts/overview/working-with-objects/namespaces/)

### 정의 및 역할
- 오브젝트들을 논리적으로 그룹화하여 구분
- 서로 다른 팀, 프로젝트, 환경(개발/운영 등) 간 리소스 충돌 방지 

### YAML 속성

```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: default
  labels:
    kubernetes.io/metadata.name: default
```

## Deployment
> [Kubernetes Docs - Deployment](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/)

### 정의 및 역할
- Pod의 생성과 업데이트를 관리하는 컨트롤러
- 선언한 상태에 맞춰 Pod 유지

### YAML 속성

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-app
  labels:
    app: my-app
spec:
  replicas: 3
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxUnavailable: 1
      maxSurge: 1
  selector:
    matchLabels:
      app: my-app
  template:
    metadata:
      labels:
        app: my-app
    spec:
      nodeSelector:
        disktype: ssd
      containers:
        - name: my-container
          image: nginx:1.25
          env:
            - name: CONFIG_VALUE
              valueFrom:
                configMapKeyRef:
                  name: my-config
                  key: config-key
          ports:
            - containerPort: 80
          startupProbe:
            httpGet:
              path: /
              port: 80
            failureThreshold: 30
            periodSeconds: 10
          readinessProbe:
            httpGet:
              path: /healthz
              port: 80
            initialDelaySeconds: 5
            periodSeconds: 10
          livenessProbe:
            httpGet:
              path: /healthz
              port: 80
            initialDelaySeconds: 15
            periodSeconds: 20
          resources:
            requests:
              memory: "128Mi"
              cpu: "250m"
            limits:
              memory: "256Mi"
              cpu: "500m"
          volumeMounts:
            - name: config-volume
              mountPath: /etc/config
              readOnly: true
      volumes:
        - name: config-volume
          configMap:
            name: my-config

```

- replicas: 원하는 Pod 개수
- strategy: 롤링 업데이트 등 업데이트 방식
- template: 실제 생성될 Pod 정의
- nodeSelector: Pod가 스케줄될 노드 선택
- containers.image: 사용할 컨테이너 이미지
- env: 환경변수 설정 (ConfigMap/Secret 연동)
- startupProbe: 애플리케이션 최초 기동 확인
- readinessProbe: 서비스로 트래픽을 보낼 수 있는 상태인지 확인
- livenessProbe: 컨테이너가 살아있는지 확인
- resources: CPU/Memory 요청 및 제한
- volumeMounts + volumes: 볼륨 마운트 설정


## Service
> [Kubernetes Docs - Service](https://kubernetes.io/docs/concepts/services-networking/service/)

### 정의 및 역할
- 네트워크 트래픽을 적절한 Pod로 전달
- Pod의 IP가 바뀌어도 안정적으로 접근 가능

### YAML 속성

```yaml
apiVersion: v1
kind: Service
metadata:
  name: my-service
spec:
  selector:
    app.kubernetes.io/name: MyApp
  ports:
    - protocol: TCP
      port: 80
      targetPort: 9376
```

- selector: 연결할 Pod 라벨
- port: 서비스 포트
- targetPort: 실제 컨테이너 포트


## Configmap
> [Kubernetes Docs - Configmap](https://kubernetes.io/ko/docs/concepts/configuration/configmap/)

### 정의 및 역할
- 설정 데이터를 환경변수나 파일 형태로 주입
- 애플리케이션의 설정을 외부에서 분리 가능

### YAML 속성

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: game-demo
data:
  player_initial_lives: "3"
  ui_properties_file_name: "user-interface.properties"

  game.properties: |
    enemy.types=aliens,monsters
    player.maximum-lives=5    
  user-interface.properties: |
    color.good=purple
    color.bad=yellow
    allow.textmode=true    
```

## secret
> [Kubernetes Docs - Secret](https://kubernetes.io/docs/concepts/configuration/secret/)

### 정의 및 역할
- 민감한 정보를 환경변수나 파일 형태로 주입
- ConfigMap과 유사하지만 base64 인코딩 필요

### YAML 속성

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: dotfile-secret
data:
  .secret-file: dmFsdWUtMg0KDQo=
```

## PV(Persistent Volumes) / PVC(PersistentVolumeClaim)
> [Kubernetes Docs - pv/pvc](https://kubernetes.io/docs/concepts/storage/persistent-volumes/)

### 정의 및 역할
- Pod의 생명주기와 무관하게 데이터를 저장
- PV는 클러스터에 제공된 저장소, PVC는 Pod가 요청하는 저장소

### YAML 속성

```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: foo-pv
spec:
  storageClassName: ""
  claimRef:
    name: foo-pvc
    namespace: foo
---
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: foo-pvc
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 1Gi
```

## HPA
> [Kubernetes Docs - HPA](https://kubernetes.io/ko/docs/tasks/run-application/horizontal-pod-autoscale/)

### 정의 및 역할
- CPU/메모리 등 리소스 사용량에 따라 자동으로 Pod 수 조절

### YAML 속성

```yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: php-apache
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: php-apache
  minReplicas: 1
  maxReplicas: 10
  metrics:
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 50
status:
  observedGeneration: 1
  lastScaleTime: <some-time>
  currentReplicas: 1
  desiredReplicas: 1
  currentMetrics:
  - type: Resource
    resource:
      name: cpu
      current:
        averageUtilization: 0
        averageValue: 0
```

- scaleTargetRef: 스케일 대상 리소스
- minReplicas / maxReplicas: 최소/최대 Pod 수
- metrics: 기준 리소스 및 임계값
- behavior: 스케일 조정 정책


# 쿠버네티스에서 오브젝트 간 연결
쿠버네티스 오브젝트들은 label, selector, namespace, name 등을 통해 서로 관계를 맺는다.

## 네이밍 TIP
- 이름은 가독성과 관리 용이성을 고려하여 작성
- 운영 중에는 변경이 어렵기 때문에 처음부터 명확하게 네이밍
- 네임스페이스는 서비스나 도메인 기준으로 구분

# Label & Selector

## label
- 오브젝트에 메타데이터를 부여하여 그룹화
- 형식: key=value
- prefix를 붙여 충돌 방지 가능 (team.dev/app=foo)

## selector
- 특정 라벨을 가진 오브젝트를 선택할 때 사용
- matchLabels 또는 matchExpressions 방식

### Selector 사용 예시

<div style="overflow-x: auto;">
  <table style="width: auto;">
    <thead>
      <tr>
        <th>오브젝트 유형</th>
        <th>필드 위치</th>
        <th>역할 및 설명</th>
      </tr>
    </thead>
    <tbody>
      <tr>
        <td>Service</td>
        <td>spec.selector</td>
        <td>연결할 대상 Pod의 라벨을 기준으로 트래픽 전달</td>
      </tr>
      <tr>
        <td>Deployment</td>
        <td>spec.selector.matchLabels</td>
        <td>관리할 Pod 정의. template.labels와 일치해야 함</td>
      </tr>
      <tr>
        <td>ReplicaSet</td>
        <td>spec.selector.matchLabels</td>
        <td>생성 및 관리할 Pod 그룹 선택</td>
      </tr>
      <tr>
        <td>HorizontalPodAutoscaler</td>
        <td>spec.scaleTargetRef.name</td>
        <td>스케일 대상 리소스 (Deployment 등) 이름 지정</td>
      </tr>
      <tr>
        <td>PersistentVolume</td>
        <td>spec.claimRef</td>
        <td>PVC와 연결될 PV 식별</td>
      </tr>
      <tr>
        <td>PersistentVolumeClaim</td>
        <td>metadata.name</td>
        <td>PV의 claimRef.name과 매칭됨</td>
      </tr>
      <tr>
        <td>Pod</td>
        <td>spec.nodeSelector</td>
        <td>특정 라벨을 가진 노드에만 Pod 스케줄링</td>
      </tr>
      <tr>
        <td>NetworkPolicy</td>
        <td>spec.podSelector</td>
        <td>정책 적용 대상 Pod 선택</td>
      </tr>
    </tbody>
  </table>
</div>


# 회고
이번 강의를 통해 쿠버네티스가 왜 선언적인 구조인지, 그리고 그 선언들이 어떻게 연결되는지를 조금 더 명확히 이해할 수 있었다.

각 오브젝트는 단독으로 존재하지 않고, label, selector, namespace 등을 통해 서로 연결된다.
Deployment, Service, HPA 같은 컨트롤러들이 이 선언된 상태를 유지시키며 시스템이 원하는 상태로 작동하게 만든다.

이 구조 덕분에 쿠버네티스는 복잡한 운영 환경에서도 예측 가능한 상태를 유지할 수 있고,
우리는 선언만으로도 안정적인 인프라를 구성할 수 있게 된다는 것을 깨달았다.