# Ansible Playbook for filtering contents

- [Ansible Playbook for filtering contents](#ansible-playbook-for-filtering-contents)
  - [Overview](#overview)
  - [Requirement](#requirement)
  - [Usage](#usage)
    - [Preparation](#preparation)
      - [Target server](#target-server)
      - [Client PC](#client-pc)
    - [Download and edit the Playbook](#download-and-edit-the-playbook)
    - [Execute the Playbook](#execute-the-playbook)
    - [Proxy setting](#proxy-setting)
  - [ToDo](#todo)

## Overview

![no-no-channel](https://images.app.goo.gl/DFCPrfCfjBwzzz9j7)

- This [Ansible](https://docs.ansible.com/ansible/2.9/index.html) Playbook deploys the proxy server using [Squid](http://www.squid-cache.org/) to filter the contents which you do not want to see (e.g., adult or violent contents, advertisements, and so on).
- For just showing the example of filtering, the setting listed on [Easylist](https://easylist.to/) is used. If you delete this setting, change `roles/proxy/templates/banned_list.txt.j2` as below.

```roles/proxy/templates/banned_list.txt.j2:j2
{% for item in banned_url %}
{{ item }}
{% endfor %}
```

## Requirement

- Target server:
  - OS: Raspbian or Ubuntu
  - location: local network
- Client PC:
  - OS: macOS
  - required software: ansible, sshpass

## Usage

### Preparation

#### Target server

- Make a Ubuntu VM or use Raspberry Pi on your local network.
- Here is the minimal setting to use this Playbook.
  - Fix the private IP address (e.g., 192.168.0.100).
  - enable and start sshd.

#### Client PC

- Install CommandLineTools.

```
sudo xcode-select --install
```

- Install sshpass.

```
wget https://sourceforge.net/projects/sshpass/files/latest/download/sshpass/1.08/sshpass-1.08.tar.gz
tar xvf sshpass-1.08.tar.gz
cd sshpass-1.08
./configure
make
make install
```

- Install Homebrew.

```
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
```

- Install Ansible.

```
brew install ansible
```

### Download and edit the Playbook

- Clone this Playbook.

```
git clone https://github.com/MathWistLight/contents_filtering_with_squid.git
```

- Edit the following files.
  - `hosts`
    - \[Must\] Replace "YOUR HOST IP ADDRESS" to the private IP of target server.
  - `host_vars/target.yml`
    - \[Must\] Replace "YOUR USERNAME" and "YOUR PASSWORD" to the account information which you use to connect to the server via ssh.
    - \[Must\] Replace "YOUR SUDO PASSWORD" to the sudo password.
  - `setting_proxy.yml`
    - \[Must\] Replace "YOUR USERNAME" and "YOUR GROUP NAME" to the user and group name which can be checkd when you execute `ls -l` on the home directory of ssh user on the target server.
    - \[Must\] Replace "YOUR LOCAL NETWORK ADDRESS" to the network address used on your local network(e.g., 192.168.1.0/24).
    - \[Optional\] Edit the `banned_list:` block if you want to change or add the banned URLs as below. If you want to know more, see [YAML Syntax on Ansible](https://docs.ansible.com/ansible/2.9/reference_appendices/YAMLSyntax.html).

```setting_proxy.yml:yml
# simple example
- name: setting proxy server.
  hosts: target
  become: true
  vars:
    user: pi
    group: pi
    localaddr: 192.168.1.0/24
    banned_url:
      - ^(https*://)*([^/][^/]*\.)*rr.*(-*)sn-oguel(.*)\.googlevideo\.com(:443|:80).*$

    # This setting allows "yyy.com", "example.com", and all subdomains of "example.com"(e.g., "aaa.example.com", "hogehoge.example.com",...) to be blocked
    banned_list:
      - yyy.com
      - .example.com
```

### Execute the Playbook

- Execute the Ansible Playbook.

```
cd <this directory>
ansible-playbook -i hosts setting_proxy.yml
```

### Proxy setting

- Make settings of the proxy server on your devices.

## ToDo

- Support multiple OS.
  - [x] Raspbian
  - [x] Ubuntu
  - [ ] CentOS
  - [ ] macOS
  - [ ] Windows
- [ ] Write the role description.
