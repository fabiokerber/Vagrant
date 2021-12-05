# 01.Alura
## Vagrant & Puppet & Ansible

[^1]: Início

PowerShell - precise (Ubuntu 12.04)
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

### Port Forwarding

PowerShell - bionic (Ubuntu 18.04.6)
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

### Private Network

PowerShell - bionic (Ubuntu 18.04.6)
```
!!!! edit Vagranfile !!!!
    config.vm.network "private_network", ip: "192.168.50.4"

> vagrant up / vagrant reload
> vagrant ssh
```

### Public Network (Bridge)

PowerShell - bionic (Ubuntu 18.04.6)
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

|Tools      |Links|
|-------------|-----------|
|`Vagrant`| https://www.vagrantup.com/downloads
|`Virtualbox`| https://www.virtualbox.org/wiki/Downloads
