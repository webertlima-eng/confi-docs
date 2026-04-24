---
title: Renovação de certificado
description: 
published: true
date: 2026-04-24T15:24:06.584Z
tags: 
editor: markdown
dateCreated: 2026-04-24T15:24:06.584Z
---

---
# Certificado Wildcard Let's Encrypt
description: Guia para emitir e renovar o certificado wildcard no modo manual com desafio DNS-01 via Certbot
tags:
  - devops
  - ssl
  - letsencrypt
  - certbot
  - dns
---

Guia para emitir (e trocar) o certificado no **modo manual** com **desafio DNS-01**, seguindo o fluxo oficial do [Certbot](https://certbot.eff.org/), usado quando o app exige upload manual do certificado e a renovação não é automática no servidor.

| Campo | Valor |
|---|---|
| **Wildcard coberto** | `*.delivery.after.sale` |
| **Host de exemplo** | `demo.delivery.after.sale` |
| **Desafio ACME (TXT)** | `_acme-challenge.delivery.after.sale` |

**SANs típicos do certificado:**
- `*.delivery.after.sale` — wildcard
- `delivery.after.sale` — apex *(opcional, se o tráfego ou redirecionamento também usar a raiz da zona)*

> 📄 **Referências oficiais**
> - [Certbot — User Guide](https://eff-certbot.readthedocs.io/en/latest/using.html)
> - [Modo manual com DNS-01](https://eff-certbot.readthedocs.io/en/latest/using.html#manual)

---

## Pré-requisitos

- **Certbot** instalado na máquina emissora (ou host temporário com acesso à internet e ao DNS)
- Acesso **administrativo ao DNS** da zona `after.sale` (ou provedor que delega `delivery.after.sale`) para criar registro **TXT** em `_acme-challenge.delivery.after.sale`
- **E-mail** monitorado da equipe para avisos da CA (expiração, alterações de termos)

---

## 1. Instalação do Certbot

### 1.1 Linux (pacote nativo)

Escolha a linha correspondente à sua distro. Documentação oficial por SO: [Certbot — instruções](https://certbot.eff.org/instructions).

**Debian / Ubuntu**
```bash
sudo apt update
sudo apt install -y certbot
certbot --version
```

> ⚠️ Em versões LTS muito antigas, se o pacote do repositório estiver defasado, prefira backports ou o método Snap abaixo.

**Fedora**
```bash
sudo dnf install -y certbot
certbot --version
```

**RHEL / AlmaLinux / Rocky**
```bash
# EPEL costuma ser necessário em RHEL-derivados
sudo dnf install -y certbot
certbot --version
```

**Arch Linux**
```bash
sudo pacman -S certbot
certbot --version
```

**Snap (funciona em várias distros)**
```bash
sudo snap install core; sudo snap refresh core
sudo snap install --classic certbot
sudo ln -sf /snap/bin/certbot /usr/bin/certbot
certbot --version
```

Após a emissão, os certificados ficarão em `/etc/letsencrypt/` no host Linux (veja a seção 4).

---

### 1.2 Docker (imagem oficial `certbot/certbot`)

Útil quando não se quer instalar o Certbot no host ou é necessário isolar a versão. Imagem oficial: [certbot/certbot no Docker Hub](https://hub.docker.com/r/certbot/certbot).

> ℹ️ O modo **manual é interativo** — use sempre o flag `-it`.

Monte `/etc/letsencrypt` e `/var/lib/letsencrypt` no host via bind mounts. Crie as pastas **antes** do primeiro `run`.

**Comando validado** (usando `$HOME/Confi/letsencrypt-certs` como raiz):

```bash
BASE="$HOME/Confi/letsencrypt-certs"
mkdir -p "$BASE/etc/letsencrypt" "$BASE/var/lib/letsencrypt"
cd "$BASE"

docker run -it --rm --name certbot \
  -v "$PWD/etc/letsencrypt:/etc/letsencrypt" \
  -v "$PWD/var/lib/letsencrypt:/var/lib/letsencrypt" \
  certbot/certbot certonly \
  --manual \
  --preferred-challenges dns \
  --agree-tos \
  --no-eff-email \
  -m "devops@after.sale" \
  -d "*.delivery.after.sale" \
  -d "delivery.after.sale"
```

> 💡 O flag `--no-eff-email` evita o prompt interativo da EFF. Sem ele, o Certbot pergunta se deseja compartilhar o e-mail — responda **N** se não quiser (o e-mail em `-m` continua sendo usado para a conta Let's Encrypt).

Após a emissão bem-sucedida, os arquivos estarão em:

```
$HOME/Confi/letsencrypt-certs/etc/letsencrypt/live/delivery.after.sale/fullchain.pem
$HOME/Confi/letsencrypt-certs/etc/letsencrypt/live/delivery.after.sale/privkey.pem
```

> 🔍 Confirme o nome exato da pasta com:
> ```bash
> ls etc/letsencrypt/live/
> ```

> ⚠️ **`dig` no container:** o utilitário pode não estar disponível na imagem. Instale `bind-utils` / `dnsutils` no host ou use um container efêmero para validar o DNS.

---

## 2. Comando de emissão (DNS manual)

**Wildcard + apex** (recomendado quando ambos são necessários no app):

```bash
certbot certonly \
  --manual \
  --preferred-challenges dns \
  --agree-tos \
  --no-eff-email \
  -m "devops@after.sale" \
  -d "*.delivery.after.sale" \
  -d "delivery.after.sale"
```

**Somente wildcard** (sem apex):

```bash
certbot certonly \
  --manual \
  --preferred-challenges dns \
  --agree-tos \
  --no-eff-email \
  -m "devops@after.sale" \
  -d "*.delivery.after.sale"
```

> ⚠️ O Certbot vai **pausar** e exibir um ou mais valores TXT. Com wildcard + apex, podem aparecer **dois tokens para o mesmo nome DNS**. Não avance o prompt até o `dig` confirmar **todos** os tokens.

---

## 3. Criar o registro DNS (TXT)

Para cada desafio exibido pelo Certbot:

**1. Crie o registro TXT no painel DNS:**

| Campo | Valor |
|---|---|
| **Nome / host** | `_acme-challenge.delivery` *(dentro da zona `after.sale`)* ou o FQDN `_acme-challenge.delivery.after.sale`, conforme o provedor |
| **Tipo** | `TXT` |
| **Valor** | Token exibido pelo Certbot *(sem espaços extras)* |

**2. Com dois valores (wildcard + apex):** publique **os dois TXT** no mesmo nome antes de confirmar o último **Enter**. Se o provedor suportar múltiplos TXT no mesmo nome, crie dois registros separados.

**3. Aguarde a propagação** (pode levar de minutos a uma hora).

**4. Valide antes de continuar:**

```bash
dig +short TXT _acme-challenge.delivery.after.sale
# ou forçando o resolver do Google
dig +short TXT @8.8.8.8 _acme-challenge.delivery.after.sale
```

**5. Confirme no Certbot** com **Enter** somente quando o `dig` retornar **todos** os tokens esperados.

---

## 4. Onde ficam os arquivos após o sucesso

### Certbot instalado no Linux (host)

| Uso | Arquivo |
|---|---|
| Certificado + cadeia completa | `/etc/letsencrypt/live/<nome>/fullchain.pem` |
| Chave privada | `/etc/letsencrypt/live/<nome>/privkey.pem` |
| Só o certificado leaf | `/etc/letsencrypt/live/<nome>/cert.pem` |
| Cadeia intermediária | `/etc/letsencrypt/live/<nome>/chain.pem` |

Para o par wildcard + apex, a pasta costuma ser `live/delivery.after.sale/`. Confirme com:

```bash
sudo ls -la /etc/letsencrypt/live/
```

### Via Docker (layout validado)

```
$HOME/Confi/letsencrypt-certs/etc/letsencrypt/live/delivery.after.sale/fullchain.pem
$HOME/Confi/letsencrypt-certs/etc/letsencrypt/live/delivery.after.sale/privkey.pem
```

```bash
# Listar a partir da raiz do projeto
ls -la etc/letsencrypt/live/
```

### Upload no app

O par necessário para a maioria das plataformas é:

- `fullchain.pem` — certificado + cadeia
- `privkey.pem` — chave privada

> ⚠️ Alguns painéis exigem formato **PFX** ou certificado + chave em campo único — converta com `openssl` se necessário. **Nunca versione a chave privada em repositórios ou tickets.**

---

## 5. Renovação (fluxo manual)

- Certificados Let's Encrypt têm **~90 dias** de validade.
- Com `--manual` sem plugin DNS, o comando `certbot renew` **não** renova de forma desatendida (sem [hooks de autenticação](https://eff-certbot.readthedocs.io/en/latest/using.html#renewal)).
- O fluxo esperado é **repetir este procedimento** próximo ao vencimento e fazer o **novo upload** no app.

> 📅 **Recomendação:** registrar no calendário da equipe para reemitir e enviar com **30–60 dias de antecedência** ao fim do certificado ativo.

---

## 6. Pós-emissão: o que fazer no app

1. Obter `fullchain.pem` e `privkey.pem` (ou o formato exigido pela plataforma)
2. Aplicar no serviço / CDN / PaaS conforme a documentação do produto
3. Validar em `https://demo.delivery.after.sale/` — cadeia válida, data de expiração correta, sem avisos no navegador
4. Registrar a troca no **changelog interno** para auditoria

---

## 7. Problemas comuns

| Problema | Causa provável | Solução |
|---|---|---|
| Falha no desafio DNS | TXT incorreto, host errado ou propagação incompleta | Valide com `dig` antes de confirmar no Certbot |
| Dois nomes e só um TXT publicado | Faltou o segundo valor exibido pelo Certbot | Releia a saída completa do terminal e publique ambos |
| `*.delivery.after.sale` aparece como CN/SAN | Comportamento esperado | Nenhuma ação necessária |
| Permissão negada nos `.pem` (Docker) | Container grava como `root` no volume | Use `sudo` para listar/copiar ou ajuste com `chown` |
| `_acme-challenge` com CNAME | O TXT precisa existir onde o resolver enxerga o desafio | Siga o CNAME ou remova-o temporariamente durante a emissão |

**Diagnóstico completo do desafio:**

```bash
dig _acme-challenge.delivery.after.sale TXT +noall +answer
```

---

*Última revisão: documentação genérica — ajuste e-mails, lista exata de `-d` e passos de upload conforme a plataforma em produção.*
