---
title: Ansible 과 Vagrant로 HCloud-Platform 배포하는 방법 - 1편
subtitle: 
layout: post
icon: fa-book
order: 3

---



 저희 HCloud-Platform 은 Middleware 들이 운영되고 있는 Master 노드와 단일 가상 서버에서 사용할 볼륨들을 프로비저닝 해주는 Storage 노드를 통해 단일 가상 서버를 구성하여 배포합니다.



 이 글에서는 Vagrant 파일을 작성하여 Virtual Box 에 Master 노드와 Storage 노드를 구성하고 Ansible 을 통해 미들웨어들이 동작하도록 하는 환경을 구성해보도록 하겠습니다.



  이번 편에서는 Vagrant 로 각 노드들을 구성하고 Master 노드에서 MySQL 과 필수적인 몇몇 미들웨어만 운영을 하여 API Gateway 까지 동작하는 부분만 보여드리도록 하겠습니다.

 다음 편에서는 InfluxDB 와 RabbitMQ 을 연동하여 모든 미들웨어가 운영 가능 할 수 있도록 하는 과정에 대해서 보여드리도록 하겠습니다.



## Vagrant 란?

Vagrant 는 가상머신을 관리해주는 도구입니다. VirtualBox 가상 머신을 이용하여 가상 머신에서 사용할 프로그램들과 사전 설정들을 미리 자동으로 구성할 수 있도록 해줍니다.

Vagrantfile 이라는 파일을 작성하여 이를 동일한 환경을 구성하도록 관리할 수 있습니다. 



## Vagrantfile 작성

먼저 미들웨어들을 실행할 master 노드와 볼륨을 프로비저닝할 FreeBSD 운영체제를 사용하는 Storage 노드를 구성해주는 Vagrant 파일을 작성해 줍니다.

Vagrantfile 의 문법은 Ruby 언어를 따릅니다.



```shell
$ vi Vagrantfile
```

```vagrant
{% raw %}
# -*- mode: ruby -*-
# vi: set ft=ruby :

Vagrant.configure("2") do |config|

  # VM1 - hcc-master
  config.vm.define "hcc-master" do |master|
    master.vm.box = "ubuntu/focal64"
    master.vm.provider "virtualbox" do |vb|
      vb.name = "hcc-master"
      vb.cpus = 2
      vb.memory = 2000
    end
    master.vm.hostname = "master"
    master.vm.network "private_network", ip: "172.168.10.100"

    # Enable SSH Password Authentication
    master.vm.provision "shell", inline: <<-SHELL
      sed -i 's/ChallengeResponseAuthentication no/ChallengeResponseAuthentication yes/g' /etc/ssh/sshd_config
      sed -i 's/archive.ubuntu.com/mirror.kakao.com/g' /etc/apt/sources.list
      sed -i 's/security.ubuntu.com/mirror.kakao.com/g' /etc/apt/sources.list
      systemctl restart ssh
    SHELL
  end

  # VM2 - hcc-storage
  config.vm.define "hcc-storage" do |storage|
    storage.vm.box = "freebsd/FreeBSD-12.2-STABLE"
    storage.vm.provider "virtualbox" do |vb|
      vb.name = "hcc-storage"
      vb.cpus = 2
      vb.memory = 2000
    end
    storage.ssh.shell = "sh"
    storage.vm.hostname = "storage"
    storage.vm.network "private_network", ip: "172.168.10.101"
  end

end
{% endraw %}
```



여기서는 다음과 같은 환경을 구성하였습니다.

- hcc-master
  - 미들웨어들을 운영하고 단일 가상 서버를 구성해주는 역할을 하는 master 노드
  - 사용 이미지 : ubuntu/focal64 (20.04 LTS 64bit)
  - CPU : 2개
  - RAM : 2GB
  - HOSTNAME : master
  - Private Network IP : 172.168.10.100

- hcc-storage
  - 단일 가상 서버에서 사용되는 볼륨을 프로비저닝 해주는 역할을 하는 storage 노드
  - 사용 이미지 : freebsd/FreeBSD-12.2-STABLE
  - CPU : 2개
  - RAM : 2GB
  - HOSTNAME : storage
  - Private Network IP : 172.168.10.101



Vagrantfile 을 작성하고 난 후 `vagrant up` 명령을 실행하면 가상 머신이 만들어지고 부팅이 완료되면 SSH 접속이 가능하도록 설정이 됩니다.

