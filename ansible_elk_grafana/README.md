# ARA

```
Checar configs
# python3 -m ara.setup.action_plugins
# python3 -m ara.setup.callback_plugins

# vim /root/.ara/server/settings.yaml

default:
  ALLOWED_HOSTS:
  - ::1
  - 127.0.0.1
  - localhost
  - 192.168.0.16
  BASE_DIR: /root/.ara/server
  CORS_ORIGIN_ALLOW_ALL: false
  CORS_ORIGIN_REGEX_WHITELIST: []
  CORS_ORIGIN_WHITELIST:
  - http://127.0.0.1:8000
  - http://localhost:3000
  CSRF_TRUSTED_ORIGINS: []
  DATABASE_CONN_MAX_AGE: 0
  DATABASE_ENGINE: django.db.backends.sqlite3
  DATABASE_HOST: null
  DATABASE_NAME: /root/.ara/server/ansible.sqlite
  DATABASE_OPTIONS: {}
```

```
# ara-manage runserver

# ara result list

# ansible-playbook /vagrant/playbook.yaml
```

https://github.com/ansible-community/ara-collection<br>
https://speakerdeck.com/tonk/ara-on-rhel7-welcome-to-hell?slide=12<br>
http://tonkersten.com/files/makeara<br>
https://ara.readthedocs.io/en/latest/api-security.html<br>
https://sebiwi.github.io/blog/ara/<br>
https://ara.readthedocs.io/en/stable-0.x/webserver.html<br>
https://github.com/ansible-community/ara<br>
https://github.com/ansible-community/ara-collection<br>
https://buildmedia.readthedocs.org/media/pdf/ara/latest/ara.pdf<br>
https://ara.readthedocs.io/en/stable-0.x/configuration.html<br>

```
#!/bin/bash
# vi: set sw=4 ts=4 ai:

T="	"

# Install packages
yum -y install				\
	python36				\
	python36-virtualenv		\
	npm

# Create Python virtual environment
rm -rf /root/.ara
mkdir -p /root/.ara/server
#virtualenv-3.6 -p python3 /root/.ara/venv
#source /root/.ara/venv/bin/activate

# Install packages in the virtualenv
#pip3 install --upgrade pip
pip3 install 'django==2.1.*'
pip3 install ansible
pip3 install --user ansible 'ara[server]'

CB="$(python3 -m ara.setup.callback_plugins)"
export ANSIBLE_CALLBACK_PLUGINS="${CB}"

cat << @EOF > /root/.ara/server/settings.yaml
---
default:
  ALLOWED_HOSTS: ['*']
  BASE_DIR: /root/.ara/server
  CORS_ORIGIN_ALLOW_ALL: true
  CORS_ORIGIN_WHITELIST:
    - http://127.0.0.1:8000
    - http://localhost:3000
  DATABASE_ENGINE: django.db.backends.sqlite3
  DATABASE_HOST: null
  DATABASE_NAME: /root/.ara/server/ansible.sqlite
  DATABASE_PASSWORD: null
  DATABASE_PORT: null
  DATABASE_USER: null
  DEBUG: false
  LOGGING:
    disable_existing_loggers: false
    formatters:
      normal:
        format: '%(asctime)s %(levelname)s %(name)s: %(message)s'
    handlers:
      console:
        class: logging.StreamHandler
        formatter: normal
        level: INFO
        stream: ext://sys.stdout
    loggers:
      ara:
        handlers:
          - console
        level: INFO
        propagate: 0
    root:
      handlers:
        - console
      level: INFO
    version: 1
  LOG_LEVEL: INFO
  PAGE_SIZE: 100
  READ_LOGIN_REQUIRED: false
  SECRET_KEY: vtyIMy90dxT1xU7gnofpjPstv9O29XxuFbUFOW4B4XAeBAPnIT
  WRITE_LOGIN_REQUIRED: false
@EOF
cd /root/.ara/server
/root/.local/bin/ara-manage migrate

# Make sure the lastest nodejs is available
cd /tmp
VER="v13.5.0"
rm -f node-${VER}-linux-x64.tar.gz
wget http://nodejs.org/dist/${VER}/node-${VER}-linux-x64.tar.gz
cd /usr/local
rm -rf nodejs node-${VER}-linux-x64
tar -xf /tmp/node-${VER}-linux-x64.tar.gz
ln -s node-${VER}-linux-x64 nodejs
export PATH=/usr/local/nodejs/bin:${PATH}

# Get ara-web and compile
cd /root/.ara
git clone https://github.com/ansible-community/ara-web
cd ara-web
npm install
npm audit fix
npm update

cat <<- @EOF > /root/.ara/ara-web/public/config.json
{
    "apiURL": "http://192.168.122.101:8000"
}
@EOF

cat <<- '@EOF' >> ${HOME}/.bashrc

	if [[ -d /usr/local/nodejs/bin ]]
	then
	    export PATH=/usr/local/nodejs/bin:${PATH}
	fi

	if [[ -d /root/.local/bin ]]
	then
	    export PATH=/root/.local/bin:${PATH}
	fi

	TOPD="/usr/local/lib/python3.6/site-packages/ansible/plugins"
	AANS="${TOPD}/action:/etc/ansible/plugins/action"
	CANS="${TOPD}/callback:/etc/ansible/plugins/callback"
	export ANSIBLE_ACTION_PLUGINS=${AANS}:$(python3 -m ara.setup.action_plugins)
	export ANSIBLE_CALLBACK_PLUGINS=${CANS}:$(python3 -m ara.setup.callback_plugins)
	export ARA_API_CLIENT=http
	export ARA_API_SERVER="http://127.0.0.1:8000"
	export ARA_API_TIMEOUT=15
@EOF

echo "Make sure you re-login to activate all needed settings"

exit

#
# To start the API-server and web interface
#
/root/.local/bin/ara-manage runserver 0.0.0.0:8000 >/dev/null 2>&1 &
cd /root/.ara/ara-web
npm start &
```

```
ansible.vm.provision 'shell', inline: 'pip3.9 install ansible ara[server]'
ansible.vm.provision 'shell', inline: 'export ANSIBLE_CALLBACK_PLUGINS="$(python3 -m ara.setup.callback_plugins)"'
ansible.vm.provision 'shell', inline: 'export ARA_SETTINGS="/root/.ara/server/settings.yaml"'
ansible.vm.provision 'shell', inline: 'ara-manage migrate'
```

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

```
Ansible.cfg
[callback_foreman]
url = 'https://ansible.aut.lab'

# foreman-installer --enable-foreman-plugin-{remote-execution,ansible} --enable-foreman-proxy-plugin-{ansible,remote-execution-ssh}
# foreman-installer --enable-foreman-plugin-{remote-execution,ansible} --enable-foreman-proxy-plugin-{ansible,remote-execution-ssh} --foreman-proxy-plugin-remote-execution-ssh-install-key=true

Import hosts to the Foreman
# sudo -u foreman-proxy -s /bin/bash

# vim /tmp/inventory
srv01 ansible_host=192.168.0.190
srv02 ansible_host=192.168.0.191

# vim /tmp/hosts_setup.yml
---

- hosts: all
  tasks:
    - setup:

...

Generate keygen
# ssh-keygen -t rsa auto yes

# ansible-playbook -u root --private-key ~/.ssh/id_rsa_foreman_proxy -i /tmp/inventory
 - Verificar envio dessa chave aos hosts

Import roles
# /etc/ansible/roles

Assign roles to hosts and them execute

Foreman Services Status
# foreman-maintain service status -b
 - Verificar comando

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