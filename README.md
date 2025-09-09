# 🚀 Instalar Wazuh via Ansible

![Ansible](https://img.shields.io/badge/Ansible-2.17-blue)
![Wazuh](https://img.shields.io/badge/Wazuh-4.12.0-orange)

**Descrição:** Passo a passo visual para instalação e configuração do **Wazuh Agent** usando Ansible.  

---

## 📖 Sumário

1. [Pré-requisitos](#-pré-requisitos)  
2. [Instalação do Ansible](#-instalação-do-ansible)  
3. [Estrutura de diretórios e roles](#-estrutura-de-diretórios-e-roles)  
4. [Configuração do Inventário](#-configuração-do-inventário)  
5. [Configuração do Playbook](#-configuração-do-playbook)  
6. [Desativar temporariamente o Host Key Checking](#-desativar-temporariamente-o-host-key-checking)  
7. [Testando a Conexão](#-testando-a-conexão)  
8. [Executando o Playbook](#-executando-o-playbook)  
9. [Verificação no Wazuh Dashboard](#-verificação-no-wazuh-dashboard)  

---

## 🛠️ 1. Pré-requisitos

- Servidor Linux (Ubuntu/Debian ou RHEL/CentOS) dedicado para Ansible  
- Acesso root ou usuário com privilégios sudo  
- Conectividade SSH entre o servidor Ansible e os hosts de destino  
- IP/Hostname do **Wazuh Manager** e credenciais de API (`api_user` / `api_password`)  

> 🔗 [Documentação oficial Wazuh](https://documentation.wazuh.com/)  
> 🔗 [Documentação oficial Ansible](https://docs.ansible.com/)

---

## 💻 2. Instalação do Ansible

### Ubuntu/Debian

```bash

sudo apt-get update -y
sudo apt-get install ansible git -y
ansible --version

````

> ✅ Deve retornar algo como: `ansible [core 2.17.13]`

---

## 📂 3. Estrutura de diretórios e roles

### 3.1 Clonar repositório oficial do Wazuh

```bash

sudo mkdir -p /etc/ansible/roles
cd /etc/ansible

git clone https://github.com/wazuh/wazuh-ansible.git
cd wazuh-ansible
git checkout v4.12.0

```

### 3.2 Copiar role para o diretório padrão

```bash

cp -r roles/wazuh /etc/ansible/roles/

```

---

## 📋 4. Configuração do Inventário (`/etc/ansible/hosts`)

```ini

[wazuh_agents]
192.168.15.152 ansible_user=usuario ansible_ssh_pass=senha

```

> ⚠️ O grupo `wazuh_agents` deve coincidir com o playbook
> 🔹 Substitua `usuario` e `senha` pelas credenciais SSH corretas

---

## 📄 5. Configuração do Playbook (`/etc/ansible/wazuh-agent.yml`)

```yaml

---
- name: Deploy Wazuh agent
  hosts: wazuh_agents
  become: yes
  vars:
    wazuh_managers:
      - address: 192.168.15.87    # IP do Wazuh Manager
        port: 1514
        protocol: tcp
        api_port: 55000
        api_proto: https
        api_user: wazuh-wui       # usuário da API
        api_password: ChangeMe123 # senha da API

    wazuh_agent_authd:
      enable: true
      port: 1515
      ssl_agent_ca: null
      ssl_auto_negotiate: true

  roles:
    - wazuh/ansible-wazuh-agent

```

> ⚙️ Ajuste `address`, `api_user` e `api_password` conforme sua instalação

---

## 🔑 6. Desativar temporariamente o Host Key Checking

```bash

sudo apt update
sudo apt install -y sshpass

```

Edite ou crie `/etc/ansible/ansible.cfg`:

```ini

[defaults]
host_key_checking = False

```

---

## 📡 7. Testando a Conexão

```bash

ansible wazuh_agents -m ping

```

> ✅ Deve retornar: `"ping": "pong"`

---

## ▶️ 8. Executando o Playbook

```bash

ansible-playbook /etc/ansible/wazuh-agent.yml -b -K -vvvv

```

* `-b` : executa com privilégios sudo
* `-K` : solicita senha sudo
* `-vvvv` : modo detalhado para debug

---

## 👀 9. Verificação no Wazuh Dashboard

1. Acesse o **Wazuh Dashboard**
2. Vá em **Agents**
3. Confirme que os hosts aparecem como **Active** ✅

---

## 🗂️ Estrutura final de diretórios

```
/etc/ansible/
├── hosts                  # inventário
├── wazuh-agent.yml        # playbook
├── roles/
│   └── wazuh/
│       └── ansible-wazuh-agent/
└── wazuh-ansible/         # repositório clonado (opcional)
```

---
