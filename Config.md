### Configurations

<details>

<summary>Terraform</summary>

#### /home/admin/terraform/config.tf

``` go
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

<summary>Ansible</summary>
```
hello world!
```

</details>