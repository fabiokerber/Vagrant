# 01.Alura

### Vagrant & Puppet & Ansible
<br />

**Start**

*PowerShell - precise (Ubuntu 12.04)*
```
> vagrant version
> vagrant init hashicorp/precise64 
> vagrant up 
> vagrant status
> vagrant halt
> vagrant suspend
> vagrant reload
> vagrant ssh (u vagrant / s vagrant - default)
> vagrant ssh-config
```
<br />

**Port Forwarding**

*PowerShell - bionic (Ubuntu 18.04.6)*
```
!!!! edit Vagranfile !!!!
    Vagrant.configure("2") do |config|
        config.vm.box = "ubuntu/bionic64"
    end

> vagrant up
> vagrant ssh
    $ sudo apt update
    $ sudo apt install -y nginx
    $ netstat -lntp
    $ curl http://localhost
    $ exit
> vagrant halt

!!!! edit Vagranfile !!!!
    config.vm.network "forwarded_port", guest: 80, host:8089

> vagrant up

!!!! navegador http://localhost:8089 !!!!
```
<br />

**Private Network**

*PowerShell - bionic (Ubuntu 18.04.6)*
```
!!!! edit Vagranfile !!!!
    config.vm.network "private_network", ip: "192.168.50.4"

> vagrant up / vagrant reload
> vagrant ssh
```
<br />

**Public Network (Bridge)**

*PowerShell - bionic (Ubuntu 18.04.6)*
```
!!!! edit Vagranfile !!!!
    config.vm.network "public_network"

> vagrant up / vagrant reload
> vagrant ssh

!!!! edit Vagranfile !!!!
    config.vm.network "public_network", ip: "192.168.0.50"

> vagrant up / vagrant reload
> vagrant ssh
```
<br />

**Connecting by SSH (Private Key)**

*PowerShell - bionic (Ubuntu 18.04.6)*
```
> vagrant ssh-config
    IdentityFile ...

>  ssh -i D:/git_projects/Vagrant/1.Alura/bionic/.vagrant/machines/default/virtualbox/private_key vagrant@192.168.0.50
```
<br />

**Add SSH Key to VM(Private Key)**

*PowerShell - bionic (Ubuntu 18.04.6)*
```
> ssh-keygen -t rsa
    D:/git_projects/Vagrant/1.Alura/bionic/id_bionic

> vagrant ssh
    $ ls /vagrant
    $ cp /vagrant/id_bionic.pub ~
    $ cat /vagrant/id_bionic.pub >> .ssh/authorized_keys
```
<br />

|Tools      |Links/Tips|
|-------------|-----------|
|`Vagrant`| https://www.vagrantup.com/downloads
|`Virtualbox`| https://www.virtualbox.org/wiki/Downloads
|`PowerShell`| Set-PSReadlineOption -BellStyle None

