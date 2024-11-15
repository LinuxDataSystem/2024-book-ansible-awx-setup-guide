# 제6장: Ansible AWX 기본 사용법

## 6.1. 로그인
### 6.1.1. 웹 UI 정보 확인
```bash
# ip -br addr
```

```bash
# kubectl get svc -l "app.kubernetes.io/managed-by=awx-operator"
```

### 6.1.2. admin 비밀번호 확인
```bash
# kubectl get secret awx-demo-admin-password -o jsonpath="{.data.password}" | base64 --decode ; echo
```

### 6.1.3. AWX UI 로그인

## 6.2. 사용자 인터페이스
### 6.2.1. Views
### 6.2.2. Resources
### 6.2.3. Access
### 6.2.4. Administration
### 6.2.5. Settings

## 6.3. Access
### 6.3.1. Organizations
### 6.3.2. Users
### 6.3.3. Teams

## 6.4. Resources
### 6.4.1. Credentials
### 6.4.2. Projects
#### 6.4.2.2. 생성
##### 6.4.2.2.2. Project 생성 - 실습 - 준비
###### 6.4.2.2.2.1. Persistent Volume에 디렉토리 생성
```bash
# kubectl get deployment.apps/awx-demo-web -o yaml 

# kubectl get pvc

# kubectl get pv
```

```bash
# kubectl get pv pvc-[NAME] -o yaml
```

```bash
# cd [PERSISTENT_VOLUME_HOST_PATH]

# mkdir lds-project

# chmod -R 777 lds-project
```

```bash
# kubectl exec -it awx-demo-web-[POD_NAME] /bin/bash

# cd /var/lib/awx/projects

# ls
```


### 6.4.3. Inventories
#### 6.4.3.2. 추가
##### 6.4.3.2.2. Inventory 추가 - 실습 
```ini
# inventory.ini
---
awx_servers:
  hosts:
    awx_vm:
      ansible_host: 192.168.50.110
awx_guests:
  hosts:
    guest_vm:
      ansible_host: 192.168.50.120
awx:
  children:
    awx_servers:
    awx_guests:

```

###### 6.4.3.2.2.4. Inventory 추가 - 실습 - Hosts - awx_vm
```yaml
# [그림 : Ansible AWX - Resources - Inventories - lds-inventory - Groups - awx_servers]
---
ansible_host: 192.168.50.110
```

###### 6.4.3.2.2.5. Inventory 추가 - 실습 - Hosts - guest_vm
```YAML
# [그림 : Ansible AWX - Resources - Inventories - lds-inventory - Groups - awx_guests - Host - guest_vm - Result]
---
ansible_host: 192.168.50.120
```

### 6.4.4. Templates
#### 6.4.4.2. 추가 - Job Template
##### 6.4.4.2.2. 추가 - Job Template - 실습 - 준비
###### 6.4.4.2.2.1. Project directory에 Playbook 파일  생성
```bash
# cd [PERSISTENT_VOLUME_HOST_PATH]

# cd lds-project

# vi playbook.yaml
```

```yaml
# playbook.yaml
- name: My first play
  hosts: localhost
  tasks:
  - name: Ping my hosts
    ansible.builtin.ping:

  - name: Print message
    ansible.builtin.debug:
      msg: Hello world
```