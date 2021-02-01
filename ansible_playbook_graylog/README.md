# Ansible Playbooks Graylog

Playbook sử dụng để cài đặt graylog-server và graylog-sidecar trên CentOS 7

Playbook sẽ thực hiện các tác vụ trên máy chủ được quản lý như: 

**Graylog-server:**

- Cài đặt và cấu hình đồng bộ thời gian
- Cài đặt elasticsearch
- Cài đặt MongoDB
- Cài đặt và cấu hình Graylog-Server

**Graylog-sidecar:**

- Cài đặt và cấu hình đồng bộ thời gian
- Cài đặt filebeat và graylog-sidecar

## Roles

| Role | Description |
|-------|------------|
| setup_env | Cấu hình môi trường cần thiết trước khi bắt đầu cài đặt graylog-server |
| elastic_install | Cài đặt và cấu hình Elasticsearch | 
| mongodb_install | Cài đặt MongoDB |
| graylog-server | Cài đặt và cấu hình các bước cần thiết để chạy graylog-server |
| sidecar_install | Cài đặt và cấu hình graylog-sidecar đẩy log về graylog-server |

## Tree 

Cấu trúc của các role được thể hiện như sau: 

```
roles 
├── elastic_install
│   ├── defaults
│   │   └── main.yml
│   ├── tasks
│   │   └── main.yml
│   └── templates
│       └── repo-elastic.j2
├── graylog-server
│   ├── defaults
│   │   └── main.yml
│   └── tasks
│       └── main.yml
├── mongodb_install
│   ├── defaults
│   │   └── main.yml
│   ├── tasks
│   │   └── main.yml
│   └── templates
│       └── repo-mongodb.j2
├── setup_env
│   ├── defaults
│   │   └── main.yml
│   └── tasks
│       ├── config_selinux.yml
│       ├── main.yml
│       └── setup_ntp.yml
└── sidecar-install
    ├── defaults
    │   └── main.yml
    └── tasks
        ├── install_sidecar_centos7.yml
        ├── main.yml
        └── setup_env.yml
```

## Guide

Để bắt đầu sử dụng playbook này, bạn chỉ cần làm theo các bước sau: 

## 1. Cấu hình ban đầu

### 1.1 Clone repository

```
cd
git clone https://github.com/hungviet99/ansible_playbooks.git
```
### 1.2 Truy cập vào thư mục playbook

```
cd /root/ansible_playbooks/ansible_playbook_graylog
```
### 1.3 Cấu hình file inventory

Thêm vào các máy chủ được quản lý trong file `hosts`. 

Ví dụ như sau: 

```
[graylogsrv]
graylog-server ansible_host=10.10.10.10 ansible_port =22 ansible_user=root ansible_ssh_pass=password
[sidecar]
client1 ansible_host=10.10.10.11 ansible_port=22 ansible_user=root ansible_ssh_pass=password
```

trong đó: 

- Ở cột đầu tiên là `hostname` của các máy chủ. 

- `ansible_host` là địa chỉ của máy chủ được quản lý (máy chủ graylog)

- `ansible_port` là port sử dụng để ssh vào máy chủ từ xa

- `ansible_user` là thông tin user sử dụng để đăng nhập ssh 

- `ansible_ssh_pass` là password của user đó

## 2. Sử dụng playbook để cài graylog-server

### 2.1. Cấu hình file playbook

Ban đầu ta sẽ có 1 file playbook như sau: 

```
---
- hosts: graylogsrv
  become: yes
  remote_user: root
  vars:
    ip_ntp_server:
    graylog_version: '4.0'
    pass_login_graylog:
    timezone_for_root: 'Asia/Ho_Chi_Minh'
    http_bind_address: '0.0.0.0'
    http_publish_uri: '0.0.0.0'
    mongodb_version: '4.2'
  roles:
    - {role: setup_env, tags: ['setup_env']}
    - {role: elastic_install, tags: ['elastic_install']}
    - {role: mongodb_install, tags: ['mongodb_install']}
    - {role: graylog-server, tags: ['graylog-server']}
```

