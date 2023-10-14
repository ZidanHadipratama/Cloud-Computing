# DNS and Web Server Setup using Vagrant and Ansible 🌐🚀

In this project, I learned how to use Vagrant to automate the creation of virtual machines and Ansible to configure and provision these machines as DNS and web servers. Below, I will provide a detailed walkthrough of the entire project, including the code and steps involved.

## Vagrant Setup 🔧

1. First, navigate to the project folder and initialize Vagrant using the `vagrant init` command.

2. Create a Vagrantfile with the following configuration to define three virtual machines:
```ruby
Vagrant.configure("2") do |config|
  
  config.vm.define "dns1" do |dns1|
    dns1.vm.box = "ubuntu/focal64"
    dns1.vm.network "private_network", type: "dhcp"
    dns1.vm.provider "virtualbox" do |vb|
      vb.memory = 1024
      vb.cpus = 1
    end
  end

  config.vm.define "dns2" do |dns2|
    dns2.vm.box = "ubuntu/focal64"
    dns2.vm.network "private_network", type: "dhcp"
    dns2.vm.provider "virtualbox" do |vb|
      vb.memory = 1024
      vb.cpus = 1
    end
  end

  config.vm.define "web" do |web|
    web.vm.box = "centos/7"
    web.vm.network "private_network", type: "dhcp"
    web.vm.provider "virtualbox" do |vb|
      vb.memory = 1024
      vb.cpus = 1
    end
  end
end
```
This configuration sets up three virtual machines, two running Ubuntu and one running CentOS, with private network connections and specific resource allocations.

3. Run `vagrant up` to create and start the virtual machines. You can use `vagrant status` to verify that the VMs are running.

4. Use `vagrant ssh-config` to identify the location of the private keys for each VM. Save this information, including the path to the private key, in a notepad for future reference.

5. To find the IP addresses of each VM, SSH into each VM (e.g., `vagrant ssh dns1`) and run `ip a`. Typically, the IP format is something like 192.168.xx.xx. Note down all the IP addresses for later use.

## Ansible Configuration 🧩

6. Navigate to the `/etc/ansible` directory and create a `hosts` file. Here's an example of the `hosts` file for this project:

```yaml
all:
    vars:
        ansible_user: vagrant

dns_server:
    hosts:
        dns1:
            ansible_host: 192.168.xx.xx
            ansible_ssh_private_key_file: /path/to/dns1/private_key
        dns2:
            ansible_host: 192.168.xx.xx
            ansible_ssh_private_key_file: /path/to/dns2/private_key

web_server:
    hosts:
        web:
            ansible_host: 192.168.xx.xx
            ansible_ssh_private_key_file: /path/to/web/private_key
```

In this configuration, we have grouped the VMs into `dns_server` and `web_server` for easier Ansible script management. Replace the IP addresses and private key paths with the information you collected earlier.

## Ansible Playbook 📜

7. Create an Ansible playbook, e.g., `master_playbook.yaml`, with the following content:

```yaml
---
- name: Setup DNS Servers
  hosts: dns_server
  become: yes

  tasks:
    - name: Update package cache (Ubuntu)
      apt:
        update_cache: yes

    - name: Install BIND9
      apt:
        name: bind9
        state: present

    - name: Create Zone Files (Primary)
      template:
        src: zone1.txt
        dest: /etc/bind/db.tka.a02.com
      when: inventory_hostname == "dns1"

    - name: Create Zone Files (Secondary)
      template:
        src: zone2.txt
        dest: /var/cache/bind/db.tka.a02.com
      when: inventory_hostname == "dns2"

    - name: Configure DNS Server (Primary)
      template:
        src: named.conf.primary.j2
        dest: /etc/bind/named.conf.local
      when: inventory_hostname == "dns1"

    - name: Configure DNS Server (Secondary)
      template:
        src: named.conf.secondary.j2
        dest: /etc/bind/named.conf.local
      when: inventory_hostname == "dns2"

    - name: Start and Enable bind9
      service:
        name: bind9
        state: started
        enabled: yes

    - name: Reload BIND9
      systemd:
        name: bind9
        state: restarted

- name: Setup Web Server
  hosts: web_server
  become: yes

  tasks:
    - name: Install Apache2 (CentOS)
      yum:
        name: httpd
        state: present

    - name: Start Apache2 (CentOS)
      service:
        name: httpd
        state: started
```

This Ansible playbook installs BIND9 on Ubuntu-based DNS servers and Apache on the CentOS-based web server. It also copies necessary configuration files for the DNS servers.

8. Run the playbook using the following command:
```bash
ansible-playbook master_playbook.yaml
```

Now, Ansible will execute the tasks specified in the playbook on the VMs, setting up the DNS and web servers according to the assignment requirements.

Congratulations! You have successfully set up DNS and web servers using Vagrant and Ansible. Enjoy your cloud computing journey! 🎉👏