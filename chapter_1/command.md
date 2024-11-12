# 제1장: Rocky Linux 설치
## 1.1. Rocky Linux 소개
### 1.1.1. Rocky Linux란 무엇인가?
### 1.1.2. Rocky Linux의 특징
## 1.2. Rocky Linux 설치 준비
### 1.2.1. OS 설치 사전 준비 사항
```
# wget https://download.rockylinux.org/pub/rocky/9/isos/x86_64/Rocky-9.3-x86_64-dvd.iso
```

### 1.2.2. 설치 프로그램 ISO 파일 확인
```
# wget https://download.rockylinux.org/pub/rocky/9/isos/x86_64/CHECKSUM
# sha256sum -c CHECKSUM --ignore-missing
```
### 1.2.3. 부팅 ISO 만들기
## 1.3. Rocky Linux 설치
### 1.3.1. 설치 요약
### 1.3.2. Localization 부분
#### 1.2.3.2. Linux 환경에서 부팅 가능한 USB 만들기
```
# dmesg
# dmesg | grep disk
```

```
$ su -
```

### 1.3.3. Software 부분
### 1.3.4. System 부분
### 1.3.5. Network 및 Hostname 부분
### 1.3.6. User 설정 부분
### 1.3.7. 설치 진행 단계
## 1.4. 보안 배포 체크리스트
### 1.4.1. SSH 연결에 대한 액세스 제한
```
# cp /etc/ssh/sshd_config /etc/ssh/sshd_config.orig

# sed -i -E 's/^#?(PermitRootLogin).*/\1 yes/' /etc/ssh/sshd_config

# systemctl restart sshd

# systemctl status sshd
```

## 1.5. Host 서버 기본 설정
### 1.5.1. timezone
### 1.5.2. Hostname 설정
### 1.5.3.  Firewalld and selinux
### 1.5.4. Network Interface 설정 - Host

