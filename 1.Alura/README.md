# 01.Alura
## Vagrant & Puppet & Ansible
### Início

PowerShell
```
> vagrant version
> cd precise
> vagrant init hashicorp/precise64 
> vagrant up (Ubuntu 12.04)
> vagrant status
> vagrant halt
> vagrant suspend
> vagrant reload
> vagrant ssh (u vagrant / s vagrant - default)
> vagrant ssh-config
```

### Port Forwarding

PowerShell
```
> cd bionic
> vagrant up (Ubuntu 18.04.2)
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

### Private Network

PowerShell
```
> cd bionic

!!!! edit Vagranfile !!!!
    config.vm.network "private_network", ip: "192.168.50.4"

> vagrant up / vagrant reload (Ubuntu 18.04.2)
```

|Requisitos      |Links|
|-------------|-----------|
|`Vagrant`| https://www.vagrantup.com/downloads
|`Virtualbox`| https://www.virtualbox.org/wiki/Downloads
