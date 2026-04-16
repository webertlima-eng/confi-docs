# 📖 Confi DevOps Wiki

Documentação centralizada da infraestrutura e engenharia da Confi. Este repositório utiliza o conceito de **Docs as Code**, onde toda a base de conhecimento é versionada via Git e renderizada utilizando o [MkDocs Material](https://squidfunk.github.io/mkdocs-material/).

---

## 🚀 Estrutura do Repositório

A organização dos arquivos segue uma lógica de domínios técnicos para facilitar a busca e manutenção:

* `docs/architecture/`: Desenhos de solução, diagramas OCI e ADRs (Architecture Decision Records).
* `docs/infrastructure/`: Specs de clusters OKE, Terraform, redes e serviços Cloud.
* `docs/engineering/`: Padrões de desenvolvimento, Git Flow e guias de CI/CD.
* `docs/runbooks/`: Procedimentos operacionais de emergência e troubleshooting.

---

## 🛠️ Como Contribuir (Local Workflow)

Para garantir a qualidade visual antes de realizar um `push`, você pode rodar a Wiki localmente via Docker.

### Pré-requisitos
* Docker & Docker Compose instalado.

### Passo a Passo
1.  Clone o repositório:
    ```bash
    git clone git@github.com:webertlima-eng/confi-docs.git
    cd confi-docs
    ```
2.  Suba o servidor de desenvolvimento:
    ```bash
    docker compose up
    ```
3.  Acesse: `http://localhost:8000`

> **Nota:** O servidor possui *Hot Reload*. Qualquer alteração nos arquivos `.md` será refletida instantaneamente no navegador.

---

## 🤖 Fluxo de CI/CD (GitOps)

A atualização da Wiki na instância da OCI é totalmente automatizada. 

1.  **Commit/Push:** As alterações são enviadas para a branch `main`.
2.  **Validation:** (Opcional) O GitHub Actions valida a sintaxe do Markdown.
3.  **Deploy:** O servidor na OCI realiza um `git pull` automático (via cron/webhook) e o container MkDocs reflete as mudanças.

**URL de Produção:** [http://wiki.confi.com.vc:8000](http://wiki.confi.com.vc)

---

## ⚖️ Padrões de Escrita

* Utilize **títulos (H1, H2, H3)** para organizar a hierarquia.
* Para blocos de código, sempre especifique a linguagem (ex: ` ```yaml `).
* Adicione notas de destaque quando necessário:
    ```markdown
    !!! info "Nota importante"
        Este é um exemplo de admonition do Material Theme.
    ```

---

## 👤 Mantenedores
* **DevOps Team** - [webert.lima@confi.com.vc]