```shell
$ vagrant up
Bringing machine 'hcc-master' up with 'virtualbox' provider...
Bringing machine 'hcc-storage' up with 'virtualbox' provider...
==> hcc-master: Importing base box 'ubuntu/focal64'...
==> hcc-master: Matching MAC address for NAT networking...
==> hcc-master: Checking if box 'ubuntu/focal64' version '20211001.0.0' is up to date...
==> hcc-master: Setting the name of the VM: hcc-master
==> hcc-master: Clearing any previously set network interfaces...
==> hcc-master: Preparing network interfaces based on configuration...
    hcc-master: Adapter 1: nat
    hcc-master: Adapter 2: hostonly
==> hcc-master: Forwarding ports...
    hcc-master: 22 (guest) => 2222 (host) (adapter 1)
==> hcc-master: Running 'pre-boot' VM customizations...
==> hcc-master: Booting VM...
==> hcc-master: Waiting for machine to boot. This may take a few minutes...
    hcc-master: SSH address: 127.0.0.1:2222
    hcc-master: SSH username: vagrant
    hcc-master: SSH auth method: private key
    hcc-master: Warning: Connection reset. Retrying...
    hcc-master: 
    hcc-master: Vagrant insecure key detected. Vagrant will automatically replace
    hcc-master: this with a newly generated keypair for better security.
    hcc-master: 
    hcc-master: Inserting generated public key within guest...
    hcc-master: Removing insecure key from the guest if it's present...
    hcc-master: Key inserted! Disconnecting and reconnecting using new SSH key...
==> hcc-master: Machine booted and ready!
==> hcc-master: Checking for guest additions in VM...
==> hcc-master: Setting hostname...
==> hcc-master: Configuring and enabling network interfaces...
==> hcc-master: Mounting shared folders...
    hcc-master: /vagrant => /home/jollaman999/1111
==> hcc-master: Running provisioner: shell...
    hcc-master: Running: inline script
==> hcc-storage: Importing base box 'freebsd/FreeBSD-12.2-STABLE'...
==> hcc-storage: Generating MAC address for NAT networking...
==> hcc-storage: Checking if box 'freebsd/FreeBSD-12.2-STABLE' version '2021.09.30' is up to date...
==> hcc-storage: Setting the name of the VM: hcc-storage
==> hcc-storage: Fixed port collision for 22 => 2222. Now on port 2200.
==> hcc-storage: Clearing any previously set network interfaces...
==> hcc-storage: Preparing network interfaces based on configuration...
    hcc-storage: Adapter 1: nat
    hcc-storage: Adapter 2: hostonly
==> hcc-storage: Forwarding ports...
    hcc-storage: 22 (guest) => 2200 (host) (adapter 1)
==> hcc-storage: Running 'pre-boot' VM customizations...
==> hcc-storage: Booting VM...
==> hcc-storage: Waiting for machine to boot. This may take a few minutes...
    hcc-storage: SSH address: 127.0.0.1:2200
    hcc-storage: SSH username: vagrant
    hcc-storage: SSH auth method: private key
    hcc-storage: Warning: Connection reset. Retrying...
    hcc-storage: Warning: Connection reset. Retrying...
    hcc-storage: Warning: Connection reset. Retrying...
    hcc-storage: Warning: Connection reset. Retrying...
    hcc-storage: Warning: Remote connection disconnect. Retrying...
    hcc-storage: Warning: Connection reset. Retrying...
    hcc-storage: Warning: Connection reset. Retrying...
    hcc-storage: Warning: Connection reset. Retrying...
    hcc-storage: Warning: Connection reset. Retrying...
    hcc-storage: Warning: Remote connection disconnect. Retrying...
    hcc-storage: Warning: Connection reset. Retrying...
    hcc-storage: Warning: Connection reset. Retrying...
    hcc-storage: Warning: Connection reset. Retrying...
    hcc-storage: Warning: Connection reset. Retrying...
    hcc-storage: Warning: Connection reset. Retrying...
    hcc-storage: Warning: Remote connection disconnect. Retrying...
    hcc-storage: Warning: Connection reset. Retrying...
    hcc-storage: Warning: Connection reset. Retrying...
    hcc-storage: Warning: Connection reset. Retrying...
    hcc-storage: 
    hcc-storage: Vagrant insecure key detected. Vagrant will automatically replace
    hcc-storage: this with a newly generated keypair for better security.
    hcc-storage: 
    hcc-storage: Inserting generated public key within guest...
    hcc-storage: Removing insecure key from the guest if it's present...
    hcc-storage: Key inserted! Disconnecting and reconnecting using new SSH key...
==> hcc-storage: Machine booted and ready!
==> hcc-storage: Checking for guest additions in VM...
==> hcc-storage: Setting hostname...
==> hcc-storage: Configuring and enabling network interfaces...
==> hcc-storage: Mounting shared folders...
    hcc-storage: /vagrant => /home/jollaman999/1111
```



VirtualBox 를 열어서 확인해 보면 다음과 같이 생성한 가상 머신들이 동작중인 것을 확인 할 수 있습니다.

![image-20211026191744571](/assets/images/docs/Ansible 과 Vagrant로 HCloud-Platform 배포하는 방법 - 1편/image-20211026191744571.png)



현재 로컬 머신에서 가상 머신으로 SSH 접속을 키 인증 방식으로 접속하기 위해 `ssh-copy-id` 명령어를 통하여 로컬 머신의 SSH PublicKey 를 가상 머신에 등록합니다.

