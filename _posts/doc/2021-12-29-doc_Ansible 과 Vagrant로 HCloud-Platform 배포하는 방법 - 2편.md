---
title: Ansible 과 Vagrant로 HCloud-Platform 배포하는 방법 - 2편
subtitle: 
layout: post
icon: fa-book
order: 3

---



 저희 HCloud-Platform 은 Middleware 들이 운영되고 있는 Master 노드와 단일 가상 서버에서 사용할 볼륨들을 프로비저닝 해주는 Storage 노드를 통해 단일 가상 서버를 구성하여 배포합니다.



 이전 편에서는 Vagrant 로 각 노드들을 구성하고 Master 노드에서 MySQL 과 필수적인 몇몇 미들웨어만 운영을 하여 API Gateway 까지 동작하는 부분을 보여드렸습니다.

 이번 편에서는 InfluxDB 와 RabbitMQ 그리고 Network 담당 모듈인 Harp 미들웨어 모듈에서 필요로 하는 DHCP 서버와 VnStat 을 설치하여 모든 미들웨어 모듈들이 정상적으로 동작하는 것을 보여 드리도록 하겠습니다.



 이전 편에 이어서 설정파일을 작성 하오니 이전 편을 참고하시려면 아래 링크를 클릭해 주시기 바랍니다.

