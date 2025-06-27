# ☁️ Gerenciando Máquinas Virtuais no Azure  
## 🚀 Implantação de Máquinas Virtuais e 🔌 Desanexando Discos

> **📘 Objetivo:** Aprender de forma prática e estruturada como implantar uma VM no Microsoft Azure e desanexar discos adicionais de forma segura e eficiente.

---

## 📌 Tópicos abordados

- [📦 Introdução às VMs no Azure](#-introdução-às-vms-no-azure)
- [🛠️ Implantação de uma Máquina Virtual](#-implantação-de-uma-máquina-virtual)
- [💾 Como desanexar um disco de uma VM](#-como-desanexar-um-disco-de-uma-vm)
- [💡 Dicas práticas e boas práticas](#-dicas-práticas-e-boas-práticas)
- [📚 Recursos adicionais](#-recursos-adicionais)

---

## 📦 Introdução às VMs no Azure

As Máquinas Virtuais (VMs) do Azure oferecem a flexibilidade de virtualização da infraestrutura com controle total do sistema operacional, armazenamento e rede. São usadas para:

- Hospedar aplicações
- Realizar testes e desenvolvimento
- Rodar serviços em ambientes escaláveis

---

## 🛠️ Implantação de uma Máquina Virtual

> Vamos criar uma VM Linux (Ubuntu) usando o portal Azure.

### ✅ Passo a passo

1. **Acesse o Portal do Azure**  
   🔗 [portal.azure.com](https://portal.azure.com)

2. **Pesquise por “Máquinas Virtuais”**  
   Clique em **"Criar" > "Máquina Virtual"**.

3. **Configure os parâmetros básicos**:
   - **Assinatura:** Selecione a sua
   - **Grupo de Recursos:** Crie um novo ou reutilize um existente
   - **Nome da VM:** ex: `vm-producao`
   - **Região:** Escolha a mais próxima (ex: Brazil South)
   - **Imagem:** Ubuntu 22.04 LTS
   - **Tamanho:** Standard B2s (2 vCPU, 4 GB RAM é uma boa escolha inicial)
   - **Autenticação:** Chave SSH ou Senha

4. **Configurar disco**
   - Disco do SO: SSD padrão
   - Adicionar disco de dados se desejar (pode ser feito posteriormente também)

5. **Configurar rede**
   - Criar uma nova rede virtual ou usar uma existente
   - Habilite IP público se necessário

6. **Revisar e criar**
   - Clique em "Revisar + Criar"
   - Depois clique em "Criar"

🎉 **Pronto! Sua VM está em processo de provisionamento.**

---

## 💾 Como desanexar um disco de uma VM

> Desanexar um disco de dados de uma VM pode ser útil para manutenção, backup ou migração.

### ⚠️ Antes de começar:

- **Nunca remova o disco do SO**
- **Desmonte o disco dentro da VM antes de desanexar (Linux: `umount`)**

### 🔧 Passo a passo

1. **Acesse o Portal do Azure**
2. Vá para **Máquinas Virtuais** e selecione a VM desejada
3. No menu lateral, clique em **Discos**
4. Identifique o **disco de dados** (normalmente chamado `datadisk1`, `datadisk2`, etc.)
5. Clique no botão **"Desanexar"** ao lado do disco
6. Confirme a ação

✅ O disco foi desanexado com sucesso!  
📦 Ele continua disponível em **"Discos"** no portal e pode ser reutilizado em outra VM.

---

## 💡 Dicas práticas e boas práticas

🧠 **Dica 1: Nomeação clara**
> Use nomes padrão e descritivos: `vm-dev-ubuntu`, `disk-vm-analytics`, `rg-produtos-vm`

🧠 **Dica 2: Tags são poderosas**
> Adicione tags como `ambiente:produção`, `projeto:ecommerce`, `owner:caio`  
> Isso facilita rastrear custos e gerenciar recursos

🧠 **Dica 3: Backup antes de remover**
> Se o disco contiver dados importantes, faça um snapshot antes de desanexar

🧠 **Dica 4: Automação com CLI**
> Use o **Azure CLI** para automatizar o gerenciamento de discos:

```bash
# Desanexar disco com Azure CLI
az vm disk detach --resource-group MeuGrupo --vm-name MinhaVM --name MeuDisco
```

## 🐍 Python Para Tudo

### 🧰 Pré-requisitos

Antes de rodar os códigos abaixo, instale os pacotes:
```python
pip install azure-identity azure-mgmt-compute azure-mgmt-resource
```

### ✅ Exemplo 1: Criar uma Máquina Virtual (Linux)

```python
from azure.identity import DefaultAzureCredential
from azure.mgmt.compute import ComputeManagementClient
from azure.mgmt.resource import ResourceManagementClient

subscription_id = "SEU_ID_DE_ASSINATURA"
resource_group = "meu-grupo"
location = "brazilsouth"
vm_name = "minha-vm-python"

# Autenticação
credential = DefaultAzureCredential()
compute_client = ComputeManagementClient(credential, subscription_id)
resource_client = ResourceManagementClient(credential, subscription_id)

# 1. Criar grupo de recursos (caso não exista)
resource_client.resource_groups.create_or_update(resource_group, {"location": location})

# 2. Criar VM simples (modelo básico, customizável)
vm_parameters = {
    "location": location,
    "storage_profile": {
        "image_reference": {
            "publisher": "Canonical",
            "offer": "UbuntuServer",
            "sku": "22_04-lts",
            "version": "latest"
        }
    },
    "hardware_profile": {"vm_size": "Standard_B1s"},
    "os_profile": {
        "computer_name": vm_name,
        "admin_username": "azureuser",
        "admin_password": "SenhaForte123!"  # Evite hardcode de senhas em produção
    },
    "network_profile": {
        "network_interfaces": [{
            "id": "/subscriptions/SEU_ID/resourceGroups/meu-grupo/providers/Microsoft.Network/networkInterfaces/NIC_ID"
        }]
    }
}

# 3. Criar VM
async_vm_creation = compute_client.virtual_machines.begin_create_or_update(
    resource_group,
    vm_name,
    vm_parameters
)

vm_result = async_vm_creation.result()
print(f"✅ VM '{vm_name}' criada com sucesso!")
```

> ⚠️ Este exemplo assume que você já criou a interface de rede (NIC) e sub-rede. Para scripts 100% completos com criação de VNet, IP, NSG e NIC, me avise que posso incluir.

### 🔌 Exemplo 2: Desanexar um disco de dados de uma VM

```python
from azure.identity import DefaultAzureCredential
from azure.mgmt.compute import ComputeManagementClient

subscription_id = "SEU_ID_DE_ASSINATURA"
resource_group = "meu-grupo"
vm_name = "minha-vm-python"
data_disk_name = "meu-disco-dados"

# Autenticação
credential = DefaultAzureCredential()
compute_client = ComputeManagementClient(credential, subscription_id)

# 1. Obter a VM atual
vm = compute_client.virtual_machines.get(resource_group, vm_name)

# 2. Filtrar discos e remover o desejado
original_disks = vm.storage_profile.data_disks
vm.storage_profile.data_disks = [
    disk for disk in original_disks if disk.name != data_disk_name
]

# 3. Atualizar a VM
async_vm_update = compute_client.virtual_machines.begin_create_or_update(
    resource_group,
    vm_name,
    vm
)
async_vm_update.result()
print(f"🔌 Disco '{data_disk_name}' desanexado com sucesso da VM '{vm_name}'!")
```

### 💡 Dica Extra: Listar todos os discos anexados

```python
vm = compute_client.virtual_machines.get(resource_group, vm_name)
for disk in vm.storage_profile.data_disks:
    print(f"📦 Disco: {disk.name} | LUN: {disk.lun}")
```