최초 사용자명과 비밀번호는 `vagrant` 입니다.

```shell
$ ssh-copy-id vagrant@172.168.10.100
/usr/bin/ssh-copy-id: INFO: attempting to log in with the new key(s), to filter out any that are already installed
/usr/bin/ssh-copy-id: INFO: 1 key(s) remain to be installed -- if you are prompted now it is to install the new keys
Password: 

Number of key(s) added: 1

Now try logging into the machine, with:   "ssh 'vagrant@172.168.10.100'"
and check to make sure that only the key(s) you wanted were added.
```

```
$ ssh-copy-id vagrant@172.168.10.101
The authenticity of host '172.168.10.101 (172.168.10.101)' can't be established.
ECDSA key fingerprint is SHA256:Z9UXJxKLgiD6HP039vokYWqWHRpwgk0CrCD4hQiQuL4.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
/usr/bin/ssh-copy-id: INFO: attempting to log in with the new key(s), to filter out any that are already installed
/usr/bin/ssh-copy-id: INFO: 1 key(s) remain to be installed -- if you are prompted now it is to install the new keys
Password for vagrant@storage:

Number of key(s) added: 1

Now try logging into the machine, with:   "ssh 'vagrant@172.168.10.101'"
and check to make sure that only the key(s) you wanted were added.
```



`vagrant ssh [가상머신 이름]` 명령을 통해 가상 머신에 SSH 로 접속 할 수 있습니다.

```shell
$ vagrant ssh hcc-master
Welcome to Ubuntu 20.04.3 LTS (GNU/Linux 5.4.0-88-generic x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/advantage

  System information as of Wed Oct 27 01:54:41 UTC 2021

  System load:  0.0               Processes:               121
  Usage of /:   3.5% of 38.71GB   Users logged in:         0
  Memory usage: 13%               IPv4 address for enp0s3: 10.0.2.15
  Swap usage:   0%                IPv4 address for enp0s8: 172.168.10.100

 * Super-optimized for small spaces - read how we shrank the memory
   footprint of MicroK8s to make it the smallest full K8s around.

   https://ubuntu.com/blog/microk8s-memory-optimisation

1 update can be applied immediately.
To see these additional updates run: apt list --upgradable


The list of available updates is more than a week old.
To check for new updates run: sudo apt update

vagrant@master:~$ 
```



## Ansible 인벤토리 파일 작성

인벤토리 파일은 관리노드에 접속하기 위해 호스트들이 정보가 작성되어 있는 파일입니다.

INI 또는 YAML 파일로 작성할 수 있는데, 보통 INI 파일로 많이 사용하고 작성하기도 쉽습니다.



```shell
$ vi inventory.ini
```

```
172.168.10.1 ansible_connection=local

[mgmt]
172.168.10.100
172.168.10.200
```



여기서는 위에서 Vagrant 로 만든 가상 머신들이 사용하는 게이트웨이 주소를 localhost 로 지정하고,

두 가상 머신을 mgmt 라는 그룹으로 지정하였습니다.



## Ansible Config 파일 작성

다음으로 Ansible 구성 파일을 작성합니다.

Ansible 의 작동방식을 구성하는 파일입니다.

`ansible` 또는 `ansible-playbook` 명령을 실행할때 제일 처음에 현재 폴더에 `ansible.cfg` 파일이 있는지 확인합니다.

다음으로 홈폴더에 `.ansible.cfg` 파일이 있는지 확인한 후 둘 다 존재 하지 않을 경우 `/etc/ansible/ansible.cfg` 설정파일을 참고합니다.



여기서는 현재 폴더에 Ansible 구성파일을 작성하도록 하겠습니다.

```shell
$ vi ansible.cfg
```

```
{% raw %}
[defaults]
remote_user = vagrant
inventory = inventory.ini
ask_pass = false
callback_whitelist = profile_tasks, profile_roles
ansible_managed = Ansible managed: {file} modified on %Y-%m-%d %H:%M:%S by {uid} on {host} {file}

[privilege_escalation]
become = true
become_method = sudo
become_user = root
become_ask_pass = false

[ssh_connection]
pipelining = true
{% endraw %}
```



- defaults
  - remote_user : SSH 로 접속할 계정 명입니다. 여기서는 Vagrant 기본 사용자인 vagrant 로 접속하도록 했습니다.
  - inventory : 인벤토리 파일을 지정합니다. 위에서 만든 `inventory.ini` 파일을 참고합니다.
  - ask_pass : SSH 접속시 비밀번호를 묻게 할 건지 설정하는 옵션입니다. `ssh-copy-id` 를 통해 로컬 머신의 공개키를 가상머신에 복사하였기 때문에 해당 옵션을 비활성화 하고 키 인증 방식으로 접속합니다.
  - callback_whitelist : profile_tasks 와 profile_roles 를 설정하여 각 task 와 role 이 끝날때 마다 시간이 얼마나 걸렸는지 표시해 줍니다.

