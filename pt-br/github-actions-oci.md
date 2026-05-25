---
title: GitHub Actions + OCI Authentication (OIDC e API Key)
description: 
published: true
date: 2026-05-25T15:15:27.007Z
tags: 
editor: markdown
dateCreated: 2026-05-18T15:01:08.546Z
---

# GitHub Actions + OCI Authentication (OIDC e API Key)

## Objetivo

Documentar a integração entre GitHub Actions e Oracle Cloud Infrastructure (OCI) para execução de pipelines CI/CD, Terraform e deploys no OKE.

Este documento descreve:

* OIDC (OpenID Connect) com GitHub
* Limitações atuais do OCI Identity Domain Free
* Modelo recomendado para produção
* Configuração segura com usuário técnico + API Key
* Estrutura IAM necessária
* Workflow GitHub Actions funcional
* Boas práticas de segurança

---

# Resultado Final da POC

A POC validou com sucesso:

* GitHub Actions funcionando
* OCI CLI autenticado
* Listagem de compartments OCI
* Integração GitHub → OCI
* Execução segura via GitHub Secrets
* Estrutura IAM funcional

Foi possível listar os compartments reais da tenancy OCI utilizando GitHub Actions.

---

# OIDC na OCI

## O que foi validado

Foi validado:

* Emissão de JWT OIDC pelo GitHub
* Claims do token JWT
* Audience `sts.oraclecloud.com`
* Dynamic Groups OCI
* Policies OCI
* OAuth Client OCI

O GitHub gerou corretamente tokens JWT temporários.

---

# Limitação Encontrada

A tenancy OCI utilizada possui:

* Identity Domain Free
* Recursos limitados de Workload Identity Federation

Durante os testes foi identificado que o OCI Identity Domain Free não suporta completamente o fluxo moderno de:

```text
GitHub JWT
→ OCI Token Exchange
→ OCI Session Token
→ OCI CLI Federation
```

O endpoint OAuth da OCI rejeitou o fluxo RFC Token Exchange.

Por este motivo, o modelo OIDC puro (sem API Key) não ficou funcional para uso completo do OCI CLI/Terraform.

---

# Conclusão Técnica

## OIDC

OIDC funciona parcialmente:

| Recurso            | Status                       |
| ------------------ | ---------------------------- |
| GitHub JWT         | OK                           |
| Claims JWT         | OK                           |
| Audience OCI       | OK                           |
| OAuth Client OCI   | OK                           |
| Dynamic Groups     | OK                           |
| Policies           | OK                           |
| OCI Token Exchange | Limitado                     |
| OCI CLI Federation | Não funcional no Domain Free |

---

# Arquitetura Recomendada

## Recomendação Final

Para ambientes reais utilizando OCI + GitHub Actions:

### Utilizar:

* GitHub Actions
* Usuário técnico OCI
* API Key OCI
* Policies mínimas
* Compartments segregados

---

# Modelo Recomendado

```text
GitHub Actions
↓
Workflow autorizado
↓
GitHub Repository Secrets
↓
OCI API Key (usuário técnico)
↓
Terraform / OCI CLI
↓
OKE / Infraestrutura OCI
```

---

# Por que este modelo?

Porque atualmente:

* OCI Federation ainda possui limitações
* Identity Domain Free possui recursos reduzidos
* OCI CLI/Terraform possuem suporte mais estável para API Key
* É o modelo mais utilizado em produção na OCI

---

# Segurança do Modelo

Mesmo utilizando API Key:

* As credenciais ficam protegidas nos GitHub Secrets
* O usuário possui permissões mínimas
* Pode ser criado exclusivamente para CI/CD
* Permite rotação simples das chaves
* Não utiliza contas pessoais

---

# Estrutura Recomendada na OCI

## 1. Criar usuário técnico

Exemplo:

```text
svc-github-actions
```

Este usuário será utilizado exclusivamente pelo CI/CD.

Nunca utilizar usuários pessoais.

---

# 2. Criar grupo IAM

Exemplo:

```text
github-actions-ci
```

---

# 3. Adicionar usuário ao grupo

```text
svc-github-actions → github-actions-ci
```

---

# 4. Gerar API Key

No usuário técnico:

```text
Identity & Security
→ Domains
→ Users
→ svc-github-actions
→ API Keys
→ Add API Key
```

Gerar:

* Private Key
* Public Key
* Fingerprint

---

# 5. Upload da Public Key

Enviar a public key para o usuário OCI.

A OCI irá gerar:

* fingerprint
* vinculação da chave

---

# Policies OCI

## Exemplo para Terraform/OKE

