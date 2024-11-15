# 제3장: K3s 구성


## 3.1. K3s란 무엇인가?

### 3.1.1. K3s 개요

### 3.1.2. K3s Architecture

## 3.2. K3s 설치 요구 사항

### 3.2.1. K3s 설치 전제 조건

### 3.2.2. Architecture

### 3.2.3. Operating System
#### 3.2.3.1. 추가 OS 준비 사항
```bash
# systemctl disable firewalld --now

# systemctl stop firewalld

# systemctl status firewalld
```

```bash
# firewall-cmd --permanent --add-port=6443/tcp #apiserver

# firewall-cmd --permanent --zone=trusted --add-source=10.42.0.0/16 #pods

# firewall-cmd --permanent --zone=trusted --add-source=10.43.0.0/16 #services

# firewall-cmd --reload
```

```bash
# systemctl disable nm-cloud-setup.service nm-cloud-setup.timer

# reboot
```

```bash
# setenforce 0

# sed -i 's/enforcing/disabled/g' /etc/selinux/config

# grep "^SELINUX" /etc/selinux/config

# sestatus
```
### 3.2.4. Hardware

### 3.2.5. Networking

## 3.3. K3s 설치

### 3.3.1. K3s 설치 - 설치 스크립트

#### 3.3.1.1. K3s 설치 스크립트
```bash
# curl -sfL https://get.k3s.io | sh -
```

#### 3.3.1.2. 파일 권한 설정
```bash
# chmod 644 /etc/rancher/k3s/k3s.yaml
```


### 3.3.2. K3s 설치 상태 점검
#### 3.3.2.1. K3s 설치 내역 확인
```bash
# k3s --version
```

```bash
# kubectl version
```

#### 3.3.2.2. K3s 서비스 점검
```bash
# systemctl start k3s

# systemctl enable k3s

# systemctl status k3s
```

#### 3.3.2.3. K3s 상태 점검
```bash
# kubectl config get-clusters
```

```bash
# kubectl cluster-info
```

```bash
# kubectl get nodes
```

```bash
# kubectl get namespaces
```

```bash
# kubectl get endpoints -n kube-system
```

```bash
# kubectl get pods -n kube-system
```

```bash
# crictl ps
```