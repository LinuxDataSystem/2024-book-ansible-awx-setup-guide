# 제7장: Ansible AWX로 서버 유지 관리 및 업데이트

## 7.1. 서버 패키지 관리

### 7.1.1. 실습 환경 구성

### 7.1.2. 필수 시스템 소프트웨어 설치
#### 7.1.2.1. Project directory에 Playbook 파일 생성
```bash
# cd [PERSISTENT_VOLUME_HOST_PATH]

# cd lds-project

# vi install-required-package.yaml
```

```yaml
# install-required-package.yaml
- name: Install essential system software
  hosts: all
  become: yes

  tasks:
    - name: Install a list of required packages
      yum:
        name:
          - vim
          - curl
          - git
        state: present

```

### 7.1.3. 서버 패키지 업데이트
#### 7.1.3.1. Project directory에 Playbook 파일 생성
```bash
# cd [PERSISTENT_VOLUME_HOST_PATH]

# cd lds-project

# vi update-package.yaml
```

```yaml
# update-package.yaml
---
- name: Update all packages on the servers
  hosts: all
  become: yes  

  tasks:
    - name: Ensure all packages are up to date
      yum:
        name: '*'
        state: latest
      register: update_result

    - name: Check if kernel updates were installed
      set_fact:
        kernel_updated: "{{ update_result.results | selectattr('name', 'defined') | map(attribute='name') | select('search', 'kernel') | list | length > 0 }}"
      when: update_result is defined and update_result.changed

    - name: Reboot the machine if kernel updates were installed
      reboot:
        msg: "Reboot initiated by Ansible due to kernel updates"
        connect_timeout: 5
        reboot_timeout: 300
        pre_reboot_delay: 5
        post_reboot_delay: 30
        test_command: whoami
      when: kernel_updated | default(false)
```

## 7.2. 서버 보안 업데이트

### 7.2.1. 보안 패치 적용
#### 7.2.1.1. Project directory에 Playbook 파일 생성
```bash
# cd [PERSISTENT_VOLUME_HOST_PATH]

# cd lds-project

# vi apply-security-patches.yaml
```

```yaml
# apply-security-patches.yaml
---
- name: Apply Security Patches
  hosts: all
  become: yes

  tasks:
  - name: Install security updates on CentOS/RHEL or Rocky
    yum:
      name: '*'
      state: latest
      security: yes
    when: ansible_os_family == "RedHat"
    register: yum_updates

  - name: Install security updates on Debian/Ubuntu
    apt:
      upgrade: 'dist'
      update_cache: yes
      force_apt_get: yes
      only_upgrade: yes
    when: ansible_os_family == "Debian"
    register: apt_updates

  - name: Check if a reboot is needed on Debian/Ubuntu
    stat:
      path: /var/run/reboot-required
    register: reboot_required
    when: ansible_os_family == "Debian"

  - name: Reboot the machine if needed for Debian/Ubuntu
    reboot:
      msg: "Rebooting because of security updates (Debian/Ubuntu)"
      connect_timeout: 5
      reboot_timeout: 600
    when: ansible_os_family == "Debian" and reboot_required.stat.exists

  - name: Reboot the machine if needed for CentOS/RHEL or Rocky
    reboot:
      msg: "Rebooting because of security updates (CentOS/RHEL or Rocky)"
      connect_timeout: 5
      reboot_timeout: 600
    when: ansible_os_family == "RedHat" and yum_updates.changed
```

### 7.2.2. 취약점 스캔 및 보고
#### 7.2.2.1. Project directory에 Playbook 파일 생성
```bash
# cd [PERSISTENT_VOLUME_HOST_PATH]

# cd lds-project

# vi vulnerability-scanning.yaml
```

```yaml
# vulnerability-scanning.yaml
---
- name: Vulnerability Scanning and Reporting
  hosts: all
  become: yes

  tasks:
    - name: Install EPEL Repository for CentOS/RHEL/Rocky
      yum:
        name: epel-release
        state: present
      when: ansible_os_family == "RedHat"

    - name: Install ClamAV on Debian/Ubuntu
      apt:
        name:
          - clamav
          - clamav-freshclam
        state: present
      when: ansible_os_family == "Debian"

    - name: Install ClamAV on CentOS/RHEL/Rocky
      yum:
        name:
          - clamav
          - clamav-update
        state: present
      when: ansible_os_family == "RedHat"

    - name: Update ClamAV database
      command: freshclam
      ignore_errors: yes

    - name: Check if /home exists
      stat:
        path: /home
      register: home_dir

    - name: Perform a ClamAV scan on /home if it exists
      command: clamscan -r /home --log=/var/log/clamscan_home.log
      register: scan_result
      when: home_dir.stat.exists

    - name: Print scan results
      debug:
        msg: "Scan completed. See /var/log/clamscan_home.log for details."
      when: scan_result is defined and scan_result.rc == 0
```