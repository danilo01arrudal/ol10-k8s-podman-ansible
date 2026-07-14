# 🐘 Provisionamento Automatizado do PostgreSQL 18 com Ansible

[![GitHub](https://img.shields.io/badge/Repository-danilo01arrudal/ol10--pg18--ansible-blue?logo=github)](https://github.com/danilo01arrudal/ol10-pg18-ansible)
[![Ansible](https://img.shields.io/badge/Ansible-2.9+-black?logo=ansible)](https://www.ansible.com/)
[![PostgreSQL](https://img.shields.io/badge/PostgreSQL-18-336791?logo=postgresql)](https://www.postgresql.org/)
[![Oracle Linux](https://img.shields.io/badge/Oracle%20Linux-10-red?logo=oracle)](https://www.oracle.com/linux/)

---

## 📋 Visão Geral

Este projeto tem como objetivo **automatizar o processo de instalação, configuração e gerenciamento do PostgreSQL 18 em ambientes Oracle Linux 10**, utilizando **Ansible** como ferramenta de automação.

A solução foi projetada para ser:

- **Reprodutível**: todo o processo é descrito como código (Infrastructure as Code).
- **Flexível**: suporte a diferentes perfis de instância (pequena, média, grande) com parâmetros de performance pré-definidos.
- **Integrado**: pipeline CI/CD via GitHub Actions para execução automatizada.
- **Seguro**: acesso via SSH com chave pública e gerenciamento de secrets.


---

## 🚀 Principais Funcionalidades

| Funcionalidade | Descrição |
|----------------|-----------|
| **Instalação automatizada** | Instalação do PostgreSQL 18 a partir do repositório oficial PGDG. |
| **Configuração dinâmica** | Aplicação de parâmetros de performance conforme a classe da instância. |
| **Inicialização do cluster** | Execução do `initdb` e criação da estrutura de dados. |
| **Gerenciamento de serviço** | Habilitação e inicialização do serviço `postgresql-18` via systemd. |
| **Desinstalação controlada** | Playbook para remoção completa do ambiente. |
| **CI/CD com GitHub Actions** | Pipelines para deploy e desinstalação acionados manualmente ou via push. |

---

## 🏗️ Estrutura do Projeto

```plaintext
ol10-pg18-ansible/
├── .github/
│   └── workflows/
│       ├── deploy-postgres.yml     # Pipeline de deploy
│       └── uninstall.yml           # Pipeline de desinstalação
├── group_vars/
│   ├── all/
│   │   └── vars.yml                # Variáveis comuns (versão, repositório, etc.)
│   ├── pg_classe_pequena/
│   │   └── vars.yml                # Parâmetros para instâncias pequenas
│   ├── pg_classe_media/
│   │   └── vars.yml                # Parâmetros para instâncias médias
│   └── pg_classe_grande/
│       └── vars.yml                # Parâmetros para instâncias grandes
├── inventory/
│   └── production                  # Inventário dos hosts gerenciados
├── playbooks/
│   ├── site.yml                    # Playbook principal de provisionamento
│   └── uninstall.yml               # Playbook de desinstalação
├── roles/
│   └── postgresql/
│       ├── handlers/
│       │   └── main.yml            # Handlers (ex: restart do serviço)
│       ├── tasks/
│       │   ├── main.yml            # Orquestração das tarefas
│       │   ├── install.yml         # Instalação dos pacotes
│       │   ├── init.yml            # Inicialização do cluster
│       │   ├── configure.yml       # Configuração dos templates
│       │   └── uninstall.yml       # Remoção do ambiente
│       ├── templates/
│       │   ├── postgresql.conf.j2  # Template do arquivo de configuração
│       │   └── pg_hba.conf.j2      # Template das regras de autenticação
│       └── vars/
│           └── main.yml            # Variáveis padrão da role
├── ansible.cfg                     # Configuração do Ansible
└── README.md                       # Este arquivo
```

---

## 🧩 Principais Arquivos e suas Funções

### `playbooks/site.yml`
Playbook principal que orquestra o provisionamento completo do PostgreSQL. Executa a role `postgresql` nos hosts definidos no inventário.

### `playbooks/uninstall.yml`
Playbook para desinstalação controlada do PostgreSQL, removendo pacotes, diretórios de dados e desabilitando o serviço.

### `group_vars/`
Diretório que organiza as variáveis por grupo de hosts:
- `all/vars.yml`: define a versão do PostgreSQL, repositório, diretórios e usuário.
- `pg_classe_*`: define parâmetros de performance (`max_connections`, `shared_buffers`, `work_mem`, etc.) para cada perfil de instância.

### `roles/postgresql/`
Role Ansible que encapsula toda a lógica de instalação e configuração:
- **install.yml**: adiciona o repositório PGDG e instala o pacote `postgresql18-server`.
- **init.yml**: executa o `initdb` para criar o cluster de dados (apenas se o arquivo `PG_VERSION` não existir).
- **configure.yml**: aplica os templates `postgresql.conf.j2` e `pg_hba.conf.j2` com os parâmetros definidos.
- **uninstall.yml**: remove pacotes, diretórios e o serviço.

### `templates/`
Arquivos Jinja2 que geram as configurações finais do PostgreSQL com base nas variáveis fornecidas.

---

## ⚙️ Tecnologias Utilizadas

| Tecnologia | Versão | Finalidade |
|------------|--------|------------|
| **Oracle Linux** | 10 | Sistema operacional base (host e alvo). |
| **PostgreSQL** | 18 | Sistema de gerenciamento de banco de dados. |
| **Ansible** | 2.9+ | Automação de provisionamento e configuração. |
| **GitHub Actions** | — | Pipeline CI/CD para execução automatizada. |
| **Jinja2** | — | Template engine para arquivos de configuração dinâmicos. |

---

## 🔄 Integração Contínua (CI/CD) com GitHub Actions

O projeto conta com dois workflows configurados no diretório `.github/workflows/`:

### `deploy-postgres.yml`
- **Trigger**: `workflow_dispatch` (execução manual).
- **Executor**: `self-hosted` (runner instalado na própria VM Oracle Linux 10).
- **Etapas**:
  1. Checkout do código.
  2. Instalação do Ansible (se necessário).
  3. Execução do playbook `site.yml`.

### `uninstall.yml`
- **Trigger**: `workflow_dispatch` (execução manual).
- **Executor**: `self-hosted`.
- **Etapas**:
  1. Checkout do código.
  2. Execução do playbook `uninstall.yml`.

> **Observação**: Os workflows utilizam um **runner auto-hospedado** configurado na VM alvo, garantindo que a execução ocorra no ambiente Oracle Linux 10 real, sem dependência de containers ou emuladores.

---

## 📦 Pré‑requisitos

Antes de executar o projeto, certifique-se de que:

- ✅ A **máquina de controle** (onde o Ansible será executado) possui o Ansible instalado.
- ✅ A **VM alvo** está rodando **Oracle Linux 10** com Python 3 instalado.
- ✅ O **acesso SSH** da máquina de controle para a VM alvo está configurado (de preferência com chave pública).
- ✅ O repositório GitHub está clonado localmente ou acessível via CI/CD.

---

## 🛠️ Como Utilizar

### 1. Clonar o repositório
```bash
git clone https://github.com/danilo01arrudal/ol10-pg18-ansible.git
cd ol10-pg18-ansible
```

### 2. Ajustar o inventário
Edite o arquivo `inventory/production` com o IP e usuário da VM alvo:
```ini
[servidores]
ol10pg18 ansible_host=192.168.1.100 ansible_user=root

[servidores:vars]
ansible_python_interpreter=/usr/bin/python3
```

### 3. Escolher a classe da instância
No inventário, adicione a variável `pg_classe` com o valor desejado (`pequena`, `media` ou `grande`):
```ini
ol10pg18 ansible_host=192.168.1.100 ansible_user=root pg_classe=media
```

### 4. Executar o playbook
```bash
ansible-playbook -i inventory/production playbooks/site.yml
```

### 5. Desinstalar (se necessário)
```bash
ansible-playbook -i inventory/production playbooks/uninstall.yml
```

---

## 🔐 Configuração do Runner Auto-Hospedado (para CI/CD)

Para que os workflows do GitHub Actions funcionem, é necessário instalar um runner na VM Oracle Linux 10:

```bash
# Criar diretório e baixar o runner
mkdir ~/actions-runner && cd ~/actions-runner
curl -o actions-runner-linux-x64-2.335.1.tar.gz -L https://github.com/actions/runner/releases/download/v2.335.1/actions-runner-linux-x64-2.335.1.tar.gz
tar xzf ./actions-runner-linux-x64-2.335.1.tar.gz

# Configurar (substitua o token pelo fornecido pelo GitHub)
./config.sh --url https://github.com/danilo01arrudal/ol10-pg18-ansible --token SEU_TOKEN

# Instalar como serviço systemd
./svc.sh install
./svc.sh start
```

---

## 📝 Personalização

### Adicionar uma nova classe de instância
1. Crie um diretório em `group_vars/pg_classe_nova/`.
2. Dentro, crie um arquivo `vars.yml` com os parâmetros desejados.
3. No inventário, defina `pg_classe=nova`.

### Alterar parâmetros padrão
Edite o arquivo `roles/postgresql/vars/main.yml` para modificar os valores padrão da role.

---

## 🤝 Contribuição

Contribuições são bem-vindas! Sinta-se à vontade para abrir issues ou pull requests com melhorias, correções ou novas funcionalidades.

---

## 📄 Licença

Este projeto está sob a licença MIT. Consulte o arquivo [LICENSE](LICENSE) para mais detalhes.

---

## 👤 Autor

**Danilo Arruda**  
- GitHub: [@danilo01arrudal](https://github.com/danilo01arrudal)

---

## 🙏 Agradecimentos

- [PostgreSQL Global Development Group](https://www.postgresql.org/) pela excelente base de dados.
- [Ansible](https://www.ansible.com/) pela poderosa ferramenta de automação.
- [Oracle Linux](https://www.oracle.com/linux/) pela plataforma estável e confiável.
```
