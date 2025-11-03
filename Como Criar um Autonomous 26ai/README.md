
# Passo a Passo para Provisionar o Autonomous AI Database 26ai na OCI

> **Objetivo:** Este guia descreve como **provisionar** um Autonomous AI Database 26ai na OCI e como **criar/configurar um usuário** para cenários de RAG (Retrieval-Augmented Generation).

---

## Provisionamento do Autonomous AI Database 26ai

1. Acesse o **Oracle Cloud Infrastructure Console** clicando no ícone de navegação ao lado de “Cloud”.
2. No menu de navegação esquerdo, clique em **Oracle Database** e depois em **Autonomous AI Database**.
3. Selecione sua **região (region)** e **compartimento (compartment)**.
4. Na página de *Autonomous AI Databases*, clique em **Create Autonomous AI Database**.
5. Forneça informações básicas:
   - **Display name:** Insira um nome amigável (ex.: `MeuDB26ai`).
   - **Database name:** Insira um nome único (letras e números, máx. 30 caracteres, ex.: `db26ai`).

6. **Escolha o tipo de workload:**  
   - **Lakehouse** para workloads de IA/RAG.  
   - **Transaction Processing (TP)** para transações gerais **com suporte a vetores**.

7. **Configure o database (modelo de billing ECPU):**
   - **Always Free:** Ative para uma instância gratuita (*limitada, apenas na região home*).
   - **Developer:** Ative para uma instância de desenvolvimento (4 ECPUs fixos, 20 GB de storage).
   - **Choose database version:** Selecione **Oracle AI Database 26ai**.
   - **ECPU count:** Defina o número de CPUs (*mínimo 2*).
   - **Compute auto scaling:** Ative para escalar automaticamente até **3x** mais recursos.
   - **Storage:** Defina em **TB** (para Lakehouse) ou **GB/TB** (para outros). Ative **Storage auto scaling** para expansão automática até **3x**.
   - Clique em **Show advanced options** para opções como **elastic pool**, **modelo de compute (ECPU recomendado)** ou **Bring Your Own License (BYOL)**.

8. **Configure a retenção de backups:**
   - Defina o período de **retenção automática** (1 a 60 dias).
   - Ative **Immutable backup retention** se quiser bloquear alterações (*irreversível sem suporte*).

9. **Crie credenciais de administrador:**
   - **Username:** `ADMIN` (somente leitura para o campo; o usuário é fixo).
   - **Password:** Defina uma senha forte (12–30 caracteres, com maiúscula, minúscula, número; siga as regras de complexidade da OCI).

10. **Escolha o acesso de rede:**
    - **Secure access from everywhere:** Padrão, permite conexões seguras de qualquer lugar.
    - **Secure access from allowed IPs and VCNs only:** Restrinja com **ACLs**.
    - **Private endpoint access only:** Use para acesso privado via **VCN** (*recomendado para produção segura*).

11. *(Opcional)* Adicione **contatos para notificações operacionais** (até 10 e-mails).

12. Clique em **Show advanced options** para configurações avançadas:
    - **Encryption key:** Use chave **gerenciada pela Oracle** ou **customer-managed** (via **Vault**).
    - **Maintenance:** Defina **patch level** (*Regular* ou *Early*).
    - **Management:** Escolha **character set** (ex.: `UTF8`).
    - **Tools:** Configure ferramentas *built-in* (ex.: **Database Actions** para gerenciamento).
    - **Security attributes:** Adicione atributos para **Zero Trust Packet Routing (ZPR)**.
    - **Tags:** Adicione tags para organização.

13. *(Opcional)* Salve a configuração como **stack** para uso com **Resource Manager**.
14. Clique em **Create Autonomous AI Database**. O estado mudará para **Provisioning** até ficar disponível (*pode levar minutos*).

> **Após o provisionamento:** Baixe a **wallet** de credenciais para conexões seguras (via **Database Actions** ou **OCI Console**).

---

## Passo a Passo para Criar e Configurar um Usuário para RAG

Para RAG, crie um usuário com privilégios para consultas semânticas e análises (ex.: `CONNECT` para sessões e `DWROLE` para warehousing/analytics, ideal para buscas vetoriais). Use **Database Actions (GUI)** ou um **cliente SQL** (ex.: *SQL Developer*).

### Método 1: Usando Database Actions (Recomendado para Iniciantes)

1. Conecte-se como **ADMIN** via **Database Actions** (acesso via *OCI Console* → *Autonomous Database* → **Database Actions**).
2. No menu, vá em **Administration** → **Database Users**.
3. Clique em **+ Create User**.
4. Insira **username** (ex.: `rag_user`), **senha** (forte, confirme) e **quota** no tablespace **DATA** (ex.: `unlimited` para RAG com dados grandes).
5. Na aba **Granted Roles**, conceda:
   - **CONNECT:** Para criar sessões e consultas básicas.
   - **DWROLE:** Para privilégios de warehousing (útil para RAG com buscas em grandes datasets).
   - **Opcional:** Habilite **Graph**, **OML** ou **Web Access** para integração com ferramentas de IA.
6. Clique em **Create User**.
7. Forneça a **URL de Database Actions** e a **wallet** ao usuário.

### Método 2: Usando SQL (via Cliente como SQL Developer)

1. Conecte-se como **ADMIN** usando a **wallet** baixada.
2. Execute:
   ```sql
   CREATE USER rag_user IDENTIFIED BY "SenhaForte123!";
   GRANT CREATE SESSION TO rag_user;  -- Equivalente a CONNECT
   GRANT DWROLE TO rag_user;          -- Para análises e RAG
   ```

3. *(Opcional para RAG)* Conceda `SELECT` em tabelas específicas:
   ```sql
   GRANT SELECT ON schema.tabela_rag TO rag_user;
   ```

4. Forneça a **wallet** ao usuário para conexões.

---

## Configurações Básicas para RAG no 26ai

Após o setup, use **AI Vector Search** para RAG:

- **Converter dados em vetores (embeddings)** usando modelos como *Cohere* ou **OCI GenAI**.
- **Criar índices vetoriais:**
  ```sql
  CREATE VECTOR INDEX idx ON tabela(coluna_vector);
  ```
- **Integrar com apps** via *LangChain/Python* para buscas semânticas e *prompt augmentation*.
- **Testar com `SELECT AI`** com RAG para *queries* em linguagem natural.

> Para pacotes prontos de RAG, use **RAG-in-a-Box** da Oracle.

---

## Links para Mais Informações

- **Documentação de Provisionamento:** https://docs.oracle.com/en/cloud/paas/autonomous-database/adbsa/autonomous-provision.html  
- **Criação de Usuários:** https://docs.oracle.com/en/cloud/paas/autonomous-database/adbsa/manage-users-create.html  
- **AI Vector Search e RAG:** https://www.oracle.com/artificial-intelligence/easily-build-a-rag-solution/ e https://docs.oracle.com/en/database/oracle/oracle-database/26/vecse/index.html *(para 26ai, similar ao 23ai)*  
- **Visão Geral do 26ai:** https://www.oracle.com/database/26ai/  
- **Tutorial de RAG com Autonomous:** https://blogs.oracle.com/machinelearning/indatabase-retrieval-augmented-generation-rag-using-oracle-autonomous-database-select-ai

---

### Notas & Boas Práticas
- Mantenha a **wallet** em local seguro e **não versione** em repositórios públicos.
- Para produção, prefira **Private Endpoint** e controle de acesso via **VCN/ACLs**.
- Habilite **auto scaling** (compute e storage) para absorver picos de demanda.
- Utilize **tags** e **nomenclatura padrão** para facilitar auditoria e governança.
