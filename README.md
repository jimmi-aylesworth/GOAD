# GOAD

* GOAD Repo: https://github.com/Orange-Cyberdefense/GOAD
* GOAD Docs: https://orange-cyberdefense.github.io/GOAD

## Virtualbox & Vagrant Setup

I choose to utilize Virtualbox as my hypervisor.
To create the lab on a windows computer you will need C++ runtimes and Vagrant. 
Vagrant will be leveraged to automate the process of vm download and creation.

* Download and install
  * virtualbox: https://www.virtualbox.org/wiki/Downloads
  * visual c++ 2019 : https://aka.ms/vs/17/release/vc_redist.x64.exe
  * vagrant : https://developer.hashicorp.com/vagrant/install
  * vagrant plugins: `vagrant.exe plugin install vagrant-reload vagrant-vbguest winrm winrm-fs winrm-elevated`

---

Make your life easier, and update the following as needed.

### GOAD\globalsettings.ini

```text
[all:vars]
; This is the global inventory file, data here will override all lab or provider inventory datas
; modify this to add layouts to VMs
; https://learn.microsoft.com/en-us/windows-hardware/manufacture/desktop/windows-language-pack-default-values
; French  : 0000040C
; US      : 00000409
; German  : 00000407
; Spanish : 0000040A
; the first in the list will be the default layout (here: FR | US)
keyboard_layouts=["0000040C", "00000409"]

; Uncoment to not use SSL in ansible (usefull if you get Digest initialization failed: initialization error with vagrant)
ansible_winrm_transport=basic
ansible_port=5985

; modify this to add a default route
add_route=no
route_gateway=192.168.56.1
route_network=10.0.0.0/8

; modify this to enable http proxy
enable_http_proxy=no
proxy_ip=x.x.x.x
proxy_port=8080
ad_http_proxy="http://{{proxy_ip}}:{{proxy_port}}"
ad_https_proxy="http://{{proxy_ip}}:{{proxy_port}}"

; dns server fallback forwarder
;dns_server_forwarder=1.1.1.1

[default]
; lab: goad / goad-light / minilab / nha / sccm
lab = GOAD
; provider : virtualbox / vmware / aws / azure / proxmox
provider = virtualbox
; provisioner method : local / remote
provisioner = local
; ip_range (3 first ip digits)
ip_range = 192.168.56
```

### $HOME\.goad\goad.ini

```text
[default]
; lab: goad / goad-light / minilab / nha / sccm
lab = GOAD
; provider : virtualbox / vmware / aws / azure / proxmox
provider = virtualbox
; provisioner method : local / remote
provisioner = local
; ip_range (3 first ip digits)
ip_range = 192.168.56
```

---

## GOAD - Pre-flight Check and Install

```text
git clone https://github.com/Orange-Cyberdefense/GOAD.git
cd C:\GOAD														# switch to GOAD folder
. \.venv\Scripts\Activate.ps1									# activate venv vars
pip install -r .\noansible_requirements.yml						# ONLY run if installing (first time)
# Check Pre-reqs
python .\goad.py -d ludus -t check -p virtualbox -d vmware -d azure -d aws -d proxmox -d remove -d docker
# Install (VM is Virtual Machine; Not VMWare)
python .\goad.py -d ludus -t install -p virtualbox -d vmware -d azure -d aws -d proxmox -d remove -d docker
```

Answer **y** to if you want to install and you're good to go.

---

- Workspaces are located within the _workspace_ folder of the GOAD install - i.e. `C:\\GOAD\workspace`
- GOAD configuration file is located at `C:\.goad\goad.ini`

---

## Troubleshooting

> In most case if you get errors during install, don't think. Select the failed instance ̀load <instance_id> and just replay the install with provision_lab to relaunch all or provision_lab_from <playbook> if you know the last failed playbook (most of the errors which could came up are due to windows latency during installation, wait few minutes and replay the install)

```text
GOAD/vmware/local/192.168.56.X > instances
┏━━━━━━━━━━━━━━━━━━━━━━━━┳━━━━━━┳━━━━━━━━━━━━┳━━━━━━━━━━━━━━━━━┳━━━━━━━━━━━━━━━━━━━━━━━━┳━━━━━━━━━━━━┳━━━━━━━━━━━━┓
┃ Instance ID            ┃ Lab  ┃ Provider   ┃ IP Range        ┃ Status                 ┃ Is Default ┃ Extensions ┃
┡━━━━━━━━━━━━━━━━━━━━━━━━╇━━━━━━╇━━━━━━━━━━━━╇━━━━━━━━━━━━━━━━━╇━━━━━━━━━━━━━━━━━━━━━━━━╇━━━━━━━━━━━━╇━━━━━━━━━━━━┩
│ 9f1f6f-goad-virtualbox │ GOAD │ virtualbox │ 192.168.56.0/24 │ ready for provisioning │ No         │            │
└────────────────────────┴──────┴────────────┴─────────────────┴────────────────────────┴────────────┴────────────┘
GOAD/vmware/local/192.168.56.X > load 9f1f6f-goad-virtualbox
GOAD/virtualbox/vm/192.168.56.X (9f1f6f-goad-virtualbox) > provision_lab
```

If you're still having issues with server connections timing out, specifically "fatal: [srv02]: UNREACHABLE! => {"changed": false, "msg": "ssl: Server execution failed  (extended fault data: {'transport_message': 'Bad HTTP response returned from server. Code 500', 'http_status_code': 500, 'wsmanfault_code': 2148007941, 'fault_code': 's:Receiver', 'fault_subcode': 'w:InternalError'})", "unreachable": true}", and you didn't use the above settings files, check the `C:\Users\stops\GOAD\ad\GOAD\data\inventory` file and remove the comments on following lines (~35-36). Make sure to push the settings to the provisioning box.

```text
ansible_winrm_transport=basic
ansible_port=5985
```

After editing the file, you'll need to push the updated file to the provisioning box via `sync_source_jumpbox`.