- privilege_escalation
  - become, become_method  : SSH 접속후 sudo 명령을 실행하도록 합니다.
  - become_user : root 계정으로 권한 상승이 되도록 합니다.
  - become_ask_pass : sudo 명령을 실행할때 비밀번호를 묻지 않게 합니다. vagrant 기본 계정으로 sudo 명령을 실행할때 기본적으로 비밀번호를 묻지 않게 설정이 되므로, 해당 옵션을 비활성화 하였습니다.

- ssh_connection
  - pipelining : Ansible 이 기본적으로 임시디렉터리를 생성하고 실행할 명령들을 대상 호스트에 복사한 후에 실행을 합니다. pipelining 옵션을 키면 바로 SSH 로 접속하여 명령들을 실행하기 때문에 성능을 개선할 수 있습니다.



## Ansible Playbook 작성

각  실행 단위의 작업들은 Task 라는 단위로 구분됩니다.

관련 있는 여러 Task 들을 모아 하나의 Role 로 구성할 수 있습니다.

또한, 작업을 실행하고 난 후 다른 작업을 호출하여 실행할 수 있도록 Handler 가 존재합니다.



그리고 여러 Role 들은 다음과 같은 폴더 구조를 가집니다.

![image-20211027134751606](/assets/images/docs/Ansible 과 Vagrant로 HCloud-Platform 배포하는 방법 - 1편/image-20211027134751606.png)



- tasks/main.yml : 여러 task 들을 정의 합니다.
- handlers/main.yml : 여러 hanler 들을 정의 합니다.
- tests/test.yml : 역할을 테스트 하기 위한 플레이 북입니다.
- defaults/main.yml : 기본 변수들을 정의 합니다. (매우 낮은 우선순위를 가집니다.)
- vars/main.yml : 변수들을 정의 합니다. (상대적으로 높은 우선순위를 가집니다.)



본 글에서는 다음과 같은 구조로 Playbook 을 작성하였습니다.



### MySQL 구성하기

먼저 MySQL 을 구성하기 위해서 `roles/mysql` 폴더를 만들어 주고 `roles/mysql/tasks/main.yml` 에 작업들이 정의 되어 있는 파일을 import 해줍니다.

```shell
$ mkdir -p roles/mysql/tasks
$ vi roles/mysql/tasks/main.yml
```

```yaml
- import_tasks: ubuntu_mysql_package.yml
- import_tasks: mysql_setup.yml
```



다음으로 Ubuntu 가 설치 되어 있는 Master 노드에 MySQL 을 설치 하기 위한 작업들을 정의한 파일을 작성해 줍니다.

```shell
$ vi roles/mysql/tasks/ubuntu_mysql_package.yml
```

```yaml
- name: Update and Install Package for MySQL
  apt:
    name: mysql-server, mysql-client, python3-pymysql
    update_cache: true
    state: present
```



이제 MySQL 을 재시작하는 작업을 수행하는 Handler 를 작성해 줍니다.

```shell
$ mkdir -p roles/mysql/handlers
$ vi roles/mysql/handlers/main.yml
```

```yaml
- name: Restart MySQL Service
  service:
    name: mysql
    state: restarted
    enabled: true
```



MySQL 을 설치했으니 데이터베이스와 테이블을 구성해야 합니다.

기존에 HCloud Platform 에서 dump 한 SQL 파일을 복사해 주었습니다.

```shell
$ mkdir -p roles/mysql/backup
$ cp master_2021-10-26.sql roles/mysql/backup/
```



다음에는 MySQL 에 사용할 socket 과 사용자명, 비밀번호 그리고 dump 파일이 위치한 곳과 가상 머신 내에 복사할 디렉토리들을 정의한 변수 파일을 작성해 줍니다.

```shell
$ mkdir -p roles/mysql/vars
$ vi roles/mysql/vars/main.yml
```

```yaml
mysql_socket: /var/run/mysqld/mysqld.sock
mysql_root_password: P@assWord!
mysql_hcc_user_name: hcc
mysql_hcc_user_password: hcCP@assWord!

mysql_hcc_backup_filename: "master_2021-10-26.sql"
mysql_hcc_dump_directory: "/root/hcc_mysql"
```



마지막으로 MySQL 을 구성하기 위한 파일을 작성해 줍니다.

