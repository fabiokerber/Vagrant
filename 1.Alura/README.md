# 1.Alura

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
> vagrant destroy
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

**Shell Provisioner**

*PowerShell - bionic (Ubuntu 18.04.6)*
```
!!!! edit Vagranfile !!!!
    config.vm.provision "shell", inline: "echo Hello, World >> hello.txt"

> vagrant up / vagrant reload (Necessário forçar com o comando abaixo, pois o provision é somente no ato da criação)
> vagrant provision

> vagrant ssh
    $ cat hello.txt

```
<br />

**Synced Folder & +Shell**

*PowerShell - bionic (Ubuntu 18.04.6)*
```
!!! create folder configs !!!
!!! copy id_bionic.pub to configs !!!

!!!! edit Vagranfile !!!!
    config.vm.synced_folder "./configs", "/configs" (Mapeia pasta configs dentro de bionic em /configs)
    config.vm.synced_folder ".", "/vagrant", disabled: true (Desabilita o mapeamento do conteúdo da pasta padrão "bionic")

> vagrant destroy -f
> vagrant up

```
<br />

**MySQL Provisioning**

*PowerShell - bionic (Ubuntu 18.04.6)*
```
!!!! edit Vagranfile !!!!
    $script_mysql = <<-SCRIPT
        apt-get update && \
        apt-get install -y mysql-server-5.7 && \
        mysql -e "create user 'phpuser'@'%' identified by 'pass';"
    SCRIPT

    config.vm.provision "shell", inline: $script_mysql

> vagrant destroy -f
> vagrant up
> vagrant ssh
    $ sudo mysql
    mysql> select user from mysql.user
    mysql> ;
    $ cat /etc/mysql/mysql.conf.d/mysqld.cnf >> /configs/mysqld.cnf

!!!! /configs/mysqld.cnf !!!!
    bind-address		= 0.0.0.0 (Permitir conexão externa a partir de qualquer host)

!!!! edit Vagranfile !!!!
    config.vm.provision "shell", inline: "cat /configs/mysqld.cnf > /etc/mysql/mysql.conf.d/mysqld.cnf"
    config.vm.provision "shell", inline: "service mysql restart"

> vagrant destroy -f
> vagrant up
```
<br />

**Multi-Machine**

*PowerShell - bionic (Ubuntu 18.04.6)*
```
!!!! edit Vagranfile !!!!
    $script_mysql = <<-SCRIPT
        apt-get update && \
        apt-get install -y mysql-server-5.7 && \
        mysql -e "create user 'phpuser'@'%' identified by 'pass';"
    SCRIPT

    Vagrant.configure("2") do |config|
        config.vm.box = "ubuntu/bionic64"

        config.vm.define "mysqldb" do |mysql|

            mysql.vm.network "public_network", ip: "192.168.0.50"

            mysql.vm.provision "shell", inline: "cat /configs/id_bionic.pub >> .ssh/authorized_keys"
            mysql.vm.provision "shell", inline: $script_mysql
            mysql.vm.provision "shell", inline: "cat /configs/mysqld.cnf > /etc/mysql/mysql.conf.d/mysqld.cnf"
            mysql.vm.provision "shell", inline: "service mysql restart"

            mysql.vm.synced_folder "./configs", "/configs"
            mysql.vm.synced_folder ".", "/vagrant", disabled: true

        end

        config.vm.define "phpweb" do |phpweb|

            phpweb.vm.network "forwarded_port", guest:80, host:8089
            phpweb.vm.network "public_network", ip: "192.168.0.100"

            phpweb.vm.provision "shell", inline: apt update && apt install -y puppet

        end

    end

> vagrant destroy -f
> vagrant up
> vagrant ssh phpweb
> vagrant ssh mysqldb
```
<br />

**Multi-Machine**

*PowerShell - bionic (Ubuntu 18.04.6)*
```
!!! create folder config/manifests !!!
!!!! edit config/manifests/phpweb.pp !!!!
    exec { 'apt-update':
      command => '/usr/bin/apt-get update' 
    }

    package { ['php7.2' ,'php7.2-mysql'] :
      require => Exec['apt-update'],
      ensure => installed,
    }

    exec { 'run-php7':
      require => Package['php7.2'],
      command => '/usr/bin/php -S 192.168.1.25:8888 -t /vagrant/src &'
    }



```
<br />

|Tools      |Links/Tips|
|-------------|-----------|
|`Vagrant Downloads`| https://www.vagrantup.com/downloads
|`Vagrant Docs`| https://www.vagrantup.com/docs
|`Virtualbox Downloads`| https://www.virtualbox.org/wiki/Downloads
|`Puppet Lamp`| https://www.digitalocean.com/community/tutorials/getting-started-with-puppet-code-manifests-and-modules
|`PowerShell`| Set-PSReadlineOption -BellStyle None
