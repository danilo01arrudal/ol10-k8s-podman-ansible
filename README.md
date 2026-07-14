# Provisionamento Automatizado de Cluster Kubernetes Multi-Node com Suporte a GPU NVIDIA via CDI no Oracle Linux 10.1

Este projeto fornece uma automação fim a fim baseada em **Ansible Roles** para implantar um cluster Kubernetes (v1.31) altamente resiliente sobre o **Oracle Linux 10.1**. O ambiente é configurado utilizando o **CRI-O** como runtime de container e integra suporte nativo a workloads de aceleração gráfica (GPU NVIDIA RTX 3060) utilizando a especificação **CDI (Container Device Interface)**.

## 🚀 Visão Geral do Design

A arquitetura de automação separa rigidamente a infraestrutura de controle dos nós de processamento com GPU, garantindo que as dependências do NVIDIA Container Toolkit e as regras CDI sejam injetadas exclusivamente no nó worker.

* **SO Host:** Oracle Linux 10.1 (Kernel UEK ou Red Hat Compatible Kernel)
* **Orquestrador:** Kubernetes v1.31 (kubeadm, kubelet, kubectl)
* **Container Runtime:** CRI-O v1.31 (configurado com Systemd Cgroup driver)
* **Engine do Host:** Podman (nativo do OL10)
* **Suporte a GPU:** NVIDIA Container Toolkit + CDI (Container Device Interface) para isolamento de hardware limpo no CRI-O.

---

## 📂 Estrutura do Projeto

```text
ol10-k8s-gpu-ansible/
├── .github/
│   └── workflows/
│       ├── deploy-k8s.yml
│       └── uninstall-k8s.yml
├── group_vars/
│   └── all/
│       └── vars.yml
├── inventory/
│   └── production
├── playbooks/
│   ├── site.yml
│   └── uninstall.yml
├── roles/
│   └── kubernetes-gpu/
│       ├── handlers/
│       │   └── main.yml
│       └── tasks/
│           └── main.yml
├── ansible.cfg
└── README.md

```

---

## 🛠️ Tecnologias Utilizadas

* **Ansible 2.16+** (Sintaxe compatível com Python 3 no Oracle Linux 10)
* **Kubernetes 1.31** (Repositórios oficiais da comunidade Kubernetes)
* **CRI-O 1.31** (Gerenciado via pacotes oficiais e repositórios openSUSE/K8s)
* **NVIDIA Container Toolkit** (Configuração de runtime para geração de specs CDI v1)
* **GitHub Actions** (Orquestração de CI/CD utilizando runners locais self-hosted)

---

## 🔄 Integração Contínua (CI/CD)

O pipeline é executado em **Runners Self-Hosted** instalados localmente na sua rede interna. Isso permite que o runner do GitHub Actions tenha conectividade SSH direta com os IPs privados dos nós do cluster.

### Triggers do Workflow

* **`workflow_dispatch`**: Permite o disparo manual via interface do GitHub, permitindo selecionar o branch de execução e garantindo controle absoluto sobre quando implantar ou destruir a infraestrutura.

---

## 📋 Pré-requisitos

Antes de iniciar a execução do pipeline ou playbook manual, certifique-se de preencher os seguintes requisitos nos hosts:

1. **Oracle Linux 10.1** instalado de forma limpa em ambas as máquinas.
2. **Driver NVIDIA oficial instalado** no nó `KUBERWORKER1` (recomenda-se a instalação via repositório DKMS da NVIDIA ou runfile oficial do driver de produção).
3. **Acesso SSH sem senha (SSH Keys)** do Runner Self-Hosted do GitHub Actions para ambos os nós do cluster como usuário `root` (ou usuário com privilégios sudo sem senha).
4. O runner self-hosted deve possuir o binário do `ansible-playbook` instalado localmente.

---

## 🏁 Como Utilizar

### Execução via GitHub Actions

1. Configure um **Self-Hosted Runner** no seu repositório do GitHub e garanta que ele esteja ativo (status `Idle`).
2. Adicione as chaves SSH privadas necessárias para acesso aos nós nas configurações do Runner ou garanta que a chave SSH do usuário do runner esteja devidamente distribuída nos nós do inventário.
3. Vá até a aba **Actions** no GitHub.
4. Selecione o workflow **Deploy Kubernetes Cluster** e clique em **Run workflow**.
5. Para destruir o cluster, selecione o workflow **Uninstall Kubernetes Cluster** e execute-o.

### Execução Manual local via CLI

Caso queira disparar a automação diretamente da sua estação de trabalho (dentro do mesmo segmento de rede):

```bash
# Executar a instalação completa
ansible-playbook -i inventory/production playbooks/site.yml

# Executar a desinstalação controlada (Purge)
ansible-playbook -i inventory/production playbooks/uninstall.yml

```
