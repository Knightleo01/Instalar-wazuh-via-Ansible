# Instalar-wazuh-via-Ansible
Instalar wazuh via Ansible
Documentação: Instalação e Configuração do Wazuh Agent via Ansible

________________________________________
1.	Clonar wazuh-ansible e fazer checkout da versão correta.
2.	Copiar a role wazuh para /etc/ansible/roles/.
3.	Configurar inventário /etc/ansible/hosts.
4.	Criar ou ajustar o playbook /etc/ansible/wazuh-agent.yml.
5.	Testar ping Ansible.
6.	Desativar temporariamente o Host Key Checking
7.	Testando a Conexão
8.	Rodar playbook.
9.	Verificação Wazuh Dashboard
________________________________________
Estrutura de diretórios final
/etc/ansible/
├── hosts                  # inventário
├── wazuh-agent.yml        # playbook
├── roles/
│   └── wazuh/
│       └── ansible-wazuh-agent/
└── wazuh-ansible/         # repositório clonado (opcional)
________________________________________
1. Pré-requisitos
•	Servidor Linux (Ubuntu/Debian ou RHEL/CentOS) dedicado para rodar o Ansible.
•	Acesso root ou usuário com privilégios sudo.
•	Conectividade SSH entre o servidor Ansible e os hosts onde o Wazuh Agent será instalado.
•	IP/Hostname do Wazuh Manager e credenciais de API (api_user / api_password).

2. Instalação do Ansible
Ubuntu/Debian
apt-get update -y 
apt-get install ansible -y
ansible –version
 
________________________________________
3. Estrutura de diretórios e roles
3.1. Clonar repositório oficial do Wazuh
sudo mkdir -p /etc/ansible/roles
cd /etc/ansible

git clone https://github.com/wazuh/wazuh-ansible.git
cd wazuh-ansible
git checkout v4.12.0
3.2. Copiar a role para o diretório padrão do Ansible
cp -r roles/wazuh /etc/ansible/roles/________________________________________
4. Configuração do Inventário (/etc/ansible/hosts)
[wazuh_agents]
192.168.15.152 ansible_user=usuario ansible_ssh_pass=senha
⚠️ O grupo wazuh_agents deve bater exatamente com o playbook.
🔹 Substitua usuario e senha pelas credenciais SSH corretas.________________________________________
5. Configuração do Playbook (/etc/ansible/wazuh-agent.yml)
---
- name: Deploy Wazuh agent
  hosts: wazuh_agents
  become: yes
  vars:
    wazuh_managers:
      - address: 192.168.15.87	# IP do Wazuh Manager
        port: 1514
        protocol: tcp
        api_port: 55000
        api_proto: https
        api_user: wazuh-wui #usuario da API
        api_password: ChangeMe123 #Senha da API

    wazuh_agent_authd:
      enable: true
      port: 1515
      ssl_agent_ca: null
      ssl_auto_negotiate: true

  roles:
    - wazuh/ansible-wazuh-agent

•	# Ajuste address, api_user e api_password conforme sua instalação.
________________________________________
6. Desativar temporariamente o Host Key Checking

sudo apt update
sudo apt install -y sshpass
Edite ou crie o arquivo /etc/ansible/ansible.cfg e adicione:
[defaults]
host_key_checking = False
________________________________________
7. Testando a Conexão
ansible wazuh_agents -m ping
Deve retornar pong.
 
________________________________________
8. Executando o Playbook
ansible-playbook /etc/ansible/wazuh-agent.yml -b -K -vvvv
•	-b: executa com privilégios sudo.
•	-K: solicita senha sudo.
•	-vvvv: modo detalhado.
 
________________________________________
9. Verificação
1.	Acesse o Wazuh Dashboard.
2.	Vá em Agents e confirme que os hosts aparecem como Active.
 


