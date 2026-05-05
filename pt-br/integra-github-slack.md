---
title: Integração GitHub + Slack (Padrão DevOps)
description: Estabelecer a integração entre GitHub e Slack para centralizar notificações relacionadas ao ciclo de desenvolvimento, garantindo maior visibilidade, rastreabilidade e agilidade na comunicação do time.
published: true
date: 2026-05-05T20:40:39.945Z
tags: slack, github
editor: markdown
dateCreated: 2026-05-05T20:40:39.945Z
---

# Integração GitHub + Slack (Padrão DevOps)

## 1. Objetivo

Estabelecer a integração entre GitHub e Slack para centralizar notificações relacionadas ao ciclo de desenvolvimento, garantindo maior visibilidade, rastreabilidade e agilidade na comunicação do time.

---

## 2. Escopo

Esta configuração contempla notificações de:

* Issues (incluindo comentários)
* Pull Requests
* Workflows (CI/CD)
* Deployments

A integração é baseada em eventos de repositórios. O GitHub Projects (Kanban) não possui integração direta com o Slack.

---

## 3. Conceitos Importantes

### 3.1 GitHub Projects

O GitHub Projects é uma camada de organização visual baseada em issues.

* Cada card representa uma issue ou item relacionado
* Movimentações no board (colunas) não geram eventos para o Slack

### 3.2 Fonte das Notificações

O Slack recebe eventos provenientes de:

* Issues
* Pull Requests
* Workflows
* Deployments

Não há suporte nativo para eventos de movimentação de cards no Kanban.

---

## 4. Pré-requisitos

* Acesso ao repositório GitHub
* Acesso ao workspace Slack
* Permissão para instalar aplicações no Slack

---

## 5. Instalação da Integração

### 5.1 Instalar o aplicativo GitHub no Slack

1. Acessar: https://slack.com/apps
2. Buscar por "GitHub"
3. Selecionar "Adicionar ao Slack"
4. Autorizar a integração

---

### 5.2 Autenticação

No Slack, executar:

```
/github signin
```

---

### 5.3 Adicionar o bot ao canal

No canal desejado:

```
/invite @GitHub
```

---

## 6. Repositório Utilizado

Organização: `ConfiNeotrust`
Repositório: `devops`

---

## 7. Estrutura de Canais

| Canal          | Finalidade                          |
| -------------- | ----------------------------------- |
| #devops-issues | Gestão de tarefas (Kanban / Issues) |
| #devops-prs    | Pull Requests e Code Review         |
| #devops-ci-cd  | Execução de pipelines               |
| #devops-deploy | Eventos de deploy                   |
| #alerts-prod   | Incidentes críticos                 |

---

## 8. Configuração por Canal

### 8.1 Canal: #devops-issues

Responsável por representar o Kanban operacional.

```
/github subscribe ConfiNeotrust/devops issues
/github subscribe ConfiNeotrust/devops comments
```

Eventos recebidos:

* Criação de issues
* Atualizações
* Comentários

---

### 8.2 Canal: #devops-prs

Focado no fluxo de revisão de código.

```
/github subscribe ConfiNeotrust/devops pulls
/github subscribe ConfiNeotrust/devops reviews
```

Eventos recebidos:

* Abertura de Pull Request
* Solicitação de review
* Aprovação
* Merge

---

### 8.3 Canal: #devops-ci-cd

Monitoramento de pipelines.

```
/github subscribe ConfiNeotrust/devops workflows
```

Recomendado (redução de ruído):

```
/github subscribe ConfiNeotrust/devops workflows:failed
```

---

### 8.4 Canal: #devops-deploy

Eventos de deploy.

```
/github subscribe ConfiNeotrust/devops deployments
```

---

### 8.5 Canal: #alerts-prod

Eventos críticos filtrados por label.

```
/github subscribe ConfiNeotrust/devops +label:"incident"
```

---

## 9. Boas Práticas

### 9.1 Separação de responsabilidades

Cada canal deve possuir um único propósito claro.
Evitar centralizar múltiplos tipos de eventos em um único canal.

---

### 9.2 Controle de ruído

Evitar o uso de assinaturas genéricas:

```
/github subscribe repo everything
```

---

### 9.3 Uso de labels

Recomendado padronizar labels:

* incident
* bug
* infra
* urgent

Permite filtragem e melhor organização.

---

### 9.4 Fluxo DevOps recomendado

```
Issue → Pull Request → CI/CD → Deploy
```

---

## 10. Testes de Validação

Executar os seguintes testes:

1. Criar uma issue
2. Adicionar comentário na issue
3. Criar um Pull Request
4. Executar pipeline
5. Realizar deploy

Validar recebimento correto em cada canal.

---

## 11. Limitações

* Não há notificação de mudança de coluna no GitHub Projects
* Não há eventos de movimentação visual no Kanban
* Alto volume de eventos pode gerar ruído se não houver filtragem adequada

---

## 12. Evolução

Possíveis melhorias futuras:

* Integração com GitHub Actions para alertas customizados
* Integração com ferramentas de observabilidade
* Automação de incidentes
* Políticas de SLA e alertas inteligentes

---

## 13. Governança

* Definir responsáveis por cada canal
* Definir padrões de uso
* Revisar periodicamente o volume de notificações

---

## 14. Responsáveis

Responsável: Webert Natan
Time: DevOps

---

## 15. Status

Versão: 1.0
Situação: Em uso
