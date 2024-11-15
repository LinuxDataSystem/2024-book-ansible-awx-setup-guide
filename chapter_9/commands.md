# 제9장: 애플리케이션 배포 및 관리

## 9.1. 웹 애플리케이션 관리

### 9.1.1. 웹 애플리케이션 프로비저닝
#### 9.1.1.1. Project directory에 Playbook 파일 생성
```bash
# cd [PERSISTENT_VOLUME_HOST_PATH]

# cd lds-project

# vi provision-web-applications.yaml
```

```yaml
# provision-web-applications.yaml
---
- name: Provision Web Applications on Linux Servers
  hosts: all
  become: true

  tasks:
    - name: Install Apache httpd and Tomcat on Rocky Linux
      block:
        - name: Install httpd
          yum:
            name: httpd
            state: present
        - name: Start and enable httpd
          systemd:
            name: httpd
            state: started
            enabled: true
        - name: Install Tomcat
          yum:
            name: tomcat
            state: present
        - name: Start and enable Tomcat
          systemd:
            name: tomcat
            state: started
            enabled: true
        - name: Open firewall for Tomcat on Rocky Linux
          firewalld:
            port: "8080/tcp"
            permanent: true
            state: enabled
      when: ansible_os_family == "RedHat"

    - name: Install Apache httpd and Tomcat on Debian
      block:
        - name: Install httpd
          apt:
            name: apache2
            state: present
        - name: Start and enable httpd
          systemd:
            name: apache2
            state: started
            enabled: true
        - name: Install Tomcat
          apt:
            name: tomcat9
            state: present
        - name: Start and enable Tomcat
          systemd:
            name: tomcat9
            state: started
            enabled: true
      when: ansible_os_family == "Debian"

    - name: Open firewall for HTTP and HTTPS on Rocky Linux
      firewalld:
        service: "{{ item }}"
        permanent: true
        state: enabled
      loop: ['http', 'https']
      when: ansible_os_family == "RedHat"
```

### 9.1.2. 웹 애플리케이션 삭제
#### 9.1.2.1. Project directory에 Playbook 파일 생성
```bash
# cd [PERSISTENT_VOLUME_HOST_PATH]

# cd lds-project

# vi remove-web-applications.yaml
```

```bash
# remove-web-applications.yaml
---
- name: Remove Web Applications from Linux Servers
  hosts: all
  become: true

  tasks:
    - name: Stop and disable httpd on Rocky Linux
      systemd:
        name: httpd
        state: stopped
        enabled: no
      when: ansible_os_family == "RedHat"

    - name: Remove httpd from Rocky Linux
      yum:
        name: httpd
        state: absent
      when: ansible_os_family == "RedHat"

    - name: Stop and disable Tomcat on Rocky Linux
      systemd:
        name: tomcat
        state: stopped
        enabled: no
      when: ansible_os_family == "RedHat"

    - name: Remove Tomcat from Rocky Linux
      yum:
        name: tomcat
        state: absent
      when: ansible_os_family == "RedHat"

    - name: Stop and disable apache2 on Debian
      systemd:
        name: apache2
        state: stopped
        enabled: no
      when: ansible_os_family == "Debian"

    - name: Remove apache2 from Debian
      apt:
        name: apache2
        state: absent
      when: ansible_os_family == "Debian"

    - name: Stop and disable Tomcat9 on Debian
      systemd:
        name: tomcat9
        state: stopped
        enabled: no
      when: ansible_os_family == "Debian"

    - name: Remove Tomcat9 from Debian
      apt:
        name: tomcat9
        state: absent
      when: ansible_os_family == "Debian"
```

## 9.2. Jenkins 관리

### 9.2.1. Jenkins 프로비저닝
#### 9.2.1.1. Project directory에 Playbook 파일 생성
```bash
# cd [PERSISTENT_VOLUME_HOST_PATH]

# cd lds-project

# vi install-jenkins.yaml
```

```yaml
# install-jenkins.yaml
---
- name: Install and Configure Jenkins on Rocky Linux
  hosts: all
  become: true

  tasks:
    - name: Install Java
      yum:
        name: java-11-openjdk
        state: present

    - name: Add Jenkins repository
      yum_repository:
        name: jenkins
        description: Jenkins
        baseurl: https://pkg.jenkins.io/redhat-stable/
        gpgkey: https://pkg.jenkins.io/redhat-stable/jenkins.io.key
        gpgcheck: yes
        enabled: yes

    - name: Import Jenkins repository key
      command: rpm --import https://pkg.jenkins.io/redhat-stable/jenkins.io.key

    - name: Refresh repository cache
      command: yum makecache

    - name: Install Jenkins
      yum:
        name: jenkins
        state: present
        disable_gpg_check: yes  

    - name: Start and enable Jenkins service
      systemd:
        name: jenkins
        state: started
        enabled: true

    - name: Open firewall for Jenkins
      firewalld:
        service: http
        zone: public
        permanent: true
        state: enabled

    - name: Open Jenkins port for external access
      firewalld:
        port: "8080/tcp"
        zone: public
        permanent: true
        state: enabled

    - name: Get initialAdminPassword
      shell: cat /var/lib/jenkins/secrets/initialAdminPassword
      register: jenkins_admin_password
      changed_when: false

    - name: Display Jenkins initial admin password
      debug:
        msg: "The initial Jenkins admin password is: {{ jenkins_admin_password.stdout }}"

    - name: Wait for Jenkins to start up and log in
      uri:
        url: "http://{{ ansible_default_ipv4.address }}:8080/login"
        method: GET
        return_content: yes
        status_code: 200
      register: login_page
      retries: 10
      delay: 10
      until: login_page.status == 200
      when: jenkins_admin_password.stdout is defined

    - name: Reload firewalld
      systemd:
        name: firewalld
        state: reloaded

    - name: Restart Jenkins
      systemd:
        name: jenkins
        state: restarted
```