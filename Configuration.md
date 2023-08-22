Содержание
==========
* [Terraform](#Terraform)
* [Ansible](#Ansible)
   * [node-exporter](#node-exporter)
   * [nginx](#nginx)
   * [prometheus](#prometheus)
   * [grafana](#grafana)
   * [elasticsearch](#elasticsearch)
   * [filebeat](#filebeat)
   * [kibana](#kibana)
   * [bastion](#bastion)

---------
## Terraform

`/home/admin/terraform/`

<details>

*<summary>metadata.txt</summary>*

``` GO

#cloud-config
users:
  - name: admin
    groups: sudo
    shell: /bin/bash
    sudo: ['ALL=(ALL) NOPASSWD:ALL']
    ssh-authorized-keys:
       - ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQDCtAda4X6uzM8J2IvQrJhixXrLrov1OCjADgwYqlPz9rUMG7glFeOJFf1YiQSJArRymqED/dIj0hYhBxfBvrZLnG+P3rEi0UsdlM0CEEn5pMlRZdvg3/EQvarNL88exchWOjSyDE7xBinLr+SANuLl4x8bV5YwPK2yUUNyw6taDYfHJBOpMMsKOBYj6PYg6dBSZs/+XwIbZVoRL8XLIq1YfC75Ao/MF/hmB3mPwkHssmWWUIPtu/eU0jr1EbwPdjZfaIMpkLtIzcP1JRyBXWqJ4mR+OhycYlENW+0TJcQ/EVrV1VObOfPbS1chUwBlvw3Eut+ILtPN2NPT6unVdSKR rsa-key-20230821
```
</details>

<details>

*<summary>config.tf</summary>*

``` GO

terraform {

  required_providers {

    yandex = {

      source = "yandex-cloud/yandex"

    }

  }

  required_version = ">= 0.13"

}

provider "yandex" {

    cloud_id  = "b1gcvt5l6bsrvg3nfac5"

  folder_id = "b1g0bhh4bik34mog3r9m"

  zone      = "ru-central1-a"

}

////////////////////////////////////////////////////////////////////////

resource "yandex_vpc_network" "anders" {

  name = "anders"

}

resource "yandex_vpc_subnet" "anders2" {
  name           = "anders2"
  zone           = "ru-central1-a"
  network_id     = yandex_vpc_network.anders.id
  v4_cidr_blocks = ["192.168.2.0/24"]
}

resource "yandex_vpc_subnet" "anders3" {
  name           = "anders3"
  zone           = "ru-central1-b"
  network_id     = yandex_vpc_network.anders.id
  v4_cidr_blocks = ["192.168.3.0/24"]
}

////////////////////////////////////////////////////////////////////////

resource "yandex_compute_instance" "vm2" {
  name = "web1"
  allow_stopping_for_update = true

  resources {
    cores  = 2
    memory = 2
	core_fraction = 20
  }

  boot_disk {
    initialize_params {
      image_id = "fd843htdp8usqsiji0bb"
    }
  }

  network_interface {
    subnet_id = yandex_vpc_subnet.anders2.id
    nat       = false
  }

  metadata = {
    user-data = "${file("/home/admin/terraform/metadata.txt")}"
  }
}

output "internal_ip_address_vm_2" {
  value = yandex_compute_instance.vm2.network_interface.0.ip_address
}

////////////////////////////////////////////////////////////////////////

resource "yandex_compute_instance" "vm3" {
  name = "web2"
  allow_stopping_for_update = true

  zone = "ru-central1-b"
  resources {
    cores  = 2
    memory = 2
	core_fraction = 20
   }

  boot_disk {
    initialize_params {
      image_id = "fd843htdp8usqsiji0bb"
    }
  }

  network_interface {
    subnet_id = yandex_vpc_subnet.anders3.id
    nat       = false
  }

  metadata = {
    user-data = "${file("/home/admin/terraform/metadata.txt")}"
  }
}

output "internal_ip_address_vm_3" {
  value = yandex_compute_instance.vm3.network_interface.0.ip_address
}

////////////////////////////////////////////////////////////////////////

resource "yandex_compute_instance" "vm4" {
  name = "prometheus"
  allow_stopping_for_update = true

  resources {
    cores  = 2
    memory = 2
	core_fraction = 20
  }

  boot_disk {
    initialize_params {
      image_id = "fd843htdp8usqsiji0bb"
    }
  }

  network_interface {
    subnet_id = yandex_vpc_subnet.anders2.id
    nat       = false

  }
  metadata = {
    user-data = "${file("/home/admin/terraform/metadata.txt")}"
  }
}

output "internal_ip_address_vm_4" {
  value = yandex_compute_instance.vm4.network_interface.0.ip_address
}

////////////////////////////////////////////////////////////////////////

resource "yandex_compute_instance" "vm5" {
  name = "grafana"
  allow_stopping_for_update = true
    resources {
    cores  = 2
    memory = 2
	core_fraction = 20
   }

  boot_disk {
    initialize_params {
      image_id = "fd843htdp8usqsiji0bb"
    }
  }

  network_interface {
    subnet_id = yandex_vpc_subnet.anders2.id
    nat       = true
    security_group_ids = ["enphroi8q7uccvfdo28t"]
  }

  metadata = {
    user-data = "${file("/home/admin/terraform/metadata.txt")}"
  }
}

output "internal_ip_address_vm_5" {
  value = yandex_compute_instance.vm5.network_interface.0.ip_address
}

output "external_ip_address_vm_5" {
  value = yandex_compute_instance.vm5.network_interface.0.nat_ip_address
}

////////////////////////////////////////////////////////////////////////

resource "yandex_compute_instance" "vm6" {
  name = "elasticsearch"
  allow_stopping_for_update = true

  resources {
    cores  = 2
    memory = 2
	core_fraction = 20
   }

  boot_disk {
    initialize_params {
      image_id = "fd843htdp8usqsiji0bb"
    }
  }

  network_interface {
    subnet_id = yandex_vpc_subnet.anders2.id
    nat       = false
  }

  metadata = {
    user-data = "${file("/home/admin/terraform/metadata.txt")}"
  }
}

output "internal_ip_address_vm_6" {
  value = yandex_compute_instance.vm6.network_interface.0.ip_address
}

////////////////////////////////////////////////////////////////////////

resource "yandex_compute_instance" "vm7" {
  name = "kibana"
  allow_stopping_for_update = true

  resources {
    cores  = 2
    memory = 2
	core_fraction = 20
  }

  boot_disk {
    initialize_params {
      image_id = "fd843htdp8usqsiji0bb"
    }
  }

  network_interface {
    subnet_id = yandex_vpc_subnet.anders2.id
    nat       = true
    security_group_ids = ["enphroi8q7uccvfdo28t"]
}
  metadata = {
    user-data = "${file("/home/admin/terraform/metadata.txt")}"
  }
}

output "internal_ip_address_vm_7" {
  value = yandex_compute_instance.vm7.network_interface.0.ip_address
}

output "external_ip_address_vm_7" {
  value = yandex_compute_instance.vm7.network_interface.0.nat_ip_address
}

////////////////////////////////////////////////////////////////////////

resource "yandex_compute_instance" "vm8" {
  name = "bastion"
  allow_stopping_for_update = true

  resources {
    cores  = 2
    memory = 2
	core_fraction = 20
  }

  boot_disk {
    initialize_params {
      image_id = "fd843htdp8usqsiji0bb"
    }
  }

  network_interface {
    subnet_id = yandex_vpc_subnet.anders2.id
    nat       = true
    security_group_ids = ["enppkv83r7cv81fh9hti"]
  }
  metadata = {
    user-data = "${file("/home/admin/terraform/metadata.txt")}"
  }
}

output "internal_ip_address_vm_8" {
  value = yandex_compute_instance.vm8.network_interface.0.ip_address
}

output "external_ip_address_vm_8" {
  value = yandex_compute_instance.vm8.network_interface.0.nat_ip_address
}

output "schedule_disk" {
  value =  [   resource.yandex_compute_instance.vm2.boot_disk[0].disk_id   ]
}
```
</details>

<details>

*<summary>targetgroup/config.tf</summary>*

``` GO

terraform {
  required_providers {
    yandex = {
      source = "yandex-cloud/yandex"
    }
  }
  required_version = ">= 0.13"
}

provider "yandex" {
  cloud_id  = "b1gcvt5l6bsrvg3nfac5"
  folder_id = "b1g0bhh4bik34mog3r9m"
  zone      = "ru-central1-a"
}

resource "yandex_alb_target_group" "foo" {
  name           = "target-group"

  target {
    subnet_id    = "e9b6slku7liimdj5jqgq"
    ip_address   = "192.168.2.15"
  }

  target {
    subnet_id    = "e2lmbuqsgrlau186thlb"
    ip_address   = "192.168.3.4"
  }
}
```
</details>

<details>

*<summary>backendgroup/config.tf</summary>*

``` GO

terraform {
  required_providers {
    yandex = {
      source = "yandex-cloud/yandex"
    }
  }
  required_version = ">= 0.13"
}

#provider "yandex" {
#  zone = "ru-central1-a"
#}

provider "yandex" {
  cloud_id  = "b1gcvt5l6bsrvg3nfac5"
  folder_id = "b1g0bhh4bik34mog3r9m"
  zone      = "ru-central1-a"
}

resource "yandex_alb_backend_group" "test-backend-group" {
  name                     = "backend-group"
  session_affinity {
    connection {
      source_ip = false
    }
  }

  http_backend {
    name                   = "backend-group"
    weight                 = 1
    port                   = 80
    target_group_ids       = ["ds7nroqreljl3ccncm90"]
    load_balancing_config {
      panic_threshold      = 90
    }    
    healthcheck {
      timeout              = "10s"
      interval             = "2s"
      healthy_threshold    = 10
      unhealthy_threshold  = 15 
      http_healthcheck {
        path               = "/"
      }
    }
  }
}
```
</details>

<details>

*<summary>router/config.tf</summary>*

``` GO

terraform {
  required_providers {
    yandex = {
      source = "yandex-cloud/yandex"
    }
  }
  required_version = ">= 0.13"
}


provider "yandex" {
  cloud_id  = "b1gcvt5l6bsrvg3nfac5"
  folder_id = "b1g0bhh4bik34mog3r9m"
  zone      = "ru-central1-a"
}

resource "yandex_alb_http_router" "tf-router" {
  name          = "router"
  labels        = {
    tf-label    = "tf-label-value"
    empty-label = ""
  }
}

resource "yandex_alb_virtual_host" "my-virtual-host" {
  name                    = "vm-main"
  http_router_id          = yandex_alb_http_router.tf-router.id
  route {
    name                  = "main"
    http_route {
      http_route_action {
        backend_group_id  = "ds7r41hec33n5sc8jp3o"
        timeout           = "60s"
      }
    }
  }
}
```
</details>

<details>

*<summary>balancer/config.tf</summary>*

``` GO

terraform {
  required_providers {
    yandex = {
      source = "yandex-cloud/yandex"
    }
  }
  required_version = ">= 0.13"
}


provider "yandex" {
  cloud_id  = "b1gcvt5l6bsrvg3nfac5"
  folder_id = "b1g0bhh4bik34mog3r9m"
  zone      = "ru-central1-a"
}



resource "yandex_alb_load_balancer" "test-balancer" {
  name        = "balancer"
  network_id  = "enp558euc92kq4tn1fpi" 

  allocation_policy {
    location {
      zone_id   = "ru-central1-a"
      subnet_id = "e9b6slku7liimdj5jqgq"   
    }
  }

  listener {
    name = "listener"
    endpoint {
      address {
        external_ipv4_address {
        }
      }
      ports = [ 80 ]
    }
    http {
      handler {
        http_router_id = "ds7823ms07o6impuu7k3"
      }
    }
  }
}
```
</details>

<details>

*<summary>security/config.tf</summary>*

``` GO

terraform {
  required_providers {
    yandex = {
      source = "yandex-cloud/yandex"
    }
  }
  required_version = ">= 0.13"
}

provider "yandex" {
  cloud_id  = "b1gcvt5l6bsrvg3nfac5"
  folder_id = "b1g0bhh4bik34mog3r9m"
  zone      = "ru-central1-a"
}

resource "yandex_vpc_security_group" "security" {
  name        = "security"
  description = "Description for security group"
  network_id  = "enp558euc92kq4tn1fpi"

  ingress {
    protocol       = "TCP"
    description    = "grafana"
    v4_cidr_blocks = ["0.0.0.0/0"]
    port           = 3000
  }

ingress {
    protocol       = "TCP"
    description    = "kibana"
    v4_cidr_blocks = ["0.0.0.0/0"]
    port           = 5601
  }

ingress {
    protocol       = "TCP"
    description    = "application load balancer"
    v4_cidr_blocks = ["0.0.0.0/0"]
    port           = 80
  }

ingress {
    protocol       = "TCP"
    description    = "SSH-permission"
    v4_cidr_blocks = ["192.168.2.0/24"]
    port           = 22
  }


  egress {
    protocol       = "ANY"
    description    = "Rule description 2"
    v4_cidr_blocks = ["0.0.0.0/0"]
    from_port      = 0
    to_port        = 65535
  }
}

resource "yandex_vpc_security_group" "bastion" {
  name        = "bastion"
  description = "Description for security group"
  network_id  = "enp558euc92kq4tn1fpi"

  ingress {
    protocol       = "TCP"
    description    = "bastion"
    v4_cidr_blocks = ["0.0.0.0/0"]
    port           = 22
  }

  egress {
    protocol       = "ANY"
    description    = "Rule description 2"
    v4_cidr_blocks = ["0.0.0.0/0"]
    from_port      = 0
    to_port        = 65535
  }
}
```
</details>

<details>

*<summary>snapshots/config.tf</summary>*

``` GO

terraform {
  required_providers {
    yandex = {
      source = "yandex-cloud/yandex"
    }
  }
  required_version = ">= 0.13"
}

provider "yandex" {
  cloud_id  = "b1gcvt5l6bsrvg3nfac5"
  folder_id = "b1g0bhh4bik34mog3r9m"
  zone      = "ru-central1-a"
}

data "yandex_compute_instance" "vm2" {
  name = "web1"
}

data "yandex_compute_instance" "vm3" {
  name = "web2"

}

data "yandex_compute_instance" "vm4" {
  name = "prometheus"

}

data "yandex_compute_instance" "vm5" {
  name = "grafana"

}

data "yandex_compute_instance" "vm6" {
  name = "elasticsearch"

}

data "yandex_compute_instance" "vm7" {
  name = "kibana"

}

data "yandex_compute_instance" "vm8" {
  name = "bastion"

}

resource "yandex_compute_snapshot_schedule" "default" {
  name = "snap"

  schedule_policy {
    expression = "00 19 ? * *"
  }

  snapshot_count = 7

  disk_ids = [data.yandex_compute_instance.vm2.boot_disk.0.disk_id,
              data.yandex_compute_instance.vm3.boot_disk[0].disk_id,
              data.yandex_compute_instance.vm4.boot_disk[0].disk_id,
              data.yandex_compute_instance.vm5.boot_disk[0].disk_id,
              data.yandex_compute_instance.vm6.boot_disk[0].disk_id, 
              data.yandex_compute_instance.vm7.boot_disk[0].disk_id,           
              data.yandex_compute_instance.vm8.boot_disk[0].disk_id,
             ]

}
```
</details>

## Ansible

`/etc/ansible/`

<details>

*<summary>ansible.cfg</summary>*

``` GO

[defaults]
remote_user = admin
```
</details>

<details>

*<summary>hosts</summary>*

``` GO

[all:vars]
ansible_python_interpreter=/usr/bin/python3

[web_servers]
vm2 ansible_host=192.168.2.15
vm3 ansible_host=192.168.3.4

[prometheus]
vm4 ansible_host=192.168.2.33

[grafana]
vm5 ansible_host=192.168.2.36

[Elasticsearch]
vm6 ansible_host=192.168.2.23

[kibana]
vm7 ansible_host=192.168.2.14

[bastion]
vm8 ansible_host=192.168.2.17
```
</details>

<details>

*<summary>play.yml</summary>*

``` GO

- hosts: web_servers
  become: true
  become_method: sudo
  become_user: root
  remote_user: admin
  roles:
   - role: nginx
   - role: node-exporter
     tags: exporter
   - role: filebeat
     tags: filebeat

  vars:
    nginx_user: www-data

- hosts: prometheus
  user: admin
  become: true
  become_method: sudo
  become_user: root
  roles:
    - role: Prometheus
      tags: prom
- hosts: grafana
  user: admin
  become: true
  become_method: sudo
  become_user: root
  roles:
    - role: Grafana
      tags: graf
- hosts: Elasticsearch
  user: admin
  become: true
  become_method: sudo
  become_user: root
  roles:
    - role: Elasticsearch
      tags: elastic

- hosts: kibana
  user: admin
  become: true
  become_method: sudo
  become_user: root
  roles:
    - role: kibana
      tags: kibana

- hosts: bastion
  user: admin
  become: true
  become_method: sudo
  become_user: root
  roles:
    - role: bastion
      tags: bastion
```
</details>

#### *node-exporter*

`/etc/ansible/roles/node-exporter/`

<details>

*<summary>tasks/main.yml</summary>*

``` GO

---

# tasks file for node-exporter
- name: Install Node_exporter
  include_tasks: tasks/install_node_exporter.yml

- name: Install Nginx Log Exporter
  include_tasks: tasks/install_nginx_log_exporter.yml
```
</details>

<details>

*<summary>tasks/install_node_exporter.yml</summary>*

``` GO

---

- name: Create User node-exporter
  user:
    name: nodeusr
    create_home: no
    shell: /bin/false

- name: Create directories for node-exporter
  file:
    path: "/tmp/node-exporter"
    state: directory
       
- name: Download And Unzipped node-exporter
  unarchive:
    src: https://github.com/prometheus/node_exporter/releases/download/v{{ node_exporter_version }}/node_exporter-{{ node_exporter_version }}.linux-amd64.tar.gz
    dest: /tmp/node-exporter
    creates: /tmp/node-exporter/node-exporter-{{ node_exporter_version }}.linux-amd64
    remote_src: yes

- name: Copy Bin Files From Unzipped to node-exporter
  copy: 
    src: /tmp/node-exporter/node_exporter-{{ node_exporter_version }}.linux-amd64/node_exporter
    dest: /usr/local/bin/
    remote_src: yes
    mode: preserve
    owner: nodeusr
    group: nodeusr

- name: Create File for node-exporter Systemd
  template:
    src=templates/node_exporter.service
    dest=/etc/systemd/system/
  notify:
    - systemd reload

- name: Systemctl node-exporters Start
  systemd:
    name: node_exporter.service
    state: started
    enabled: yes
```
</details>

<details>

*<summary>tasks/install_nginx_log_exporter.yml</summary>*

``` GO

---

- name: Create Directories For nginx-log-exporter
  file:
    path: "/tmp/nginx-log-exporter"
    state: directory
    
- name: Download And Unzipped nginx-log-exporter
  unarchive:
    src: https://github.com/martin-helmich/prometheus-nginxlog-exporter/releases/download/v{{ nginx_log_exporter }}/prometheus-nginxlog-exporter_{{ nginx_log_exporter }}_linux_amd64.tar.gz
    dest: /tmp/nginx-log-exporter
    creates: /tmp/nginx-log-exporter/nginx-log-exporter-{{ nginx_log_exporter }}.linux-amd64
    remote_src: yes

- name: Copy Bin Files From Unzipped to nginx-log-exporter
  copy: 
    src: /tmp/nginx-log-exporter/prometheus-nginxlog-exporter
    dest: /usr/sbin/
    remote_src: yes
    mode: preserve
    owner: nodeusr
    group: nodeusr


- name: Create File for nginx-log-exporter Config
  template:
    src=templates/prometheus-nginxlog-exporter.hcl
    dest=/etc/


- name: Create File for nginx-log-exporter Systemd
  template:
    src=templates/prometheus-nginxlog-exporter.service
    dest=/etc/systemd/system
  notify:
    - systemd reload

- name: Systemctl prometheus-nginxlog-exporter Start
  systemd:
    name: prometheus-nginxlog-exporter
    state: started
    enabled: yes
```
</details>

<details>

*<summary>vars/main.yml</summary>*

``` GO

---

node_exporter_version : 1.6.1
nginx_log_exporter : 1.9.2
```
</details>

#### *nginx*

`/etc/ansible/roles/nginx/`

<details>

/etc/ansible/roles/nginx/

*<summary>tasks/main.yml</summary>*

``` GO

---

- name: Install Nginx Web Server on Debian Family
  apt:
    name=nginx
    state=latest
  when:
    ansible_os_family == "Debian"
  notify:
    - nginx systemd

- name: Replace nginx.conf
  template:
    src=templates/nginx.conf
    dest=/etc/nginx/nginx.conf

- name: Create home directory
  file:
    path: /var/lib/www
    state: directory

- name: copy the nginx config file and restart nginx
  copy:
    src: /etc/ansible/roles/nginx/templates/static_site.cfg
    dest: /etc/nginx/sites-available/static_site.cfg

- name: create symlink
  file:
    src: /etc/nginx/sites-available/static_site.cfg
    dest: /etc/nginx/sites-enabled/default
    state: link

- name: copy the content of the web site
  copy:
    src: /etc/ansible/roles/nginx/static/
    dest: /var/lib/www
```
</details>

<details>

*<summary>vars/main.yml</summary>*

``` GO

---

worker_processes: auto
worker_connections: 2048
client_max_body_size: 512M
```
</details>

<details>

*<summary>static/index.html</summary>*

``` HTML

<center><h1>До новых встреч!</h1><center>
<img src="https://media1.giphy.com/media/Z21HJj2kz9uBG/giphy.gif?cid=ecf05e475ooe8qyeye1vjhbrmdo91n2u3fucegrsxzahnggt&ep=v1_gifs_search&rid=giphy.gif&ct=g" alt="GIF">
```
</details>

#### *prometheus*

/etc/ansible/roles/Prometheus/

<details>

*<summary>tasks/main.yml</summary>*

``` GO

---

- name: Install Prometheus
  include_tasks: tasks/install_prometheus.yml

- name: Install Alertmanager
  include_tasks: tasks/install_alertmanager.yml

```
</details>

<details>

*<summary>tasks/install_prometheus.yml</summary>*

``` GO

---

- name: Create User prometheus
  user:
    name: prometheus
    create_home: no
    shell: /bin/false

- name: Create directories for prometheus
  file:
    path: "{{ item }}"
    state: directory
    owner: prometheus
    group: prometheus
  loop:
    - '/tmp/prometheus'
    - '/etc/prometheus'
    - '/var/lib/prometheus'

- name: Download And Unzipped Prometheus
  unarchive:
    src: https://github.com/prometheus/prometheus/releases/download/v{{ prometheus_version }}/prometheus-{{ prometheus_version }}.linux-amd64.tar.gz
    dest: /tmp/prometheus
    creates: /tmp/prometheus/prometheus-{{ prometheus_version }}.linux-amd64
    remote_src: yes

- name: Copy Bin Files From Unzipped to Prometheus
  copy: 
    src: /tmp/prometheus/prometheus-{{ prometheus_version }}.linux-amd64/{{ item }}
    dest: /usr/local/bin/
    remote_src: yes
    mode: preserve
    owner: prometheus
    group: prometheus
  loop: [ 'prometheus', 'promtool' ]

- name: Copy Conf Files From Unzipped to Prometheus
  copy: 
    src: /tmp/prometheus/prometheus-{{ prometheus_version }}.linux-amd64/{{ item }}
    dest: /etc/prometheus/
    remote_src: yes
    mode: preserve
    owner: prometheus
    group: prometheus
  loop: [ 'console_libraries', 'consoles', 'prometheus.yml' ]

- name: Add template
  template:
    src=templates/prometheus.yml
    dest=/etc/prometheus/prometheus.yml
  notify:
    - systemd reload

- name: Create File for Prometheus Systemd
  template:
    src=templates/prometheus.service
    dest=/etc/systemd/system/
  notify:
    - systemd reload

- name: Systemctl Prometheus Start
  systemd:
    name: prometheus
    state: started
    enabled: yes
```
</details>

<details>

*<summary>tasks/install_alertmanager.yml</summary>*

``` GO

---

- name: Create User Alertmanager
  user:
    name: alertmanager
    create_home: no
    shell: /bin/false

- name: Create Directories For Alertmanager
  file:
    path: "{{ item }}"
    state: directory
    owner: alertmanager
    group: alertmanager
  loop:
    - '/tmp/alertmanager'
    - '/etc/alertmanager'
    - '/var/lib/prometheus/alertmanager'

- name: Download And Unzipped Alertmanager
  unarchive:
    src: https://github.com/prometheus/alertmanager/releases/download/v{{ alertmanager_version }}/alertmanager-{{ alertmanager_version }}.linux-amd64.tar.gz
    dest: /tmp/alertmanager
    creates: /tmp/alertmanager/alertmanager-{{ alertmanager_version }}.linux-amd64
    remote_src: yes

- name: Copy Bin Files From Unzipped to Alertmanager
  copy: 
    src: /tmp/alertmanager/alertmanager-{{ alertmanager_version }}.linux-amd64/{{ item }}
    dest: /usr/local/bin/
    remote_src: yes
    mode: preserve
    owner: alertmanager
    group: alertmanager
  loop: [ 'alertmanager', 'amtool' ]

- name: Copy Conf File From Unzipped to Alertmanager
  copy: 
    src: /tmp/alertmanager/alertmanager-{{ alertmanager_version }}.linux-amd64/alertmanager.yml
    dest: /etc/alertmanager/
    remote_src: yes
    mode: preserve
    owner: alertmanager
    group: alertmanager

- name: Create File for Alertmanager Systemd
  template:
    src=templates/alertmanager.service
    dest=/etc/systemd/system/
  notify:
    - systemd reload

- name: Systemctl Alertmanager Start
  systemd:
    name: alertmanager
    state: started
    enabled: yes
```
</details>

<details>

*<summary>templates/prometheus.yml</summary>*

``` GO

# my global config
global:
  scrape_interval:     15s # Set the scrape interval to every 15 seconds. Default is every 1 minute.
  evaluation_interval: 15s # Evaluate rules every 15 seconds. The default is every 1 minute.
  # scrape_timeout is set to the global default (10s).

# Alertmanager configuration
alerting:
  alertmanagers:
  - static_configs:
    - targets:
      # - alertmanager:9093

# Load rules once and periodically evaluate them according to the global 'evaluation_interval'.
rule_files:
  # - "first_rules.yml"
  # - "second_rules.yml"

# A scrape configuration containing exactly one endpoint to scrape:
# Here it's Prometheus itself.
scrape_configs:
   - job_name: 'node_exporter_clients'
    scrape_interval: 5s
    static_configs:
      - targets:
          - 192.168.3.4:9100
          - 192.168.2.15:9100
    # metrics_path defaults to '/metrics'
    # scheme defaults to 'http'.
   - job_name: 'nginx_exporter'
    scrape_interval: 5s
    static_configs:
      - targets: 
          - 192.168.3.4:4040
          - 192.168.2.15:4040
```
</details>

<details>

*<summary>vars/main.yml</summary>*

``` GO

---

prometheus_version : 2.23.0
alertmanager_version : 0.21.0
```
</details>

#### *grafana*

`/etc/ansible/roles/Grafana/`

<details>

*<summary>tasks/main.yml</summary>*

``` GO

---

- name: Create User grafana
  user:
    name: grafana
    create_home: no
    shell: /bin/false
  
- name: Create directories for Grafana
  file:
    path: "/tmp/grafana"
    state: directory

- name: Download Grafana
  copy:
    src: "/etc/ansible/roles/Grafana/static/grafana_10.0.2_amd64.deb"
    dest: /tmp/grafana

- name: Install Grafana
  apt:
    deb: "/tmp/grafana/grafana_10.0.2_amd64.deb"

- name: Copy template
  copy:
    src: "/etc/ansible/roles/Grafana/templates/main.yml"
    dest: /etc/grafana/provisioning/datasources

- name: Create directories for Dashboards
  file:
    path: "/var/lib/grafana/dashboards"
    state: directory

- name: Copy json
  copy:
    src: "/etc/ansible/roles/Grafana/templates/metrics.json"
    dest: /var/lib/grafana/dashboards
    owner: grafana
    group: grafana
```
</details>

<details>

*<summary>templates/main.yml</summary>*

``` GO

apiVersion: 1
 
datasources:
  - name: Prometheus
    type: prometheus
    version: 1
    access: proxy
    orgId: 1
    basicAuth: false
    editable: false
    url: http://192.168.2.33:9090
```
</details>


#### *elasticsearch*

`/etc/ansible/roles/Elasticsearch/`

<details>

*<summary>tasks/main.yml</summary>*

``` GO

---

- name: Create User elasticsearch
  user:
    name: elasticsearch
    create_home: no
    shell: /bin/false

- name: Create directories for elasticsearch
  file:
    path: "/tmp/elasticsearch"
    state: directory

- name: Download elasticsearch
  copy:
    src: "/etc/ansible/roles/Elasticsearch/static/elasticsearch-8.8.2-amd64.deb"
    dest: /tmp/elasticsearch

- name: Install java
  apt:
    name=default-jre
    state=latest

- name: Install elasticsearch
  apt:
    deb: "/tmp/elasticsearch/elasticsearch-8.8.2-amd64.deb"


- name: Copy CA to ansible
  ansible.builtin.fetch:
    src: /etc/elasticsearch/certs/http_ca.crt
    dest: /etc/ansible/roles/Elasticsearch/static/
```
</details>


#### *filebeat*

`/etc/ansible/roles/filebeat/`

<details>

*<summary>tasks/main.yml</summary>*

``` GO

---

- name: Create directories for filebeat
  file:
    path: "/tmp/filebeat"
    state: directory

- name: Download filebeat
  copy:
    src: "/etc/ansible/roles/filebeat/static/filebeat-8.5.2-amd64.deb"
    dest: /tmp/filebeat

- name: Install filebeat
  apt:
    deb: "/tmp/filebeat/filebeat-8.5.2-amd64.deb"

- name: Copy template
  copy:
    src: "/etc/ansible/roles/filebeat/templates/filebeat.yml"
    dest: /etc/filebeat

- name: Copy module
  copy:
    src: "/etc/ansible/roles/filebeat/templates/nginx.yml"
    dest: /etc/filebeat/modules.d/

- name: Copy ca
  copy:
    src: "/etc/ansible/roles/kibana/static/http_ca.crt"
    dest: /etc/filebeat
```
</details>

<details>

*<summary>templates/filebeat</summary>*

``` GO

---

filebeat.inputs:
- type: log
  enabled: false
  paths:
      - /var/log/nginx/access.log
  fields:
    type: nginx_access
  fields_under_root: true
  scan_frequency: 5s

- type: log
  enabled: false
  paths:
      - /var/log/nginx/error.log
  fields:
    type: nginx_error
  fields_under_root: true
  scan_frequency: 5s

output.elasticsearch:
  hosts: ["https://192.168.2.23:9200"]
  username: "elastic"
  password: "gJ56Irber-Jh8QOio=YS" 
  ssl:
    enabled: true
    certificate_authorities: ["/etc/filebeat/http_ca.crt"]
```
</details>

<details>

*<summary>templates/nginx</summary>*

``` GO

---

- module: nginx
  access:
    enabled: true
    var.paths: ["/var/log/nginx/access.log*"]
  error:
    enabled: true
    var.paths: ["/var/log/nginx/error.log*"
```
</details>

#### *kibana*

`/etc/ansible/roles/kibana/`

<details>

*<summary>tasks/main.yml</summary>*

``` GO

---

- name: Create directories for kibana
  file:
    path: "/tmp/kibana"
    state: directory

- name: Download kibana
  copy:
    src: "/etc/ansible/roles/kibana/static/kibana-8.8.2-amd64.deb"
    dest: /tmp/kibana

- name: Install kibana
  apt:
    deb: "/tmp/kibana/kibana-8.8.2-amd64.deb"

- name: Copy template
  copy:
    src: "/etc/ansible/roles/kibana/templates/kibana.yml"
    dest: /etc/kibana

- name: Create directories for ca
  file:
    path: "/etc/kibana/certs/"
    state: directory

- name: Copy ca
  copy:
    src: "/etc/ansible/roles/kibana/static/http_ca.crt"
    dest: /etc/kibana/certs/
```
</details>

<details>

*<summary>templates/kibana.yml</summary>*

``` GO

# =================== System: Kibana Server ===================
server.host: "0.0.0.0"

# =================== System: Elasticsearch ===================
elasticsearch.hosts: ["https://192.168.2.23:9200"]
elasticsearch.username: "kibana_system"
elasticsearch.password: "2sefwI0SFcHpQYzzevpb"

# =================== System: Elasticsearch (Optional) ===================
elasticsearch.ssl.certificateAuthorities: [ "/etc/kibana/certs/http_ca.crt" ]

# =================== System: Logging ===================
logging:
  appenders:
    file:
      type: file
      fileName: /var/log/kibana/kibana.log
      layout:
        type: json
  root:
    appenders:
      - default
      - file

# =================== System: Other ===================
pid.file: /run/kibana/kibana.pid
```
</details>

#### *bastion*

`/etc/ansible/roles/bastion/`

<details>

*<summary>tasks/main.yml</summary>*

``` GO

---

- name: Copy id_rsa
  copy:
    src: /home/admin/.ssh/id_rsa
    dest: /home/admin/.ssh
    owner: admin
    group: admin
    mode: '0600'

```
</details>