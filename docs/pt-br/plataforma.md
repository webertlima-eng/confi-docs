---
title: Plataforma
description: 
published: true
date: 2026-04-16T21:49:20.824Z
tags: 
editor: markdown
dateCreated: 2026-04-16T20:52:16.420Z
---

#  Kubernetes - Documentação Completa (DevOps)

---

#  1. Overview

O Kubernetes é uma plataforma open-source para orquestração de containers, responsável por automatizar deploy, escalabilidade e gerenciamento de aplicações.

## Objetivos

* Alta disponibilidade
* Escalabilidade automática
* Auto-healing
* Deploy contínuo

---

# 2. Arquitetura

## Componentes de Control plane

* API Server
* Scheduler
* Controller Manager
* etcd

## Componentes de Node

* Kubelet
* Kube Proxy
* Container Runtime

---

#  3. Comandos Essenciais

## Ver recursos

`kubectl get pods -A
`
`kubectl get svc -A
`
`kubectl get nodes
`
## Descrever recursos

`kubectl describe pod <pod>
`
## Logs

`kubectl logs <pod>
`
## Acessar container

`kubectl exec -it <pod> -- /bin/bash
`
## Aplicar configuração

`kubectl apply -f deployment.yaml
`
## Deletar recurso

`kubectl delete -f deployment.yaml
`
---

#  4. Deploy (Exemplo)

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
spec:
  replicas: 2
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:latest
        ports:
        - containerPort: 80
```

Aplicar:
`kubectl apply -f deployment.yaml
`
---

#  5. Service

```yaml
apiVersion: v1
kind: Service
metadata:
  name: nginx-service
spec:
  selector:
    app: nginx
  ports:
    - protocol: TCP
      port: 80
      targetPort: 80
  type: ClusterIP
```

---

#  6. Troubleshooting

## Pod em CrashLoopBackOff

### Diagnóstico

`kubectl describe pod <pod>
`
`kubectl logs <pod>
`
### Possíveis causas

* Erro na aplicação
* Configuração incorreta
* Dependência indisponível

---

## Pod não sobe

### Ver eventos

`kubectl get events`

### Ver logs

`kubectl logs <pod>
`
---

#  7. Segurança

## RBAC

Controle de acesso baseado em roles.

## Secrets

Armazenamento seguro de senhas e tokens.

Exemplo:
`kubectl create secret generic meu-secret --from-literal=senha=123
`
---

#  8. Observabilidade

## Logs

`kubectl logs <pod>
`
## Monitoramento

* Prometheus
* Grafana

## Métricas

`kubectl top pods
`
---

#  9. Rollback

`kubectl rollout undo deployment nginx-deployment
`
---

#  10. Boas Práticas

* Usar namespaces
* Definir requests/limits
* Versionar YAMLs
* Usar probes (liveness/readiness)
* Evitar usar latest em produção

---

#  11. Runbook de Incidente

## Passos para serviço fora do ar


1. kubectl get pods -A
2. Verificar pods com erro
3. kubectl describe pod
4. kubectl logs
5. Reiniciar deployment se necessário