```yaml
{% raw %}
- name: mysql change root password
  mysql_user:
    name: root
    login_user: root
    login_unix_socket: "{{ mysql_socket }}"
    login_password: ""
    password: "{{ mysql_root_password }}"
    state: present
  ignore_errors: yes

- name: Adding user 'hcc'
  mysql_user:
    name: "{{ mysql_hcc_user_name }}"
    login_user: root
    login_unix_socket: "{{ mysql_socket }}"
    login_password: "{{ mysql_root_password }}"
    password: "{{ mysql_hcc_user_password }}"
    priv: "{{ mysql_hcc_user_name }}.*:ALL,GRANT"
    host: "%"
    state: present
  notify:
  - Restart MySQL Service

- name: Creates mysql dump directory
  file:
    path: "{{ mysql_hcc_dump_directory }}"
    state: directory

- name: Copying mysql backup
  copy:
    src: "{{ item }}"
    dest: "{{ mysql_hcc_dump_directory }}"
    owner: root
    mode: 644
  with_fileglob:
    - backup/*

- name: Importing mysql backup
  mysql_db:
    name: all
    state: import
    target: "/{{ mysql_hcc_dump_directory }}/{{ mysql_hcc_backup_filename }}"
    login_user: "root"
    login_unix_socket: "{{ mysql_socket }}"
    login_password: "{{ mysql_root_password }}"
{% endraw %}
```

각 task 들을 순서대로 설명드리면 다음과 같습니다.

- MySQL 의 root 패스워드를 변경합니다
- MySQL 에 hcc 사용자를 추가하고 MySQL 서비스를 재시작합니다.
- Master Node 에 SQL 덤프 파일을 복사할 디렉터리를 생성합니다.
- Master Node 에 생성한 디렉토리에 SQL 덤프 파일을 복사합니다.
- SQL 덤프 파일로 부터 MySQL 데이터베이스를 구성합니다.



### Middleware 구성하기

먼저 Middleare을 구성하기 위해서 `roles/middleware` 폴더를 만들어 주고 `roles/middleware/tasks/main.yml` 에 작업들이 정의 되어 있는 파일을 import 해줍니다.

```shell
$ mkdir -p roles/middleware/tasks
$ vi roles/middleware/tasks/main.yml
```

```yaml
- import_tasks: middleware_setup.yml
```



이제 미들웨어의 각 서비스들을 재시작 해주는 Handler 들을 작성해 줍니다.

```shell
$ mkdir -p roles/middleware/handlers
$ vi roles/middleware/handlers/main.yml
```

```yaml
- name: Reload daemon
  command: "systemctl daemon-reload"

- name: Restart flute service
  service:
    name: flute
    state: restarted
    enabled: true

- name: Restart harp service
  service:
    name: harp
    state: restarted
    enabled: true

- name: Restart piano service
  service:
    name: piano
    state: restarted
    enabled: true

- name: Restart piccolo service
  service:
    name: piccolo
    state: restarted
    enabled: true

- name: Restart violin service
  service:
    name: violin
    state: restarted
    enabled: true

- name: Restart violin-novnc service
  service:
    name: violin-novnc
    state: restarted
    enabled: true

- name: Restart violin-scheduler service
  service:
    name: violin-scheduler
    state: restarted
    enabled: true
```



Middleware 바이너리 파일들을 복사해 줍니다.

```shell
$ mkdir -p roles/middleware/binaries
$ cp clarinet flute harp hcc_status piano piccolo violin violin-novnc violin-scheduler roles/middleware/binaries
```



Middleware 에서 사용하는 설정파일들을 복사해 줍니다.

```shell
$ mkdir -p roles/middleware/configs/hcc
$ cp clarinet/ flute/ harp/ piano/ piccolo/ violin/ violin-novnc/ violin-scheduler/ roles/middleware/configs/hcc
```



Middleware 의 systemd 서비스 파일들을 복사해 줍니다.

```shell
$ mkdir -p roles/middleware/systemd-services
$ cp flute.service harp.service piano.service piccolo.service violin-novnc.service violin-scheduler.service violin.service
```



Middleware 에 사용할 설정파일들이 위치한 디렉토리, 바이너리 파일들이 위피한 디렉토리, systemd 서비스 파일들이 위치한 디렉토리들을 정의한 변수 파일을 작성해 줍니다.

```shell
$ mkdir -p roles/middleware/vars
$ vi roles/middleware/vars/main.yml
```

```yaml
middleware_config_directory: "/etc/hcc/"
middleware_binary_directory: "/usr/local/bin/"

systemd_service_files_location: "/etc/systemd/system"
```



마지막으로 Middleare의 설정파일, 바이너리파일, systemd 서비스 파일들을 복사하고 Middleware 서비스들을 실행해 주는 Task 들을 정의해 줍니다.

```shell
$ vi roles/mysql/tasks/middleware_setup.yml
```

```yaml
{% raw %}
- name: Creates middleware configs directory
  file:
    path: "{{ middleware_config_directory }}"
    state: directory

- name: Copying middleware configs
  copy:
    src: "configs/hcc"
    dest: "/etc/"
    owner: root

- name: Copying middleware binaries
  copy:
    src: "{{ item }}"
    dest: "{{ middleware_binary_directory }}"
    owner: root
    mode: 755
  with_fileglob:
    - binaries/*

- name: Copying middleware systemd services
  copy:
    src: "{{ item }}"
    dest: "{{ systemd_service_files_location }}"
    owner: root
    mode: 644
  with_fileglob:
    - systemd-services/*
  changed_when: true
  notify:
    - Reload daemon
    - Restart piccolo service
    - Restart flute service
    - Restart violin service
{% endraw %}
```



