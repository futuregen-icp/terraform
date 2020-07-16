## erraform install

```
https://www.terraform.io/downloads.html

wget https://releases.hashicorp.com/terraform/0.12.28/terraform_0.12.28_linux_amd64.zip

unzip terraform_0.12.28_linux_amd64.zip

mv terraform /usr/local/bin/
```


## Verify the installation

```
[root@terraform ~]# terraform --help
Usage: terraform [-version] [-help] <command> [args]

The available commands for execution are listed below.
The most common, useful commands are shown first, followed by
less common or more advanced commands. If you're just getting
started with Terraform, stick with the common commands. For the
other commands, please read the help and docs before usage.

Common commands:
    apply              Builds or changes infrastructure
    console            Interactive console for Terraform interpolations
    destroy            Destroy Terraform-managed infrastructure
    env                Workspace management
    fmt                Rewrites config files to canonical format
    get                Download and install modules for the configuration
    graph              Create a visual graph of Terraform resources
    import             Import existing infrastructure into Terraform
    init               Initialize a Terraform working directory
    login              Obtain and save credentials for a remote host
    logout             Remove locally-stored credentials for a remote host
    output             Read an output from a state file
    plan               Generate and show an execution plan
    providers          Prints a tree of the providers used in the configuration
    refresh            Update local state file against real resources
    show               Inspect Terraform state or plan
    taint              Manually mark a resource for recreation
    untaint            Manually unmark a resource as tainted
    validate           Validates the Terraform files
    version            Prints the Terraform version
    workspace          Workspace management

All other commands:
    0.12upgrade        Rewrites pre-0.12 module source code for v0.12
    debug              Debug output management (experimental)
    force-unlock       Manually unlock the terraform state
    push               Obsolete command for Terraform Enterprise legacy (v1)
    state              Advanced state management
[root@terraform ~]# terraform --help plan
Usage: terraform plan [options] [DIR]

  Generates an execution plan for Terraform. 

  ...


  ## Enable tab completion
     (sh reload 필요 )
  terraform -install-autocomplete
```



## libvirt plugin install

```
[root@terraform ~]# mkdir /root/terraform
[root@terraform ~]# mkdir /root/terraform/libvirt

[root@terraform ~]# terraform init
Terraform initialized in an empty directory!

The directory has no Terraform configuration files. You may begin working
with Terraform immediately by creating Terraform configuration files.

<<https://github.com/dmacvicar/terraform-provider-libvirt/releases>>
버전에 맞는 provider를 다운로드 

[root@terraform ~]# wget https://github.com/dmacvicar/terraform-provider-libvirt/releases/download/v0.6.2/terraform-provider-libvirt-0.6.2+git.1585292411.8cbe9ad0.Fedora_28.x86_64.tar.gz 

[root@terraform ~]# tar zxvfp terraform-provider-libvirt-0.6.2+git.1585292411.8cbe9ad0.Fedora_28.x86_64.tar.gz
terraform-provider-libvirt

[root@terraform ~]# cp -Rp terraform-provider-libvirt /root/.terraform.d/plugins/terraform-provider-libvirt
```

테라폼 메인 파일 생성

```
[root@terraform libvirt]# vi main.tf
#provider "libvirt" {
#  uri = "qemu:///system"
#}
#
provider "libvirt" {
  uri   = "qemu+ssh://root@192.168.1.100/system"
}

resource "libvirt_volume" "centos77-qcow2" {
  name = "centos77.qcow2"
  pool = "pool-SATA"
  source = "https://cloud.centos.org/centos/7/images/CentOS-7-x86_64-GenericCloud.qcow2"
  #source = "./CentOS-7-x86_64-GenericCloud.qcow2"
  format = "qcow2"
}

# Define KVM domain to create
resource "libvirt_domain" "db1" {
  name   = "db1"
  memory = "1024"
  vcpu   = 1

  network_interface {
    network_name = "NAT`"
  }

  disk {
    volume_id = libvirt_volume.centos77-qcow2.id
  }

  console {
    type = "pty"
    target_type = "serial"
    target_port = "0"
  }

  graphics {
    type = "spice"
    listen_type = "address"
    autoport = true
  }
}
```

## 테라폼 적용을 위한 초기화

```
[root@terraform libvirt]# terraform init

Initializing the backend...

Initializing provider plugins...

Terraform has been successfully initialized!

You may now begin working with Terraform. Try running "terraform plan" to see
any changes that are required for your infrastructure. All Terraform commands
should now work.

