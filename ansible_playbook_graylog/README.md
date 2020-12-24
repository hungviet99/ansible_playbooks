# Ansible Playbooks Graylog

Playbook sử dụng để cài graylog trên CentOS 7

Playbook sẽ thực hiện các tác vụ trên máy chủ được quản lý như: 

- Cài đặt và cấu hình đồng bộ thời gian
- Cài đặt elasticsearch
- Cài đặt MongoDB
- Cài đặt và cấu hình Graylog

## Roles

| Role | Description |
|-------|------------|
| setup_env | Cấu hình môi trường cần thiết trước khi bắt đầu cài đặt graylog |
| elastic_install | Cài đặt và cấu hình Elasticsearch | 
| mongodb_install | Cài đặt MongoDB |
| graylog-server | Cài đặt và cấu hình các bước cần thiết để chạy graylog |

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
└── setup_env
    ├── defaults
    │   └── main.yml
    └── tasks
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
cd /root/ansible_playbooks/ansible_playbook_graylog
```
### 3. Cấu hình file inventory

Thêm vào các máy chủ được quản lý trong file `hosts`. 

Ví dụ như sau: 

```
[graylogsrv]
host1 ansible_host=10.10.10.10 ansible_port =22 ansible_user=root ansible_ssh_pass=password
```

trong đó: 

- `ansible_host` là địa chỉ của máy chủ được quản lý (máy chủ graylog)

- `ansible_port` là port sử dụng để ssh vào máy chủ từ xa

- `ansible_user` là thông tin user sử dụng để đăng nhập ssh 

- `ansible_ssh_pass` là password của user đó

### 4. Cấu hình file playbook

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

Sử dụng `vi` hoặc `vim` để truy cập file trên với đường dẫn `/root/ansible_playbooks/ansible_playbook_graylog/playbook-graylog.yml` và sửa 1 số thông tin sau:

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

### 5. Chạy playbook

Sau khi cấu hình các cài đặt cần thiết, bạn có thể chạy playbook như sau :

```
cd /root/ansible_playbooks/ansible_playbook_graylog

ansible-playbook -i hosts playbook-graylog.yml
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

### 6. Kết thúc cài đặt 

Mở trình duyệt web của bạn và điều hướng đến http://ip_graylog_server:9000 để truy cập graylog web interface. 

Author Information
------------------

This role was created in 2020 by Nguyen Viet Hung.