### 최종 Ansible 작성

이제 마무리 단계입니다. 최종적으로 MySQL 과 Middleware Role 들을 실행할 Playbook 을 작성해 줍니다.

```shell
$ vi hcc.yml
```

```
- name: Deploy hcc platform
  hosts: 172.168.10.100
  force_handlers: true

  roles:
    - mysql
    - middleware
```



지금까지 복사한 파일들과 Playbook 을 작성한 파일들의 구조를 보면 다음과 같습니다.

```shell
$ tree
.
├── Vagrantfile
├── ansible.cfg
├── hcc.yml
├── inventory.ini
└── roles
    ├── middleware
    │   ├── binaries
    │   │   ├── clarinet
    │   │   ├── flute
    │   │   ├── harp
    │   │   ├── hcc_status
    │   │   ├── piano
    │   │   ├── piccolo
    │   │   ├── violin
    │   │   ├── violin-novnc
    │   │   └── violin-scheduler
    │   ├── configs
    │   │   └── hcc
    │   │       ├── clarinet
    │   │       │   └── clarinet.conf
    │   │       ├── flute
    │   │       │   └── flute.conf
    │   │       ├── harp
    │   │       │   ├── dhcpd
    │   │       │   │   ├── 138c4dd4-f90f-4e4e-6e07-9673abeb57a6.conf
    │   │       │   │   └── config
    │   │       │   │       ├── 5203e92f-e57a-4d39-5136-95dd8e481e0b.conf
    │   │       │   │       ├── harp_dhcpd.conf
    │   │       │   │       └── test.conf
    │   │       │   ├── harp.conf
    │   │       │   └── harp_adaptiveip_network.conf
    │   │       ├── piano
    │   │       │   └── piano.conf
    │   │       ├── piccolo
    │   │       │   └── piccolo.conf
    │   │       ├── violin
    │   │       │   └── violin.conf
    │   │       ├── violin-novnc
    │   │       │   └── violin-novnc.conf
    │   │       └── violin-scheduler
    │   │           └── violin-scheduler.conf
    │   ├── handlers
    │   │   └── main.yml
    │   ├── systemd-services
    │   │   ├── flute.service
    │   │   ├── harp.service
    │   │   ├── piano.service
    │   │   ├── piccolo.service
    │   │   ├── violin-novnc.service
    │   │   ├── violin-scheduler.service
    │   │   └── violin.service
    │   ├── tasks
    │   │   ├── main.yml
    │   │   └── middleware_setup.yml
    │   └── vars
    │       └── main.yml
    └── mysql
        ├── backup
        │   └── master_2021-10-26.sql
        ├── handlers
        │   └── main.yml
        ├── tasks
        │   ├── main.yml
        │   ├── mysql_setup.yml
        │   └── ubuntu_mysql_package.yml
        └── vars
            └── main.yml

24 directories, 43 files
```



## Ansible Playbook 실행

이제 Ansible Playbook 을 작성하였으니 실행을 해볼 차례입니다.

`ansible-playbook [Playbook 파일명]` 명령을 실행하여 Playbook 을 실행합니다.



