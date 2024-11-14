# 제2장: 가상화 설치
## 2.1. 가상화 개요
### 2.1.1. 가상화란 무엇인가?
### 2.1.2. 가상화의 장점
### 2.1.3. 가상화의 종류
## 2.2. KVM 가상화 설치
### 2.2.1. KVM이란 무엇인가?
### 2.2.2. KVM 환경 확인
##### 2.2.2.1. KVM 지원 시스템
```bash
# lscpu | grep Virtualization

# lsmod | grep kvm

# modprobe kvm_intel

# modprobe kvm_amd

```

```bash
# grep -E ‘svm|vmx’ /proc/cpuinfo

```

## 2.3. KVM 설치
### 2.3.1. 가상화 패키지 종류
### 2.3.2. KVM 설치 및 서비스 시작
```bash
# dnf -y install qemu-kvm libvirt libvirt-client virt-install virt-viewer

# dnf -y install virt-manager
```

```bash
# systemctl start libvirtd

# systemctl enable libvirtd

# systemctl status libvirtd
```

## 2.4. 가상화 구성
### 2.4.1. 구성도
### 2.4.2. 가상화 Network 설정
```bash
# virsh net-dumpxml default

# virsh net-edit default
```

```bash
# virsh net-destroy default

# virsh net-start default

# virsh net-list default

# virsh net-dumpxml default
```

### 2.4.3. VM 환경 구성
```bash
# virt-manager
```

##### 2.4.3.2.3. Prepare ISO
```bash
# cd /var/lib/libvirt/images/ISO/

# mv /home/lds/Downloads/Rocky-9.3-x86_64-dvd.iso ./

# chown qemu:qemu Rocky-9.3-x86_64-dvd.iso

# ls -al
```

### 2.4.4. VM 구성
##### 2.4.4.1.4. Network Interface 설정
```bash
# su - 

# nmtui

# ip addr show
```

##### 2.4.4.1.5. Hostname 설정
```bash
# hostnamectl set-hostname awx.example.com
```

##### 2.4.4.2.1. Disk 생성
```bash
# cd /var/lib/libvirt/images/pool

# qemu-img create -f qcow2 ansible-guest.qcow2 100G

# chown qemu:qemu ansible-guest.qcow2

# ls -al
```

##### 2.4.4.2.3. SSH 연결에 대한 액세스 제한
```bash
# cp /etc/ssh/sshd_config /etc/ssh/sshd_config.orig

# sed -i -E 's/^#?(PermitRootLogin).*/\1 yes/' /etc/ssh/sshd_config

# systemctl restart sshd

# systemctl status sshd
```