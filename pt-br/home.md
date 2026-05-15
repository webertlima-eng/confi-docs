---
title: Início
description: 
published: true
date: 2026-05-15T15:27:14.993Z
tags: devops, overview
editor: markdown
dateCreated: 2026-04-16T20:24:23.813Z
---

# DevOps Platform - CONFI

## Visão Geral

Plataforma DevOps baseada em Oracle Cloud Infrastructure (OCI), Kubernetes (OKE), GitOps e automação CI/CD, focada em escalabilidade, segurança, governança e padronização operacional.

A plataforma foi desenhada para suportar múltiplos ambientes, pipelines reutilizáveis, deploy contínuo, segurança desacoplada e infraestrutura moderna baseada em GitOps.

---

# Objetivos

- Alta disponibilidade
- Deploy contínuo (GitOps)
- Segurança baseada em OIDC
- Padronização operacional
- Escalabilidade multiambiente
- Governança de infraestrutura
- Redução de drift
- Automação de deploy
- Centralização de secrets
- Reutilização de pipelines CI/CD

---

# Ambientes

| Ambiente | Objetivo | Status |
|---|---|---|
| LAB | Testes e homologação | Ativo |
| STAGING | Validação operacional | Ativo |
| PRODUÇÃO | Produção oficial | Planejado |

---

# Stack

## Cloud & Infraestrutura

- OCI (Oracle Cloud Infrastructure)
- OKE (Oracle Kubernetes Engine)
- Terraform
- DRG
- VCN
- VPN

---

## Plataforma Kubernetes

- Kubernetes v1.34.1
- ArgoCD
- Helm
- External Secrets
- OCI Vault
- NGINX Ingress

---

## CI/CD

- GitHub Actions
- Workflows reutilizáveis
- Actions reutilizáveis
- OCI Registry
- OIDC Federation

---

## Segurança

- OCI IAM
- Dynamic Groups
- Least Privilege
- OCI Vault
- External Secrets
- RBAC Kubernetes

---

## Observabilidade

- Grafana
- Logs
- OCI Monitoring
- OCI Logging

---

# Arquitetura Geral

A plataforma é organizada em múltiplos ambientes separados logicamente e conectados via OCI Networking.

## Estrutura Atual

```text
LAB
 ├── OKE Cluster
 ├── ArgoCD
 ├── Testes GitOps
 └── Pipelines homologação

STAGING
 ├── OKE Cluster
 ├── Deploy validação
 ├── OCI Vault
 └── Integrações OCI

PRODUÇÃO
 ├── OKE Cluster
 ├── GitOps oficial
 ├── Deploy controlado
 └── Observabilidade
```

---

# Estratégia GitOps

A ferramenta oficial escolhida foi:

- ArgoCD

## Motivações

- Facilidade operacional
- Melhor curva de aprendizado para equipe
- Interface visual madura
- Forte adoção de mercado
- Integração Kubernetes nativa
- Compatibilidade Helm
- Estratégia App of Apps

---

# Estratégia CI/CD

A plataforma utiliza pipelines reutilizáveis com GitHub Actions.

## Objetivos

- Redução de duplicação
- Padronização
- Segurança
- Governança
- Escalabilidade

---

# Segurança e Secrets

A estratégia atual utiliza:

- OIDC GitHub + OCI
- OCI Vault
- External Secrets
- Dynamic Groups
- Policies com menor privilégio

## Objetivos

- Eliminar secrets estáticas
- Evitar exposição de credenciais
- Centralizar autenticação
- Melhorar rastreabilidade

---

# Ambientes Kubernetes

## STAGING

```powershell
kubectl get nodes

NAME             STATUS   ROLES   AGE   VERSION
10.102.145.213   Ready    node    46h   v1.34.1
10.102.145.90    Ready    node    46h   v1.34.1
10.102.154.226   Ready    node    46h   v1.34.1
```

---

## LAB

```powershell
kubectl config use-context context-confi-oke-lab

kubectl get nodes

NAME             STATUS   ROLES   AGE     VERSION
10.100.156.151   Ready    node    7d11h   v1.34.1
10.100.157.249   Ready    node    7d11h   v1.34.1
```

---

# Links Rápidos

## Arquitetura

- OCI
- Kubernetes
- Networking
- DRG
- VPN

---

## Plataforma

- ArgoCD
- GitOps
- CI/CD
- Terraform
- Segurança

---

## Operação

- Runbooks
- Troubleshooting
- Deploy
- Rollback

---

# Roadmap

## Curto Prazo

- Instalação oficial ArgoCD
- OIDC GitHub + OCI
- OCI Vault
- External Secrets
- Pipelines reutilizáveis

---

## Médio Prazo

- SonarQube
- Policy Enforcement
- OPA/Kyverno
- Observabilidade avançada

---

## Longo Prazo

- Internal Developer Platform
- Golden Paths
- Self Service Deploy
- Disaster Recovery
- FinOps

---

# Responsáveis

| Área | Responsável |
|---|---|
| Kubernetes | DevOps Team |
| OCI | DevOps Team |
| GitOps | DevOps Team |
| Segurança | DevOps Team |
| CI/CD | DevOps Team |

---

# Última atualização

| Data | Descrição |
|---|---|
| 15/05/2026 | Criação inicial da plataforma DevOps CONFI |
