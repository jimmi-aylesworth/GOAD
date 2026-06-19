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

## GOAD - Pre-flight Check and Install

```text
git clone https://github.com/Orange-Cyberdefense/GOAD.git
cd C:\GOAD														# switch to GOAD folder
. \.venv\Scripts\Activate.ps1									# activate venv vars
pip install -r .\noansible_requirements.yml						# ONLY run if installing (first time)
python .\goad.py -d ludus -t check -p virtualbox				# Check Pre-reqs
python .\goad.py -d ludus -t install -p virtualbox -m vm		# Install (VM is Virtual Machine; Not VMWare)
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

If you're still having issues with server connections timing out, specifically "fatal: [srv02]: UNREACHABLE! => {"changed": false, "msg": "ssl: Server execution failed  (extended fault data: {'transport_message': 'Bad HTTP response returned from server. Code 500', 'http_status_code': 500, 'wsmanfault_code': 2148007941, 'fault_code': 's:Receiver', 'fault_subcode': 'w:InternalError'})", "unreachable": true}", check the `C:\Users\stops\GOAD\ad\GOAD\data\inventory` file and remove the comments on following lines (~35-36)

```text
ansible_winrm_transport=basic
ansible_port=5985
```

After editing the file, you'll need to push the updated file to the provisioning box via `sync_source_jumpbox`.