```shell
$ ansible-playbook hcc.yml 
[DEPRECATION WARNING]: [defaults]callback_whitelist option, normalizing names to new standard, use 
callbacks_enabled instead. This feature will be removed from ansible-core in version 2.15. Deprecation 
warnings can be disabled by setting deprecation_warnings=False in ansible.cfg.

PLAY [Deploy hcc platform] *****************************************************************************

TASK [Gathering Facts] *********************************************************************************
수요일 27 10월 2021  12:01:24 +0900 (0:00:00.023)       0:00:00.023 *************** 
수요일 27 10월 2021  12:01:24 +0900 (0:00:00.022)       0:00:00.022 *************** 
ok: [172.168.10.100]

TASK [mysql : Update and Install Package for MySQL] ****************************************************
수요일 27 10월 2021  12:01:25 +0900 (0:00:01.271)       0:00:01.294 *************** 
수요일 27 10월 2021  12:01:25 +0900 (0:00:01.271)       0:00:01.294 *************** 
changed: [172.168.10.100]

TASK [mysql : mysql change root password] **************************************************************
수요일 27 10월 2021  12:02:17 +0900 (0:00:51.557)       0:00:52.852 *************** 
수요일 27 10월 2021  12:02:17 +0900 (0:00:51.557)       0:00:52.851 *************** 
changed: [172.168.10.100]

TASK [mysql : Adding user 'hcc'] ***********************************************************************
수요일 27 10월 2021  12:02:17 +0900 (0:00:00.307)       0:00:53.159 *************** 
수요일 27 10월 2021  12:02:17 +0900 (0:00:00.307)       0:00:53.159 *************** 
changed: [172.168.10.100]

TASK [mysql : Creates mysql dump directory] ************************************************************
수요일 27 10월 2021  12:02:17 +0900 (0:00:00.225)       0:00:53.385 *************** 
수요일 27 10월 2021  12:02:17 +0900 (0:00:00.225)       0:00:53.385 *************** 
changed: [172.168.10.100]

TASK [mysql : Copying mysql backup] ********************************************************************
수요일 27 10월 2021  12:02:17 +0900 (0:00:00.270)       0:00:53.656 *************** 
수요일 27 10월 2021  12:02:17 +0900 (0:00:00.270)       0:00:53.656 *************** 
changed: [172.168.10.100] => (item=/home/jollaman999/1111/roles/mysql/backup/master_2021-10-26.sql)

TASK [mysql : Importing mysql backup] ******************************************************************
수요일 27 10월 2021  12:02:18 +0900 (0:00:00.648)       0:00:54.305 *************** 
수요일 27 10월 2021  12:02:18 +0900 (0:00:00.648)       0:00:54.304 *************** 
changed: [172.168.10.100]

TASK [middleware : Creates middleware configs directory] ***********************************************
수요일 27 10월 2021  12:02:19 +0900 (0:00:01.000)       0:00:55.305 *************** 
수요일 27 10월 2021  12:02:19 +0900 (0:00:01.000)       0:00:55.304 *************** 
changed: [172.168.10.100]

TASK [middleware : Copying middleware configs] *********************************************************
수요일 27 10월 2021  12:02:19 +0900 (0:00:00.189)       0:00:55.494 *************** 
수요일 27 10월 2021  12:02:19 +0900 (0:00:00.189)       0:00:55.494 *************** 
changed: [172.168.10.100]

TASK [middleware : Copying middleware binaries] ********************************************************
수요일 27 10월 2021  12:02:25 +0900 (0:00:05.480)       0:01:00.975 *************** 
수요일 27 10월 2021  12:02:25 +0900 (0:00:05.480)       0:01:00.975 *************** 
changed: [172.168.10.100] => (item=/home/jollaman999/1111/roles/middleware/binaries/hcc_status)
changed: [172.168.10.100] => (item=/home/jollaman999/1111/roles/middleware/binaries/piano)
changed: [172.168.10.100] => (item=/home/jollaman999/1111/roles/middleware/binaries/clarinet)
changed: [172.168.10.100] => (item=/home/jollaman999/1111/roles/middleware/binaries/flute)
changed: [172.168.10.100] => (item=/home/jollaman999/1111/roles/middleware/binaries/violin)
changed: [172.168.10.100] => (item=/home/jollaman999/1111/roles/middleware/binaries/harp)
changed: [172.168.10.100] => (item=/home/jollaman999/1111/roles/middleware/binaries/violin-scheduler)
changed: [172.168.10.100] => (item=/home/jollaman999/1111/roles/middleware/binaries/violin-novnc)
changed: [172.168.10.100] => (item=/home/jollaman999/1111/roles/middleware/binaries/piccolo)

TASK [middleware : Copying middleware systemd services] ************************************************
수요일 27 10월 2021  12:02:33 +0900 (0:00:07.851)       0:01:08.827 *************** 
수요일 27 10월 2021  12:02:33 +0900 (0:00:07.851)       0:01:08.826 *************** 
changed: [172.168.10.100] => (item=/home/jollaman999/1111/roles/middleware/systemd-services/violin.service)
changed: [172.168.10.100] => (item=/home/jollaman999/1111/roles/middleware/systemd-services/violin-scheduler.service)
changed: [172.168.10.100] => (item=/home/jollaman999/1111/roles/middleware/systemd-services/harp.service)
changed: [172.168.10.100] => (item=/home/jollaman999/1111/roles/middleware/systemd-services/piccolo.service)
changed: [172.168.10.100] => (item=/home/jollaman999/1111/roles/middleware/systemd-services/violin-novnc.service)
changed: [172.168.10.100] => (item=/home/jollaman999/1111/roles/middleware/systemd-services/piano.service)
changed: [172.168.10.100] => (item=/home/jollaman999/1111/roles/middleware/systemd-services/flute.service)

RUNNING HANDLER [mysql : Restart MySQL Service] ********************************************************
수요일 27 10월 2021  12:02:36 +0900 (0:00:03.349)       0:01:12.176 *************** 
수요일 27 10월 2021  12:02:36 +0900 (0:00:03.349)       0:01:12.176 *************** 
changed: [172.168.10.100]

RUNNING HANDLER [middleware : Reload daemon] ***********************************************************
수요일 27 10월 2021  12:02:39 +0900 (0:00:03.079)       0:01:15.255 *************** 
수요일 27 10월 2021  12:02:39 +0900 (0:00:03.079)       0:01:15.255 *************** 
changed: [172.168.10.100]

RUNNING HANDLER [middleware : Restart flute service] ***************************************************
수요일 27 10월 2021  12:02:40 +0900 (0:00:00.505)       0:01:15.760 *************** 
수요일 27 10월 2021  12:02:40 +0900 (0:00:00.505)       0:01:15.760 *************** 
changed: [172.168.10.100]

RUNNING HANDLER [middleware : Restart piccolo service] *************************************************
수요일 27 10월 2021  12:02:40 +0900 (0:00:00.500)       0:01:16.261 *************** 
수요일 27 10월 2021  12:02:40 +0900 (0:00:00.500)       0:01:16.261 *************** 
changed: [172.168.10.100]

RUNNING HANDLER [middleware : Restart violin service] **************************************************
수요일 27 10월 2021  12:02:41 +0900 (0:00:00.486)       0:01:16.747 *************** 
수요일 27 10월 2021  12:02:41 +0900 (0:00:00.486)       0:01:16.747 *************** 
changed: [172.168.10.100]

PLAY RECAP *********************************************************************************************
172.168.10.100             : ok=16   changed=15   unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   

수요일 27 10월 2021  12:02:41 +0900 (0:00:00.479)       0:01:17.227 *************** 
=============================================================================== 
mysql : Update and Install Package for MySQL --------------------------------------------------- 51.56s
middleware : Copying middleware binaries -------------------------------------------------------- 7.85s
middleware : Copying middleware configs --------------------------------------------------------- 5.48s
middleware : Copying middleware systemd services ------------------------------------------------ 3.35s
mysql : Restart MySQL Service ------------------------------------------------------------------- 3.08s
Gathering Facts --------------------------------------------------------------------------------- 1.27s
mysql : Importing mysql backup ------------------------------------------------------------------ 1.00s
mysql : Copying mysql backup -------------------------------------------------------------------- 0.65s
middleware : Reload daemon ---------------------------------------------------------------------- 0.51s
middleware : Restart flute service -------------------------------------------------------------- 0.50s
middleware : Restart piccolo service ------------------------------------------------------------ 0.49s
middleware : Restart violin service ------------------------------------------------------------- 0.48s
mysql : mysql change root password -------------------------------------------------------------- 0.31s
mysql : Creates mysql dump directory ------------------------------------------------------------ 0.27s
mysql : Adding user 'hcc' ----------------------------------------------------------------------- 0.23s
middleware : Creates middleware configs directory ----------------------------------------------- 0.19s
수요일 27 10월 2021  12:02:41 +0900 (0:00:00.480)       0:01:17.227 *************** 
=============================================================================== 
mysql ------------------------------------------------------------------ 57.09s
middleware ------------------------------------------------------------- 18.84s
gather_facts ------------------------------------------------------------ 1.27s
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~ 
total ------------------------------------------------------------------ 77.21s
```

