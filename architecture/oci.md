---
title: Arquitetura OCI
description: 
published: true
date: 2026-04-16T20:26:46.778Z
tags: 
editor: markdown
dateCreated: 2026-04-16T20:26:46.778Z
---

#  Arquitetura OCI

##  Visão Geral

Arquitetura baseada em camadas com isolamento de rede e clusters separados.

## 🔹 Camadas

* CI/CD
* Rede (VCN)
* Kubernetes (OKE)
* Segurança
* Dados

---

## 🌐 Rede

### VCN

CIDR: 10.0.0.0/16

### Subnets

* Public → LB / Bastion
* Private → OKE
* DB/Security → Vault / DB

---

## ☸️ Kubernetes

### DEV

* Tools + Apps DEV/HML

### PROD

* Cluster isolado

---

## 🔄 Fluxo

Internet → LB → Gateway API → Pods
