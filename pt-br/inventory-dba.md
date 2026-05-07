---
title: Inventário de Bancos de Dados — Ambiente Cloud
description: 
published: true
date: 2026-05-07T16:45:50.214Z
tags: 
editor: markdown
dateCreated: 2026-05-07T16:45:50.214Z
---

# Inventário de Bancos de Dados — Ambiente Cloud

## Objetivo

Documentar os bancos de dados atualmente provisionados nos ambientes da organização, permitindo rastreabilidade operacional, governança de infraestrutura e apoio às atividades de DevOps/SRE.

---

# Ambientes Mapeados

* `production`
* `staging`
* `homolog`
* `dev`
* `sprint`
* `vault`

---

# Tecnologias Encontradas

| Tecnologia | Quantidade Aproximada |
| ---------- | --------------------- |
| PostgreSQL | 16                    |
| MongoDB    | 10                    |
| Redis      | 15                    |
| MySQL      | 7                     |
| RabbitMQ   | 4                     |

---

# Inventário

| Conta          | Recurso                                            | Ambiente   | Tecnologia | Observações              | Região      |
| -------------- | -------------------------------------------------- | ---------- | ---------- | ------------------------ | ----------- |
| aftersale-prod | aftersale-prod-mongodb-production                  | production | MongoDB    | Replica Set `mdb-p`      | confi-1     |
| aftersale-prod | aftersale-prod-mongodb-transaction-logs-production | production | MongoDB    | Replica Set `mdb-p-logs` | confi-1     |
| aftersale-prod | aftersale-prod-mysql-production                    | production | MySQL      | —                        | confi-1     |
| aftersale-prod | aftersale-prod-postgresql-production               | production | PostgreSQL | —                        | confi-1     |
| aftersale-prod | aftersale-prod-rabbitmq-production                 | production | RabbitMQ   | —                        | confi-1     |
| aftersale-prod | aftersale-prod-redis-production                    | production | Redis      | —                        | confi-1     |
| aftersale-v2   | aftersale-v2-billing-v2-production                 | production | PostgreSQL | —                        | confi-1     |
| aftersale-v2   | aftersale-v2-billing-staging                       | staging    | PostgreSQL | —                        | confi-stg-2 |
| aftersale-v2   | aftersale-v2-mongodb-production                    | production | MongoDB    | `MONGODB_OLD` / `mdb-p`  | confi-1     |
| aftersale-v2   | aftersale-v2-mongodb-2-staging                     | staging    | MongoDB    | `MONGODB_OLD` / `mdb-s`  | confi-stg-2 |
| aftersale-v2   | aftersale-v2-rabbitmq-2-production                 | production | RabbitMQ   | —                        | confi-1     |
| aftersale-v2   | aftersale-v2-rabbitmq-staging                      | staging    | RabbitMQ   | —                        | confi-stg-2 |
| aftersale      | aftersale-mongodb-legacy-staging                   | staging    | MongoDB    | `mdb-legacy-s`           | confi-stg-2 |
| aftersale      | aftersale-mysql-staging                            | staging    | MySQL      | —                        | confi-stg-2 |
| aftersale      | aftersale-postgresql-staging                       | staging    | PostgreSQL | —                        | confi-stg-2 |
| confi-infra    | confi-infra-vault-prod                             | vault      | PostgreSQL | Vault Database           | confi-1     |
| confi          | confi-mongodb-app-production-production            | production | MongoDB    | `MONGODB_OLD` / `mdb-p`  | confi-1     |
| confi          | confi-mysql-wordpress-production                   | production | MySQL      | WordPress                | confi-1     |
| confi          | confi-redis-app-production-production              | production | Redis      | —                        | confi-1     |
| neotrust       | neotrust-analytics-production                      | production | PostgreSQL | Analytics                | confi-1     |
| neotrust       | neotrust-metabase-prod                             | prod       | PostgreSQL | Metabase                 | confi-1     |
| neotrust       | neotrust-mongodb-development-dev                   | dev        | MongoDB    | `mdb-d`                  | confi-stg-2 |
| neotrust       | neotrust-mongodb-neoweb-production                 | production | MongoDB    | `mdb-p`                  | confi-1     |
| neotrust       | neotrust-mysql-neo-strapi-prod-production          | production | MySQL      | Strapi                   | confi-1     |
| neotrust       | neotrust-neo-scaleup-prod-2-production             | production | PostgreSQL | ScaleUp                  | confi-1     |
| neotrust       | neotrust-pg-airflow-2-production                   | production | PostgreSQL | Airflow                  | confi-1     |
| neotrust       | neotrust-redis-airflow-2-production                | production | Redis      | Airflow Cache/Broker     | confi-1     |
| neotrust       | neotrust-redis-neoweb-production                   | production | Redis      | NeoWeb                   | confi-1     |

---

# Observações Técnicas

* Nenhuma versão dos bancos está publicamente exposta.
* Ambientes utilizam separação lógica por tenancy (`confi-1` e `confi-stg-2`).
* Instâncias MongoDB utilizam replica sets distintos por ambiente.
* Redis está amplamente utilizado como cache, broker e suporte a aplicações.
* PostgreSQL é a principal tecnologia utilizada para workloads transacionais e analytics.
* RabbitMQ está presente em ambientes críticos de mensageria.

---

# Recomendações DevOps/SRE

* Validar política de backup e retenção por ambiente.
* Garantir monitoramento centralizado via Grafana/Prometheus.
* Revisar exposição de endpoints privados.
* Padronizar nomenclatura de recursos.
* Validar disaster recovery dos replica sets MongoDB.
* Consolidar documentação em CMDB/Wiki central.
* Automatizar inventário via Terraform/OCI CLI/API.

---

# Responsáveis

| Área                    | Responsável     |
| ----------------------- | --------------- |
| DevOps / Infraestrutura | Webert Lima     |
| DevOps / Infraestrutura | Rodrigo Turatti |
