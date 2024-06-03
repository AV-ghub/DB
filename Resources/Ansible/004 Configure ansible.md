[src](https://habr.com/ru/companies/nixys/articles/668458/)   
[How to install ssh on Ubuntu Linux using apt-get](https://www.cyberciti.biz/faq/how-to-install-ssh-on-ubuntu-linux-using-apt-get/)
## SSH
```
sudo apt-get update
sudo apt-get upgrade
sudo apt-get install openssh-client
```
```
sudo systemctl start ssh
sudo systemctl stop ssh
sudo systemctl restart ssh
```
```
sudo systemctl status ssh
```
## Прокидываем ключ ssh на удаленный сервер
На основном хосте   
```
$ ssh-keygen -t rsa

Generating public/private rsa key pair.
Enter file in which to save the key (/.../.ssh/id_rsa): 
Enter passphrase (empty for no passphrase): 
Enter same passphrase again: 
Your identification has been saved in /.../.ssh/id_rsa
Your public key has been saved in /.../.ssh/id_rsa.pub
The key fingerprint is:
SHA256:U5iQZ4hB9... anisimov@...pg-03
The key's randomart image is:
+---[RSA 3072]----+
|  .o++.=*+       |
...
+----[SHA256]-----+
```
```
$ cat ~/.ssh/id_rsa.pub

ssh-rsa AAAAB3NzaC1yc2EA....
```
Подключаемся к удаленному серверу и сохраняем этот ключ в authorized_keys.   
По адресу ~/.ssh/authorized_keys   
```
touch ~/.ssh/authorized_keys
echo "ssh-rsa AAAAB3NzaC1yc2EA....." >> ~/.ssh/authorized_keys
```
Возвращаемся на master. Открываем inventory-файл.   
```
cd /var/ansible
nano hosts.txt
```
```
[adm]
anisimov...03-1 ansible_host=0.0.0.0 ansible_port=22 ansible_user=anisimov ansible_ssh_private_key_file=/.../.ssh/id_rsa
```
Для проверки подключения нам необходимо запустить команду ansible с ключами.
```
$ ansible -i hosts.txt all -m ping

The authenticity of host '10.30.41.7 (10.30.41.7)' can't be established.
ED25519 key fingerprint is SHA256:quJaKOxdioqoHh7tugy+vA5xWZpiCmdwI3q0OfZmzr4.
This host key is known by the following other names/addresses:
    ~/.ssh/known_hosts:1: [hashed name]
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes

anisimov-ubuntu-pg-03-1 | SUCCESS => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python3"
    },
    "changed": false,
    "ping": "pong"
}
```
На наш запрос ping нам пришел ответ pong. Это значит, что соединение установлено, и мы можем делать с сервером все, что нам необходимо.   
## Конфигурация
[Configuring Ansible](https://docs.ansible.com/ansible/latest/installation_guide/intro_configuration.html)   

> ansible configuration file is only created automatically during the installation
> if you perform the installation with package managers like yum or apt-get.
> If you installed ansible using pip you should create the configuration file manually.

[ansible config file not found; using defaults](https://stackoverflow.com/questions/63672853/ansible-config-file-not-found-using-defaults)    
[The configuration file](https://docs.ansible.com/ansible/latest/reference_appendices/config.html#the-configuration-file)
Changes can be made and used in a configuration file which will be searched for in the following order:
```
ANSIBLE_CONFIG (environment variable if set)
ansible.cfg (in the current directory)
~/.ansible.cfg (in the home directory)
/etc/ansible/ansible.cfg
```
Ansible will process the above list and use the first file found, all others are ignored.   

### Предварительные установки
```
mkdir /etc/ansible
mkdir /var/log/ansible
sudo touch ansible.log
chmod ugo+w ansible.log
```

Редактируем файл конфигурации Ansible.   
```
sudo nano /etc/ansible/ansible.cfg
```
```
[defaults]
host_key_checking = false
inventory = /var/ansible/hosts.txt
log_path = /var/log/ansible/ansible.log
```
```
$ ansible all -m ping

anisimov-ubuntu-pg-03-1 | SUCCESS => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python3"
    },
    "changed": false,
    "ping": "pong"
}
```

## Командные модули ansible
Все командные модули вызывается ключом -m.   
### command - выполняет команды на выбранных группах без участие оболочки shell.
> Команда для выполнение вызывается ключом -a (аргумент) и помещается в кавычки.
> Пример: ansible test -m command -a "date"   

```
ansible adm -m command -a "date"

anisimov-ubuntu-pg-03-1 | CHANGED | rc=0 >>
Пн 03 июн 2024 12:49:14 MSK
```
> У этого модуля есть минус: не будут работать переменные "<", ">", "|", ";", "&".
> Например, вы не сможете делать grep с помощью данного модуля.   

[Официальный мануал](https://docs.ansible.com/ansible/2.5/modules/command_module.html#command-module)

### shell - Делает все тоже самое, что и модуль command, только с участием оболочки shell.
```
$ ansible adm -m shell -a "cat /var/log/syslog | tail -5"

anisimov-ubuntu-pg-03-1 | CHANGED | rc=0 >>
Jun  3 12:49:14 anisimov-ubuntu-pg-03-1 python3[3189]: ansible-ansible.legacy.command Invoked with _raw_params=date warn=True _uses_shell=False stdin_add_newline=True strip_empty_ends=True argv=None chdir=None executable=None creates=None removes=None stdin=None
Jun  3 12:50:14 anisimov-ubuntu-pg-03-1 systemd[1]: session-41.scope: Deactivated successfully.
Jun  3 12:50:14 anisimov-ubuntu-pg-03-1 systemd[1]: session-41.scope: Consumed 1.055s CPU time.
Jun  3 12:52:28 anisimov-ubuntu-pg-03-1 systemd[1]: Started Session 42 of User anisimov.
Jun  3 12:52:29 anisimov-ubuntu-pg-03-1 python3[3261]: ansible-ansible.legacy.command Invoked with _raw_params=cat /var/log/syslog | tail -5 _uses_shell=True warn=True stdin_add_newline=True strip_empty_ends=True argv=None chdir=None executable=None creates=None removes=None stdin=None
```   
[Официальный мануал](https://docs.ansible.com/ansible/2.5/modules/shell_module.html#shell-module)   

### copy - модуль позволяет скопировать файл с master на удаленный сервер.

> Аргументы:
> scr - адрес файла на master;
> dest - адрес куда сохранить файл на удаленный сервер;
> owner - имя пользователя, кому будет принадлежать файл;
> group - группа, кому будет принадлежать файл;
> mode - права на файл.
```
ansible adm -m copy -a "src=/var/ansible/test.txt dest=/var owner=test group=test mode=644"
```   
Если не из под root пользователя, то необходимо добавить -b (sudo) в конце
```
ansible adm -m copy -a "src=/var/ansible/test.txt dest=/var owner=test group=test mode=644" -b
```
[Официальный мануал](https://docs.ansible.com/ansible/2.5/modules/copy_module.html?highlight=copy)   
    
### file - выполняет действие с файлом. Удаляет, изменяет права пользователя и много чего еще.

> Аргументы:
> path - путь до файла на удаленном хранилище;
> state - состояние.   

Удалим файл, который мы скачали с помощью предыдущего модуля
```
ansible adm -m file -a "path=/var/test.txt state=absent"
```

[Официальный мануал](https://docs.ansible.com/ansible/2.5/modules/file_module.html?highlight=file)

### apt - модуль для установки пакетов на удаленном сервере при условии использования операционных системах Debian и основанных на них (Ubuntu, Linux Mint и т. п.). 
При использовании Red Hat ОС можно использовать модуль yum.

> Аргументы:
> name - имя сервиса, пакета;
> tate - команда установить, удалить, обновить.   

```
ansible adm -m apt -a "name=htop state=latest"
```

[Официальный мануал](https://docs.ansible.com/ansible/2.5/modules/apt_module.html)

### service - позволяет запускать/останавливать/перезагружать сервисы на удаленном сервере.

> Аргументы:
> name - "название сервиса";
> state - команда для изменения состояния;
> enabled - когда необходимо добавить в автозагрузку.

```
adm -m service -a "name=nginx state=started"
```

[Официальный мануал](https://docs.ansible.com/ansible/2.5/modules/service_module.html)

## Ansible-playbooks
С помощью playbook можем все объединить в один файл и запускать с помощью одной команды.
```
ansible-playbook Адрес playbook
```

### Default directory 
```
$ sudo mkdir /etc/ansible/ansible
$ cd /etc/ansible/ansible
$ sudo chmod go+w playbooks/
$ cd playbooks/
$ touch playbook.yml
$ nano playbook.yml
```

> формат yml, не любит TAB, поэтому никогда его там не используйте

```
Ключи:
name - присваивание названия группе/команде.
Пример: name: Test PING

hosts - указание группы серверов, на которых необходимо выполнить данную команду.
Пример: hosts: all

become - запуск sudo (если вы делаете настройки не под root-пользователем).
tasks - указываем, что дальше будет идти исполняемая команда.
name - присваивание названия команде.
```
```
- name: Test PING.
  hosts: all
  become: no
  tasks:
    - name: ping
      ping:
```
```
$ ansible-playbook playbook.yml

PLAY [Test PING.] **************************************************************************************************************************************************************
TASK [Gathering Facts] *********************************************************************************************************************************************************
ok: [anisimov-ubuntu-pg-03-1]
TASK [ping] ********************************************************************************************************************************************************************
ok: [anisimov-ubuntu-pg-03-1]
PLAY RECAP *********************************************************************************************************************************************************************
anisimov-ubuntu-pg-03-1    : ok=2    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
```

## Ansible Roles
[Официальный мануал](https://docs.ansible.com/ansible/latest/user_guide/playbooks_reuse_roles.html)   
Для создания структуры директорий используется команда:
```
ansible-galaxy init <Название>
``` 


[Missing sudo password in Ansible](https://stackoverflow.com/questions/25582740/missing-sudo-password-in-ansible)
```
$ ansible-playbook playbook.yml --extra-vars 'ansible_sudo_pass=<sudo_pass>'

PLAY [Install LEMP server] *****************************************************************************************************************************************************
TASK [Gathering Facts] *********************************************************************************************************************************************************
ok: [anisimov-ubuntu-pg-03-1]
TASK [LEMP : include_tasks] ****************************************************************************************************************************************************
included: /etc/ansible/ansible/playbooks/LEMP/tasks/default_settings.yml for anisimov-ubuntu-pg-03-1
TASK [LEMP : update repo.] *****************************************************************************************************************************************************
changed: [anisimov-ubuntu-pg-03-1]
PLAY RECAP *********************************************************************************************************************************************************************
anisimov-ubuntu-pg-03-1    : ok=3    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0 
```











