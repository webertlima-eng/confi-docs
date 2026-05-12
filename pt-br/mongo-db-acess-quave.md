---
title: Acesso e Liberação de Permissões MongoDB
description: 
published: true
date: 2026-05-12T21:51:36.768Z
tags: 
editor: markdown
dateCreated: 2026-05-12T21:51:36.768Z
---

# MongoDB - Acesso e Liberação de Permissões (Quave)

## Objetivo

Este documento descreve:

* Como acessar o MongoDB hospedado na Quave;
* Como conectar utilizando MongoDB Compass;
* Como validar permissões;
* Como liberar acesso para usuários específicos;
* Boas práticas de segurança.

---

# Visão Geral

O ambiente MongoDB utilizado na Quave possui:

* Replica Set;
* SSL/TLS habilitado;
* Autenticação via usuário e senha;
* Bancos internos e bancos de aplicação.

Exemplo de databases:

* `aftersale`
* `aftersale-demo`
* `admin`
* `config`
* `local`
* `test`

---

# Bancos Internos

Os bancos abaixo são internos do MongoDB:

| Database | Finalidade                              |
| -------- | --------------------------------------- |
| admin    | Administração do cluster e autenticação |
| config   | Metadados internos do cluster/sharding  |
| local    | Replicação e operações locais           |

> Evite conceder acesso desnecessário a esses bancos.

---

# Pré-requisitos

## Informações necessárias

Obter na Quave:

* Host MongoDB
* Usuário
* Senha
* Replica Set
* SSL habilitado

---

# Acessando via MongoDB Compass

## Download

[https://www.mongodb.com/products/tools/compass](https://www.mongodb.com/products/tools/compass)

---

## Connection String

Exemplo:

```txt
mongodb://root:SENHA@mdb-s-0.aftersale-v2-ddmnqz3qhzrwszojr.db.zcloud.ws:56446,mdb-s-1.aftersale-v2-ddmnqz3qhzrwszojr.db.zcloud.ws:59802,mdb-s-2.aftersale-v2-ddmnqz3qhzrwszojr.db.zcloud.ws:58863/?authSource=admin&ssl=true&replicaSet=mdb-s
```

---

# Validando Acesso

## Listar databases

```javascript
show dbs
```

---

## Validar usuário autenticado

```javascript
use admin

db.runCommand({ connectionStatus: 1 })
```

---

## Validar usuário específico

```javascript
use admin

db.getUser("wesley.palmeira")
```

---

# Roles MongoDB

## Somente leitura

```javascript
{ role: "read", db: "aftersale-demo" }
```

Permite:

* Consultar dados
* Exportar dados
* Visualizar collections

Não permite:

* Inserir
* Alterar
* Excluir

---

## Leitura e escrita

```javascript
{ role: "readWrite", db: "aftersale-demo" }
```

Permite:

* Ler dados
* Inserir documentos
* Atualizar documentos
* Excluir documentos

---

## Administração do banco

```javascript
{ role: "dbAdmin", db: "aftersale-demo" }
```

Permite:

* Gerenciar índices
* Estatísticas
* Operações administrativas do DB

---

# Liberando Permissão para Usuário

## Exemplo: liberar leitura e escrita

```javascript
use admin

db.grantRolesToUser(
  "wesley.palmeira",
  [
    { role: "readWrite", db: "aftersale-demo" }
  ]
)
```

---

## Exemplo: liberar somente leitura

```javascript
use admin

db.grantRolesToUser(
  "wesley.palmeira",
  [
    { role: "read", db: "aftersale-demo" }
  ]
)
```

---

# Validando Roles do Usuário

```javascript
use admin

db.getUser("wesley.palmeira")
```

Exemplo esperado:

```json
{
  "user" : "wesley.palmeira",
  "roles" : [
    {
      "role" : "readWrite",
      "db" : "aftersale"
    },
    {
      "role" : "readWrite",
      "db" : "aftersale-demo"
    }
  ]
}
```

---

# Revogando Permissões

## Remover acesso

```javascript
use admin

db.revokeRolesFromUser(
  "wesley.palmeira",
  [
    { role: "readWrite", db: "aftersale-demo" }
  ]
)
```

---

# Boas Práticas

## Recomendado

* Utilizar usuários individuais;
* Liberar somente os DBs necessários;
* Evitar compartilhamento de usuários root;
* Utilizar `read` sempre que possível;
* Auditar permissões periodicamente.

---

## Evitar

* Compartilhar senha root;
* Liberar `root` para desenvolvedores;
* Liberar acesso aos DBs internos sem necessidade;
* Utilizar usuários sem rastreabilidade.

---

# Troubleshooting

## Unauthorized

Erro:

```txt
MongoServerError[Unauthorized]
```

Causas comuns:

* Usuário sem permissão;
* authSource incorreto;
* senha inválida.

---

## ETIMEDOUT

Erro:

```txt
MongoServerSelectionError: connect ETIMEDOUT
```

Causas comuns:

* Firewall;
* VPN necessária;
* IP não liberado;
* Banco privado.

---

# Referências

## MongoDB Compass

[https://www.mongodb.com/products/tools/compass](https://www.mongodb.com/products/tools/compass)

---

## MongoDB Roles

[https://www.mongodb.com/docs/manual/reference/built-in-roles/](https://www.mongodb.com/docs/manual/reference/built-in-roles/)
