---
layout: post
title:  "Spring Cloud Bus on Kubernetes"
date:   2025-07-05
categories: [spring, kubernetes]
description: ConfigMap 변경을 Spring 앱 재기동 없이 반영하는 방법
comments: true
bird_image: "hummingbirds.webp"
bird_name: "벌새 (Humming Birds)"
bird_scientific_name: "Trochilidae"
bird_description: "벌새는 세계에서 가장 작은 조류 중 하나이며, 빠르게 움직이는 날갯짓으로 공중에 정지해 있을 수 있는 독특한 능력을 지닌다. 주로 꽃의 꿀을 먹으며 긴 부리와 혀를 이용해 섬세하게 먹이를 채취한다. 몸길이는 5~20cm 정도이고, 깃털은 금속성 광택이 나는 화려한 색을 띤다."
---

# ConfigMap 변경을 Spring 앱 재기동 없이 반영하는 방법
Kubernetes 환경에서 Spring 기반 서비스를 운영할 때 애플리케이션 내부에서 관리되던 설정 파일을 Kubernetes의 ConfigMap 리소스를 통해 외부에서 관리할 수 있다. 이때 ConfigMap 변경 시 애플리케이션의 설정을 런타임 중 자동으로 반영하려면 설정 리로드를 트리거하는 메커니즘이 필요하다.  
  
이를 위해 자주 사용되는 조합이 **Spring Cloud Bus와 Spring Cloud Kubernetes Configuration Watcher**이다.  
> **Spring Cloud Bus**: Kafka, RabbitMQ 등 메시지 브로커를 통해 설정 변경 이벤트를 서비스 간에 전달한다.  
> **Spring Cloud Kubernetes Configuration Watcher**: Kubernetes의 ConfigMap 또는 Secret 변경을 감지하여 설정 리로드를 트리거하는 역할을 한다.

# Watcher의 트리거 방식
Watcher는 두 가지 방식으로 설정 변경 트리거를 수행할 수 있다.

## HTTP 방식

<img src="{{ '/assets/images/20250705_spring_kubernetes_watcher_http.png' | prepend: site.baseurl }}" alt="spring kubernetes watcher http">

변경 감지 시, DiscoveryClient를 통해 관련 애플리케이션 인스턴스를 찾고, 해당 인스턴스의 /actuator/refresh 엔드포인트로 HTTP POST 요청을 전송한다.

- 변경을 감지할 ConfigMap에는 `spring.cloud.kubernetes.configmap.apps` 애노테이션을 통해 대상 서비스 이름을 설정해야 하고, 해당 서비스에 연결된 Pod의 IP로 요청을 전송한다.
- 서비스에 context-path나 포트 변경이 있는 경우, 해당 서비스에 `boot.spring.io/actuator: http://:{port}/{context-path}/actuator` 애노테이션을 추가해야한다.

> #### SPRING_CLOUD_KUBERNETES_CONFIGURATION_WATCHER_REFRESHDELAY
> Kubernetes에서 ConfigMap을 volume으로 마운트하면, kubelet은 **기본 1분 주기(60초)**로 변경을 감지하고, 파일을 갱신한다.
> 따라서 Pod 내부에 마운트된 파일 내용이 변경되기까지 최대 1분의 지연이 발생할 수 있다. 이로 인해 spring cloud kubernetes watcher의 SPRING_CLOUD_KUBERNETES_CONFIGURATION_WATCHER_REFRESHDELAY 설정도 kubelet의 동기화 주기와 유사하게 설정하는 것이 좋다.

## 메시지 브로커 방식 (Kafka, RabbitMQ 등)

<img src="{{ '/assets/images/20250705_spring_kubernetes_watcher_msg.png' | prepend: site.baseurl }}" alt="spring kubernetes watcher message">

변경 감지 시, 지정된 topic으로 설정 변경 메시지를 발행한다. Spring Cloud Bus가 해당 메시지를 구독하고, 이를 통해 각 서비스에 설정 리로드를 유도한다.
