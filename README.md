# Guia de Instalação e Configuração do N8N na Oracle Cloud Infrastructure (OCI)

## 1. Criar a infraestrutura necessária

### Criar uma conta trial na OCI

Para começar, é necessário criar uma conta gratuita na Oracle Cloud Infrastructure (OCI).

**Link para criar conta trial:**  
[https://www.oracle.com/br/cloud/free/](https://www.oracle.com/br/cloud/free/)

**Passo a passo detalhado:**  
[https://docs.oracle.com/pt-br/iaas/Content/GSG/Tasks/signingup.htm](https://docs.oracle.com/pt-br/iaas/Content/GSG/Tasks/signingup.htm)

---

### Criar uma VM e rede com acesso liberado à Internet

Crie uma instância de máquina virtual (Compute Instance) no OCI com sistema operacional **Ubuntu 24.04** e configure uma **Virtual Cloud Network (VCN)** com acesso público.

**Documentação e laboratório passo a passo:**  
- Guia oficial OCI: [https://docs.oracle.com/pt-br/iaas/Content/Compute/Tasks/launchinginstance.htm](https://docs.oracle.com/pt-br/iaas/Content/Compute/Tasks/launchinginstance.htm)  
- Laboratório Oracle Learning: [https://apexapps.oracle.com/pls/apex/dbpm/r/livelabs/view-workshop?wid=732](https://apexapps.oracle.com/pls/apex/dbpm/r/livelabs/view-workshop?wid=732)

**Baixar as chaves SSH para acesso à VM:**  
Durante a criação da instância, selecione a opção **“Gerar par de chaves SSH”** e baixe o arquivo `.pem`.  
Salve-o em local seguro, por exemplo:  
`~/Downloads/oci_key.pem`

---

### Acessar a VM via SSH

Após a VM ser criada, copie o IP público exibido no painel da OCI e acesse via terminal:

```bash
# Dar permissão para o arquivo de chave
chmod 600 ~/Downloads/oci_key.pem

# Acessar a instância
ssh -i ~/Downloads/oci_key.pem ubuntu@<IP_Público_da_VM>
```

---

## 2. Instalação do N8N no Ubuntu 24

### Instalar dependências básicas

```bash
sudo apt update
sudo apt install unzip -y
```

### Instalar o fnm (Fast Node Manager)

```bash
curl -o- https://fnm.vercel.app/install | bash
```

Aplicar variáveis de ambiente do perfil:

```bash
source /home/ubuntu/.bashrc
```

### Instalar o Node.js v22

```bash
fnm install 22
```

### Instalar o N8N globalmente

```bash
npm install -g n8n
```

### Testar a instalação

```bash
n8n start
```

Se tudo estiver correto, o servidor local do N8N será iniciado em `http://localhost:5678`.

---

## 3. Liberar acesso externo (Firewall e Rede)

### Liberação temporária (sessão atual)

```bash
sudo iptables -F
sudo iptables -X
```

### Liberação permanente (salvar configuração)

```bash
sudo iptables -P INPUT ACCEPT
sudo iptables -P FORWARD ACCEPT
sudo iptables -P OUTPUT ACCEPT
sudo netfilter-persistent save
```

---

## 4. Configurar o N8N para acesso externo

**Atenção:** As variáveis abaixo desabilitam o uso de cookies seguros (apenas para ambiente de testes).  
Em produção, é altamente recomendado configurar HTTPS e domínio com TLS.

```bash
export N8N_SECURE_COOKIE=false
export N8N_PROTOCOL=http
export N8N_HOST=0.0.0.0
export N8N_PORT=5678
export N8N_EDITOR_BASE_URL=http://<IP_Público>:5678
export N8N_PUBLIC_URL=http://<IP_Público>:5678

n8n start
```

Agora, acesse no navegador:  
**http://<IP_Público>:5678**

---

## 5. Conclusão
Agora que vc abriu o N8N no navegador baixe e abra o arquivo [RAG_Agent_Manual_Configuracao_N8N_v1.pdf](RAG_Agent_Manual_Configuracao_N8N_v1.pdf
) e siga as instruções para configurar o RAG Agent!