모든 단계가 정상적으로 진행된 모습입니다.



## MySQL 및 API Gateway 구동 확인

Master 노드에 접속하여 MySQL 이 정상적으로 구성되었는지 확인해 봅니다.

```shell
$ vagrant ssh hcc-master
Welcome to Ubuntu 20.04.3 LTS (GNU/Linux 5.4.0-88-generic x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/advantage

  System information as of Wed Oct 27 05:41:55 UTC 2021

  System load:  0.02              Processes:               120
  Usage of /:   5.4% of 38.71GB   Users logged in:         0
  Memory usage: 31%               IPv4 address for enp0s3: 10.0.2.15
  Swap usage:   0%                IPv4 address for enp0s8: 172.168.10.100


28 updates can be applied immediately.
19 of these updates are standard security updates.
To see these additional updates run: apt list --upgradable


Last login: Wed Oct 27 03:02:40 2021 from 172.168.10.1
vagrant@master:~$ mysql -uroot -p
Enter password: 
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 8
Server version: 8.0.27-0ubuntu0.20.04.1 (Ubuntu)

Copyright (c) 2000, 2021, Oracle and/or its affiliates.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

mysql> show databases;
+--------------------+
| Database           |
+--------------------+
| cello              |
| flute              |
| harp               |
| information_schema |
| mysql              |
| performance_schema |
| piano              |
| piccolo            |
| sys                |
| viola              |
| violin             |
| violin_novnc       |
+--------------------+
12 rows in set (0.01 sec)
```



다음은 API Gateway 가 정상적으로 구동되는지 확인해봅니다.

![image-20211027144638190](/assets/images/docs/Ansible 과 Vagrant로 HCloud-Platform 배포하는 방법 - 1편/image-20211027144638190.png)



모두 정상적으로 구동되는 것을 확인할 수 있습니다.



지금까지 Vagrant 와 Ansible 로 HCloud-Platform 실행환경을 구성하는 방법에 대해서 알아보았습니다.

매번 실행환경을 구성하기 위해 설정과 설치 작업을 반복하게 되는데, 이러한 방법을 잘 활용하여 실제 개발을 하고 테스트를 하는데에 있어서 시간과 노력을 줄일 수 있도록 도움이 되셨으면 합니다.



읽어 주셔서 감사합니다.

