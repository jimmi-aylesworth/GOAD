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

```powershell
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
