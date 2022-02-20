---
title: Percobaan keempat deployment
---
Ansible adalah sebuah provisioning tool yang dikembangkan oleh RedHat. Dimana kamu dapat mencatat setiap proses deployment ataupun konfigurasi yang biasa dilakukan berulang - ulang terhadap beberapa server. Misal saat pertama kali kita memasang Ubuntu Server di 10 mesin, maka kita akan melakuan apt-get update serta memasang beberapa komponen seperti PHP5 dan Apache2. Sebenarnya tidak akan menjadi masalah, bila kita hanya melakukan sedikit hal. Tapi bayangkan bila harus melakukan konfigurasi yang cukup kompleks dan dilakukan secara berulang - ulang ke 10 mesin tersebut.

Mungkin kita dapat membuat BASH script atau dengan menggunakan bahasa pemrograman lain untuk mempermudah masalah kita. Namun dengan Ansible, kita dapat melakukan pengelolaan server yang lebih mudah dan terstruktur. Selain itu dapat menghemat waktu kita untuk mengeksekusi proses konfigurasi atau deployment yang sama terhadap beberapa server yang ada.

Untuk memasang Ansible, kamu dapat memasangnya dengan cara berikut.

Mac:
```sh
$ brew install ansible
```
Ubuntu:
```sh
$ sudo apt-get install ansible
```
Kemudian buatlah **ansible.cfg** di dalam direktori `~/.ansible` dan isi dengan konfigurasi berikut:
```sh
[defaults]
inventory=/usr/local/etc/ansible/hosts

[ssh_connection]
control_path=%(directory)s/%%h-%%r
control_path_dir=~/.ansible/cp

[privilege_escalation]
become = yes
become_method = sudo
```
Konfigurasi diatas akan membaca daftar host dari file /usr/local/etc/ansible/hosts, dan membuatnya sebagai sudo. Sekarang kita coba tambahkan server yang akan ditembak oleh Ansible dengan mencatatkannya di file hosts.
```sh
[ubuntu-server]
192.168.11.57 ansible_ssh_user=ubuntu ansible_ssh_private_key_file=./ubuntu-server.pem
```
Sebagai langkah awal, kamu dapat mengeksekusi ad hoc command seperti berikut:
```sh
$ ansible ubuntu-server -m ping
$ ansible ubuntu-server -a "free -m"
$ ansible ubuntu-server -a "hostname && uname -a"
$ ansible ubuntu-server -a "ps ax | grep python"
$ ansible ubuntu-server -a "touch ~/README.txt"
```
Sebagai contoh berikut adalah contoh script Ansible untuk memasang Apache2 dan PHP5 di Ubuntu Server:
```sh
- hosts: ubuntu-server
  tasks:
    - command: apt-get update
    - apt: "name=apache2 state=present"
    - apt: "name=php5 state=present"
```
Untuk memanggilnya, kamu dapat mengeksekusinya dengan menggunakan perintah berikut:
```sh
$ ansible-playbook initial-setup.yml
```
Kamu pun dapat menambahkan banyak server di dalam inventory Ansible. Kemudian bila proses konfigurasi hampir sama, kamu dapat mengganti hosts menjadi all seperti berikut:
```sh
- hosts: all
  tasks:
    - command: apt-get update
    - apt: "name=apache2 state=present"
    - apt: "name=php5 state=present"
```
