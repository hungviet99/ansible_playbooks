# Ansible Playbooks ELK

Playbook sử dụng để cài elk trên CentOS 7 và Ubuntu 18.04

Playbook sẽ thực hiện các tác vụ trên máy chủ được quản lý như: 

- Cài đặt và cấu hình đồng bộ thời gian
- Cài đặt và cấu hình elasticsearch
- Cài đặt logstash
- Cài đặt và cấu hình kibana

## Roles

| Role | Description |
|-------|------------|
| setup_env | Cấu hình môi trường cần thiết trước khi bắt đầu cài đặt ELK |
| elastic_install | Cài đặt và cấu hình Elasticsearch | 
| logstash_install | Cài đặt logstash |
| kibana_install | Cài đặt và cấu hình kibana |

## Tree 

Cấu trúc của các role được thể hiện như sau: 

```
roles 
├── elastic_install
│   ├── defaults
│   │   └── main.yml
│   ├── tasks
│   │   ├── install_elastic_centos_7.yml
│   │   ├── install_elastic_ubuntu_18.yml
│   │   └── main.yml
│   └── templates
│       └── repo-elastic.j2
├── kibana_install
│   ├── defaults
│   │   └── main.yml
│   ├── tasks
│   │   ├── install_kibana_centos_7.yml
│   │   ├── install_kibana_ubuntu_18.yml
│   │   └── main.yml
│   └── templates
│       └── repo-kibana.j2
├── logstash_install
│   ├── defaults
│   │   └── main.yml
│   ├── tasks
│   │   ├── install_logstash_centos_7.yml
│   │   ├── install_logstash_ubuntu_18.yml
│   │   └── main.yml
│   └── templates
│       └── repo-logstash.j2
└── setup_env
    ├── defaults
    │   └── main.yml
    └── tasks
        ├── config_env_centos_7.yml
        ├── config_env_ubuntu_18.yml
        ├── config_selinux.yml
        ├── main.yml
        └── setup_ntp.yml
```

## Guide

Để bắt đầu sử dụng playbook này, bạn chỉ cần làm theo các bước sau: 

### 1. Clone repository

```
cd
git clone https://github.com/hungviet99/ansible_playbooks.git
```

### 2. Truy cập vào thư mục playbook

```
cd /root/ansible_playbooks/ansible_playbook_elk
```

### 3. Cấu hình file inventory

Thêm vào các máy chủ được quản lý trong file `hosts`. 

Ví dụ như sau: 

```
[elksrv]
host1 ansible_host=10.10.10.10 ansible_port =22 ansible_user=root ansible_ssh_pass=password
```

trong đó: 

- `ansible_host` là địa chỉ của máy chủ được quản lý (máy chủ elk)

- `ansible_port` là port sử dụng để ssh vào máy chủ từ xa

- `ansible_user` là thông tin user sử dụng để đăng nhập ssh 

- `ansible_ssh_pass` là password của user đó

### 4. Cấu hình file playbook

Ban đầu ta sẽ có 1 file playbook như sau: 

```
---
- hosts: elksrv
  become: yes
  remote_user: root
  vars:
    ip_ntp_server: ''
    elastic_version: '7.x'
    logstash_version: '6.x'
    kibana_version: '6.x'
  roles: 
    - {role: setup_env, tags: ['setup_env']}
    - {role: elastic_install, tags: ['elastic_install']}
    - {role: logstash_install, tags: ['logstash_install']}
    - {role: kibana_install, tags: ['kibana_install']}
```

Sử dụng `vi` hoặc `vim` để truy cập file trên với đường dẫn `/root/ansible_playbooks/ansible_playbook_elk/playbook_elk.yml` và sửa 1 số thông tin sau:

- Sửa thông tin remote user:

Có thể sửa remote user `root` thành các user có quyền truy cập vào các host được quản lý.

- Sửa thông tin biến các biến `vars`: 

    - Biến để đặt địa chỉ của máy chủ ntp trong mạng của bạn : `ip_ntp_server`
    - Biến chỉ định version cho elasticsearch (version mặc định là 7.x): `elastic_version`
    - Biến chỉ định version cho logstash (version mặc định là 6.x): `logstash_version`
    - Biến chỉ định version cho kibana (version mặc định là 6.x): `kibana_version`

### 5. Chạy playbook

Sau khi cấu hình các cài đặt cần thiết, bạn có thể chạy playbook như sau :

```
cd /root/ansible_playbooks/ansible_playbook_elk

ansible-playbook -i hosts playbook-elk.yml
```

Ta sẽ nhận được đầu ra tương tự như sau: 

```
PLAY [elksrv] ****************************************************************************************************************************************************************************

TASK [Gathering Facts] *******************************************************************************************************************************************************************
ok: [host1]

TASK [setup_env : include_tasks] *********************************************************************************************************************************************************
included: /root/ansible_playbooks/ansible_playbook_elk/roles/setup_env/tasks/config_env_centos_7.yml for host1

TASK [setup_env : install elpel-release] *************************************************************************************************************************************************
ok: [host1]

...

TASK [logstash_install : add repo logtash] ***********************************************************************************************************************************************
changed: [host1]

TASK [logstash_install : Install logstash] ***********************************************************************************************************************************************
changed: [host1]

TASK [logstash_install : Restart logstash] ***********************************************************************************************************************************************
changed: [host1]

...

TASK [kibana_install : Install kibana] ***************************************************************************************************************************************************
changed: [host1]

TASK [kibana_install : Restart kibana] ***************************************************************************************************************************************************
changed: [host1]

PLAY RECAP *******************************************************************************************************************************************************************************
host1         : ok=27   changed=19   unreachable=0    failed=0    skipped=4    rescued=0    ignored=0

```
Khi `failed` = 0 tức là không có lỗi xảy ra trong quá trình cài đặt. Quá trình chạy playbook hoàn tất !

### 6. Kết thúc cài đặt 

Mở trình duyệt web của bạn và điều hướng đến http://ip_elk_server:9000 để truy cập kibana trên trình duyệt.  

Author Information
------------------

This role was created in 2020 by Nguyen Viet Hung.

