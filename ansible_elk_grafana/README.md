# FOREMAN

https://www.youtube.com/watch?v=PQYCiJlnpHM<br>
https://www.youtube.com/watch?v=jC0c3kv2ofA<br>
https://theforeman.org/manuals/3.3/index.html<br>
https://github.com/ATIX-AG/foreman_acd<br>
https://groups.google.com/g/ansible-project/c/IJL4SXWPgL8<br>
https://pt.slideshare.net/NikhilKathole/ansible-integration-in-foreman<br>
https://docs.w3cub.com/ansible/collections/theforeman/foreman/index<br>
https://theforeman.org/plugins/foreman_ansible/3.x/index.html#2.Installation<br>
https://www.linuxtechi.com/install-configure-foreman-1-16-debian-9-ubuntu-16-04/<br>
https://www.itzgeek.com/how-tos/linux/centos-how-tos/install-foreman-on-centos-7-rhel-7-ubuntu-14-04-3.html<br>
https://docs.theforeman.org/nightly/Managing_Hosts/index-foreman-el.html#Synchronizing_Templates_Repositories_managing-hosts<br>

**Log Foreman**
```
> /var/log/foreman/production.log
```

**Install/Configure**
```
# foreman-installer --enable-foreman-plugin-{remote-execution,ansible} --enable-foreman-proxy-plugin-{ansible,remote-execution-ssh}
# foreman-installer --enable-foreman-plugin-{remote-execution,ansible} --enable-foreman-proxy-plugin-{ansible,remote-execution-ssh} --foreman-proxy-plugin-remote-execution-ssh-install-key=true (não recomendado em Prod)

(Atenção)
> /etc/ansible/ansible.cfg
> /etc/foreman-proxy/ansible.cfg
> /usr/share/foreman-proxy/.ansible.cfg
```

**Settings (FE)**
```
Settings > Authentication > Trusted hosts
"List of hostnames, IPv4, IPv6 addresses or subnets to be trusted in addition to Smart Proxies for access to fact/report importers and ENC output"
e.g.: mgm-puppetmaster01.manager, mgm-puppetmaster02.manager, mgm-puppetmaster03.manager

Settings > Facts > Create new host when facts are uploaded
"Foreman will create the host when new facts are received"
> "No"

Settings > Remote Execution > SSH User
"Default user to use for SSH. You may override per host by setting a parameter called remote_execution_ssh_user (e.g.: host parameters)."
> ansible

Settings > Remote Execution > Default SSH password
"Default password to use for SSH. You may override per host by setting a parameter called remote_execution_ssh_password (e.g.: host parameters)."
> <ansible_user_password>
```

**Remote Execution SSH Another User (FE)**
```
Edit host parameters:
remote_execution_ssh_user | string | ansible
remote_execution_ssh_password | string | <ansible_user_password> (Somente quando solicita senha do usuario ansible para casos que não enviou a key ao destino)

"The granularity is per host/host group/subnet/domain/os/organization/location"
https://community.theforeman.org/t/remote-execution-ssh-user/6091
```

**Create Ansible User (srv01, srv02)**
```
$ sudo useradd -r -m ansible
$ tr -cd '[:alnum:]' < /dev/urandom | fold -w10 | head -n1 (generate password)
$ sudo passwd ansible

$ sudo sed -i 's/PasswordAuthentication no/PasswordAuthentication yes/g' /etc/ssh/sshd_config
$ sudo bash -c 'echo "AllowUsers root" >> /etc/ssh/sshd_config'
$ sudo bash -c 'echo "AllowUsers ansible" >> /etc/ssh/sshd_config'
$ sudo bash -c 'echo "AllowUsers fabio" >> /etc/ssh/sshd_config'
$ sudo systemctl restart sshd

$ sudo update-alternatives --config editor (Ubuntu Server somente)
$ sudo bash -c 'visudo'
  ansible ALL=(ALL) NOPASSWD:ALL
```

**Import hosts to the Foreman (user foreman-proxy)**
```
# sudo -u foreman-proxy -s /bin/bash

Generate keygen
$ ssh-keygen -t rsa
$ ssh-copy-id -i ~foreman-proxy/.ssh/id_rsa_foreman_proxy.pub ansible@srv01.aut.lab
$ ssh-copy-id -i ~foreman-proxy/.ssh/id_rsa_foreman_proxy.pub ansible@srv02.aut.lab

$ vim ~/inventory
srv01 ansible_host=192.168.0.190
srv02 ansible_host=192.168.0.191

$ vim ~/hosts_setup.yml
---

- hosts: all
  tasks:
    - setup:

...

$ ansible-playbook ~/hosts_setup.yml -u ansible --private-key ~/.ssh/id_rsa_foreman_proxy -i ~/inventory
$ ansible-playbook ~/hosts_setup.yml -u ansible -k -i ~/inventory (Solicita senha do usuario ansible para casos que não enviou a key ao destino)
```

**Install Package**
```
Select Action > Schedule Remote Job
  Job category: Packages
  Job template: Package Action - SSH Default
  Action: install
  Package: nginx
```

**Job Templates**
https://www.youtube.com/watch?v=jC0c3kv2ofA<br>
```
Hosts > Templates > Job Templates
  Template
    Name: SAPO - SSH Install Package
    Editor:
      sudo apt install -y <%= input('package') %>
      sudo apt list --installed | grep <%= input('package') %>

  Inputs
    Name: package
    Required: Yes
    Input Type: User input
    Value Type: Plain
    Description: Name of the package to be installed

  Job
    Job Category: SAPO - SSH Install Package
    Provider Type: SSH
    Overridable: Yes
```

