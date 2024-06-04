---
date: "2024-06-04T00:00:00Z"
title: How to automate testing webapps 
---


For today's post I will walk you through the process of setting up a web-app testing environment automatically using ansible.

I will split this into 2 parts. 
First part will focus on creating the automation needed to deploy the containers on proxmox.
Second part is about creating the tester tool solution.

*Assumption:*
* you already have a proxmox server in place.
* your ansible controller has passwordless ssh access enabled to the proxmox server.
* you know what ansible is and how to use it.


ok, so the first step is to install ansible. I recommend doing this in a virtual environment:
```
costin@hp:~$ python -m venv ansible
costin@hp:~$ . ansible/bin/activate
(ansible) costin@hp:~$ pip install ansible
Collecting ansible
  Downloading ansible-6.7.0-py3-none-any.whl (42.8 MB)
     |████████████████████████████████| 42.8 MB 16.4 MB/s
Collecting ansible-core~=2.13.7
  Downloading ansible_core-2.13.13-py3-none-any.whl (2.1 MB)
     |████████████████████████████████| 2.1 MB 23.6 MB/s
Collecting resolvelib<0.9.0,>=0.5.3
  Downloading resolvelib-0.8.1-py2.py3-none-any.whl (16 kB)
Collecting packaging
  Downloading packaging-24.0-py3-none-any.whl (53 kB)
     |████████████████████████████████| 53 kB 3.2 MB/s
Collecting PyYAML>=5.1
  Downloading PyYAML-6.0.1-cp38-cp38-manylinux_2_17_x86_64.manylinux2014_x86_64.whl (736 kB)
     |████████████████████████████████| 736 kB 24.5 MB/s
Collecting jinja2>=3.0.0
  Downloading jinja2-3.1.4-py3-none-any.whl (133 kB)
     |████████████████████████████████| 133 kB 31.6 MB/s
Collecting cryptography
  Downloading cryptography-42.0.7-cp37-abi3-manylinux_2_17_x86_64.manylinux2014_x86_64.whl (3.8 MB)
     |████████████████████████████████| 3.8 MB 13.3 MB/s
Collecting MarkupSafe>=2.0
  Using cached MarkupSafe-2.1.5-cp38-cp38-manylinux_2_17_x86_64.manylinux2014_x86_64.whl (26 kB)
Collecting cffi>=1.12; platform_python_implementation != "PyPy"
  Downloading cffi-1.16.0-cp38-cp38-manylinux_2_17_x86_64.manylinux2014_x86_64.whl (444 kB)
     |████████████████████████████████| 444 kB 33.8 MB/s
Collecting pycparser
  Downloading pycparser-2.22-py3-none-any.whl (117 kB)
     |████████████████████████████████| 117 kB 36.4 MB/s
Installing collected packages: resolvelib, packaging, PyYAML, MarkupSafe, jinja2, pycparser, cffi, cryptography, ansible-core, ansible
Successfully installed MarkupSafe-2.1.5 PyYAML-6.0.1 ansible-6.7.0 ansible-core-2.13.13 cffi-1.16.0 cryptography-42.0.7 jinja2-3.1.4 packaging-24.0 pycparser-2.22 resolvelib-0.8.1
(ansible) costin@hp:~$
```

Next, I've created an ansible inventory for my proxmox server. For this to work, you also need to setup passwordless ssh access based on ssh keys.

```
(ansible) costin@hp:~/ansible/proxmox$ cat inventory.yml
---
all:
  hosts:
    pmox:
      ansible_host: 192.168.0.100
      ansible_user: root
      ansible_port: 22
      
(ansible) costin@hp:~/ansible/proxmox$ ansible -m ping pmox -i inventory.yml
pmox | SUCCESS => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python3"
    },
    "changed": false,
    "ping": "pong"
}
```

Now that we're able to run Ansible code on the proxmox server, next step is to start deploying containers and VMs over it.

For this we'll use a collection: https://docs.ansible.com/ansible/latest/collections/community/general/proxmox_module.html

```
(ansible) costin@hp:~/ansible/proxmox$ ansible-galaxy collection install community.general
Starting galaxy collection install process
Process install dependency map
Starting collection install process
Downloading https://galaxy.ansible.com/api/v3/plugin/ansible/content/published/collections/artifacts/community-general-9.0.1.tar.gz to /home/costin/.ansible/tmp/ansible-local-41867639bgiz5qc/tmpp5j008n7/community-general-9.0.1-mwhmm563
Installing 'community.general:9.0.1' to '/home/costin/.ansible/collections/ansible_collections/community/general'
community.general:9.0.1 was installed successfully
(ansible) costin@hp:~/ansible/proxmox$
```

This collection comes with some requirements on the proxmox server: `proxmoxer` and `requests` which were installed manually on proxmox server using pip.

Created a playbook to deploy a container on Proxmox host:

```
(ansible) costin@hp:~/ansible/proxmox$ cat play-container.yml
---
- hosts: pmox
  tasks:
  - name: Create new container with minimal options
    community.general.proxmox:
      vmid: 200 #CONTAINER_ID
      node: pmox #PROXMOX_SERVER
      api_user: root@pam #PROXMOX_ACCOUNT
      api_password: <PROXMOX_ACCOUNT_PASSWORD>
      api_host: pmox  #PROXMOX_HOST
      password: CTpassword123 #CONTAINER_PASSWORD
      hostname: CT200 #HOSTNAME
      ostemplate: 'local:vztmpl/debian-12-standard_12.2-1_amd64.tar.zst' #CONTAINER_TEMPLATE
      storage: local-lvm #STORAGE
      state: started
```

```
(ansible) costin@hp:~/ansible/proxmox$ ansible-playbook -i inventory.yml play-container.yml

PLAY [pmox] ***************************************************************************************************************

TASK [Gathering Facts] ****************************************************************************************************
ok: [pmox]

TASK [Create new container with minimal options] **************************************************************************
changed: [pmox]

PLAY RECAP ****************************************************************************************************************
pmox                       : ok=2    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
```

Then you should be able to see the newly created CT on proxmox server:
```
root@pmox:~# pct status 200
status: running
```
we can even connect to it using the above defined CONTAINER_PASSWORD:

```
root@pmox:~# pct console 200

Connected to tty 1
Type <Ctrl+a q> to exit the console, <Ctrl+a Ctrl+a> to enter Ctrl+a itself

Debian GNU/Linux 12 CT200 tty1

CT200 login: root
Password:
Linux CT200 6.5.13-5-pve #1 SMP PREEMPT_DYNAMIC PMX 6.5.13-5 (2024-04-05T11:03Z) x86_64

The programs included with the Debian GNU/Linux system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Debian GNU/Linux comes with ABSOLUTELY NO WARRANTY, to the extent
permitted by applicable law.
root@CT200:~#
```

In Part 2 we'll see how to deploy more containers that we'll be using in our webapp tool tester solution.
