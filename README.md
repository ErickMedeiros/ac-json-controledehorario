# Controle de Horário de Logon — Conditional Access (Entra ID)

Este repositório armazena o arquivo JSON utilizado para criar uma política de **Acesso Condicional (Conditional Access)** no **Microsoft Entra ID** com restrição de horário de logon.

---

## 📋 Sobre a Política

A política bloqueia o acesso de usuários fora do horário comercial permitido. O JSON disponível neste repositório configura as seguintes condições:

- **Estado:** Habilitada (`enabled`)
- **Aplicativos:** Todos os recursos do Entra ID (`AllAgentIdResources`)
- **Plataformas:** Todas (`all`)
- **Tipos de cliente:** Browser, aplicativos móveis/desktop, Exchange ActiveSync e outros
- **Controle de horário:** Bloqueia acessos fora do intervalo de **08:00 às 18:00 UTC**, de segunda a sexta-feira
- **Ação:** Bloquear (`block`)

### Usuários

> ⚠️ Atualize os IDs de usuário/grupo conforme o seu ambiente antes de aplicar a política.

---

## ✅ Requisitos

### Licenciamento

- **Microsoft Entra ID P1** ou **P2** (ou Microsoft 365 Business Premium)  
  > Conditional Access com controle de horário requer licença P1 no mínimo.

### Permissões

- Conta com a função **Administrador de Acesso Condicional** ou **Administrador Global** no Entra ID.

### Funcionalidades necessárias habilitadas

- **Conditional Access** habilitado no tenant (acesse: *Entra ID > Proteção > Acesso Condicional*)
- O recurso de **filtro por horário (Time-based conditions)** deve estar disponível no tenant — verifique se o portal exibe a opção de configuração de horário na criação de políticas.

---

## 🚀 Como Aplicar a Política

### Opção 1: Importar via Portal do Entra ID

1. Acesse [https://entra.microsoft.com](https://entra.microsoft.com)
2. Navegue até **Proteção > Acesso Condicional > Políticas**
3. Clique em **+ Nova política** > **Criar nova política**
4. Configure manualmente os campos conforme o JSON deste repositório

> O portal ainda não oferece importação direta de JSON para políticas de Acesso Condicional na interface gráfica padrão.

### Opção 2: Aplicar via Microsoft Graph API

1. Obtenha um token de acesso com o escopo `Policy.ReadWrite.ConditionalAccess`
2. Faça uma requisição `POST` para o endpoint:

```
POST https://graph.microsoft.com/v1.0/identity/conditionalAccess/policies
Content-Type: application/json
Authorization: ******
```

3. Utilize o conteúdo do arquivo [`ca.json`](./ca.json) como corpo da requisição.

### Opção 3: Aplicar via PowerShell (Microsoft.Graph)

```powershell
Connect-MgGraph -Scopes "Policy.ReadWrite.ConditionalAccess"

$body = Get-Content -Path "./ca.json" -Raw | ConvertFrom-Json

New-MgIdentityConditionalAccessPolicy -BodyParameter $body
```

---

## ⚙️ Estrutura do JSON

```json
{
  "state": "enabled",
  "conditions": {
    "times": {
      "excludeDays": {
        "daysOfWeek": ["monday", "tuesday", "wednesday", "thursday", "friday"],
        "timeZone": "UTC",
        "startTime": "08:00:00",
        "endTime": "18:00:00"
      }
    }
  },
  "grantControls": {
    "builtInControls": ["block"]
  }
}
```

> O campo `excludeDays` combinado com `startTime`/`endTime` define o horário **permitido**. Fora desse intervalo, o acesso é bloqueado.

---

## ⚠️ Considerações Importantes

- **Fuso horário:** O horário está configurado em **UTC**. Ajuste o campo `timeZone` para o fuso horário do seu ambiente (ex: `"E. South America Standard Time"` para Brasília).
- **Conta de break-glass:** Certifique-se de que contas de acesso de emergência estejam **excluídas** da política para evitar lockout do tenant.
- **Teste antes de habilitar:** Utilize o modo **"Somente relatório" (Report-only)** antes de ativar a política em produção para validar o impacto.
- **Monitoramento:** Acompanhe os logs de acesso em *Entra ID > Monitoramento > Entradas* para verificar bloqueios.

---

## 📚 Referências

- [Documentação oficial de Acesso Condicional — Microsoft](https://learn.microsoft.com/pt-br/entra/identity/conditional-access/overview)
- [Criar política via Graph API](https://learn.microsoft.com/pt-br/graph/api/conditionalaccessroot-post-policies)
- [Condições de tempo no Acesso Condicional](https://learn.microsoft.com/pt-br/entra/identity/conditional-access/concept-conditional-access-conditions#time-based-conditions)
