Содержание
==========
* [Terraform](#Terraform)
* [Ansible](#Ansible)
* [NEW](#NEW)

---------
### Terraform

<details>

*<summary>/home/admin/terraform/metadata.txt</summary>*

``` GO

#cloud-config
users:
  - name: admin
    groups: sudo
    shell: /bin/bash
    sudo: ['ALL=(ALL) NOPASSWD:ALL']
    ssh-authorized-keys:
      - ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABgQDpOo0+4sY1ac9Ki86DIdstDUaAskdZaOFv80gSdo2ohseAXTK+8lbYMMrxVOEtWyJqu9N0Pu0KC8Dyi05QOJxCvqrbIEjWc7RhLyUEQJTM7LXWo1z7gCEZo5Je7z2v1wM3z120WjSAdPx3Bg9jqbCUKofzpWSU9TtZ+nFCM1elSU3NFwQCbtC+o8IOjg5Q/tGBxTAgkuZtvoI5+O+CFYYr3SQ4Qiu3w1UUcXHYaWw2UEijS/rfnOCcWy+kWLx+obni+iKmOq5sOzvvGXOMpLcf08U1EIidHUKwOBAaG1DotI5u+cC/5t1xSb/ODo5Np4tnTyFStPSPyFal8fWLI1Vw4dKhntA5xDZtjmMwM3lxZJOWnAtSHZ8RpMDbuWDWWfpzElkU3EnA4fWknQmz1bXXOjFNmwiujK9xQsnSQ4SZBUehdewz269TAdg6HiFdOzQbZUmYTQyHfzjmk2tpd7eTCdO4tXrkL7ooJCXBRzrtdhHFDBapZvxzWta4KJ5pk7M= admin@terraform
```
</details>

<details>

*<summary>/home/admin/terraform/config.tf</summary>*

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
    nat       = true
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
    nat       = true
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
    nat       = true

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
    nat       = true
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

*<summary>/home/admin/terraform/targetgroup/config.tf</summary>*

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

*<summary>/home/admin/terraform/backendgroup/config.tf</summary>*

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

*<summary>/home/admin/terraform/router/config.tf</summary>*

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

*<summary>/home/admin/terraform/balancer/config.tf</summary>*

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

*<summary>/home/admin/terraform/security/config.tf</summary>*

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

*<summary>/home/admin/terraform/snapshots/config.tf</summary>*

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
    expression = "0 9 * * *"
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

### Ansible

<details>

*<summary>/etc/ansible/ansible.cfg</summary>*

``` GO

[defaults]
remote_user = user
```
</details>

<details>

*<summary>/etc/ansible/hosts</summary>*

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

*<summary>/etc/ansible/play.yml</summary>*

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

### NEW

<details>

*<summary>NEW</summary>*

``` GO

hello world!
```
</details>

<details>

*<summary>NEW</summary>*

``` GO

hello world!
```
</details>
