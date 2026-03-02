# 📊 Dashboard API Planner — Power BI + Microsoft Graph

> Dashboard Power BI com atualização automática no Service, consumindo dados do **Microsoft Planner** via **Graph API** e autenticação por **Service Principal (Azure AD)** — sem gateway, sem Python, sem credencial organizacional.

---

## 🧩 O Problema

Ao usar a API do Microsoft Graph no Power BI Desktop, tudo funciona. Mas ao publicar no **Power BI Service**, a atualização quebra.

Dois motivos principais:

- **URL dinâmica**: o Service precisa saber o destino da requisição com antecedência. URL montada em tempo de execução = bloqueio.
- **Autenticação Organizacional**: não funciona de forma confiável no Service sem gateway.

---

## ✅ A Solução

| Problema | Solução Aplicada |
|---|---|
| URL dinâmica bloqueada | `BaseUrl` fixa + `RelativePath` variável |
| Autenticação organizacional | Service Principal com Client Credentials |
| Necessidade de gateway | Fonte configurada como **Anônima** |

O próprio Power Query gera o token automaticamente a cada requisição via chamada `POST` para o Azure AD — sem intervenção manual.

---

## 📁 Conteúdo do Repositório

```
📦 Dashboard_API_Planner
 ┣ 📄 Dashboard_API_Planner.pbit   # Template do Power BI (sem dados cacheados)
 ┗ 📄 README.md
```

---

## 🚀 Pré-requisitos

- Power BI Desktop instalado
- Acesso ao [Portal do Azure](https://portal.azure.com)
- Permissão para registrar aplicativos no Azure AD
- Acesso ao Microsoft Planner da sua organização

---

## 🔐 Passo a Passo — Configurando o Azure AD

### 1. Registrar o Aplicativo

1. Acesse [portal.azure.com](https://portal.azure.com)
2. Vá em **Azure Active Directory → App registrations → New registration**
3. Preencha:
   - **Name**: `PowerBI-Planner` (ou o nome que preferir)
   - **Supported account types**: *Accounts in this organizational directory only*
4. Clique em **Register**

---

### 2. Coletar as Credenciais

Após registrar, na tela do aplicativo copie:

| Campo | Onde encontrar |
|---|---|
| **Tenant ID** | Overview → Directory (tenant) ID |
| **Client ID** | Overview → Application (client) ID |

---

### 3. Criar o Client Secret

1. No menu lateral, vá em **Certificates & secrets**
2. Clique em **New client secret**
3. Defina uma descrição e prazo de expiração
4. Clique em **Add**
5. **Copie o valor imediatamente** — ele não será exibido novamente

> ⚠️ Guarde o Secret em local seguro.

---

### 4. Adicionar Permissões do Planner

1. Vá em **API permissions → Add a permission**
2. Selecione **Microsoft Graph → Application permissions**
3. Adicione as seguintes permissões:

| Permissão | Motivo |
|---|---|
| `Tasks.Read.All` | Leitura de tarefas do Planner |
| `Group.Read.All` | Acesso aos grupos/planos |

4. Clique em **Grant admin consent** (requer conta de administrador)

---

## ⚙️ Configurando o PBIT

1. Baixe o arquivo `Dashboard_API_Planner.pbit`
2. Abra no Power BI Desktop
3. Ao abrir, será solicitado os parâmetros:

| Parâmetro | Valor |
|---|---|
| `TenantID` | ID do diretório coletado no passo 2 |
| `ClientID` | ID do aplicativo coletado no passo 2 |
| `ClientSecret` | Secret criado no passo 3 |
| `PlanID` | ID do seu Plano no Planner* |

> *Para pegar o **Plan ID**: abra o Planner no navegador e copie o ID da URL:
> `https://planner.cloud.microsoft.com/webui/plan/**{SEU_PLAN_ID}**/view/board`

4. Clique em **Carregar**
5. Vá em **Transformar Dados → Configurações de Fonte de Dados**
6. Certifique-se que a fonte `https://graph.microsoft.com` está como **Anônima**

---

## ☁️ Publicando no Power BI Service

1. Publique o relatório normalmente (**Arquivo → Publicar**)
2. No Service, vá em **Conjunto de Dados → Configurações → Credenciais da fonte de dados**
3. Confirme que a autenticação está como **Anônima**
4. Configure o agendamento de atualização normalmente

✅ A atualização automática funcionará sem gateway.

---

## 💡 Como Funciona a Autenticação (resumo técnico)

```
Power BI Service
      │
      ▼
POST https://login.microsoftonline.com/{TenantID}/oauth2/v2.0/token
      │  (Client ID + Secret + scope: graph.microsoft.com/.default)
      ▼
   Access Token (válido por 1h)
      │
      ▼
GET https://graph.microsoft.com/v1.0/planner/...
      │  (Authorization: Bearer {token})
      ▼
   Dados do Planner
```

---

## 📌 Observações

- O token é renovado automaticamente a cada atualização do dataset
- O Client Secret tem prazo de validade — lembre de renovar antes de expirar
- Este template foi desenvolvido e testado com o **Planner do Microsoft 365**

---

## 🤝 Créditos

Solução desenvolvida com base no post de **[Denise Barcelos](https://www.linkedin.com/in/denisebarcelos/)** sobre paginação na Graph API — obrigado pelo ponto de partida!

---

## 👤 Autor

**Victor Araujo**
Analytics Engineer · Accesstage

[![LinkedIn](https://img.shields.io/badge/LinkedIn-Victor%20Araujo-0077B5?style=flat&logo=linkedin)](https://www.linkedin.com/in/victoraraujo965/)
[![GitHub](https://img.shields.io/badge/GitHub-Victoraraujo965-181717?style=flat&logo=github)](https://github.com/Victoraraujo965)