```text
Allow group github-actions-ci to manage cluster-family in compartment staging
Allow group github-actions-ci to manage instance-family in compartment staging
Allow group github-actions-ci to manage virtual-network-family in compartment staging
Allow group github-actions-ci to manage volume-family in compartment staging
```

---

# Princípio do Menor Privilégio

Sempre limitar:

* compartments
* recursos
* ambientes
* operações

Evitar:

```text
manage all-resources in tenancy
```

---

# GitHub Repository Secrets

## Secrets necessários

| Secret           | Descrição                   |
| ---------------- | --------------------------- |
| OCI_USER_OCID    | OCID do usuário técnico     |
| OCI_TENANCY_OCID | OCID da tenancy             |
| OCI_FINGERPRINT  | Fingerprint da API Key      |
| OCI_PRIVATE_KEY  | Conteúdo da private key PEM |

---

# Workflow GitHub Actions

## Exemplo funcional

```yaml
name: OCI API Key Test

on:
  workflow_dispatch:

permissions:
  contents: read

jobs:
  test:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout
        uses: actions/checkout@v4

      - name: Install OCI CLI
        run: |
          bash -c "$(curl -L https://raw.githubusercontent.com/oracle/oci-cli/master/scripts/install/install.sh)" -- --accept-all-defaults
          echo "$HOME/bin" >> $GITHUB_PATH

      - name: Configure OCI CLI
        run: |
          mkdir -p ~/.oci

          cat > ~/.oci/config <<EOF
          [DEFAULT]
          user=${{ secrets.OCI_USER_OCID }}
          fingerprint=${{ secrets.OCI_FINGERPRINT }}
          tenancy=${{ secrets.OCI_TENANCY_OCID }}
          region=sa-saopaulo-1
          key_file=/tmp/oci_key.pem
          EOF

          echo "${{ secrets.OCI_PRIVATE_KEY }}" > /tmp/oci_key.pem

          chmod 600 /tmp/oci_key.pem
          chmod 600 ~/.oci/config

      - name: List Compartments
        run: |
          oci iam compartment list \
            --compartment-id "${{ secrets.OCI_TENANCY_OCID }}" \
            --all
```

---

# Resultado Esperado

Exemplo:

```json
{
  "data": [
    {
      "name": "staging"
    },
    {
      "name": "production"
    }
  ]
}
```

---

# Terraform

## Provider OCI

Exemplo:

```hcl
provider "oci" {
  region       = "sa-saopaulo-1"
  tenancy_ocid = var.tenancy_ocid
  user_ocid    = var.user_ocid
  fingerprint  = var.fingerprint
  private_key  = var.private_key
}
```

---

# Deploy OKE

Este modelo suporta:

* Terraform
* kubectl
* Helm
* ArgoCD
* OCI CLI
* OKE
* Registry OCI

---

# Sobre OIDC

## Vale a pena manter?

Sim.

Mesmo sem federation completa da OCI.

OIDC ainda pode ser utilizado para:

* validação de workflows
* branch protection
* controle de origem
* Zero Trust
* validação de repositórios

---

# Futuro

Caso a OCI evolua o suporte de:

* Workload Identity Federation
* Identity Propagation Trust
* Token Exchange RFC

será possível remover completamente:

* API Keys
* PEM privadas

---

# Recomendação Final

## Produção

Recomendado:

* Usuário técnico OCI
* API Key dedicada CI/CD
* Policies mínimas
* Compartments segregados
* GitHub Secrets
* Rotação periódica das chaves

---

# Não recomendado

Evitar:

* Usuários pessoais
* Policies amplas
* Chaves compartilhadas
* Credenciais hardcoded
* Deploy manual

---

# Status Final da Implementação

| Item                    | Status                  |
| ----------------------- | ----------------------- |
| GitHub Actions          | OK                      |
| OCI CLI                 | OK                      |
| Compartments OCI        | OK                      |
| API Key Auth            | OK                      |
| Terraform Ready         | OK                      |
| OKE Ready               | OK                      |
| OIDC JWT                | Parcial                 |
| Federation OCI completa | Limitada no Domain Free |

---

# Limitações do GitHub Free

## GitHub Actions Free

O GitHub Free possui limitações importantes que precisam ser consideradas para ambientes corporativos.

---

# Principais limitações

| Recurso                         | GitHub Free    |
| ------------------------------- | -------------- |
| GitHub Actions minutos          | Limitado       |
| Concurrent jobs                 | Limitado       |
| Environment approvals avançados | Limitado       |
| Secret scanning avançado        | Limitado       |
| Enterprise policies             | Não disponível |
| OIDC Federation avançada        | Parcial        |
| Auditoria avançada              | Limitada       |
| Runners enterprise              | Não disponível |

---

# Impacto prático

## 1. Minutos de execução

Pipelines Terraform/OKE podem consumir muitos minutos.

