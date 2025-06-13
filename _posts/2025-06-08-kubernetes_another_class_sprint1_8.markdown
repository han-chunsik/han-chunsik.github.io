---
layout: post
title:  "쿠버네티스 컴포넌트 동작 흐름"
date: 2025-06-10
categories: [kubernetes, container, devops]
description: "[Inflearn Warm-Up Club - DevOps] 매일 1% 성장하기"
comments: true
bird_image: "warbling_white_eye.webp"
bird_name: "동박새 (Warbling white-eye)"
bird_scientific_name: "Zosterops japonicus"
bird_description: "동박새는 눈 주위의 하얀 테두리가 특징인 새로, 이름 그대로 겨울에도 푸른 잎(동박나무) 사이에서 자주 보이기 때문에 ‘동박새’라는 이름이 붙었다. 몸길이는 약 10~12cm로 작고, 녹색빛이 도는 등 색깔과 활발한 움직임이 인상적이다."
---

> [**쿠버네티스 어나더 클래스 (지상편) - Sprint 1, 2**](https://www.inflearn.com/course/%EC%BF%A0%EB%B2%84%EB%84%A4%ED%8B%B0%EC%8A%A4-%EC%96%B4%EB%82%98%EB%8D%94-%ED%81%B4%EB%9E%98%EC%8A%A4-%EC%A7%80%EC%83%81%ED%8E%B8-sprint1)

# 컴포넌트 동작 흐름
쿠버네티스를 깊게 이해하기 위해서는 각 컴포넌트의 역할과 그 동작 흐름을 파악하는 것이 중요하다.

## Pod 생성과 Probe 요청

<img src="{{ '/assets/images/20250613_kubernetes_flow_pod_probe.png' | prepend: site.baseurl }}" alt="kubernetes flow pod probe">

### 1. Deployment 생성 요청
사용자가 `kubectl`을 통해 Deployment를 생성하면, `kube-apiserver`가 리소스 정보를 `etcd`에 저장한다.

### 2. ReplicaSet 및 Pod 생성
`kube-controller-manager`는 `kube-apiserver`를 통해 새로운 Deployment가 생성된 사실을 감지하고, 해당 Deployment에 대한 ReplicaSet을 생성한다. 이후 ReplicaSet을 기반으로 원하는 수의 Pod 생성 API 요청을 다시 `kube-apiserver`에 전달한다.

### 3. 스케줄링
`kube-scheduler`는 생성된 Pod들을 확인하고, 클러스터 내 각 노드의 자원을 모니터링한 뒤, 적절한 노드에 해당 Pod을 할당한다. 이때, `nodeSelector`, `resources` 등의 스펙도 고려된다.

### 4. Kubelet이 Pod 감지
각 노드의 `kubelet`은 자신에게 스케줄링된 Pod이 있는지 `kube-apiserver`를 통해 주기적으로 확인한다. 해당 Pod이 감지되면, `container-runtime`에게 컨테이너 생성을 요청한다.

### 5. Probe 요청 수행
컨테이너가 생성된 후, `kubelet`은 Pod에 설정된 `readinessProbe`, `livenessProbe`, `startupProbe` 정보를 기반으로 주기적인 헬스 체크를 수행한다. Probe 방식은 HTTP, TCP, Exec 중 하나로 구성되며, 이를 통해 애플리케이션의 상태를 지속적으로 점검한다.

---

## Service 동작

<img src="{{ '/assets/images/20250613_kubernetes_flow_service.png' | prepend: site.baseurl }}" alt="kubernetes flow service">

### 1. Service 설정
사용자가 `Service` 오브젝트를 생성하면서 `type: NodePort`를 지정하면, 해당 포트가 클러스터 노드에 개방된다.

### 2. 네트워크 생성 요청
`kubelet`은 Pod 생성 시, 네트워크 인터페이스를 생성하고 IP를 할당하며, 트래픽 경로와 네트워크 정책을 설정하기 위해 CNI 플러그인(Calico 등)을 호출한다.

### 3. 트래픽 라우팅 규칙 설정
`kube-proxy`는 `kube-apiserver`를 통해 Service 정보를 감지하고, 해당 정보를 기반으로 `iptables`(또는 ipvs)에 트래픽 라우팅 규칙을 설정한다.

### 4. 트래픽 전달
외부에서 들어온 트래픽이 NodePort를 통해 들어오면, 설정된 `iptables` 규칙에 따라 적절한 Pod으로 전달된다. 이 과정에서 CNI가 설정한 네트워크 경로를 따라 트래픽이 해당 컨테이너로 전달된다.

---

## HPA 동작 

<img src="{{ '/assets/images/20250613_kubernetes_flow_hpa.png' | prepend: site.baseurl }}" alt="kubernetes flow hpa">

### 1. CPU/Memory 조회
`containerd`는 실제 컨테이너의 자원 사용량(CPU, Memory 등)을 알고 있으며, `kubelet`은 이를 주기적으로 조회하여 리소스 사용 정보를 수집한다.

### 2. 사용량 수집
`metrics-server`는 각 노드의 `kubelet`으로부터 수집된 리소스 사용량 데이터를 주기적으로 받아 저장한다.

### 3. 임계값 및 metric 확인
`kube-controller-manager`는 HPA 설정 정보를 바탕으로`metrics-server`에 저장된 리소스 사용량을 확인한다. 이때, 설정된 임계값과 비교하여 스케일링이 필요한지를 판단한다.

### 4. 스케일링
사용량이 설정된 임계값을 초과하거나 부족할 경우,  `kube-controller-manager`는 Deployment의 Replica 수를 자동으로 조정하여 Pod 수를 늘리거나 줄인다.

---

## (참고)Secret 사용 시 주의사항
Kubernetes의 Secret은 민감한 정보를 저장하고 Pod에 주입하는 데 사용된다. 하지만 Secret을 사용할 때는 다음과 같은 동작 특성과 주의사항을 이해하고 있어야 한다.

### 1. Secret은 메모리 볼륨에 마운트된다
Secret을 `volumeMount` 형태로 마운트하면, 해당 파일은 노드의 **메모리(tmpfs)** 영역에 생성된다.  
메모리는 휘발성 자원이기 때문에 전원이 꺼지면 사라지며, Secret을 많이 생성하고 마운트할 경우 노드 메모리를 많이 점유하게 된다.

### 2. Secret 수정은 실시간 반영되지 않는다
Secret 객체를 수정하더라도, 이를 마운트한 컨테이너에서는 즉시 반영되지 않는다. 이는 `kubelet`이 Secret을 **주기적으로 감시(polling)** 하여 변경사항을 감지한 후 해당 파일을 업데이트하기 때문이다.

