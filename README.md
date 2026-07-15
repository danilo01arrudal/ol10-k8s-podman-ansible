# ☸️ Provisionamento Automatizado de um Cluster Kubernetes Single-Node com Suporte a GPU NVIDIA via CDI no Oracle Linux 10.1

[![GitHub](https://img.shields.io/badge/Repository-danilo01arrudal/ol10--k8s--podman--ansible-blue?logo=github)](https://github.com/danilo01arrudal/ol10-k8s-podman-ansible)
[![Ansible](https://img.shields.io/badge/Ansible-2.16+-black?logo=ansible)](https://www.ansible.com/)
[![Kubernetes](https://img.shields.io/badge/Kubernetes-1.31-326ce5?logo=kubernetes)](https://kubernetes.io/)
[![Oracle Linux](https://img.shields.io/badge/Oracle%20Linux-10.1-red?logo=oracle)](https://www.oracle.com/linux/)

---

## 📋 Visão Geral

[cite_start]Este projeto tem como objetivo **automatizar o processo de instalação, configuração, orquestração e gerenciamento de um cluster Kubernetes v1.31 Single-Node (All-in-One) em ambiente Oracle Linux 10.1**, utilizando **Ansible** como ferramenta de automação[cite: 13].
[cite_start]A solução foi projetada especificamente para consolidar toda a infraestrutura no host físico **192.168.0.40**, acumulando os papéis de Control Plane e nó de processamento (Worker)[cite: 14].

[cite_start]O grande diferencial desta arquitetura de infraestrutura como código (IaC) é a injeção nativa de suporte a workloads de Inteligência Artificial e aceleração gráfica para a GPU **NVIDIA RTX 3060** usando a especificação **CDI (Container Device Interface)** integrada diretamente sobre o Container Runtime Interface **CRI-O 1.31** e utilizando o motor de containers do host **Podman**[cite: 15].

A solução foi projetada para ser:
- [cite_start]**Reprodutível**: Todo o ciclo de vida do cluster (instalação e expurgo) é descrito estritamente como código (IaC)[cite: 16].
- [cite_start]**Otimizada para IA & Single-Host**: Ideal para laboratórios de desenvolvimento, engenharia de dados e inferência local, eliminando o overhead de múltiplos servidores físicos[cite: 17].
- [cite_start]**Altamente Integrada**: Provisionamento dinâmico via pipeline de CI/CD GitHub Actions executado em runners locais auto-hospedados (*self-hosted*)[cite: 18].
- [cite_start]**Segura**: Comunicação baseada em SSH Key pairs de nível administrativo e total conformidade com Cgroupsv2 (Systemd driver)[cite: 19].

<img width="1408" height="768" alt="image" src="https://github.com/danilo01arrudal/ol10-k8s-podman-ansible/blob/main/images/0000001.png" />

---

## 🚀 Principais Funcionalidades

| Funcionalidade | Descrição |
|----------------|-----------|
| **Preparação do Host** | [cite_start]Carga automatizada dos módulos `overlay`/`br_netfilter` e parametrização do Sysctl para segurança e roteamento de rede do nó único[cite: 21]. |
| **Runtime de Container Isolado** | [cite_start]Instalação e provisionamento do CRI-O 1.31 como o motor do Kubernetes sob as regras do Systemd Cgroup Driver[cite: 22]. |
| **Instalação do Kubernetes** | [cite_start]Provisionamento estruturado dos binários `kubeadm`, `kubectl` e `kubelet` v1.31 fixados via gerenciador DNF[cite: 23]. |
| **Aceleração via CDI Local** | [cite_start]Detecção automatizada de hardware NVIDIA no host único, injeção do NVIDIA Container Toolkit e geração dinâmica da especificação CDI `/etc/cdi/nvidia.yaml`[cite: 24]. |
| **Desinstalação Controlada (Purge)** | [cite_start]Limpeza profunda do host físico, incluindo remoção completa de pacotes do DNF, exclusão de interfaces de rede residuais do CRI-O e reversão de regras do IPtables[cite: 25]. |

---

## 🏗️ Estrutura do Projeto

```plaintext
ol10-k8s-podman-ansible/
├── .github/
│   └── workflows/
[cite_start]│       ├── deploy-k8s.yml          # Pipeline de deploy automatizado via Actions [cite: 26]
[cite_start]│       └── uninstall-k8s.yml       # Pipeline de desinstalação controlada via Actions [cite: 26]
├── inventory/
│   ├── group_vars/
│   │   └── all/
[cite_start]│   │       └── vars.yml            # Variáveis globais do cluster (K8s, CRI-O, Repositórios) [cite: 1, 26]
[cite_start]│   └── production                  # Inventário simplificado mapeando o host único (All-in-One) [cite: 1, 26, 27]
├── playbooks/
[cite_start]│   ├── site.yml                    # Playbook principal de provisionamento da role [cite: 1, 27]
[cite_start]│   └── uninstall.yml               # Playbook de destruição, limpeza e rollback total do host [cite: 1, 27]
├── roles/
│   └── kubernetes-gpu/
│       ├── handlers/
[cite_start]│       │   └── main.yml            # Gatilhos para recarga do CRI-O e parâmetros de Sysctl [cite: 1, 27, 28]
│       └── tasks/
[cite_start]│           └── main.yml            # Tasks lineares ordenadas para execução da arquitetura no nó único [cite: 1, 28]
[cite_start]├── ansible.cfg                     # Configurações de conexão estáveis para o Oracle Linux 10 [cite: 1, 28]
[cite_start]└── README.md                       # Este arquivo descritivo [cite: 1, 28, 29]

```

---

## 🧩 Principais Arquivos e suas Funções

#### `playbooks/site.yml`

O playbook principal que orquestra todo o processo de deploy. Ele chama de forma declarativa a role de automação `kubernetes-gpu` aplicando as tarefas de forma direta ao host único mapeado no grupo `k8s_node`.

#### `playbooks/uninstall.yml`

Playbook de emergência e governança focado no estado de expurgo total. Remove todos os binários, travas de arquivos, dados de volumes do Kubelet, limpa as regras de IPtables que poluem as chains de roteamento de rede e retorna o estado operacional limpo para o servidor.

#### `inventory/group_vars/all/vars.yml`

Centraliza a declaração de versões globais estáveis (`kubernetes_version: "1.31"` e `crio_version: "1.31"`), módulos obrigatórios do kernel Linux e endpoints de repositórios confiáveis de RPM para o Oracle Linux 10.

#### `roles/kubernetes-gpu/tasks/main.yml`

Contém a sequência de automação fim a fim. Como o ambiente é Single-Node, as tarefas de preparação de rede, instalação do Kubernetes, injeção do NVIDIA Container Toolkit e mapeamento CDI são executadas de forma homogênea e direta no host físico, sem a necessidade de filtros condicionais por grupo.

---

## ⚙️ Tecnologias Utilizadas

| Tecnologia | Versão | Finalidade |
| --- | --- | --- |
| **Oracle Linux** | 10.1 | Sistema operacional corporativo estável para hosts e nós.
| **Kubernetes** | 1.31 | Plataforma de orquestração de containers de nível de produção.
| **CRI-O** | 1.31 | Container Runtime nativo leve otimizado para o ecossistema Kubernetes.
| **Podman** | Nativo | Motor nativo de gerenciamento local de imagens no host Oracle Linux 10.
| **NVIDIA CDI** | v1 | Container Device Interface para exposição direta da GPU RTX 3060 ao CRI-O.
| **Ansible** | 2.16+ | Orquestração declarativa da Infraestrutura como Código (IaC).

---

## 🔄 Integração Contínua (CI/CD) com GitHub Actions

O projeto possui automação integrada via dois fluxos declarativos no GitHub Actions (`.github/workflows/`):

#### 1. `deploy-k8s.yml`
*  **Trigger**: `workflow_dispatch` (Executado de forma manual sob demanda na aba "Actions").
  
*  **Executor**: `self-hosted` (Executado no runner interno instalado em sua infraestrutura).
  
*  **Escopo**: Garante integridade do checkout do código, validação de sintaxe e chamada de provisionamento do `playbooks/site.yml`.

#### 2. `uninstall-k8s.yml`

*  **Trigger**: `workflow_dispatch` (Executado manualmente).

*  **Executor**: `self-hosted`.
 
*  **Escopo**: Invoca de maneira controlada a remoção completa da infraestrutura do cluster a partir do playbook `playbooks/uninstall.yml`.

---

## 📦 Pré-requisitos

Antes de iniciar a execução da automação, certifique-se de validar os seguintes pontos de conformidade no nó de destino:

* ✅ **Acesso SSH Sem Senha**: O runner local deve possuir chave SSH pública implantada no arquivo `/root/.ssh/authorized_keys` do host físico (`192.168.0.40`).


* ✅ **NVIDIA Drivers Instalados**: O driver proprietário estável da NVIDIA deve estar previamente configurado e operacional no host (com validação funcional via comando `nvidia-smi`).


* ✅ **Python 3 Instalado**: O host deve ter o pacote `python3` nativo no caminho `/usr/bin/python3`.



---

## 🛠️ Como Utilizar

#### 1. Clonar o repositório

```bash
git clone [https://github.com/danilo01arrudal/ol10-k8s-podman-ansible.git](https://github.com/danilo01arrudal/ol10-k8s-podman-ansible.git)
cd ol10-k8s-podman-ansible

```

#### 2. Validar ou Ajustar o Inventário

Verifique as configurações das suas instâncias no arquivo `inventory/production`:

```ini
[k8s_node]
KUBERSINGLE ansible_host=192.168.0.40

```

#### 3. Execução Manual via Terminal local

Para disparar localmente toda a infraestrutura diretamente da sua máquina de controle:

```bash
ansible-playbook -i inventory/production playbooks/site.yml

```

#### 4. Validação Pós-Instalação (Automatizado)

Com a nova arquitetura da nossa automação Ansible, você **não precisa executar nenhum comando manual** de inicialização (`kubeadm init`), configuração de credenciais administrativas, instalação do plugin de rede Flannel ou destravamento de taints. Tudo isso já é executado de forma 100% autônoma pela pipeline.

Para interagir com o seu cluster recém-criado, basta acessar o terminal do servidor `192.168.0.40` e executar as validações diretamente:

```bash
# 1. Verificar se o nó único está ativo e pronto (Ready)
kubectl get nodes

# 2. Confirmar se todos os Pods de sistema (Flannel, CoreDNS, Kube-Proxy) estão rodando
kubectl get pods -A

# 3. Testar a injeção nativa de aceleração gráfica por GPU RTX 3060 via CDI
# (O Ansible já valida essa injeção no final do deploy, mas você pode checar quando quiser)
podman run --rm --device [nvidia.com/gpu=all](https://nvidia.com/gpu=all) docker.io/nvidia/cuda:12.0.0-base-ubuntu22.04 nvidia-smi

```

**Ações automatizadas executadas por baixo dos panos pelo Ansible:**

1. **Inicialização do Control Plane** utilizando o socket correto do CRI-O.
2. **Configuração nativa de credenciais** sob `/root/.kube/config` para uso imediato do `kubectl`.
3. **Implantação automática do Flannel** ( plugin de rede CNI).
4. **Remoção de taints** de Control Plane e Master, liberando o nó único para agendar workloads comuns de forma imediata.
5. **Autovalidação CDI de GPU** no Host.

#### 5. Execução de Desinstalação (Reset Completo do Ambiente)

Para reverter todas as modificações aplicadas ao servidor e limpar completamente os módulos de kernel, credenciais e pacotes instalados:

```bash
ansible-playbook -i inventory/production playbooks/uninstall.yml

```

---

## 🔐 Configuração do Runner Auto-Hospedado (Self-Hosted para CI/CD)

Para integrar a automação nativamente aos workflows do GitHub Actions do seu repositório, configure o seu **Runner Self-Hosted** diretamente no host local ou em um nó que possua acesso de rede ao servidor do cluster:

```bash
# Criar o diretório de destino do runner
mkdir ~/actions-runner && cd ~/actions-runner

# Download do runner estável para plataforma Linux x64
curl -o actions-runner-linux-x64-2.335.1.tar.gz -L [https://github.com/actions/runner/releases/download/v2.335.1/actions-runner-linux-x64-2.335.1.tar.gz](https://github.com/actions/runner/releases/download/v2.335.1/actions-runner-linux-x64-2.335.1.tar.gz)
tar xzf ./actions-runner-linux-x64-2.335.1.tar.gz

# Configuração e Registro (Substitua "SEU_TOKEN" com o Token dinâmico provido pelas configurações do GitHub)
./config.sh --url [https://github.com/danilo01arrudal/ol10-k8s-podman-ansible](https://github.com/danilo01arrudal/ol10-k8s-podman-ansible) --token SEU_TOKEN

# Instalação e inicialização do serviço do runner em nível de systemd daemon
sudo ./svc.sh install
sudo ./svc.sh start

```

---

## 🤝 Contribuição

Contribuições para aprimorar as regras de configuração CDI e automação do Kubernetes no Oracle Linux 10 são sempre muito bem-vindas! Sinta-se confortável para abrir novas issues ou submeter Pull Requests.

---

## 📄 Licença

Este projeto é disponibilizado sob a licença comercial/pessoal **MIT**. Consulte o arquivo [LICENSE](https://www.google.com/search?q=https://github.com/danilo01arrudal/ol10-k8s-podman-ansible/blob/main/LICENSE) para mais detalhes.

---

## 👤 Autor

**Danilo Arruda** - GitHub: [@danilo01arrudal](https://github.com/danilo01arrudal) 

## 🙏 Agradecimentos

* 
[Ansible](https://www.ansible.com/) pela poderosa ferramenta de automação.


* 
[Oracle Linux](https://www.oracle.com/linux/) pela plataforma estável e confiável.


* 
[Kubernetes](https://kubernetes.io/) pela ferramenta de gerenciamento de containers.



```

```
