[What Is PPA and How to Use It to Install Software in Ubuntu?](https://operavps.com/docs/what-is-ppa/#:~:text=PPA%20stands%20for%20%E2%80%9CPersonal%20Package,in%20official%20operating%20system%20repositories.)  
## Installation through Apt on Ubuntu Machine
```
$ sudo apt-get update 
$ sudo apt-get install software-properties-common 
$ sudo apt-add-repository ppa:ansible/ansible $ sudo apt-get update 
$ sudo apt-get install ansible
```
### Check
```
$ ansible --version
ansible 2.10.8
  config file = None
  configured module search path = ['/home/anisimov/.ansible/plugins/modules', '/usr/share/ansible/plugins/modules']
  ansible python module location = /usr/lib/python3/dist-packages/ansible
  executable location = /usr/bin/ansible
  python version = 3.10.12 (main, Nov 20 2023, 15:14:05) [GCC 11.4.0]
```
