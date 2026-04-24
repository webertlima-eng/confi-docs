---
title: Terraform
description: 
published: true
date: 2026-04-24T15:27:43.476Z
tags: 
editor: markdown
dateCreated: 2026-04-23T21:33:30.269Z
---


# Terraform — Guia Completo

> Infraestrutura como Código para ambientes **Oracle Cloud** e **Azure**

---

##  Índice

- [Instalação](#instalação)
  - [Windows](#windows)
  - [Linux](#linux)
- [Configuração de CLIs](#configuração-de-clis)
  - [Oracle Cloud CLI — Windows](#oracle-cloud-cli--windows)
  - [Oracle Cloud CLI — Linux](#oracle-cloud-cli--linux)
  - [Azure CLI — Windows](#azure-cli--windows)
  - [Azure CLI — Linux](#azure-cli--linux)
- [Módulos do Terraform](#módulos-do-terraform)
  - [init](#terraform-init)
  - [plan](#terraform-plan)
  - [apply](#terraform-apply)

---

## Instalação

### Windows

**1. Via Chocolatey (recomendado)**

```powershell
# Instalar Chocolatey (caso ainda não tenha)
Set-ExecutionPolicy Bypass -Scope Process -Force; `
[System.Net.ServicePointManager]::SecurityProtocol = `
[System.Net.ServicePointManager]::SecurityProtocol -bor 3072; `
iex ((New-Object System.Net.WebClient).DownloadString('https://community.chocolatey.org/install.ps1'))

# Instalar o Terraform
choco install terraform -y
```

**2. Via Winget**

```powershell
winget install HashiCorp.Terraform
```

**3. Manual**

1. Acesse [https://developer.hashicorp.com/terraform/downloads](https://developer.hashicorp.com/terraform/downloads)
2. Baixe o arquivo `.zip` para Windows (AMD64)
3. Extraia o `terraform.exe` para um diretório (ex.: `C:\terraform`)
4. Adicione o diretório ao `PATH` do sistema:
   - `Painel de Controle` → `Sistema` → `Variáveis de Ambiente`
   - Em **Variáveis do sistema**, selecione `Path` → `Editar` → `Novo` → cole o caminho

**Verificar instalação:**

```powershell
terraform -version
```

---

### Linux

**1. Via repositório oficial HashiCorp (Debian/Ubuntu)**

```bash
# Instalar dependências
sudo apt-get update && sudo apt-get install -y gnupg software-properties-common

# Adicionar chave GPG
wget -O- https://apt.releases.hashicorp.com/gpg | \
gpg --dearmor | \
sudo tee /usr/share/keyrings/hashicorp-archive-keyring.gpg > /dev/null

# Adicionar repositório
echo "deb [signed-by=/usr/share/keyrings/hashicorp-archive-keyring.gpg] \
https://apt.releases.hashicorp.com $(lsb_release -cs) main" | \
sudo tee /etc/apt/sources.list.d/hashicorp.list

# Instalar
sudo apt-get update && sudo apt-get install terraform -y
```

**2. Via repositório oficial HashiCorp (RHEL/CentOS/Amazon Linux)**

```bash
sudo yum install -y yum-utils
sudo yum-config-manager --add-repo https://rpm.releases.hashicorp.com/RHEL/hashicorp.repo
sudo yum install terraform -y
```

**3. Via tfenv (gerenciador de versões — recomendado para múltiplas versões)**

```bash
git clone --depth=1 https://github.com/tfutils/tfenv.git ~/.tfenv
echo 'export PATH="$HOME/.tfenv/bin:$PATH"' >> ~/.bashrc
source ~/.bashrc

tfenv install latest
tfenv use latest
```

**Verificar instalação:**

```bash
terraform -version
```

---

## Configuração de CLIs

### Oracle Cloud CLI — Windows

**1. Instalar o OCI CLI**

```powershell
# Via PowerShell (método oficial)
Set-ExecutionPolicy RemoteSigned -Scope CurrentUser
Invoke-WebRequest https://raw.githubusercontent.com/oracle/oci-cli/master/scripts/install/install.ps1 -OutFile install.ps1
.\install.ps1
```

**2. Configurar credenciais**

```powershell
oci setup config
```

O assistente irá solicitar:

| Campo | Descrição |
|---|---|
| `User OCID` | Encontrado em: **Profile** → **User Settings** no OCI Console |
| `Tenancy OCID` | Encontrado em: **Tenancy Details** no OCI Console |
| `Region` | Ex.: `sa-saopaulo-1`, `us-ashburn-1` |
| `Key pair` | Caminho para sua chave privada `.pem` |

**3. Testar conexão**

```powershell
oci iam user get --user-id <seu-user-ocid>
```

**4. Configurar provider no Terraform**

```hcl
# providers.tf
terraform {
  required_providers {
    oci = {
      source  = "oracle/oci"
      version = "~> 5.0"
    }
  }
}

provider "oci" {
  tenancy_ocid     = var.tenancy_ocid
  user_ocid        = var.user_ocid
  fingerprint      = var.fingerprint
  private_key_path = var.private_key_path
  region           = var.region
}
```

---

### Oracle Cloud CLI — Linux

**1. Instalar o OCI CLI**

```bash
bash -c "$(curl -L https://raw.githubusercontent.com/oracle/oci-cli/master/scripts/install/install.sh)"
```

Aceite os prompts ou passe flags para instalação silenciosa:

```bash
bash -c "$(curl -L https://raw.githubusercontent.com/oracle/oci-cli/master/scripts/install/install.sh)" \
  -- --accept-all-defaults
```

Recarregue o shell:

```bash
source ~/.bashrc
# ou
source ~/.bash_profile
```

**2. Configurar credenciais**

```bash
oci setup config
```

O arquivo de config será criado em `~/.oci/config`:

```ini
[DEFAULT]
user=ocid1.user.oc1..xxxxx
fingerprint=xx:xx:xx:xx
tenancy=ocid1.tenancy.oc1..xxxxx
region=sa-saopaulo-1
key_file=~/.oci/oci_api_key.pem
```

**3. Ajustar permissões da chave**

```bash
chmod 600 ~/.oci/oci_api_key.pem
```

**4. Testar conexão**

```bash
oci iam region list
```

---

### Azure CLI — Windows

**1. Instalar o Azure CLI**

```powershell
# Via Winget
winget install Microsoft.AzureCLI

# Via MSI (instalador gráfico)
# Acesse: https://aka.ms/installazurecliwindows
```

**2. Fazer login**

```powershell
# Login interativo (abre navegador)
az login

# Login com Service Principal (para CI/CD e automações)
az login --service-principal \
  --username <APP_ID> \
  --password <CLIENT_SECRET> \
  --tenant <TENANT_ID>
```

**3. Selecionar subscription**

```powershell
# Listar subscriptions disponíveis
az account list --output table

# Definir subscription ativa
az account set --subscription "<SUBSCRIPTION_ID>"
```

**4. Configurar provider no Terraform**

```hcl
# providers.tf
terraform {
  required_providers {
    azurerm = {
      source  = "hashicorp/azurerm"
      version = "~> 3.0"
    }
  }
}

provider "azurerm" {
  features {}

  subscription_id = var.subscription_id
  tenant_id       = var.tenant_id
  # Para Service Principal:
  # client_id       = var.client_id
  # client_secret   = var.client_secret
}
```

---

### Azure CLI — Linux

**1. Instalar o Azure CLI**

```bash
# Ubuntu / Debian
curl -sL https://aka.ms/InstallAzureCLIDeb | sudo bash

# RHEL / CentOS
sudo rpm --import https://packages.microsoft.com/keys/microsoft.asc
sudo dnf install -y https://packages.microsoft.com/config/rhel/9.0/packages-microsoft-prod.rpm
sudo dnf install azure-cli -y
```

**2. Fazer login**

```bash
# Login interativo
az login

# Login via Service Principal (ambientes sem navegador / CI)
az login --service-principal \
  --username $ARM_CLIENT_ID \
  --password $ARM_CLIENT_SECRET \
  --tenant $ARM_TENANT_ID
```

**3. Configurar variáveis de ambiente (recomendado para automações)**

```bash
export ARM_SUBSCRIPTION_ID="xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx"
export ARM_TENANT_ID="xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx"
export ARM_CLIENT_ID="xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx"
export ARM_CLIENT_SECRET="sua-secret-aqui"
```

> 💡 Adicione essas variáveis no `~/.bashrc` ou `~/.bash_profile` para persistência.

**4. Testar conexão**

```bash
az account show
```

---

## Módulos do Terraform

### 📦 `terraform init`

Inicializa o diretório de trabalho do Terraform. Deve ser executado **sempre** antes de qualquer outra operação ou ao adicionar/alterar providers e módulos.

**O que ele faz:**
- Baixa os providers declarados no `required_providers`
- Inicializa o backend de estado (local, S3, Azure Blob, OCI Object Storage, etc.)
- Instala módulos externos referenciados

```bash
# Inicialização básica
terraform init

# Forçar re-download dos providers (útil após trocar versões)
terraform init -upgrade

# Inicializar apontando para um backend específico
terraform init -backend-config="storage_account_name=meu-storage"

# Inicializar sem verificar plugins (offline)
terraform init -get=false
```

**Saída esperada:**

```
Initializing the backend...
Initializing provider plugins...
- Finding oracle/oci versions matching "~> 5.0"...
- Installing oracle/oci v5.x.x...
Terraform has been successfully initialized!
```

> ⚠️ O diretório `.terraform/` e o arquivo `.terraform.lock.hcl` são gerados pelo `init`. Versionar o `.lock.hcl` no Git garante consistência entre a equipe.

---

### 🔍 `terraform plan`

Gera um **plano de execução** mostrando exatamente o que será criado, modificado ou destruído — sem aplicar nenhuma mudança.

**O que ele faz:**
- Lê o estado atual (`terraform.tfstate`)
- Compara com a configuração desejada nos arquivos `.tf`
- Exibe as diferenças (diff)

```bash
# Plano básico
terraform plan

# Salvar o plano em arquivo (recomendado para pipelines CI/CD)
terraform plan -out=tfplan

# Plano passando variáveis inline
terraform plan -var="environment=prod" -var="region=sa-saopaulo-1"

# Plano usando arquivo de variáveis
terraform plan -var-file="prod.tfvars"

# Plano de destruição (simular terraform destroy)
terraform plan -destroy
```

**Legenda da saída:**

| Símbolo | Significado |
|---|---|
| `+` | Recurso será **criado** |
| `-` | Recurso será **destruído** |
| `~` | Recurso será **modificado** |
| `-/+` | Recurso será **recriado** (destroy + create) |

**Exemplo de saída:**

```
Terraform will perform the following actions:

  # oci_core_vcn.main will be created
  + resource "oci_core_vcn" "main" {
      + cidr_block     = "10.0.0.0/16"
      + display_name   = "vcn-producao"
      + id             = (known after apply)
    }

Plan: 1 to add, 0 to change, 0 to destroy.
```

> ✅ **Boa prática:** sempre revise o `plan` antes de fazer o `apply`, especialmente em ambientes de produção.

---

### 🚀 `terraform apply`

Aplica as mudanças planejadas, **provisionando ou modificando** a infraestrutura real.

**O que ele faz:**
- Executa um `plan` internamente (a menos que um arquivo de plano seja fornecido)
- Solicita confirmação interativa antes de aplicar
- Atualiza o arquivo de estado `terraform.tfstate`

```bash
# Apply interativo (solicita confirmação)
terraform apply

# Apply usando plano salvo anteriormente (sem confirmação — ideal para CI/CD)
terraform apply tfplan

# Apply passando variáveis inline
terraform apply -var="environment=prod"

# Apply usando arquivo de variáveis
terraform apply -var-file="prod.tfvars"

# Apply sem confirmação interativa (⚠️ use com cautela)
terraform apply -auto-approve

# Limitar paralelismo (útil para evitar rate limiting de APIs)
terraform apply -parallelism=5
```

**Fluxo recomendado para produção:**

```bash
# 1. Gerar e salvar o plano
terraform plan -out=tfplan -var-file="prod.tfvars"

# 2. Revisar o plano gerado
terraform show tfplan

# 3. Aplicar exatamente o plano aprovado
terraform apply tfplan
```

**Exemplo de saída:**

```
oci_core_vcn.main: Creating...
oci_core_vcn.main: Creation complete after 3s [id=ocid1.vcn.oc1...]

Apply complete! Resources: 1 added, 0 changed, 0 destroyed.

Outputs:
  vcn_id = "ocid1.vcn.oc1.sa-saopaulo-1.xxxxx"
```

> ⚠️ **Atenção:** nunca use `-auto-approve` diretamente em produção sem antes validar o `plan`. Prefira pipelines com aprovação manual entre o `plan` e o `apply`.

---

## Estrutura de Projeto Recomendada

```
terraform/
├── main.tf               # Recursos principais
├── providers.tf          # Declaração de providers
├── variables.tf          # Declaração de variáveis
├── outputs.tf            # Outputs do módulo
├── versions.tf           # Restrições de versão do Terraform
├── terraform.tfvars      # Valores padrão das variáveis (não versionar secrets)
├── prod.tfvars           # Valores para produção
├── dev.tfvars            # Valores para desenvolvimento
└── modules/
    ├── network/          # Módulo de rede
    ├── compute/          # Módulo de computação
    └── storage/          # Módulo de armazenamento
```

---

## Referências

- [Terraform Docs — HashiCorp](https://developer.hashicorp.com/terraform/docs)
- [OCI Provider — Terraform Registry](https://registry.terraform.io/providers/oracle/oci/latest/docs)
- [AzureRM Provider — Terraform Registry](https://registry.terraform.io/providers/hashicorp/azurerm/latest/docs)
- [OCI CLI — Documentação Oracle](https://docs.oracle.com/en-us/iaas/tools/oci-cli/latest/oci_cli_docs/)
- [Azure CLI — Documentação Microsoft](https://learn.microsoft.com/pt-br/cli/azure/)