Sử dụng `vi` hoặc `vim` để truy cập file  `playbook-graylog-server.yml` và sửa 1 số thông tin sau:

- Sửa thông tin remote user:

Có thể sửa remote user `root` thành các user có quyền truy cập vào các host được quản lý.

- Sửa thông tin biến các biến `vars`: 

    - Biến để đặt địa chỉ của máy chủ ntp trong mạng của bạn : `ip_ntp_server`
    - Biến để đặt version của graylog server (mặc định là graylog phiên bản 4.0): `graylog_version`
    - Biến để đặt pass login vào graylog trên web: `pass_login_graylog`
    - Biến đặt time zone cho graylog : `timezone_for_root`
    - Biến để đặt giao diện mạng sẽ được truy cập bởi các máy trong cụm: `http_bind_address`
    - Biến để đặt giao diện mạng sẽ sử dụng để truy cập trên web interface: `http_publish_uri`
    - Biến để đặt version của mongodb (mặc định là mongodb phiên bản 4.2): `mongodb_version`

### 2.2. Chạy playbook

Sau khi cấu hình các cài đặt cần thiết, bạn có thể chạy playbook như sau :

```
cd /root/ansible_playbooks/ansible_playbook_graylog

ansible-playbook -i hosts playbook-graylog-server.yml
```

Ta sẽ nhận được đầu ra tương tự như sau: 

```
PLAY [graylogsrv] ************************************************************************************************************************************************************************

TASK [Gathering Facts] *******************************************************************************************************************************************************************
ok: [host1]

TASK [setup_env : install elpel-release] *************************************************************************************************************************************************
changed: [host1]

TASK [setup_env : update system] *********************************************************************************************************************************************************
changed: [host1]

...

TASK [mongodb_install : Install mongodb] *************************************************************************************************************************************************
changed: [host1]

TASK [mongodb_install : Restart mongodb] *************************************************************************************************************************************************
changed: [host1]

TASK [graylog-server : Repositories should be updated] ***********************************************************************************************************************************
changed: [host1]

TASK [graylog-server : update sys] *******************************************************************************************************************************************************
ok: [host1]

TASK [Install graylog-server] ************************************************************************************************************************************************************
changed: [host1]


...

TASK [graylog-server : replace] **********************************************************************************************************************************************************
changed: [host1]

TASK [graylog-server : Restart graylog] **************************************************************************************************************************************************
changed: [host1]

PLAY RECAP *******************************************************************************************************************************************************************************
host1                      : ok=31   changed=26   unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
```
Khi `failed` = 0 tức là không có lỗi xảy ra trong quá trình cài đặt. Quá trình chạy playbook hoàn tất !

### 2.3 Kết thúc cài đặt 

Mở trình duyệt web của bạn và điều hướng đến http://ip_graylog_server:9000 để truy cập graylog web interface. 

## 3. Sử dụng playbook để cài graylog-sidecar

### 3.1. Cấu hình file playbook

File playbook để cài graylog-sidecar ban đầu như sau: 

```
---
- hosts: sidecar
  become: yes
  remote_user: root
  vars:
    ip_ntp_server:
    ip_graylog_server:
    api_token:
    name_server: "{{ inventory_hostname }}"
  roles:
    - {role: sidecar-install, tags: ['sidecar-install']}
```

Sử dụng `vi` hoặc `vim` để truy cập file  `playbook-graylog-sidecar.yml` và sửa 1 số thông tin sau:

- Sửa thông tin remote user:

Có thể sửa remote user `root` thành các user có quyền truy cập vào các host được quản lý.

- Sửa thông tin biến các biến `vars`: 

    - Biến để đặt địa chỉ của máy chủ ntp trong mạng của bạn : `ip_ntp_server`
    - Biến để đặt địa chỉ của máy chủ đang chạy graylog-server (Các máy được cài graylog-server và graylog-sidecar phải cùng 1 VLAN): `ip_graylog_server`
    - Biến nhập API token của graylog server : `api_token`

### 3.2. Chạy playbook 




Author Information
------------------

This role was created in 2020 by Nguyen Viet Hung.

