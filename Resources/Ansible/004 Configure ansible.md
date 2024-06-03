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


