If you ever set or change modules or backend configuration for Terraform,
rerun this command to reinitialize your working directory. If you forget, other
commands will detect it and remind you to do so if necessary. 
```


## 테라폼 적용하여 vm 생성하기 위한 플랜 확인 

```
[root@terraform libvirt]# terraform plan
Refreshing Terraform state in-memory prior to plan...
The refreshed state will be used to calculate this plan, but will not be
persisted to local or remote state storage.


------------------------------------------------------------------------

An execution plan has been generated and is shown below.
Resource actions are indicated with the following symbols:
  + create

Terraform will perform the following actions:

  # libvirt_domain.db1 will be created
  + resource "libvirt_domain" "db1" {
      + arch        = (known after apply)
      + disk        = [
          + {
              + block_device = null
              + file         = null
              + scsi         = null
              + url          = null
              + volume_id    = (known after apply)
              + wwn          = null
            },
        ]
      + emulator    = (known after apply)
      + fw_cfg_name = "opt/com.coreos/config"
      + id          = (known after apply)
      + machine     = (known after apply)
      + memory      = 1024
      + name        = "db1"
      + qemu_agent  = false
      + running     = true
      + vcpu        = 1

      + console {
          + source_host    = "127.0.0.1"
          + source_service = "0"
          + target_port    = "0"
          + target_type    = "serial"
          + type           = "pty"
        }

      + graphics {
          + autoport       = true
          + listen_address = "127.0.0.1"
          + listen_type    = "address"
          + type           = "spice"
        }

      + network_interface {
          + addresses    = (known after apply)
          + hostname     = (known after apply)
          + mac          = (known after apply)
          + network_id   = (known after apply)
          + network_name = "NAT`"
        }
    }

  # libvirt_volume.centos77-qcow2 will be created
  + resource "libvirt_volume" "centos77-qcow2" {
      + format = "qcow2"
      + id     = (known after apply)
      + name   = "centos77.qcow2"
      + pool   = "pool-SATA"
      + size   = (known after apply)
      + source = "https://cloud.centos.org/centos/7/images/CentOS-7-x86_64-GenericCloud.qcow2"
    }

Plan: 2 to add, 0 to change, 0 to destroy.

------------------------------------------------------------------------

Note: You didn't specify an "-out" parameter to save this plan, so Terraform
can't guarantee that exactly these actions will be performed if
"terraform apply" is subsequently run.
```


## 테라폼 적용하여 vm 생성하기

```
[root@terraform libvirt]# terraform apply
libvirt_volume.centos77-qcow2: Refreshing state... [id=/VMS/sata-3t/VMS/centos77.qcow2]

An execution plan has been generated and is shown below.
Resource actions are indicated with the following symbols:
  + create

Terraform will perform the following actions:

  # libvirt_domain.db1 will be created
  + resource "libvirt_domain" "db1" {
      + arch        = (known after apply)
      + disk        = [
          + {
              + block_device = null
              + file         = null
              + scsi         = null
              + url          = null
              + volume_id    = "/VMS/sata-3t/VMS/centos77.qcow2"
              + wwn          = null
            },
        ]
      + emulator    = (known after apply)
      + fw_cfg_name = "opt/com.coreos/config"
      + id          = (known after apply)
      + machine     = (known after apply)
      + memory      = 1024
      + name        = "db1"
      + qemu_agent  = false
      + running     = true
      + vcpu        = 1

      + console {
          + source_host    = "127.0.0.1"
          + source_service = "0"
          + target_port    = "0"
          + target_type    = "serial"
          + type           = "pty"
        }

      + graphics {
          + autoport       = true
          + listen_address = "127.0.0.1"
          + listen_type    = "address"
          + type           = "spice"
        }

      + network_interface {
          + addresses    = (known after apply)
          + hostname     = (known after apply)
          + mac          = (known after apply)
          + network_id   = (known after apply)
          + network_name = "NAT"
        }
    }

Plan: 1 to add, 0 to change, 0 to destroy.

Do you want to perform these actions?
  Terraform will perform the actions described above.
  Only 'yes' will be accepted to approve.

  Enter a value: yes

libvirt_domain.db1: Creating...
libvirt_domain.db1: Creation complete after 2s [id=85e556cd-abbc-46e7-bf9b-902fbe5cc53e]

Apply complete! Resources: 1 added, 0 changed, 0 destroyed.
```

## KVM HOST Server 

```
virsh # list --all
 Id   이름             상태
-------------------------------
...
 14   rhel8.0          실행중
 15   db1              실행중
 -    centos7.0        종료
... 
```
