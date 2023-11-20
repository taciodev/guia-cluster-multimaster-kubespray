# Guia de Configuração do Cluster Kubernetes com Kubespray

Este guia fornecerá instruções passo a passo para configurar um cluster Kubernetes usando o Kubespray. Antes de começar, certifique-se de executar esses passos em um host Ansible (Notebook) com as seguintes preparações.

## 1. Preparação do Ansible Host (Notebook)

Crie uma pasta para organizar a estrutura do seu projeto (substitua "/home/nilson/Documentos/lab-k8s/" pelo seu caminho desejado):

```bash
mkdir -p /home/nilson/Documentos/lab-k8s/
```

Atualize os pacotes e instale as dependências necessárias:

```bash
sudo apt update
sudo apt install git python3 python3-pip -y
git clone https://github.com/kubernetes-sigs/kubespray.git
cd kubespray
pip install -r requirements
```

Verifique a versão do Ansible com:

```bash
ansible --version
```

## 2. Preparação do Kubespray

Crie um diretório para o ambiente Kubespray e ajuste os IPs no arquivo de hosts.yaml:

```bash
cp -rfp inventory/sample inventory/mycluster
declare -a IPS=(192.168.50.11 192.168.50.12 192.168.50.13 192.168.50.14 192.168.50.15)
CONFIG_FILE=inventory/mycluster/hosts.yaml python3 contrib/inventory_builder/inventory.py ${IPS[@]}
```

Modifique o arquivo hosts.yaml conforme necessário:

```bash
vi inventory/mycluster/hosts.yaml
```

Exemplo do conteúdo do arquivo:

```bash
all:
  hosts:
    k8s-node-1:
      ansible_host: 192.168.50.11
      ip: 192.168.50.11
      access_ip: 192.168.50.11
    k8s-node-2:
      ansible_host: 192.168.50.12
      ip: 192.168.50.12
      access_ip: 192.168.50.12
    # ... (repetir para os demais nós)
  children:
    kube_control_plane:
      hosts:
        k8s-node-1:
        k8s-node-2:
        # ... (repetir para os demais nós)
    kube_node:
      hosts:
        k8s-node-1:
        k8s-node-2:
        # ... (repetir para os demais nós)
    etcd:
      hosts:
        k8s-node-1:
        k8s-node-2:
        # ... (repetir para os demais nós)
    k8s_cluster:
      children:
        kube_control_plane:
        kube_node:
    calico_rr:
      hosts: {}
  vars:
      ansible_ssh_private_key_file: ~/.ssh/id_rsa
```
