---
layout: post
title:  "로컬에 쿠버네티스 설치하기"
date: 2025-05-31
categories: [kubernetes, container, devops]
description: "[Inflearn Warm-Up Club - DevOps] 매일 1% 성장하기"
comments: true
bird_image: "cinereous-tit.webp"
bird_name: "박새 (Cinereous Tit)"
bird_scientific_name: "Parus minor"
bird_description: "박새는 참새목 박새과의 새로, 머리와 목은 검고, 볼은 흰색이며, 등은 회색빛을 띠고 있다. 도시 공원이나 숲, 시골 마을에서도 흔히 볼 수 있으며, 곤충과 씨앗을 먹는다. 사람과 가까운 환경에서도 잘 적응해 살아가는 새다."
---

> [**쿠버네티스 어나더 클래스 (지상편) - Sprint 1, 2**](https://www.inflearn.com/course/%EC%BF%A0%EB%B2%84%EB%84%A4%ED%8B%B0%EC%8A%A4-%EC%96%B4%EB%82%98%EB%8D%94-%ED%81%B4%EB%9E%98%EC%8A%A4-%EC%A7%80%EC%83%81%ED%8E%B8-sprint1)


쿠버네티스를 더 쉽게 설치할수록 오히려 질문은 많아진다. 왜 그럴까? 문제가 생겨도 어디서 꼬였는지, 왜 안 되는지 감이 안 잡히기 때문이다. 단순히 설치만 하고 넘어가면 구조를 이해하지 못한 채 헤매기 쉽다. 그래서 중요한 건 ‘빠르고 간편한 설치’보다는 구성 원리를 이해하고 제대로 설치해보는 것이다.
  
물론 간단한 테스트를 위해서는 빠르게 로컬 환경을 준비할 수 있어야 한다. 이런 환경이 갖춰지면 직접 실험하고 문제를 마주치며 실력을 키울 수 있는 기회도 많아진다.

# 환경 정보

## 호스트 환경
- **운영체제**: macOS (Apple Silicon - M3 Pro)
- **가상화 도구**: [UTM](https://mac.getutm.app/)

## 가상 머신 환경
- **운영체제**: Rocky Linux 9.2
- **CPU**: 4Core
- **메모리**: 4GB
- **디스크**: 32GB
- **컨테이너 오케스트레이션**: Kubernetes 1.27
- **컨테이너 런타임**: containerd 1.6.2


# 환경 구성
## UTM 설치
- [UTM 공식 사이트](https://mac.getutm.app/)에서 다운로드 (앱스토어 버전은 유료이나, 무료 버전과 기능 차이 없음)

## Rocky Linux ISO 다운로드
- [Rocky Linux 9.2 arm64 - Download](https://dl.rockylinux.org/vault/rocky/9.2/isos/aarch64/Rocky-9.2-aarch64-minimal.iso)

## 가상머신 생성
1. 새 가상 머신 만들기 → 가상화 → Linux 선택
2. 부팅 ISO: 이미지에 위에서 받은 Rocky Linux ISO 지정
3. 메모리: 4096MB / CPU: 4코어 설정
4. 디스크: 32GiB 설정
5. 공유 디렉토리: 설정 안함
6. 이름 입력 후 저장
<img width="60%" src="{{ '/assets/images/20250601_utm_info.png' | prepend: site.baseurl }}" alt="utm info">

## 가상머신 추가 설정
1. 가상머신 네트워크 설정에서 고급 설정 표시 선택 후 게스트 네트워크 입력 `192.168.56.0/24`
<img width="60%" src="{{ '/assets/images/20250601_utm_network.png' | prepend: site.baseurl }}" alt="utm network">

## Rocky Linux 설치
1. 생성한 가상머신 실행 `Install Rocky Linux 9.2` 메뉴 선택
> `display output is not active` 메시지가 보이더라도 5~10초 후 설치 화면이 나타남
2. 언어: 한국어
3. Root 계정 설정
- root 비밀번호: root 비밀번호 입력
- root가 비밀번호로 SSH 로그인하도록 허용 - 체크 
4. 파티션 설정
- 설치 목적지 → 저장소 구성 : 자동 설정 체크 후 완료 클릭
5. 네트워크 설정
- 네트워크와 호스트 이름 → 호스트 이름 k8s-master 입력
- 이더넷(enp0s1): 설정 클릭 k8s-master  IPv4 설정 탭 클릭 → Method 수동 →  주소 Add 클릭 후 `주소(192.168.56.30)`, `넷마스크(255.255.255.0)`, `게이트웨이(192.168.56.1)` → 완료
<img width="60%" src="{{ '/assets/images/20250601_rocky_network.png' | prepend: site.baseurl }}" alt="utm network">
6. 설치 시작
7. 설치 완료 시 시스템 재시작 → `Install Rocky Linux 9.2` 메뉴가 나오면 가상머신 종료
8. CD/DVD 초기화 후 가상머신 재시작

## 원격 접속
- Mac 터미널을 사용하여 가상머신 접속
```
ssh root@192.168.56.30
```

# 쿠버네티스 설치
<details>
<summary>빠른 설치를 위한 스크립트</summary>
<div markdown="1">

> 출처: [큐브옵스 - 쿠버네티스 빠른설치 (Mac-m시리즈)](https://cafe.naver.com/kubeops/91) 

```shell
echo '======== [4] Rocky Linux 기본 설정 ========'
echo '======== [4-1] 패키지 업데이트 ========'
# 강의와 동일한 실습 환경을 유지하기 위해 Linux Update 주석 처리
# yum -y update

echo '======== [4-2] 타임존 설정 ========'
timedatectl set-timezone Asia/Seoul
timedatectl set-ntp true
chronyc makestep

echo '======== [4-3] [WARNING FileExisting-tc]: tc not found in system path 로그 관련 업데이트 ========'
yum install -y yum-utils iproute-tc
echo '======== [4-3] [WARNING OpenSSL version mismatch 로그 관련 업데이트 ========'
yum update openssl openssh-server -y

echo '======= [4-4] hosts 설정 =========='
cat << EOF >> /etc/hosts
192.168.56.30 k8s-master
EOF

echo '======== [5] kubeadm 설치 전 사전작업 ========'
echo '======== [5] 방화벽 해제 ========'
systemctl stop firewalld && systemctl disable firewalld

echo '======== [5] Swap 비활성화 ========'
swapoff -a && sed -i '/ swap / s/^/#/' /etc/fstab

echo '======== [6] 컨테이너 런타임 설치 ========'
echo '======== [6-1] 컨테이너 런타임 설치 전 사전작업 ========'
echo '======== [6-1] iptable 세팅 ========'
cat <<EOF |tee /etc/modules-load.d/k8s.conf
overlay
br_netfilter
EOF

modprobe overlay
modprobe br_netfilter

cat <<EOF |tee /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-iptables  = 1
net.bridge.bridge-nf-call-ip6tables = 1
net.ipv4.ip_forward                 = 1
EOF

sysctl --system

echo '======== [6-2] 컨테이너 런타임 (containerd 설치) ========'
echo '======== [6-2-1] containerd 패키지 설치 (option2) ========'
echo '======== [6-2-1-1] docker engine 설치 ========'
echo '======== [6-2-1-1] repo 설정 ========'
yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo

echo '======== [6-2-1-1] containerd 설치 ========'
yum install -y containerd.io-1.6.21-3.1.el9.aarch64
systemctl daemon-reload
systemctl enable --now containerd

echo '======== [6-3] 컨테이너 런타임 : cri 활성화 ========'
containerd config default > /etc/containerd/config.toml
sed -i 's/ SystemdCgroup = false/ SystemdCgroup = true/' /etc/containerd/config.toml
systemctl restart containerd

echo '======== [7] kubeadm 설치 ========'
echo '======== [7] repo 설정 ========'
cat <<EOF | sudo tee /etc/yum.repos.d/kubernetes.repo
[kubernetes]
name=Kubernetes
baseurl=https://pkgs.k8s.io/core:/stable:/v1.27/rpm/
enabled=1
gpgcheck=1
gpgkey=https://pkgs.k8s.io/core:/stable:/v1.27/rpm/repodata/repomd.xml.key
exclude=kubelet kubeadm kubectl cri-tools kubernetes-cni
EOF

echo '======== [7] SELinux 설정 ========'
setenforce 0
sed -i 's/^SELINUX=enforcing$/SELINUX=permissive/' /etc/selinux/config

echo '======== [7] kubelet, kubeadm, kubectl 패키지 설치 ========'
yum install -y kubelet-1.27.2-150500.1.1.aarch64 kubeadm-1.27.2-150500.1.1.aarch64 kubectl-1.27.2-150500.1.1.aarch64 --disableexcludes=kubernetes
systemctl enable --now kubelet

echo '======== [8] kubeadm으로 클러스터 생성  ========'
echo '======== [8-1] 클러스터 초기화 (Pod Network 세팅) ========'
kubeadm init --pod-network-cidr=20.96.0.0/16 --apiserver-advertise-address 192.168.56.30

echo '======== [8-2] kubectl 사용 설정 ========'
mkdir -p $HOME/.kube
cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
chown $(id -u):$(id -g) $HOME/.kube/config

echo '======== [8-3] Pod Network 설치 (calico) ========'
kubectl create -f https://raw.githubusercontent.com/k8s-1pro/install/main/ground/k8s-1.27/calico-3.26.4/calico.yaml
kubectl create -f https://raw.githubusercontent.com/k8s-1pro/install/main/ground/k8s-1.27/calico-3.26.4/calico-custom.yaml

echo '======== [8-4] Master에 Pod를 생성 할수 있도록 설정 ========'
kubectl taint nodes k8s-master node-role.kubernetes.io/control-plane-

echo '======== [9] 쿠버네티스 편의기능 설치 ========'
echo '======== [9-1] kubectl 자동완성 기능 ========'
yum -y install bash-completion
echo "source <(kubectl completion bash)" >> ~/.bashrc
echo 'alias k=kubectl' >>~/.bashrc
echo 'complete -o default -F __start_kubectl k' >>~/.bashrc
source ~/.bashrc

echo '======== [9-2] Dashboard 설치 ========'
kubectl create -f https://raw.githubusercontent.com/k8s-1pro/install/main/ground/k8s-1.27/dashboard-2.7.0/dashboard.yaml

echo '======== [9-3] Metrics Server 설치 ========'
kubectl create -f https://raw.githubusercontent.com/k8s-1pro/install/main/ground/k8s-1.27/metrics-server-0.6.3/metrics-server.yaml
```

</div>
</details>

소프트웨어를 설치할 때는 공식 문서를 참고하는 것이 가장 좋다. 신뢰할 수 있고, 버전별 차이나 필수 설정 등 실무에 필요한 정보가 잘 정리돼 있기 때문이다.  

## 쿠버네티스 문서의 kubernetes 설치 흐름
> [kubeadm 설치하기](https://kubernetes.io/ko/docs/setup/production-environment/tools/kubeadm/install-kubeadm/)

<img width="60%" src="{{ '/assets/images/20250601_k8s_install_flow.png' | prepend: site.baseurl }}" alt="kubernetes dashboard">

1. **kubeadm 설치 전 작업**
   - 방화벽 해제: 노드 간 통신을 막지 않도록 포트를 개방
   - 스왑 비활성화: Swap은 실제 메모리보다 더 많은 메모리를 쓰기 위한 기능이지만, Kubernetes는 자원을 정확히 관리해야 하므로 일관성 있는 리소스 사용을 위해 swap을 끄는 것을 권장
  
2. **컨테이너 런타임 설치**
   - iptables 세팅: 브리지 네트워크 패킷 처리를 위한 설정
   - containerd 설치: 쿠버네티스가 사용하는 컨테이너 실행 도구
     - (optional) docker engine 설치: containerd 설치 전 의존성 처리
   - CRI 활성화: 쿠버네티스가 containerd를 통해 Pod을 실행할 수 있도록 설정

3. **kubeadm 설치**
   - repo 설정: 필요한 패키지 설치를 위한 저장소 등록
   - SELinux 설정: 컨테이너 실행이나 네트워크, 볼륨 마운트 등 충돌 방지를 위해 permissive로 설정
   - kubelet, kubeadm, kubectl 설치: 쿠버네티스를 구성하는 필수 도구

## 설치 확인
- pod 상태 확인

    ```shell
    kubectl get po -A
    ```
    <img src="{{ '/assets/images/20250601_kgetpo.png' | prepend: site.baseurl }}" alt="kubectl get pod -A">
- [kubernetes 대시보드 확인](https://192.168.56.30:30000/#/login)
    <img src="{{ '/assets/images/20250601_k8s_dashboard.png' | prepend: site.baseurl }}" alt="kubernetes dashboard">

# 실습 미션

<details>
<summary>미션 1</summary>
<div markdown="1">

## 가상머신 생성 확인
### Rocky Linux 버전 확인

```shell
cat /etc/*-release
```

#### 실행결과
<img src="{{ '/assets/images/20250601_mission_os_version_check.png' | prepend: site.baseurl }}" alt="os version check">

### hostname 확인

```shell
hostname
```

#### 실행결과
<img src="{{ '/assets/images/20250601_mission_hostname_check.png' | prepend: site.baseurl }}" alt="hostname check">

### Network 확인

```shell
ip addr
```

#### 실행 결과
```shell
# UTM은 기본적으로 Shared Network 방식의 NAT 구조를 사용하며, 가상머신에서 NAT 인터페이스가 보이지 않아도 게스트와 호스트는 동일한 VLAN을 공유하므로 트래픽이 호스트 OS를 통해 직접 라우팅되어 별도 설정 없이 통신이 가능하다.

# 실제 흐름
## [VM: 192.168.56.30] → (NAT 변환) → [Host IP: 172.30.1.35] → [dns.google]
## [dns.google] → [Host IP: 172.30.1.35] → (NAT 복원) → [VM: 192.168.56.30]

# 확인 방법
## host PC
sudo tcpdump icmp

## 가상머신
ping 8.8.8.8

## 결과
16:11:59.374592 IP 192.168.56.30 > dns.google: ICMP echo request, id 6, seq 1, length 64
16:11:59.374688 IP 172.30.1.35 > dns.google: ICMP echo request, id 17475, seq 1, length 64
16:11:59.413290 IP dns.google > 172.30.1.35: ICMP echo reply, id 17475, seq 1, length 64
16:11:59.413367 IP dns.google > 192.168.56.30: ICMP echo reply, id 6, seq 1, length 64
```

<img src="{{ '/assets/images/20250601_mission_ip_addr_check.png' | prepend: site.baseurl }}" alt="ip addr check">

### 자원(cpu, memory) 확인
```shell
// cpu 확인
lscpu 

// memory 확인
free -h 
```

#### 실행 결과
<img src="{{ '/assets/images/20250601_mission_cpu_check.png' | prepend: site.baseurl }}" alt="cpu check">
<img src="{{ '/assets/images/20250601_mission_memory_check.png' | prepend: site.baseurl }}" alt="memory check">


</div>
</details>

<details>
<summary>미션 2</summary>
<div markdown="1">

## Rocky Linux 기본 설정
### 타임존 설정 확인
```shell
timedatectl
```

#### 실행 결과
<img src="{{ '/assets/images/20250601_timedate_check.png' | prepend: site.baseurl }}" alt="timedate check">

## kubeadm 설치 전 사전작업
### 방화벽 해제 확인
```shell
systemctl status firewalld
```

#### 실행 결과
<img src="{{ '/assets/images/20250601_firewall_check.png' | prepend: site.baseurl }}" alt="firewalld check">

### Swap 비활성화 확인
```shell
free && cat /etc/fstab |grep swap
```

#### 실행 결과
<img src="{{ '/assets/images/20250601_swap_check.png' | prepend: site.baseurl }}" alt="swap check">

## 컨테이너 런타임 설치
### iptables 세팅
```shell
# 설정 세팅 확인
cat /etc/modules-load.d/k8s.conf
cat /etc/sysctl.d/k8s.conf

# 모듈 적제 확인
lsmod | grep overlay
lsmod | grep br_netfilter
```

#### 실행 결과
<img src="{{ '/assets/images/20250601_iptables_check.png' | prepend: site.baseurl }}" alt="iptables check">

### docker repo 설정 확인 
```shell
yum repolist enabled
```

#### 실행 결과
<img src="{{ '/assets/images/20250601_repolist_check.png' | prepend: site.baseurl }}" alt="repolist check">

### containerd 설정 확인 
```shell
systemctl status containerd
```

#### 실행 결과
<img src="{{ '/assets/images/20250601_containerd_status_check.png' | prepend: site.baseurl }}" alt="containerd check">

### 설치 가능한 버전의 containerd.io 리스트 확인
```shell
yum list containerd.io --showduplicates | sort -r
```

#### 실행 결과
<img src="{{ '/assets/images/20250601_yumlist_check.png' | prepend: site.baseurl }}" alt="yumlist check">

### cgroup 확인 
```shell
# cri 활성화 설정 확인
# kubelet configmap 설정확인
# kubelet 설정 확인

cat /etc/containerd/config.toml | grep Systemd && \
kubectl get -n kube-system cm kubelet-config -o yaml | grep cgroup && \
cat /var/lib/kubelet/config.yaml | grep cgroup
```
#### 실행결과
<img src="{{ '/assets/images/20250601_cgroup_check.png' | prepend: site.baseurl }}" alt="cgroup check">

## kubeadm 설치

### repo 설정 확인
```shell
yum repolist enabled
```
#### 실행결과
<img src="{{ '/assets/images/20250601_repolist_check.png' | prepend: site.baseurl }}" alt="repolist check">

### SELinux 설정 확인
```shell
cat /etc/selinux/config |grep permissive && sestatus |grep permissive
```
#### 실행결과
<img src="{{ '/assets/images/20250601_selinux_check.png' | prepend: site.baseurl }}" alt="selinux check">

### kubelet, kubeadm, kubectl 패키지 설치 
```shell
# 버전 확인
kubeadm version
kubectl version

# 상태 확인
systemctl status kubelet

# 설정 파일 위치
cat /var/lib/kubelet/config.yaml

# 로그 조회
journalctl -u kubelet | tail -10
```
#### 실행결과
<img src="{{ '/assets/images/20250601_k8s_status_version_check.png' | prepend: site.baseurl }}" alt="k8s status version check">
<img src="{{ '/assets/images/20250601_k8s_config_log_check.png' | prepend: site.baseurl }}" alt="k8s config log version check">

### 설치 가능한 버전의 kubeadm 리스트 확인
```shell
yum list --showduplicates kubeadm --disableexcludes=kubernetes
```
#### 실행결과
<img src="{{ '/assets/images/20250601_k8s_install_version_check.png' | prepend: site.baseurl }}" alt="k8s install version check">

## kubeadm으로 클러스터 생성

### 클러스터 상태 확인
```shell
# master node 상태확인
kubectl get node

# pod network cidr 설정 확인
kubectl cluster-info dump | grep -m 1 cluster-cidr

# apiserver advertise address 적용 확인
kubectl cluster-info

# kubernetes component pod 확인
kubectl get pods -n kube-system
```
#### 실행결과
<img src="{{ '/assets/images/20250601_k8s_cluster_status_check.png' | prepend: site.baseurl }}" alt="k8s cluster status check">

### 인증서 설정 확인
```shell
cat ~/.kube/config
```
#### 실행결과
<img src="{{ '/assets/images/20250601_k8s_cert_check.png' | prepend: site.baseurl }}" alt="k8s cluster cert check">

### calico pod 설치 및 pod network cidr 적용 확인
```shell
# Calico Pod 상태 확인
kubectl get -n calico-system pod
kubectl get -n calico-apiserver pod

# Calico에 pod network cidr 적용 확인
kubectl get installations.operator.tigera.io default -o yaml  | grep cidr
```

#### 실행결과
<img src="{{ '/assets/images/20250601_cni_check.png' | prepend: site.baseurl }}" alt="cni check">

### Master Node에 Taint 해제 확인
```shell
kubectl describe nodes | grep Taints
```

#### 실행결과
<img src="{{ '/assets/images/20250601_master_taint_check.png' | prepend: site.baseurl }}" alt="master taint check">

## 쿠버네티스 편의 기능 설치

### kubectl 자동완성 기능 설정 확인
```shell
cat ~/.bashrc
```

#### 실행결과
<img src="{{ '/assets/images/20250601_k8s_alias_check.png' | prepend: site.baseurl }}" alt="k8s alias check">

### dashboard 설치 확인
```shell
kubectl get pod -n kubernetes-dashboard
```

#### 실행결과
<img src="{{ '/assets/images/20250601_k8s_dashboard_check.png' | prepend: site.baseurl }}" alt="k8s dashboard check">

### metrics server 설치 확인
```shell
# metric server pod 확인
kubectl get pod -n kube-system  | grep metrics

# pod 리소스 사용량 확인
kubectl top pod -A
```

#### 실행결과
<img src="{{ '/assets/images/20250601_pos_metric_check.png' | prepend: site.baseurl }}" alt="pod metric check">


</div>
</details>

# 회고
소프트웨어를 설치하면서 느낀 건, 공식 문서만 잘 읽어도 대부분의 해답이 있다는 것이다.
그동안은 자동화된 스크립트나 도구에 의존했지만, 이번엔 일부러 천천히, 단계별로 직접 확인하며 설치해봤다.
과정을 따라가다 보니 구조가 더 잘 보였고, 어디서 문제가 생겼는지도 금방 파악할 수 있었다.