```
  Template
    Name: SAPO - Ansible Install Package
    Editor:

      ---

      - name: "UBUNTU PLAYBOOK INSTALL PACKAGE"
        hosts: all
        become: true
        tasks:
          - name: UBUNTU | Apt Update
            ansible.builtin.shell: "apt update"
            when:
            - (ansible_facts['distribution'] is defined and ansible_facts['distribution'] == 'Ubuntu') and (ansible_facts['distribution_major_version'] is defined and ansible_facts['distribution_major_version'] == '20')

          - name: UBUNTU | Install package {{ package }}
            ansible.builtin.apt:
              name: "{{ package }}"
              state: latest
            when:
            - (ansible_facts['distribution'] is defined and ansible_facts['distribution'] == 'Ubuntu') and (ansible_facts['distribution_major_version'] is defined and ansible_facts['distribution_major_version'] == '20')

          - name: UBUNTU | Check version installed
            ansible.builtin.shell: "apt list --installed | grep {{ package }}"
            register: packageversion
            when:
            - (ansible_facts['distribution'] is defined and ansible_facts['distribution'] == 'Ubuntu') and (ansible_facts['distribution_major_version'] is defined and ansible_facts['distribution_major_version'] == '20')

            - name: UBUNTU | Show {{ package }} Version
              debug:
                msg: "{{ package }} Version is {{ packageversion.stdout }}"
            when:
            - (ansible_facts['distribution'] is defined and ansible_facts['distribution'] == 'Ubuntu') and (ansible_facts['distribution_major_version'] is defined and ansible_facts['distribution_major_version'] == '20')

      - name: "RHEL PLAYBOOK INSTALL PACKAGE"
        hosts: all
        become: true
        tasks:

          - name: RHEL | Install package {{ package }}
            ansible.builtin.yum:
              name: "{{ package }}"
              state: latest
            when:
            - (ansible_facts['distribution'] is defined and ansible_facts['distribution'] == 'RedHat') and (ansible_facts['distribution_major_version'] is defined and ansible_facts['distribution_major_version'] == '7.8')

          - name: RHEL | Check version installed
            ansible.builtin.shell: " rpm -qa {{ package }}"
            register: packageversion
            when:
            - (ansible_facts['distribution'] is defined and ansible_facts['distribution'] == 'RedHat') and (ansible_facts['distribution_major_version'] is defined and ansible_facts['distribution_major_version'] == '7.8')

            - name: RHEL | Show {{ package }} Version
              debug:
                msg: "{{ package }} Version is {{ packageversion.stdout }}"
            when:
            - (ansible_facts['distribution'] is defined and ansible_facts['distribution'] == 'RedHat') and (ansible_facts['distribution_major_version'] is defined and ansible_facts['distribution_major_version'] == '7.8')

      ...

  Inputs
    Name: package
    Required: Yes
    Input Type: User input
    Value Type: Plain
    Description: Name of the package to be installed

  Job
    Job Category: SAPO - Ansible Install Package
    Provider Type: Ansible
    Overridable: Yes
```

```
Hosts > Templates > Job Templates > "Run"
```

**Import roles (CLI & FE)**
```
# git clone https://github.com/fabiokerber/Ansible.git /tmp/Ansible
# cp -r /tmp/Ansible/Playbooks/install_package /etc/ansible/roles

Menu > Configure > Ansible Roles
"Import from foreman.aut.lab" & Select & "Submit"

Foreman Command:
# rm -rf /etc/ansible/roles/* && rm -rf /tmp/Ansible && git clone https://github.com/fabiokerber/Ansible.git /tmp/Ansible && cp -r /tmp/Ansible/Playbooks/install_package /etc/ansible/roles
$ ansible-playbook /etc/ansible/roles/install_package/playbook.yml -u ansible -k -i ~/inventory
```

**API (FE)**
```
https://192.168.0.180/api/hosts
```

**Hammer (user foreman-proxy)**
```
# sudo -u foreman-proxy -s /bin/bash

Foreman user list:
$ hammer user list

Foreman list architecture
$ hammer architecture list
```

**Foreman Services Status**
```
# foreman-maintain service status -b
 - Verificar comando
```

**Cockpit**
```
https://docs.theforeman.org/nightly/Managing_Hosts/index-foreman-el.html
https://cockpit-project.org/running.html
```

|**Port**|**Protocol**|**Required For**|
|-------------|-----------|-----------|
|`53`| TCP & UDP | DNS Server |
|`67, 68`| UDP | DHCP Server |
|`69`| UDP | TFTP Server |
|`80, 443`| TCP | * HTTP & HTTPS access to Foreman web UI / provisioning templates - using Apache |
|`3000`| TCP | HTTP access to Foreman web UI / provisioning templates - using standalone WEBrick service |
|`5910 - 5930`| TCP | Server VNC Consoles |
|`5432`| TCP | Separate PostgreSQL database |
|`8140`| TCP | * Puppet server |
|`8443`| TCP | Smart Proxy, open only to Foreman |

*Ports indicated with * are running by default on a Foreman all-in-one installation and should be open.*<br>