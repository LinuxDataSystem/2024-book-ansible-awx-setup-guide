# 제8장: 서버 보안 구성

## 8.1. 방화벽 설정

### 8.1.1. 서버 방화벽 규칙 구성
#### 8.1.1.1. Project directory에 Playbook 파일 생성
```bash
# cd [PERSISTENT_VOLUME_HOST_PATH]

# cd lds-project

# vi configure-firewall-rules.yaml
```

```yaml
# configure-firewall-rules.yaml
---
- name: Configure Firewall Rules
  hosts: all
  become: yes
  tasks:
    - name: Ensure firewalld is installed
      yum:
        name: firewalld
        state: present

    - name: Start and enable firewalld
      systemd:
        name: firewalld
        enabled: yes
        state: started

    - name: Allow SSH connections
      firewalld:
        service: ssh
        permanent: true
        state: enabled
        immediate: yes

    - name: Allow HTTP traffic
      firewalld:
        service: http
        permanent: true
        state: enabled
        immediate: yes

    - name: Allow HTTPS traffic
      firewalld:
        service: https
        permanent: true
        state: enabled
        immediate: yes

    - name: Reload firewalld to apply changes
      systemd:
        name: firewalld
        state: restarted

```

### 8.1.2. 방화벽 로깅 및 모니터링 설정
#### 8.1.2.1. Project directory에 Playbook 파일 생성
```bash
# cd [PERSISTENT_VOLUME_HOST_PATH]

# cd lds-project

# vi configure-firewall-logging.yaml
```

```yaml
# configure-firewall-logging.yaml
---
- name: Configure Firewall Logging
  hosts: all
  become: yes
  tasks:
    - name: Install necessary packages for logging
      yum:
        name: rsyslog
        state: present

    - name: Start and enable rsyslog
      systemd:
        name: rsyslog
        enabled: yes
        state: started

    - name: Configure firewalld to log all denied connections
      firewalld:
        rich_rule: 'rule family="ipv4" protocol value="tcp" log prefix="firewalld-denied: " level="info" limit value="1/m" accept'
        zone: public
        permanent: true
        immediate: yes
        state: enabled

    - name: Configure rsyslog to store firewalld logs
      lineinfile:
        path: /etc/rsyslog.conf
        line: 'if $programname == "firewalld" and $msg contains "denied" then /var/log/firewalld_denied.log'
        create: yes
      notify:
        - restart rsyslog

    - name: Ensure log file exists for firewalld denied logs
      file:
        path: /var/log/firewalld_denied.log
        state: touch
        owner: root
        group: root
        mode: '0644'

  handlers:
    - name: restart rsyslog
      systemd:
        name: rsyslog
        state: restarted
```


## 8.2. 액세스 및 인증 관리

### 8.2.1. 서버 패스워드 정책 구성
#### 8.2.1.1. Project directory에 Playbook 파일 생성
```bash
# cd [PERSISTENT_VOLUME_HOST_PATH]

# cd lds-project

# vi configure-password-policy.yaml
```

```yaml
# configure-password-policy.yaml
---
- name: Configure Password Policy for Multiple Distributions
  hosts: all
  become: yes
  tasks:
    - name: Install EPEL Repository for Rocky Linux
      yum:
        name: epel-release
        state: present
      when: ansible_os_family == "RedHat"

    - name: Install PAM password quality control for Rocky Linux
      yum:
        name: libpwquality  
        state: present
      when: ansible_os_family == "RedHat"

    - name: Install PAM password quality control for Debian
      apt:
        name: libpam-pwquality
        state: present
      when: ansible_os_family == "Debian"

    - name: Configure password quality requirements
      lineinfile:
        path: /etc/security/pwquality.conf
        regexp: '^{{ item.option }}'
        line: '{{ item.option }} = {{ item.value }}'
        state: present
      loop:
        - { option: 'minlen', value: '12' }
        - { option: 'dcredit', value: '-1' }
        - { option: 'ucredit', value: '-1' }
        - { option: 'ocredit', value: '-1' }
        - { option: 'lcredit', value: '-1' }
      when: ansible_os_family == "RedHat" or ansible_os_family == "Debian"
```

### 8.2.2. Sudo 권한 설정
#### 8.2.2.1. Project directory에 Playbook 파일 생성
```bash
# cd [PERSISTENT_VOLUME_HOST_PATH]

# cd lds-project

# vi configure-sudo-privileges.yaml
```

```yaml
# configure-sudo-privileges.yaml
---
- name: Configure Sudo Privileges for Specific Groups
  hosts: all
  become: true
  tasks:
    - name: Ensure sudo package is installed
      package:
        name: sudo
        state: present

    - name: Grant sudo privileges to the wheel group on Rocky Linux
      lineinfile:
        path: /etc/sudoers
        regexp: '^%wheel'
        line: '%wheel ALL=(ALL) NOPASSWD: ALL'
        validate: '/usr/sbin/visudo -cf %s'
      when: ansible_os_family == "RedHat"

    - name: Grant sudo privileges to the sudo group on Debian
      lineinfile:
        path: /etc/sudoers
        regexp: '^%sudo'
        line: '%sudo ALL=(ALL) NOPASSWD: ALL'
        validate: '/usr/sbin/visudo -cf %s'
      when: ansible_os_family == "Debian"
```