[Ansible 과 Vagrant로 HCloud-Platform 배포하는 방법 - 1편](https://hcloud-classic.github.io/2021/10/27/doc_Ansible-%EA%B3%BC-Vagrant%EB%A1%9C-HCloud-Platform-%EB%B0%B0%ED%8F%AC%ED%95%98%EB%8A%94-%EB%B0%A9%EB%B2%95-1%ED%8E%B8.html)



## Ansible Playbook 작성

### InfluxDB 구성하기

먼저 InfluxDB 를 구성하기 위해서 `roles/influxdb` 폴더를 만들어 주고 `roles/influxdb/tasks/main.yml` 에 작업들이 정의 되어 있는 파일을 import 해줍니다.

```shell
$ mkdir -p roles/influxdb/tasks
$ vi roles/influxdb/tasks/main.yml
```

```yaml
- import_tasks: install_influxdb.yml
- import_tasks: influxdb_setup.yml
```



다음으로, InfluxDB 에서 사용할 설정 파일을 복사해 줍니다.

```shell
$ mkdir -p roles/influxdb/config
$ cp influxdb.conf rroles/influxdb/config/
```



InfluxDB deb 설치파일의 URL 주소와 설정파일 경로, 생성할 데이터베이스명을 정의한 변수 파일을 작성해 줍니다.

```shell
$ mkdir -p roles/influxdb/vars
$ vi roles/influxdb/vars/main.yml
```

```yaml
influxdb_deb_download_url: "https://dl.influxdata.com/influxdb/releases/influxdb_1.8.10_amd64.deb"
influxdb_configuration_location: "/etc/influxdb/influxdb.conf"

influxdb_database_name: "telegraf"
```



Ubuntu 가 설치 되어 있는 Master 노드에 InfluxDB 를 설치 하기 위한 작업들을 정의한 파일을 작성해 줍니다.

```shell
$ vi roles/influxdb/tasks/install_influxdb.yml
```

```yaml
{% raw %}
- name: Install influxDB deb package
  apt:
    deb: "{{influxdb_deb_download_url}}"

- name: Copying influxDB configuration
  copy:
    src: "{{ item }}"
    dest: "{{ influxdb_configuration_location }}"
    owner: root
    mode: "0644"
  with_fileglob:
    - config/*

- name: Restart InfluxDB Service
  service:
    name: influxdb
    state: restarted
    enabled: true
{% endraw %}
```

각 task 들을 순서대로 설명드리면 다음과 같습니다.

- InfluxDB deb 패키지를 다운로드 하여 설치합니다.
- InfluxDB 설정 파일을 복사합니다.
- InfluxDB 서비스를 재시작합니다.



마지막으로, InfluxDB 에서 사용할 데이터베이스를 생성하는 작업을 정의한 파일을 작성해 줍니다.

```shell
$ vi roles/influxdb/tasks/influxdb_setup.yml
```

```yaml
{% raw %}
- name: Create database
  command: "influx -execute 'CREATE DATABASE {{ influxdb_database_name }}'"
{% endraw %}
```



### RabbitMQ 구성하기

먼저 RabbitMQ 를 구성하기 위해서 `roles/rabbitmq` 폴더를 만들어 주고 `roles/rabbitmq/tasks/main.yml` 에 작업들이 정의 되어 있는 파일을 import 해줍니다.

```shell
$ mkdir -p roles/rabbitmq/tasks
$ vi roles/rabbitmq/tasks/main.yml
```

```yaml
- import_tasks: install_rabbitmq.yml
```



RabbitMQ 에 사용할 사용자명과 비밀번호를 정의한 변수 파일을 작성해 줍니다.

```shell
$ mkdir -p roles/rabbitmq/vars
$ vi roles/rabbitmq/vars/main.yml
```

```yaml
rabbitmq_user_name: admin
rabbitmq_user_password: "P@assWord\\!"
```



RabbitMQ 를 설치하기 위한 스크립트 파일을 작성해 줍니다.

```shell
$ mkdir -p roles/rabbitmq/script
$ vi roles/rabbitmq/script/install_rabbitmq_server.sh
```

```bash
#!/usr/bin/sh

sudo apt-get install curl gnupg apt-transport-https -y

## Team RabbitMQ's main signing key
curl -1sLf "https://keys.openpgp.org/vks/v1/by-fingerprint/0A9AF2115F4687BD29803A206B73A36E6026DFCA" | sudo gpg --dearmor | sudo tee /usr/share/keyrings/com.rabbitmq.team.gpg > /dev/null
## Cloudsmith: modern Erlang repository
curl -1sLf https://dl.cloudsmith.io/public/rabbitmq/rabbitmq-erlang/gpg.E495BB49CC4BBE5B.key | sudo gpg --dearmor | sudo tee /usr/share/keyrings/io.cloudsmith.rabbitmq.E495BB49CC4BBE5B.gpg > /dev/null
## Cloudsmith: RabbitMQ repository
curl -1sLf https://dl.cloudsmith.io/public/rabbitmq/rabbitmq-server/gpg.9F4587F226208342.key | sudo gpg --dearmor | sudo tee /usr/share/keyrings/io.cloudsmith.rabbitmq.9F4587F226208342.gpg > /dev/null

## Add apt repositories maintained by Team RabbitMQ
sudo tee /etc/apt/sources.list.d/rabbitmq.list <<EOF
## Provides modern Erlang/OTP releases
##
deb [signed-by=/usr/share/keyrings/io.cloudsmith.rabbitmq.E495BB49CC4BBE5B.gpg] https://dl.cloudsmith.io/public/rabbitmq/rabbitmq-erlang/deb/ubuntu bionic main
deb-src [signed-by=/usr/share/keyrings/io.cloudsmith.rabbitmq.E495BB49CC4BBE5B.gpg] https://dl.cloudsmith.io/public/rabbitmq/rabbitmq-erlang/deb/ubuntu bionic main

## Provides RabbitMQ
##
deb [signed-by=/usr/share/keyrings/io.cloudsmith.rabbitmq.9F4587F226208342.gpg] https://dl.cloudsmith.io/public/rabbitmq/rabbitmq-server/deb/ubuntu bionic main
deb-src [signed-by=/usr/share/keyrings/io.cloudsmith.rabbitmq.9F4587F226208342.gpg] https://dl.cloudsmith.io/public/rabbitmq/rabbitmq-server/deb/ubuntu bionic main
EOF

## Update package indices
sudo apt-get update -y

## Install Erlang packages
sudo apt-get install -y erlang-base \
                        erlang-asn1 erlang-crypto erlang-eldap erlang-ftp erlang-inets \
                        erlang-mnesia erlang-os-mon erlang-parsetools erlang-public-key \
                        erlang-runtime-tools erlang-snmp erlang-ssl \
                        erlang-syntax-tools erlang-tftp erlang-tools erlang-xmerl

## Install rabbitmq-server and its dependencies
sudo apt-get install rabbitmq-server -y --fix-missing

```



Ubuntu 가 설치 되어 있는 Master 노드에 RabbitMQ 를 설치 하기 위한 작업들을 정의한 파일을 작성해 줍니다.

```shell
$ vi roles/rabbitmq/tasks/install_rabbitmq.yml
```

```yaml
{% raw %}
- name: Install RabbitMQ Server
  script: script/install_rabbitmq_server.sh
  notify:
  - Restart RabbitMQ Service

- name: Enable RabbitMQ Management Plugin
  command: rabbitmq-plugins enable rabbitmq_management

- name: RabbitMQ - Add admin user
  command: "rabbitmqctl add_user {{rabbitmq_user_name}} {{rabbitmq_user_password}}"

- name: RabbitMQ - Set administrator to admin
  command: rabbitmqctl set_user_tags admin administrator

- name: RabbitMQ - Set permissions
  command: rabbitmqctl set_permissions --vhost / admin '.*' '.*' '.*'

- name: RabbitMQ - Set topic permissions
  command: rabbitmqctl set_topic_permissions --vhost / admin '(AMQP default)' '.*' '.*'
{% endraw %}
```

각 task 들을 순서대로 설명드리면 다음과 같습니다.

- RabbitMQ 를 설치하는 스크립트를 실행하고, RabbitMQ 서비스를 재시작합니다.
- Web 으로 RabbitMQ 를 관리 할 수 있는 rabbitmq_management 플러그인을 설치합니다.
- RabbitMQ 서버에 admin 사용자를 추가합니다.
- admin 사용자에 관리자 권한을 부여합니다.
- RabbitMQ 의 모든 기능들을 사용할 수 있도록 권한을 부여 합니다.
- Topic 관련 메세지들을 주고 받을 수 있도록 권한을 부여합니다.



### Harp Network 모듈 구성하기

HCloud-Platform 에서 사용되는 미들웨어 모듈중 Harp 모듈은 네트워크 구성을 담당합니다.

isc-dhcp-server 를 통해 DHCP 서버를 구성하고, VnStat 을 통해 외부 트래픽량을 측정합니다. 



먼저 DHCP 서버 구성을 위해 DHCP 설정 파일을 작성해 줍니다.

Harp 모듈에서는 DHCP 설정 파일들을 단일 가상 서버 별로 생성하여 저장하고 각 파일들을 include 하는 설정 파일을 사용하는 구조로 되어 있습니다.

이 파일을 isc-dhcp-server 에서 사용하도록 하기 위해서 아래와 같이 작성하였습니다.

```shell
$ vi roles/middleware/configs/dhcpd/dhcpd.conf
```

```
include "/etc/hcc/harp/dhcpd/config/harp_dhcpd.conf";
```



앞에 1편에서 작성하였던 Middleware role 에 변수들을 정의한 파일에 VnStat 설치를 위한 deb  패키지 URL 주소와  isc-dhcp-server 설정 파일 경로를 지정한 변수를 추가해 줍니다.

`harp_vnstat_deb_download_url` 와 `harp_dhcpd_configuration_location` 변수를 추가 하였습니다.

```shell
$ vi roles/middleware/vars/main.yml
```

```
middleware_config_directory: "/etc/hcc/"
middleware_binary_directory: "/usr/local/bin/"

systemd_service_files_location: "/etc/systemd/system"

harp_vnstat_deb_download_url: "http://archive.ubuntu.com/ubuntu/pool/universe/v/vnstat/vnstat_1.18-1_amd64.deb"

harp_dhcpd_configuration_location: "/etc/dhcpd/dhcpd.conf"
```



VnStat 과 isc-dhcp-server 를 설치하기 위해서 Harp 를 위한 task 파일을 다음과 같이 추가로 작성해 주었습니다.

```shell
$ vi roles/middleware/tasks/harp_setup.yml
```

```yaml
{% raw %}
- name: Harp - Install VnStat v1.18
  apt:
    deb: "{{harp_vnstat_deb_download_url}}"

- name: Harp - Install isc-dhcp-server
  apt:
    name: isc-dhcp-server
    update_cache: true
    state: present

- name: Harp - Copy dhcpd configuration file
  copy:
    src: "{{ item }}"
    dest: "{{ harp_dhcpd_configuration_location }}"
    owner: root
    mode: "0644"
  with_fileglob:
    - configs/dhcpd*
{% endraw %}
```



이제 추가로 작성한 task 파일을 1편에서 작성했던 `roles/middleware/tasks/middleware_setup.yml` 파일에서 import 해주도 서비스를 시작하는 부분에 harp 서비스를 재시작 하는 핸들러를 호출하도록 해줍니다.

```yaml
...생략
{% raw %}
- import_tasks: harp_setup.yml

- name: Copying middleware systemd services
  copy:
    src: "{{ item }}"
    dest: "{{ systemd_service_files_location }}"
    owner: root
    mode: "0644"
  with_fileglob:
    - systemd-services/*
  changed_when: true
  notify:
    - Reload daemon
    - Restart piccolo service
    - Restart flute service
    - Restart violin service
    - Restart harp service
{% endraw %}
```



### 모든 미들웨어 실행 가능하도록 수정하기

이제 InfluxDB 와 RabbitMQ 가 구성되었고 Harp 모듈도 정상적으로 실행할 수 있게 되어, 모든 모듈을 실행하는 것이 가능해졌습니다.

1편에서 작성했던 `roles/middleware/tasks/middleware_setup.yml` 파일의 마지막에 미들웨어 서비스들을 재시작 하는 부분에 모든 미들웨어 서비스들을 재시작 할 수 있도록 아래처럼 수정해 주었습니다.

```shell
$ vi roles/middleware/tasks/middleware_setup.yml
```

```yaml
... 생략
{% raw %}
- name: Copying middleware systemd services
  copy:
    src: "{{ item }}"
    dest: "{{ systemd_service_files_location }}"
    owner: root
    mode: "0644"
  with_fileglob:
    - systemd-services/*
  changed_when: true
  notify:
    - Reload daemon
    - Restart piccolo service
    - Restart flute service
    - Restart violin service
    - Restart harp service
    - Restart piano service
    - Restart violin-novnc service
    - Restart violin-scheduler service
{% endraw %}
```



다음으로 각 role 들을 실행하는 roles 가 정의 되어있는 메인 Ansible 파일인 `hcc.yml` 을 열어 다음과 같이 수정해 주었습니다.

그리고 middleware 롤은 MySQL, InfluxDB, RabbitMQ 가 모두 준비되고 나서 수행 될 수 있도록 제일 마지막 단계에 수행될 수 있도록 하였습니다.

```shell
$ vi hcc.yml
```

```yaml
- name: Deploy hcc platform
  hosts: 172.168.10.100
  force_handlers: true

  roles:
    - mysql
    - influxdb
    - rabbitmq
    - middleware 
```



## Ansible Playbook 실행

이제 모든 준비가 완료 되었습니다. Ansible Playbook 을 실행을 해볼 차례입니다.



먼저 1편에서 만든 가상 머신들을 지우고 새로 만들기 위해서 `vagrant destroy` 명령어를 실행합니다.

그리고 각각의 질문에 y 를 입력하여 모든 가상 머신들을 삭제합니다.

```shell
$ vagrant destroy
    hcc-storage: Are you sure you want to destroy the 'hcc-storage' VM? [y/N] y
==> hcc-storage: Discarding saved state of VM...
==> hcc-storage: Destroying VM and associated drives...
    hcc-master: Are you sure you want to destroy the 'hcc-master' VM? [y/N] y
==> hcc-master: Destroying VM and associated drives...
```



다시 가상 머신들을 생성하기 위해서 `vagrant up` 명령을 실행합니다.

```shell
$ vagrant up
Bringing machine 'hcc-master' up with 'virtualbox' provider...
Bringing machine 'hcc-storage' up with 'virtualbox' provider...
==> hcc-master: Importing base box 'ubuntu/focal64'...
==> hcc-master: Matching MAC address for NAT networking...
==> hcc-master: Checking if box 'ubuntu/focal64' version '20211001.0.0' is up to date...
==> hcc-master: A newer version of the box 'ubuntu/focal64' for provider 'virtualbox' is
==> hcc-master: available! You currently have version '20211001.0.0'. The latest is version
==> hcc-master: '20211026.0.0'. Run `vagrant box update` to update.
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
    hcc-master: /vagrant => /home/jollaman999/git/ansible_hcc
==> hcc-master: Running provisioner: shell...
    hcc-master: Running: inline script
==> hcc-storage: Importing base box 'freebsd/FreeBSD-12.3-STABLE'...
==> hcc-storage: Generating MAC address for NAT networking...
==> hcc-storage: Checking if box 'freebsd/FreeBSD-12.3-STABLE' version '2021.12.23' is up to date...
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
    hcc-storage: Warning: Remote connection disconnect. Retrying...
    hcc-storage: Warning: Connection reset. Retrying...
    hcc-storage: Warning: Connection reset. Retrying...
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
    hcc-storage: Warning: Connection reset. Retrying...
    hcc-storage: Warning: Connection reset. Retrying...
    hcc-storage: Warning: Remote connection disconnect. Retrying...
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
    hcc-storage: /vagrant => /home/jollaman999/git/ansible_hcc2
```



ansible 이 새로 만든 가상 머신들에 SSH 접속을 다시 할 수 있도록 기존의 SSH 정보를 삭제하고, Vagrant 에 로컬호스트의 SSH 공개키를 복사해 줍니다.

먼저 기존의 SSH 연결 정보를 삭제합니다.

```shell
$ ssh-keygen -f ~/.ssh/known_hosts -R 172.168.10.100
$ ssh-keygen -f ~/.ssh/known_hosts -R 172.168.10.101
```



다음으로 각 가상 머신에 로컬호스트의 SSH 공개키를 복사합니다.

```shell
$ ssh-copy-id vagrant@172.168.10.100
The authenticity of host '172.168.10.100 (172.168.10.100)' can't be established.
ECDSA key fingerprint is SHA256:tNvpw+mvZ8gu62XmOGSMy6dF8oWsQxYrYzU27pZsgGY.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
/usr/bin/ssh-copy-id: INFO: attempting to log in with the new key(s), to filter out any that are already installed
/usr/bin/ssh-copy-id: INFO: 1 key(s) remain to be installed -- if you are prompted now it is to install the new keys
Password:

Number of key(s) added: 1

Now try logging into the machine, with:   "ssh 'vagrant@172.168.10.100'"
and check to make sure that only the key(s) you wanted were added.

$ ssh-copy-id vagrant@172.168.10.101
The authenticity of host '172.168.10.101 (172.168.10.101)' can't be established.
ECDSA key fingerprint is SHA256:nGG0fbV1x7n8gDNrLpKGJlbx9mm2lL6FNNYq12z7We4.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
/usr/bin/ssh-copy-id: INFO: attempting to log in with the new key(s), to filter out any that are already installed
/usr/bin/ssh-copy-id: INFO: 1 key(s) remain to be installed -- if you are prompted now it is to install the new keys
Password for vagrant@storage:

Number of key(s) added: 1

Now try logging into the machine, with:   "ssh 'vagrant@172.168.10.101'"
and check to make sure that only the key(s) you wanted were added.


```



이제 가상머신과 SSH 연결이 준비가 되었으니 `ansible-playbook hcc.yml` 명령을 실행하여 Playbook 을 실행합니다.

```shell
$ ansible-playbook hcc.yml
[DEPRECATION WARNING]: [defaults]callback_whitelist option, normalizing names to new standard, use callbacks_enabled
instead. This feature will be removed from ansible-core in version 2.15. Deprecation warnings can be disabled by
setting deprecation_warnings=False in ansible.cfg.

PLAY [Deploy hcc platform] *********************************************************************************************

TASK [Gathering Facts] *************************************************************************************************
목요일 30 12월 2021  17:38:17 +0900 (0:00:00.011)       0:00:00.011 ***************
목요일 30 12월 2021  17:38:17 +0900 (0:00:00.011)       0:00:00.011 ***************
ok: [172.168.10.100]

TASK [mysql : Update and Install Package for MySQL] ********************************************************************
목요일 30 12월 2021  17:38:18 +0900 (0:00:01.415)       0:00:01.427 ***************
목요일 30 12월 2021  17:38:18 +0900 (0:00:01.415)       0:00:01.426 ***************
changed: [172.168.10.100]

TASK [mysql : mysql change root password] ******************************************************************************
목요일 30 12월 2021  17:38:50 +0900 (0:00:31.712)       0:00:33.139 ***************
목요일 30 12월 2021  17:38:50 +0900 (0:00:31.712)       0:00:33.139 ***************
changed: [172.168.10.100]

TASK [mysql : Adding user 'hcc'] ***************************************************************************************
목요일 30 12월 2021  17:38:50 +0900 (0:00:00.313)       0:00:33.453 ***************
목요일 30 12월 2021  17:38:50 +0900 (0:00:00.313)       0:00:33.452 ***************
changed: [172.168.10.100]

TASK [mysql : Creates mysql dump directory] ****************************************************************************
목요일 30 12월 2021  17:38:50 +0900 (0:00:00.238)       0:00:33.691 ***************
목요일 30 12월 2021  17:38:50 +0900 (0:00:00.238)       0:00:33.691 ***************
changed: [172.168.10.100]

TASK [mysql : Copying mysql backup] ************************************************************************************
목요일 30 12월 2021  17:38:51 +0900 (0:00:00.269)       0:00:33.961 ***************
목요일 30 12월 2021  17:38:51 +0900 (0:00:00.269)       0:00:33.961 ***************
changed: [172.168.10.100] => (item=/home/jollaman999/git/ansible_hcc/roles/mysql/backup/master_2021-10-26.sql)

TASK [mysql : Importing mysql backup] **********************************************************************************
목요일 30 12월 2021  17:38:51 +0900 (0:00:00.650)       0:00:34.612 ***************
목요일 30 12월 2021  17:38:51 +0900 (0:00:00.650)       0:00:34.611 ***************
changed: [172.168.10.100]

TASK [influxdb : Install influxDB deb package] *************************************************************************
목요일 30 12월 2021  17:38:52 +0900 (0:00:00.974)       0:00:35.586 ***************
목요일 30 12월 2021  17:38:52 +0900 (0:00:00.974)       0:00:35.586 ***************
changed: [172.168.10.100]

TASK [influxdb : Copying influxDB configuration] ***********************************************************************
목요일 30 12월 2021  17:39:00 +0900 (0:00:07.301)       0:00:42.887 ***************
목요일 30 12월 2021  17:39:00 +0900 (0:00:07.301)       0:00:42.887 ***************
changed: [172.168.10.100] => (item=/home/jollaman999/git/ansible_hcc/roles/influxdb/config/influxdb.conf)

TASK [influxdb : Restart InfluxDB Service] *****************************************************************************
목요일 30 12월 2021  17:39:00 +0900 (0:00:00.384)       0:00:43.272 ***************
목요일 30 12월 2021  17:39:00 +0900 (0:00:00.384)       0:00:43.272 ***************
changed: [172.168.10.100]

TASK [influxdb : Create database] **************************************************************************************
목요일 30 12월 2021  17:39:06 +0900 (0:00:05.823)       0:00:49.096 ***************
목요일 30 12월 2021  17:39:06 +0900 (0:00:05.823)       0:00:49.096 ***************
changed: [172.168.10.100]

TASK [rabbitmq : Install RabbitMQ Server] ******************************************************************************
목요일 30 12월 2021  17:39:06 +0900 (0:00:00.287)       0:00:49.384 ***************
목요일 30 12월 2021  17:39:06 +0900 (0:00:00.287)       0:00:49.383 ***************
changed: [172.168.10.100]

TASK [rabbitmq : Enable RabbitMQ Management Plugin] ********************************************************************
목요일 30 12월 2021  17:40:09 +0900 (0:01:02.901)       0:01:52.285 ***************
목요일 30 12월 2021  17:40:09 +0900 (0:01:02.901)       0:01:52.285 ***************
changed: [172.168.10.100]

TASK [rabbitmq : RabbitMQ - Add admin user] ****************************************************************************
목요일 30 12월 2021  17:40:12 +0900 (0:00:02.634)       0:01:54.920 ***************
목요일 30 12월 2021  17:40:12 +0900 (0:00:02.634)       0:01:54.920 ***************
changed: [172.168.10.100]

TASK [rabbitmq : RabbitMQ - Set administrator to admin] ****************************************************************
목요일 30 12월 2021  17:40:12 +0900 (0:00:00.505)       0:01:55.426 ***************
목요일 30 12월 2021  17:40:12 +0900 (0:00:00.505)       0:01:55.426 ***************
changed: [172.168.10.100]

TASK [rabbitmq : RabbitMQ - Set permissions] ***************************************************************************
목요일 30 12월 2021  17:40:13 +0900 (0:00:00.504)       0:01:55.930 ***************
목요일 30 12월 2021  17:40:13 +0900 (0:00:00.504)       0:01:55.930 ***************
changed: [172.168.10.100]

TASK [rabbitmq : RabbitMQ - Set topic permissions] *********************************************************************
목요일 30 12월 2021  17:40:13 +0900 (0:00:00.568)       0:01:56.499 ***************
목요일 30 12월 2021  17:40:13 +0900 (0:00:00.568)       0:01:56.498 ***************
changed: [172.168.10.100]

TASK [middleware : Creates middleware configs directory] ***************************************************************
목요일 30 12월 2021  17:40:14 +0900 (0:00:00.513)       0:01:57.012 ***************
목요일 30 12월 2021  17:40:14 +0900 (0:00:00.513)       0:01:57.011 ***************
changed: [172.168.10.100]

TASK [middleware : Copying middleware configs] *************************************************************************
목요일 30 12월 2021  17:40:14 +0900 (0:00:00.210)       0:01:57.223 ***************
목요일 30 12월 2021  17:40:14 +0900 (0:00:00.210)       0:01:57.222 ***************
changed: [172.168.10.100]

TASK [middleware : Copying middleware binaries] ************************************************************************
목요일 30 12월 2021  17:40:19 +0900 (0:00:05.156)       0:02:02.379 ***************
목요일 30 12월 2021  17:40:19 +0900 (0:00:05.156)       0:02:02.379 ***************
changed: [172.168.10.100] => (item=/home/jollaman999/git/ansible_hcc/roles/middleware/binaries/hcc_status)
changed: [172.168.10.100] => (item=/home/jollaman999/git/ansible_hcc/roles/middleware/binaries/piano)
changed: [172.168.10.100] => (item=/home/jollaman999/git/ansible_hcc/roles/middleware/binaries/clarinet)
changed: [172.168.10.100] => (item=/home/jollaman999/git/ansible_hcc/roles/middleware/binaries/flute)
changed: [172.168.10.100] => (item=/home/jollaman999/git/ansible_hcc/roles/middleware/binaries/violin)
changed: [172.168.10.100] => (item=/home/jollaman999/git/ansible_hcc/roles/middleware/binaries/harp)
changed: [172.168.10.100] => (item=/home/jollaman999/git/ansible_hcc/roles/middleware/binaries/violin-scheduler)
changed: [172.168.10.100] => (item=/home/jollaman999/git/ansible_hcc/roles/middleware/binaries/violin-novnc)
changed: [172.168.10.100] => (item=/home/jollaman999/git/ansible_hcc/roles/middleware/binaries/piccolo)

TASK [middleware : Harp - Install VnStat v1.18] ************************************************************************
목요일 30 12월 2021  17:40:27 +0900 (0:00:08.181)       0:02:10.560 ***************
목요일 30 12월 2021  17:40:27 +0900 (0:00:08.181)       0:02:10.560 ***************
changed: [172.168.10.100]

TASK [middleware : Harp - Install isc-dhcp-server] *********************************************************************
목요일 30 12월 2021  17:40:31 +0900 (0:00:03.535)       0:02:14.096 ***************
목요일 30 12월 2021  17:40:31 +0900 (0:00:03.535)       0:02:14.096 ***************
changed: [172.168.10.100]

TASK [middleware : Harp - Backup previous dhcpd configuration file] ****************************************************
목요일 30 12월 2021  17:40:39 +0900 (0:00:08.041)       0:02:22.138 ***************
목요일 30 12월 2021  17:40:39 +0900 (0:00:08.041)       0:02:22.137 ***************
changed: [172.168.10.100]

TASK [middleware : Harp - Copy dhcpd configuration file] ***************************************************************
목요일 30 12월 2021  17:40:39 +0900 (0:00:00.192)       0:02:22.331 ***************
목요일 30 12월 2021  17:40:39 +0900 (0:00:00.192)       0:02:22.330 ***************
changed: [172.168.10.100] => (item=/home/jollaman999/git/ansible_hcc/roles/middleware/configs/dhcpd/dhcpd.conf)

TASK [middleware : Copying middleware systemd services] ****************************************************************
목요일 30 12월 2021  17:40:39 +0900 (0:00:00.353)       0:02:22.684 ***************
목요일 30 12월 2021  17:40:39 +0900 (0:00:00.353)       0:02:22.684 ***************
changed: [172.168.10.100] => (item=/home/jollaman999/git/ansible_hcc/roles/middleware/systemd-services/violin.service)
changed: [172.168.10.100] => (item=/home/jollaman999/git/ansible_hcc/roles/middleware/systemd-services/violin-scheduler.service)
changed: [172.168.10.100] => (item=/home/jollaman999/git/ansible_hcc/roles/middleware/systemd-services/harp.service)
changed: [172.168.10.100] => (item=/home/jollaman999/git/ansible_hcc/roles/middleware/systemd-services/piccolo.service)
changed: [172.168.10.100] => (item=/home/jollaman999/git/ansible_hcc/roles/middleware/systemd-services/violin-novnc.service)
changed: [172.168.10.100] => (item=/home/jollaman999/git/ansible_hcc/roles/middleware/systemd-services/piano.service)
changed: [172.168.10.100] => (item=/home/jollaman999/git/ansible_hcc/roles/middleware/systemd-services/flute.service)

RUNNING HANDLER [mysql : Restart MySQL Service] ************************************************************************
목요일 30 12월 2021  17:40:43 +0900 (0:00:03.409)       0:02:26.094 ***************
목요일 30 12월 2021  17:40:43 +0900 (0:00:03.409)       0:02:26.093 ***************
changed: [172.168.10.100]

RUNNING HANDLER [rabbitmq : Restart RabbitMQ Service] ******************************************************************
목요일 30 12월 2021  17:40:46 +0900 (0:00:03.175)       0:02:29.269 ***************
목요일 30 12월 2021  17:40:46 +0900 (0:00:03.175)       0:02:29.268 ***************
changed: [172.168.10.100]

RUNNING HANDLER [middleware : Reload daemon] ***************************************************************************
목요일 30 12월 2021  17:41:04 +0900 (0:00:17.566)       0:02:46.836 ***************
목요일 30 12월 2021  17:41:04 +0900 (0:00:17.566)       0:02:46.835 ***************
changed: [172.168.10.100]

RUNNING HANDLER [middleware : Restart flute service] *******************************************************************
목요일 30 12월 2021  17:41:04 +0900 (0:00:00.374)       0:02:47.210 ***************
목요일 30 12월 2021  17:41:04 +0900 (0:00:00.374)       0:02:47.210 ***************
changed: [172.168.10.100]

RUNNING HANDLER [middleware : Restart harp service] ********************************************************************
목요일 30 12월 2021  17:41:04 +0900 (0:00:00.493)       0:02:47.704 ***************
목요일 30 12월 2021  17:41:04 +0900 (0:00:00.493)       0:02:47.703 ***************
changed: [172.168.10.100]

RUNNING HANDLER [middleware : Restart piano service] *******************************************************************
목요일 30 12월 2021  17:41:05 +0900 (0:00:00.503)       0:02:48.208 ***************
목요일 30 12월 2021  17:41:05 +0900 (0:00:00.503)       0:02:48.207 ***************
changed: [172.168.10.100]

RUNNING HANDLER [middleware : Restart piccolo service] *****************************************************************
목요일 30 12월 2021  17:41:05 +0900 (0:00:00.485)       0:02:48.694 ***************
목요일 30 12월 2021  17:41:05 +0900 (0:00:00.485)       0:02:48.693 ***************
changed: [172.168.10.100]

RUNNING HANDLER [middleware : Restart violin service] ******************************************************************
목요일 30 12월 2021  17:41:06 +0900 (0:00:00.486)       0:02:49.181 ***************
목요일 30 12월 2021  17:41:06 +0900 (0:00:00.486)       0:02:49.180 ***************
changed: [172.168.10.100]

RUNNING HANDLER [middleware : Restart violin-novnc service] ************************************************************
목요일 30 12월 2021  17:41:06 +0900 (0:00:00.498)       0:02:49.679 ***************
목요일 30 12월 2021  17:41:06 +0900 (0:00:00.498)       0:02:49.678 ***************
changed: [172.168.10.100]

RUNNING HANDLER [middleware : Restart violin-scheduler service] ********************************************************
목요일 30 12월 2021  17:41:07 +0900 (0:00:00.509)       0:02:50.188 ***************
목요일 30 12월 2021  17:41:07 +0900 (0:00:00.509)       0:02:50.188 ***************
changed: [172.168.10.100]

PLAY RECAP *************************************************************************************************************
172.168.10.100             : ok=35   changed=34   unreachable=0    failed=0    skipped=0    rescued=0    ignored=0

목요일 30 12월 2021  17:41:07 +0900 (0:00:00.529)       0:02:50.718 ***************
===============================================================================
rabbitmq : Install RabbitMQ Server ----------------------------------------------------------------------------- 62.90s
mysql : Update and Install Package for MySQL ------------------------------------------------------------------- 31.71s
rabbitmq : Restart RabbitMQ Service ---------------------------------------------------------------------------- 17.57s
middleware : Copying middleware binaries ------------------------------------------------------------------------ 8.18s
middleware : Harp - Install isc-dhcp-server --------------------------------------------------------------------- 8.04s
influxdb : Install influxDB deb package ------------------------------------------------------------------------- 7.30s
influxdb : Restart InfluxDB Service ----------------------------------------------------------------------------- 5.82s
middleware : Copying middleware configs ------------------------------------------------------------------------- 5.16s
middleware : Harp - Install VnStat v1.18 ------------------------------------------------------------------------ 3.54s
middleware : Copying middleware systemd services ---------------------------------------------------------------- 3.41s
mysql : Restart MySQL Service ----------------------------------------------------------------------------------- 3.18s
rabbitmq : Enable RabbitMQ Management Plugin -------------------------------------------------------------------- 2.63s
Gathering Facts ------------------------------------------------------------------------------------------------- 1.42s
mysql : Importing mysql backup ---------------------------------------------------------------------------------- 0.97s
mysql : Copying mysql backup ------------------------------------------------------------------------------------ 0.65s
rabbitmq : RabbitMQ - Set permissions --------------------------------------------------------------------------- 0.57s
middleware : Restart violin-scheduler service ------------------------------------------------------------------- 0.53s
rabbitmq : RabbitMQ - Set topic permissions --------------------------------------------------------------------- 0.51s
middleware : Restart violin-novnc service ----------------------------------------------------------------------- 0.51s
rabbitmq : RabbitMQ - Add admin user ---------------------------------------------------------------------------- 0.51s
목요일 30 12월 2021  17:41:07 +0900 (0:00:00.530)       0:02:50.718 ***************
===============================================================================
rabbitmq --------------------------------------------------------------- 85.20s
mysql ------------------------------------------------------------------ 37.33s
middleware ------------------------------------------------------------- 32.96s
influxdb --------------------------------------------------------------- 13.80s
gather_facts ------------------------------------------------------------ 1.42s
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
total ----------------------------------------------------------------- 170.71s
```



Playbook 실행이 정상적으로 완료 되었습니다.



## 서비스 정상 동작 확인

Master Node 에 접속해 봅니다.

```shell
$ vagrant ssh hcc-master
Welcome to Ubuntu 20.04.3 LTS (GNU/Linux 5.4.0-88-generic x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/advantage

  System information as of Thu Dec 30 02:06:32 UTC 2021

  System load:  0.36              Processes:               133
  Usage of /:   6.9% of 38.71GB   Users logged in:         0
  Memory usage: 41%               IPv4 address for enp0s3: 10.0.2.15
  Swap usage:   0%                IPv4 address for enp0s8: 172.168.10.100

 * Super-optimized for small spaces - read how we shrank the memory
   footprint of MicroK8s to make it the smallest full K8s around.

   https://ubuntu.com/blog/microk8s-memory-optimisation

61 updates can be applied immediately.
35 of these updates are standard security updates.
To see these additional updates run: apt list --upgradable


Last login: Thu Dec 30 02:03:10 2021 from 172.168.10.1
vagrant@master:~$
```



각 미들웨어 서비스들이 정상적으로 동작중인지 확인해 봅니다.

```shell
vagrant@master:~$ sudo service piccolo status
● piccolo.service - HCC Piccolo Service
     Loaded: loaded (/etc/systemd/system/piccolo.service; enabled; vendor preset: enabled)
     Active: active (running) since Thu 2021-12-30 08:41:05 UTC; 4min 19s ago
   Main PID: 10471 (piccolo)
      Tasks: 8 (limit: 2286)
     Memory: 6.8M
     CGroup: /system.slice/piccolo.service
             └─10471 /usr/local/bin/piccolo

Dec 30 08:41:05 master piccolo[10471]: piccolo_logger: 2021/12/30 08:41:05 Opening Production GraphQL server on port 79>
Dec 30 08:41:05 master piccolo[10471]: piccolo_logger: 2021/12/30 08:41:05 Opening Dev Internal GraphQL server on port >
Dec 30 08:41:05 master piccolo[10471]: piccolo_logger: 2021/12/30 08:41:05 Opening gRPC server on port 7901...
Dec 30 08:41:06 master piccolo[10471]: piccolo_logger: 2021/12/30 08:41:06 checkFlute(): Flute module seems dead. Pingi>
Dec 30 08:41:06 master piccolo[10471]: piccolo_logger: 2021/12/30 08:41:06 checkHarp(): Harp module seems dead. Pinging>
Dec 30 08:41:07 master piccolo[10471]: piccolo_logger: 2021/12/30 08:41:07 checkFlute(): Ping Ok! Resetting connection.>
Dec 30 08:41:07 master piccolo[10471]: piccolo_logger: 2021/12/30 08:41:07 gRPC flute client ready
Dec 30 08:41:07 master piccolo[10471]: piccolo_logger: 2021/12/30 08:41:07 checkCello(): Cello module seems dead. Pingi>
Dec 30 08:41:10 master piccolo[10471]: piccolo_logger: 2021/12/30 08:41:10 checkHarp(): Ping Ok! Resetting connection...
Dec 30 08:41:10 master piccolo[10471]: piccolo_logger: 2021/12/30 08:41:10 gRPC harp client ready
vagrant@master:~$ sudo service flute status
● flute.service - HCC Flute Service
     Loaded: loaded (/etc/systemd/system/flute.service; enabled; vendor preset: enabled)
     Active: active (running) since Thu 2021-12-30 08:41:04 UTC; 4min 28s ago
   Main PID: 10230 (flute)
      Tasks: 8 (limit: 2286)
     Memory: 7.4M
     CGroup: /system.slice/flute.service
             └─10230 /usr/local/bin/flute

Dec 30 08:45:29 master flute[10230]: flute_logger: 2021/12/30 08:45:29 Get "https://172.31.0.3/redfish/v1/Systems/": di>
Dec 30 08:45:29 master flute[10230]: flute_logger: 2021/12/30 08:45:29 GetSerialNo(): Retrying for 172.31.0.3 1/3
Dec 30 08:45:32 master flute[10230]: flute_logger: 2021/12/30 08:45:32 Get "https://172.31.0.1/redfish/v1/Systems/": co>
Dec 30 08:45:32 master flute[10230]: flute_logger: 2021/12/30 08:45:32 GetSerialNo(): Retrying for 172.31.0.1 1/3
Dec 30 08:45:32 master flute[10230]: flute_logger: 2021/12/30 08:45:32 Get "https://172.31.0.3/redfish/v1/Systems/": di>
Dec 30 08:45:32 master flute[10230]: flute_logger: 2021/12/30 08:45:32 GetSerialNo(): Retrying for 172.31.0.3 1/3
Dec 30 08:45:32 master flute[10230]: flute_logger: 2021/12/30 08:45:32 Get "https://172.31.0.4/redfish/v1/Systems/": co>
Dec 30 08:45:32 master flute[10230]: flute_logger: 2021/12/30 08:45:32 GetSerialNo(): Retrying for 172.31.0.4 1/3
Dec 30 08:45:32 master flute[10230]: flute_logger: 2021/12/30 08:45:32 Get "https://172.31.0.2/redfish/v1/Systems/": co>
Dec 30 08:45:32 master flute[10230]: flute_logger: 2021/12/30 08:45:32 GetSerialNo(): Retrying for 172.31.0.2 1/3
vagrant@master:~$ sudo service violin status
● violin.service - HCC Violin Service
     Loaded: loaded (/etc/systemd/system/violin.service; enabled; vendor preset: enabled)
     Active: active (running) since Thu 2021-12-30 08:41:06 UTC; 4min 32s ago
   Main PID: 10547 (violin)
      Tasks: 8 (limit: 2286)
     Memory: 6.3M
     CGroup: /system.slice/violin.service
             └─10547 /usr/local/bin/violin

Dec 30 08:41:06 master violin[10547]: violin_logger: 2021/12/30 08:41:06 Connected to MySQL database
Dec 30 08:41:06 master violin[10547]: violin_logger: 2021/12/30 08:41:06 Connected to RabbitMQ server
Dec 30 08:41:06 master violin[10547]: violin_logger: 2021/12/30 08:41:06 Opened RabbitMQ channel.
Dec 30 08:41:06 master violin[10547]: violin_logger: 2021/12/30 08:41:06 gRPC flute client ready
Dec 30 08:41:06 master violin[10547]: violin_logger: 2021/12/30 08:41:06 gRPC harp client ready
Dec 30 08:41:06 master violin[10547]: violin_logger: 2021/12/30 08:41:06 gRPC violin client ready
Dec 30 08:41:06 master violin[10547]: violin_logger: 2021/12/30 08:41:06 gRPC violin-scheduler client ready
Dec 30 08:41:06 master violin[10547]: violin_logger: 2021/12/30 08:41:06 gRPC piccolo client ready
Dec 30 08:41:06 master violin[10547]: violin_logger: 2021/12/30 08:41:06 RabbitMQ forever channel ready.
Dec 30 08:41:06 master violin[10547]: violin_logger: 2021/12/30 08:41:06 Opening gRPC server on port 7500...
vagrant@master:~$ sudo service harp status
● harp.service - HCC Harp Service
     Loaded: loaded (/etc/systemd/system/harp.service; enabled; vendor preset: enabled)
     Active: active (running) since Thu 2021-12-30 08:41:04 UTC; 4min 40s ago
   Main PID: 10308 (harp)
      Tasks: 8 (limit: 2286)
     Memory: 45.2M
     CGroup: /system.slice/harp.service
             └─10308 /usr/local/bin/harp

Dec 30 08:41:09 master harp[10308]: harp_logger: 2021/12/30 08:41:09 Adding TCP input iptables rule for the master node>
Dec 30 08:41:09 master harp[10308]: harp_logger: 2021/12/30 08:41:09 Adding TCP input iptables rule for the master node>
Dec 30 08:41:09 master harp[10308]: harp_logger: 2021/12/30 08:41:09 Adding TCP input iptables rule for the master node>
Dec 30 08:41:09 master harp[10308]: harp_logger: 2021/12/30 08:41:09 Adding TCP input iptables rule for the master node>
Dec 30 08:41:09 master harp[10308]: harp_logger: 2021/12/30 08:41:09 Adding TCP input iptables rule for the master node>
Dec 30 08:41:09 master harp[10308]: harp_logger: 2021/12/30 08:41:09 Forwarding Timpani connection...
Dec 30 08:41:09 master harp[10308]: harp_logger: 2021/12/30 08:41:09 Adding AdaptiveIP Server forwarding iptables rules>
Dec 30 08:41:09 master harp[10308]: harp_logger: 2021/12/30 08:41:09 Adding TCP forwarding iptables rules for 172.168.1>
Dec 30 08:41:09 master harp[10308]: harp_logger: 2021/12/30 08:41:09 Enabling ip_forward for IPv4...
Dec 30 08:41:09 master harp[10308]: harp_logger: 2021/12/30 08:41:09 Opening gRPC server on port 7400...
vagrant@master:~$ sudo service piano status
● piano.service - HCC Piano Service
     Loaded: loaded (/etc/systemd/system/piano.service; enabled; vendor preset: enabled)
     Active: active (running) since Thu 2021-12-30 08:41:05 UTC; 4min 47s ago
   Main PID: 10392 (piano)
      Tasks: 8 (limit: 2286)
     Memory: 7.7M
     CGroup: /system.slice/piano.service
             └─10392 /usr/local/bin/piano

Dec 30 08:44:50 master piano[10392]: piano_logger: 2021/12/30 08:44:50 UpdateBillingInfo(): getVolumeBillingInfo(): rpc>
Dec 30 08:44:53 master piano[10392]: piano_logger: 2021/12/30 08:44:53 UpdateBillingInfo(): getVolumeBillingInfo(): rpc>
Dec 30 08:44:56 master piano[10392]: piano_logger: 2021/12/30 08:44:56 UpdateBillingInfo(): getVolumeBillingInfo(): rpc>
Dec 30 08:44:59 master piano[10392]: piano_logger: 2021/12/30 08:44:59 UpdateBillingInfo(): getVolumeBillingInfo(): rpc>
Dec 30 08:45:02 master piano[10392]: piano_logger: 2021/12/30 08:45:02 UpdateBillingInfo(): getVolumeBillingInfo(): rpc>
Dec 30 08:45:05 master piano[10392]: piano_logger: 2021/12/30 08:45:05 UpdateBillingInfo(): getVolumeBillingInfo(): rpc>
Dec 30 08:45:08 master piano[10392]: piano_logger: 2021/12/30 08:45:08 UpdateBillingInfo(): getVolumeBillingInfo(): rpc>
Dec 30 08:45:11 master piano[10392]: piano_logger: 2021/12/30 08:45:11 UpdateBillingInfo(): getVolumeBillingInfo(): rpc>
Dec 30 08:45:14 master piano[10392]: piano_logger: 2021/12/30 08:45:14 UpdateBillingInfo(): getVolumeBillingInfo(): rpc>
Dec 30 08:45:47 master piano[10392]: piano_logger: 2021/12/30 08:45:47 UpdateBillingInfo(): getVolumeBillingInfo(): rpc>
vagrant@master:~$ sudo service violin-novnc status
● violin-novnc.service - HCC Violin-NoVNC Service
     Loaded: loaded (/etc/systemd/system/violin-novnc.service; enabled; vendor preset: enabled)
     Active: active (running) since Thu 2021-12-30 08:41:06 UTC; 4min 56s ago
   Main PID: 10626 (violin-novnc)
      Tasks: 6 (limit: 2286)
     Memory: 6.1M
     CGroup: /system.slice/violin-novnc.service
             └─10626 /usr/local/bin/violin-novnc

Dec 30 08:41:06 master systemd[1]: Started HCC Violin-NoVNC Service.
Dec 30 08:41:06 master violin-novnc[10626]: violin-novnc_logger: 2021/12/30 08:41:06 Connected to MySQL database
Dec 30 08:41:06 master violin-novnc[10626]: violin-novnc_logger: 2021/12/30 08:41:06 gRPC harp client ready
Dec 30 08:41:06 master violin-novnc[10626]: violin-novnc_logger: 2021/12/30 08:41:06 Opening server on port 7800...
Dec 30 08:41:07 master violin-novnc[10626]: violin-novnc_logger: 2021/12/30 08:41:07 checkHarp(): Harp module seems dea>
Dec 30 08:41:10 master violin-novnc[10626]: violin-novnc_logger: 2021/12/30 08:41:10 checkHarp(): Ping Ok! Resetting co>
Dec 30 08:41:10 master violin-novnc[10626]: violin-novnc_logger: 2021/12/30 08:41:10 gRPC harp client ready
vagrant@master:~$ sudo service violin-scheduler status
● violin-scheduler.service - HCC Violin Scheduler Service
     Loaded: loaded (/etc/systemd/system/violin-scheduler.service; enabled; vendor preset: enabled)
     Active: active (running) since Thu 2021-12-30 08:41:07 UTC; 5min ago
   Main PID: 10716 (violin-schedule)
      Tasks: 8 (limit: 2286)
     Memory: 2.0M
     CGroup: /system.slice/violin-scheduler.service
             └─10716 /usr/local/bin/violin-scheduler

Dec 30 08:41:07 master systemd[1]: Started HCC Violin Scheduler Service.
Dec 30 08:41:07 master violin-scheduler[10716]: violin-scheduler_logger: 2021/12/30 08:41:07 gRPC flute client ready
Dec 30 08:41:07 master violin-scheduler[10716]: violin-scheduler_logger: 2021/12/30 08:41:07 Opening gRPC server on por>
vagrant@master:~$
```



모든 서비스들이 정상적으로 실행되고 있음을 확인할 수 있습니다.



InfluxDB 도 정상적으로 동작중이며, 데이터베이스가 생성되어있음을 확인할 수 있습니다.

```shell
root@master:~# influx
Connected to http://localhost:8086 version 1.8.10
InfluxDB shell version: 1.8.10
> show databases
name: databases
name
----
telegraf
_internal
>
```



다음은 RabbitMQ 관리 웹페이지에 정상적으로 접속되는지 확인해보겠습니다.

이전에, Harp 에서 방화벽이 동작하기 때문에 임시로 RabbitMQ 관리 웹페이지에 접속이 가능하도록 iptables 명령을 다음과 같이 실행합니다.

```shell
iptables -I INPUT 1 -p tcp --dport 15672 -j ACCEPT
iptables -I FORWARD 1 -p tcp --dport 15672 -j ACCEPT
```



다음과 같이 RabbitMQ 관리 웹페이지에 정상적으로 접속됨을 확인할 수 있습니다.

![1](/assets/images/docs/Ansible 과 Vagrant로 HCloud-Platform 배포하는 방법 - 2편/1.png)

![2](/assets/images/docs/Ansible 과 Vagrant로 HCloud-Platform 배포하는 방법 - 2편/2.png)



admin 계정에 적용한 권한 설정도 정상적으로 적용되었음을 확인할 수 있습니다.

![3](/assets/images/docs/Ansible 과 Vagrant로 HCloud-Platform 배포하는 방법 - 2편/3.png)



확인을 마쳤으면 위에서 적용했던 iptables 정책들을 지워줍니다.

```shell
iptables -D INPUT -p tcp --dport 15672 -j ACCEPT
iptables -D FORWARD -p tcp --dport 15672 -j ACCEPT
```



지금까지 Vagrant 와 Ansible 로 HCloud-Platform 실행환경을 구성하는 방법에 대해서 알아보았습니다.



긴글 읽어 주셔서 감사합니다.
