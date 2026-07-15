---
title: Troque Fácil - Gerenciamento de Roles de Usuários
description: 
published: true
date: 2026-07-15T12:35:57.441Z
tags: 
editor: markdown
dateCreated: 2026-07-15T12:35:57.441Z
---

# Inclusão de Roles para Usuários - Troque Fácil

## Objetivo

Adicionar as roles **1** e **9** para um usuário existente no sistema **Troque Fácil** através do banco de dados MySQL.

> **Importante:** Execute este procedimento apenas mediante solicitação autorizada.

---

## Pré-requisitos

- Acesso ao banco de dados do Troque Fácil.
- Cliente SQL (DBeaver).
- Permissão de escrita nas tabelas `users` e `role_user`.

---

## Passo 1 - Localizar o usuário

Localize o usuário pelo endereço de e-mail.

```sql
SELECT id, first_name, last_name, email
FROM users
WHERE email = '<EMAIL_DO_USUARIO>';
```

### Exemplo

```sql
SELECT id, first_name, last_name, email
FROM users
WHERE email = 'amanda.rosa@confi.com.vc';
```

Resultado esperado:

| id | first_name | last_name | email |
|----|------------|-----------|-------|
| 17773 | Amanda | Rosa | amanda.rosa@confi.com.vc |

Anote o **ID** retornado.

---

## Passo 2 - Verificar as roles atuais

Antes de inserir novas permissões, verifique se o usuário já possui as roles.

```sql
SELECT *
FROM role_user
WHERE user_id = <ID_DO_USUARIO>;
```

### Exemplo

```sql
SELECT *
FROM role_user
WHERE user_id = 17773;
```

Caso as roles **1** e **9** já existam, nenhuma ação adicional é necessária.

---

## Passo 3 - Inserir as roles

Caso as roles ainda não existam, execute:

```sql
INSERT INTO role_user (role_id, user_id, created_at, updated_at)
VALUES
(1, <ID_DO_USUARIO>, NOW(), NOW()),
(9, <ID_DO_USUARIO>, NOW(), NOW());
```

### Exemplo

```sql
INSERT INTO role_user (role_id, user_id, created_at, updated_at)
VALUES
(1, 17773, NOW(), NOW()),
(9, 17773, NOW(), NOW());
```

---

## Passo 4 - Validar a inclusão

Após a inserção, confirme que as roles foram adicionadas.

```sql
SELECT *
FROM role_user
WHERE user_id = <ID_DO_USUARIO>;
```

Resultado esperado:

| role_id | user_id |
|---------|---------|
| 1 | 17773 |
| 9 | 17773 |

---

## Exemplo completo

```sql
-- Localizar usuário
SELECT id, first_name, last_name, email
FROM users
WHERE email = 'usuario@empresa.com';

-- Verificar roles existentes
SELECT *
FROM role_user
WHERE user_id = 12345;

-- Inserir roles
INSERT INTO role_user (role_id, user_id, created_at, updated_at)
VALUES
(1, 12345, NOW(), NOW()),
(9, 12345, NOW(), NOW());

-- Validar inclusão
SELECT *
FROM role_user
WHERE user_id = 12345;
```

---

## Boas práticas

- Confirme o e-mail do usuário antes de realizar qualquer alteração.
- Sempre verifique se as roles já existem para evitar registros duplicados.
- Execute alterações apenas mediante solicitação autorizada.
- Valide o resultado após a inclusão das permissões.
- Em caso de inconsistências, comunique a equipe responsável antes de realizar novas alterações.

---

## Ferramentas utilizadas

- DBeaver
- MySQL

---

**Autor:** Equipe DevOps  
**Sistema:** Troque Fácil