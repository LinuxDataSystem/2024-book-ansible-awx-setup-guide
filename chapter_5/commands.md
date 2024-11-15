# 제5장: Ansible 기본 사용법

## 5.1. Ansible
### 5.1.1. Ansible 구성 요소
### 5.1.2. Ansible 소개
### 5.1.3. Ansible 개념 정리

## 5.2. Ansible 시작하기
### 5.2.1. Ansible 설치
```bash
# dnf install python3-pip
```

```bash
# pip install ansible
```

```bash
# pip install ansible-lint
```

### 5.2.2. Ansible Inventories 구축
#### 5.2.2.1. 작업 환경 구성
```bash
# mkdir ansible_quickstart && cd ansible_quickstart
```

```bash
# ssh-keygen -t rsa -b 4096
```

```bash
# ssh-copy-id -i ~/.ssh/id_rsa.pub root@192.168.50.110

# ssh-copy-id -i ~/.ssh/id_rsa.pub root@192.168.50.120
```

```bash
# ssh root@192.168.50.120 cat ~/.ssh/authorized_keys
```

#### 5.2.2.2. Ansible Inventories 구축
##### 5.2.2.2.1. Ansible Inventories 구축 - 방법
```ini
# inventory.ini
[myhosts]
192.168.50.110
192.168.50.120
```

```bash
# ansible-inventory -i inventory.ini --list

# ansible myhosts -m ping -i inventory.ini
```

#### 5.2.2.3. Ansible Inventories 구축 팁
##### 5.2.2.3.1. Ansible Inventories 구축 팁 - 방법
##### 5.2.2.3.2. Ansible Inventories 구축 팁 - 실습
```ini
# inventory.yaml
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

```bash
# ansible-inventory -i inventory.yaml --list

# ansible awx_servers -m ping -i inventory.yaml

# ansible awx_guests -m ping -i inventory.yaml

# ansible awx -m ping -i inventory.yaml
```

### 5.2.3. Ansible playbook 생성
#### 5.2.3.1. Ansible playbook 개요
##### 5.2.3.1.5. Playbook 실행 - check mode
```bash
# ansible-playbook --check playbook.yaml
```

#### 5.2.3.2. playbook 생성 - Hello world
##### 5.2.3.2.1. playbook 생성 - Hello world - 방법
```yaml
# playbook.yaml
- name: My first play
  hosts: myhosts
  tasks:
  - name: Ping my hosts
    ansible.builtin.ping:

  - name: Print message
    ansible.builtin.debug:
      msg: Hello world
```

##### 5.2.3.2.2. playbook 생성 - Hello world - 실습
```yaml
# playbook.yaml
- name: My first play
  hosts: awx
  tasks:
  - name: Ping my hosts
    ansible.builtin.ping:

  - name: Print message
    ansible.builtin.debug:
      msg: Hello world
```

```bash
# ansible-lint playbook.yaml
```

```bash
# ansible-playbook -i inventory.yaml playbook.yaml
```

#### 5.2.3.3. playbook 생성 - Httpd
##### 5.2.3.3.1. playbook 생성 - Httpd - 방법
```yaml
# playbook-httpd.yaml
---
- name: Update web servers
  hosts: awx_guests
  remote_user: root

  tasks:
  - name: Install the latest version of Apache
    ansible.builtin.dnf:
      name: httpd
      state: present

  - name: Ensure httpd is running and enabled
    ansible.builtin.service:
      name: httpd
      state: started
      enabled: yes

  - name: Open HTTP and HTTPS ports on the firewall
    ansible.builtin.firewalld:
      service: "{{ item }}"
      permanent: true
      state: enabled
    loop:
      - http
      - https

  - name: Reload firewall to apply changes
    ansible.builtin.command: firewall-cmd --reload

  - name: Print message
    ansible.builtin.debug:
      msg: Install Apache Httpd
```

##### 5.2.3.3.2. playbook 생성 - Httpd - 실습
```yaml
# playbook-httpd.yaml
---
- name: Update web servers
  hosts: awx_guests
  remote_user: root

  tasks:
  - name: Install the latest version of Apache
    ansible.builtin.dnf:
      name: httpd
      state: present

  - name: Ensure httpd is running and enabled
    ansible.builtin.service:
      name: httpd
      state: started
      enabled: yes

  - name: Open HTTP and HTTPS ports on the firewall
    ansible.builtin.firewalld:
      service: "{{ item }}"
      permanent: true
      state: enabled
    loop:
      - http
      - https

  - name: Reload firewall to apply changes
    ansible.builtin.command: firewall-cmd --reload

  - name: Print message
    ansible.builtin.debug:
      msg: Install Apache Httpd
```

```bash
# ansible-lint playbook-httpd.yaml
```

```bash
# ansible-playbook --check -i inventory.yaml playbook-httpd.yaml
```

```bash
# ansible-playbook -i inventory.yaml playbook-httpd.yaml
```

#### 5.2.3.4. Playbook 재사용
##### 5.2.3.4.1. Playbook 재사용 실습 - Httpd - 방법
```yaml
# playbook-main.yaml
---
- name: Start Playbook
  hosts: awx_guests
  gather_facts: false
  tasks:
    - ansible.builtin.debug:
        msg: play1

- import_playbook: playbook-httpd.yaml
```

```bash
# ansible-playbook -i inventory.yaml playbook-httpd.yaml
```

## 5.3. Ansible Vault
### 5.3.1. Ansible Vault 민감한 데이터 보호
### 5.3.2. Ansible Vault 비밀번호 관리 전략
### 5.3.3. Ansible Vault 개별 변수 암호화
#### 5.3.3.1. 암호화된 변수 생성
```bash
# echo PASSWORD > a_password_file

# cat a_password_file

# ansible-vault encrypt_string --vault-password-file a_password_file 'foobar' --name 'the_secret'
```

```bash
# ansible-vault encrypt_string --vault-id dev@a_password_file 'foooodev' --name 'the_dev_secret'

# ansible-vault encrypt_string --vault-id dev@a_password_file 'foooodev' --name 'the_dev_secret' > vars.yml
```

```bash
# echo -n 'letmein' | ansible-vault encrypt_string --vault-id dev@a_password_file --stdin-name 'db_password'
```

#### 5.3.3.2. 암호화된 변수 보기
```bash
# ansible localhost -m ansible.builtin.debug -a var="the_dev_secret" -e "@vars.yml" --vault-id dev@a_password_file
```

### 5.3.4. Ansible Vault 파일 암호화
#### 5.3.4.1. 암호화된 파일 생성
```bash
# ansible-vault create --vault-id test@a_password_file foo-00.yml
```

```bash
# ansible-vault create --vault-id my_new_password@prompt foo-01.yml
```

#### 5.3.4.2. 암호화된 파일 보기
```bash
# cat a_password_file

# ansible-vault view foo-00.yml
```

## 5.4. Ansible 응용하기
### 5.4.1. Ansible-Pull
```bash
# ansible-pull -U <repository_url> [options]
```

### 5.4.2. Ansible-Lint
```bash
# ansible-lint verify-apache.yml
```