Exemplo:

* terraform plan
* terraform apply
* helm deploy
* build Docker
* push OCIR

Tudo isso consome GitHub Actions runtime.

---

# 2. Segurança reduzida

No GitHub Free:

* recursos enterprise de segurança são limitados
* controles organizacionais são menores
* governança é reduzida

---

# 3. Menor controle de ambientes

Features como:

* approvals obrigatórios
* deployment rules avançadas
* environment governance

possuem limitações no plano free.

---

# Limitações do OCI Identity Domain Free

## Principais limitações

| Recurso                      | Domain Free   |
| ---------------------------- | ------------- |
| Workload Identity Federation | Limitado      |
| Token Exchange RFC           | Parcial       |
| OCI CLI Federation           | Não funcional |
| OAuth Federation avançada    | Limitada      |
| Identity Propagation Trust   | Limitado      |
| Recursos enterprise IAM      | Limitados     |
| Federation Terraform nativa  | Inconsistente |

---

# Impacto no projeto

O principal impacto encontrado foi:

```text
GitHub JWT
→ OCI Token Exchange
→ OCI Session Token
→ OCI CLI Federation
```

não funcionar completamente.

Mesmo com:

* OAuth Client OCI
* Dynamic Groups
* Policies
* JWT válido
* Audience correta

A tenancy OCI rejeitou o fluxo RFC Token Exchange.

---

# Consequência prática

Não foi possível utilizar:

```text
OIDC puro sem API Key
```

para:

* OCI CLI
* Terraform OCI
* OKE deploy federado

---

# Limitações do OCI Key Management Free Tier

## O que funciona

Mesmo no free tier é possível utilizar:

✅ OCI Vault
✅ OCI Secrets
✅ OCI KMS básico
✅ Integração OCI CLI
✅ Integração Terraform
✅ Integração GitHub Actions

---

# Limitações encontradas

| Recurso                       | Free Tier      |
| ----------------------------- | -------------- |
| Quantidade de Vaults          | Limitado       |
| Quantidade de Keys            | Limitado       |
| HSM dedicado                  | Não disponível |
| Recursos enterprise avançados | Limitados      |
| Rotação avançada automática   | Parcial        |
| Governança enterprise         | Limitada       |

---

# Estratégia adotada

Foi adotado o seguinte modelo:

```text
GitHub Actions
↓
API Key técnica OCI
↓
OCI Vault
↓
Terraform / OCI CLI / OKE
```

---

# Vantagens desse modelo

Mesmo sem federation completa:

✅ Compatível com Terraform
✅ Compatível com OKE
✅ Compatível com ArgoCD
✅ Compatível com OCI CLI
✅ Integração simples
✅ Segurança adequada
✅ Menor complexidade operacional

---

# O que fica no GitHub

Somente credenciais bootstrap:

| Secret           |
| ---------------- |
| OCI_USER_OCID    |
| OCI_TENANCY_OCID |
| OCI_FINGERPRINT  |
| OCI_PRIVATE_KEY  |

---

# O que deve ficar no OCI Vault

Exemplos:

* kubeconfig
* tokens OCI Registry
* senhas banco
* API tokens
* certificados
* Helm secrets
* secrets aplicação

---

# Segurança do Modelo Final

## Modelo implementado

```text
GitHub Actions
↓
GitHub Secrets (bootstrap)
↓
OCI API Key técnica
↓
OCI Vault
↓
Terraform / OCI CLI / OKE
```

---

# Considerações finais

Mesmo sem federation/OIDC completa, o modelo atual:

✅ é seguro
✅ é amplamente utilizado na OCI
✅ é compatível com Terraform
✅ é compatível com OKE
✅ é compatível com ArgoCD
✅ suporta crescimento futuro

---

# Evolução futura

Caso a OCI evolua federation no Identity Domain:

será possível substituir:

```text
OCI API Key
```

por:

```text
OIDC Federation completa
```

sem necessidade de alterar:

* Terraform
* Vault
* OKE
* ArgoCD
* estrutura IAM
* compartments

---

# Referências

## GitHub Actions

[https://docs.github.com/actions](https://docs.github.com/actions)

---

## OCI CLI

[https://docs.oracle.com/en-us/iaas/Content/API/SDKDocs/cliinstall.htm](https://docs.oracle.com/en-us/iaas/Content/API/SDKDocs/cliinstall.htm)

---

## OCI IAM

[https://docs.oracle.com/en-us/iaas/Content/Identity/home.htm](https://docs.oracle.com/en-us/iaas/Content/Identity/home.htm)

---

## OCI Terraform Provider

[https://registry.terraform.io/providers/oracle/oci/latest/docs](https://registry.terraform.io/providers/oracle/oci/latest/docs)
