<div align="center">

<!-- Logos -->
<img src="https://img.shields.io/badge/Microsoft-0078D4?style=for-the-badge&logo=microsoft&logoColor=white" alt="Microsoft"/>
<img src="https://img.shields.io/badge/Microsoft%20Intune-0078D4?style=for-the-badge&logo=microsoftazure&logoColor=white" alt="Microsoft Intune"/>
<img src="https://img.shields.io/badge/Microsoft%20Graph-00BCF2?style=for-the-badge&logo=microsoftazure&logoColor=white" alt="Microsoft Graph"/>

<br/><br/>

# 🛡️ Intune Security Baselines — Export & Import

**PowerShell scripts to export and import Security Baselines via Microsoft Graph REST API — without the Graph SDK**

<br/>

[![PowerShell](https://img.shields.io/badge/PowerShell-7%2B-5391FE?style=flat-square&logo=powershell&logoColor=white)](https://github.com/PowerShell/PowerShell)
[![Graph API](https://img.shields.io/badge/Graph%20API-Beta-00BCF2?style=flat-square)](https://learn.microsoft.com/en-us/graph/api/resources/intune-deviceconfigv2-devicemanagementconfigurationpolicy)
[![License](https://img.shields.io/badge/License-MIT-green?style=flat-square)](LICENSE)
[![No SDK](https://img.shields.io/badge/No%20Graph%20SDK-Required-orange?style=flat-square)](https://learn.microsoft.com/en-us/graph/sdks/sdks-overview)

<br/>

<!-- Language switcher -->
**[ 🇧🇷 Clique aqui para Português ](#-documentação-em-português) &nbsp;|&nbsp; [ 🇺🇸 Click here for English ](#-english-documentation)**

</div>

---

<br/>

<!-- ████████████████████████████████████████████████████████████ -->
<!--                   PORTUGUÊS                                   -->
<!-- ████████████████████████████████████████████████████████████ -->

## 🇧🇷 Documentação em Português

<details open>
<summary><strong>Expandir documentação em Português</strong></summary>

<br/>

### 📌 O que são Security Baselines no Intune?

As **Security Baselines** (Linhas de Base de Segurança) do Microsoft Intune são conjuntos pré-configurados de definições de segurança recomendadas pela Microsoft para dispositivos Windows. Elas agrupam centenas de configurações de segurança em um único perfil, baseadas nas melhores práticas da equipe de segurança da Microsoft, do CIS (Center for Internet Security) e do NIST (National Institute of Standards and Technology).

> **O que a Microsoft diz:**
> *"Security baselines are pre-configured groups of Windows settings that help you apply and enforce granular security settings that the relevant security teams recommend. You can also customize each baseline you deploy to enforce only those settings and values you require."*
> — [Microsoft Learn – Security baselines in Intune](https://learn.microsoft.com/en-us/mem/intune/protect/security-baselines)

#### Por que é tão importante configurar?

| Motivo | Impacto |
|--------|---------|
| 🔒 **Redução de superfície de ataque** | Desativa funcionalidades desnecessárias e potencialmente exploráveis |
| 🏛️ **Conformidade regulatória** | Auxilia no atendimento a LGPD, ISO 27001, NIST, CIS Benchmarks |
| ⚡ **Velocidade de implantação** | Em vez de configurar centenas de GPOs manualmente, uma baseline cobre tudo |
| 🧩 **Padronização** | Garante que todos os dispositivos do tenant sigam o mesmo padrão mínimo de segurança |
| 🔁 **Atualizações da Microsoft** | A cada nova versão do Windows, a Microsoft publica uma nova baseline com as proteções mais recentes |
| 🛡️ **Proteção contra ameaças comuns** | Habilita Defender Antivirus, configura Firewall, restringe execução de scripts, bloqueia protocolos inseguros |

#### O que as baselines cobrem?

As Security Baselines do Windows abrangem áreas como:

- **Microsoft Defender Antivirus** — proteção em tempo real, varredura, quarentena
- **Firewall do Windows** — regras para redes de domínio, privadas e públicas
- **BitLocker** — criptografia de disco completo
- **Windows Hello** — autenticação sem senha e biometria
- **Auditoria e logs** — rastreamento de eventos de logon, alterações de conta, uso de privilégios
- **Internet Explorer / Microsoft Edge** — hardening de navegador
- **Credenciais e senhas** — complexidade, expiração, bloqueio de conta
- **Controle de Conta de Usuário (UAC)** — elevação de privilégios
- **Windows Update** — políticas de atualização automática

> 💡 A Microsoft recomenda começar com as Security Baselines e depois ajustar as configurações conforme as necessidades específicas da organização, em vez de criar políticas do zero.

---

### 📋 Visão Geral dos Scripts

Este projeto fornece dois scripts PowerShell para exportar e importar Security Baselines do Microsoft Intune via **Microsoft Graph REST API** — sem necessidade do módulo Microsoft.Graph SDK. Isso resolve conflitos de versão de assembly MSAL comuns em ambientes com múltiplos módulos PowerShell instalados.

**Funcionalidades:**
- ✅ Autenticação por Device Code ou usuário/senha (ROPC)
- ✅ Exporta por categoria (Defender, Firewall, Auditing, etc.)
- ✅ Importa com seleção interativa via `Out-GridView`
- ✅ Permite unir settings da mesma categoria em uma única política
- ✅ Permite renomear cada política antes de importar
- ✅ Suporte a atribuição automática de grupos
- ✅ Logs detalhados na pasta do script

---

### ⚙️ Pré-requisitos

- PowerShell **7.0 ou superior**
- Acesso ao **Microsoft Entra ID** para registrar um aplicativo
- Conta com permissão **DeviceManagementConfiguration.ReadWrite.All** no tenant
- Windows com acesso à internet

---

### 🔐 Registro do Aplicativo no Entra ID

É necessário registrar um aplicativo próprio no Entra ID para obter um `$clientId` e `$tenantId` a configurar nos scripts.

> ⚠️ **Não use** o clientId genérico do *Microsoft Graph Command Line Tools* (`14d82eec-...`). Políticas de Conditional Access podem bloqueá-lo em dispositivos não gerenciados com o erro **530033**.

#### Passo a passo

**1. Criar o registro do app**
```
Entra ID → App registrations → New registration
  Nome: Intune Baselines Script
  Tipo de conta: Accounts in this organizational directory only (Single tenant)
  Redirect URI: (deixar em branco)
  → Register
```

**2. Habilitar Public Client (para Device Code funcionar)**
```
Authentication → Add a platform → Mobile and desktop applications
  ✅ https://login.microsoftonline.com/common/oauth2/nativeclient

Advanced settings:
  Allow public client flows → Yes
  → Save
```

**3. Adicionar permissão delegada**
```
API permissions → Add a permission → Microsoft Graph → Delegated permissions
  Buscar e selecionar: DeviceManagementConfiguration.ReadWrite.All
  → Add permissions

→ Grant admin consent for [seu tenant]
→ Confirmar: Yes
```

**4. Verificar que a permissão ficou aprovada**

| Permissão | Tipo | Status |
|-----------|------|--------|
| `DeviceManagementConfiguration.ReadWrite.All` | Delegated | ✅ Granted for [tenant] |

**5. Copiar os IDs**
```
Overview do app:
  Application (client) ID  →  será o $clientId nos scripts
  Directory (tenant) ID    →  será o $tenantId nos scripts
```

**6. Atualizar os scripts**

Abra `Export-SecurityBaselines.ps1` e `Import-SecurityBaselines.ps1` e substitua:
```powershell
$clientId = "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx"   # Application (client) ID
$tenantId = "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx"   # Directory (tenant) ID
```

---

### 📤 Export-SecurityBaselines.ps1

Exporta os **Security Baseline templates** disponíveis no tenant para arquivos JSON organizados por categoria.

#### O que faz

1. Lista todos os Security Baseline templates do tenant
2. Abre `Out-GridView` para seleção interativa
3. Cria um perfil temporário para capturar os valores padrão recomendados pela Microsoft
4. Salva JSON completo da baseline + JSONs individuais por categoria e configuração
5. Remove o perfil temporário automaticamente

#### Estrutura de pastas gerada

```
Exported-Baselines\
  Security Baseline for Windows 10 and later - Version 25H2\
    Security Baseline for Windows 10 and later - Version 25H2.json   ← baseline completa
    Auditing\
      001 - Audit Account Logon\
        Audit Account Logon.json
    Defender\
      001 - Allow Archive Scanning\
        Allow Archive Scanning.json
      002 - Allow Behavior Monitoring\
        Allow Behavior Monitoring.json
    Firewall\
      001 - Enable Domain Network\
        Enable Domain Network.json
      002 - Enable Private Network\
      003 - Enable Public Network\
    ...
```

#### Como usar

```powershell
# Exportar para pasta padrão (.\Exported-Baselines\<timestamp>)
.\Export-SecurityBaselines.ps1

# Exportar para pasta específica
.\Export-SecurityBaselines.ps1 -OutputPath "C:\Backup\Baselines"

# Informar UPN (pula digitação no login)
.\Export-SecurityBaselines.ps1 -AdminUPN "admin@contoso.com"
```

#### Parâmetros

| Parâmetro | Padrão | Descrição |
|-----------|--------|-----------|
| `-OutputPath` | `.\Exported-Baselines\<timestamp>` | Pasta de destino dos arquivos JSON |
| `-AdminUPN` | *(vazio)* | UPN do administrador (hint de login) |

#### Autenticação

Ao iniciar, o script pergunta o método de autenticação:

```
  Escolha o método de autenticação:
  [1] Device code  (abre navegador com código)
  [2] Usuário e senha
  Opção (1 ou 2):
```

- **[1] Device code** — abre `https://microsoft.com/devicelogin` no navegador com código temporário. Funciona com MFA.
- **[2] Usuário e senha** — digita UPN e senha direto no terminal (ROPC flow). **Não funciona com MFA obrigatório.**

---

### 📥 Import-SecurityBaselines.ps1

Importa Security Baselines para o tenant a partir de JSONs exportados previamente.

#### O que faz

1. Busca recursivamente arquivos `.json` na pasta indicada
2. Apresenta `Out-GridView` com colunas Categoria, Configuração, Template, Settings e Arquivo
3. **Agrupa settings da mesma categoria** e pergunta se deseja unir em uma única política
4. Permite **renomear** cada política antes de importar
5. Verifica duplicatas e suporta sobrescrita com `-OverwriteExisting`
6. Suporta atribuição automática a grupo do Entra ID

#### Fluxo interativo

```
  Categoria  : Firewall
  Template   : Security Baseline for Windows 10 and later (Version 25H2)
  Settings   : 3 selecionadas:
    - Enable Domain Network
    - Enable Private Network
    - Enable Public Network

  Unir em UMA política? (S/N): S
  Nome padrão: Security Baseline ... - Version 25H2 - Firewall
  Nome da política (Enter para manter): Firewall - Configurações de Rede

  Políticas a criar: 1
  Continuar com a importação? (Y/N): Y
```

#### Como usar

```powershell
# Importar como Security Baseline (aparece em Endpoint Security → Security Baselines)
.\Import-SecurityBaselines.ps1 -KeepAsBaseline

# Importar como Settings Catalog (aparece em Devices → Configuration)
.\Import-SecurityBaselines.ps1

# Pasta específica + atribuir a grupo
.\Import-SecurityBaselines.ps1 -SourcePath "C:\Baselines" `
    -GroupAssignmentId "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx" `
    -KeepAsBaseline

# Sobrescrever políticas existentes com o mesmo nome
.\Import-SecurityBaselines.ps1 -KeepAsBaseline -OverwriteExisting
```

#### Parâmetros

| Parâmetro | Padrão | Descrição |
|-----------|--------|-----------|
| `-SourcePath` | Pasta atual | Pasta com os JSONs (busca recursiva) |
| `-GroupAssignmentId` | *(vazio)* | Object ID do grupo para atribuição automática |
| `-AdminUPN` | *(vazio)* | UPN do administrador (hint de login) |
| `-KeepAsBaseline` | não | Preserva vínculo com o Security Baseline template |
| `-OverwriteExisting` | não | Deleta e recria políticas com nome idêntico |

> ✅ **Dica:** Use `-KeepAsBaseline` para que as políticas apareçam em **Endpoint Security → Security Baselines** no Intune, com o indicador de versão e conformidade com o template da Microsoft.

---

### 📄 Logs e Arquivos de Debug

Todos os arquivos de log são salvos na **mesma pasta do script**:

| Arquivo | Descrição |
|---------|-----------|
| `Export-SecurityBaselines.log` | Log completo da exportação |
| `Import-SecurityBaselines.log` | Log completo da importação |
| `DEBUG_settingTemplates_*.json` | Raw dos templates da API (pode apagar após uso) |
| `DEBUG_converted_*.json` | Settings convertidas antes do POST (pode apagar após uso) |

</details>

---

<br/>

<!-- ████████████████████████████████████████████████████████████ -->
<!--                      ENGLISH                                  -->
<!-- ████████████████████████████████████████████████████████████ -->

## 🇺🇸 English Documentation

<details>
<summary><strong>Expand English documentation</strong></summary>

<br/>

### 📌 What are Security Baselines in Intune?

**Security Baselines** in Microsoft Intune are pre-configured groups of Windows security settings recommended by Microsoft. They bundle hundreds of security configurations into a single profile, based on best practices from Microsoft's security team, the CIS (Center for Internet Security), and NIST (National Institute of Standards and Technology).

> **What Microsoft says:**
> *"Security baselines are pre-configured groups of Windows settings that help you apply and enforce granular security settings that the relevant security teams recommend. You can also customize each baseline you deploy to enforce only those settings and values you require."*
> — [Microsoft Learn – Security baselines in Intune](https://learn.microsoft.com/en-us/mem/intune/protect/security-baselines)

#### Why is it so important to configure?

| Reason | Impact |
|--------|--------|
| 🔒 **Attack surface reduction** | Disables unnecessary and potentially exploitable features |
| 🏛️ **Regulatory compliance** | Helps meet ISO 27001, NIST, CIS Benchmarks, and local compliance requirements |
| ⚡ **Deployment speed** | Instead of configuring hundreds of GPOs manually, one baseline covers everything |
| 🧩 **Standardization** | Ensures all tenant devices follow the same minimum security standard |
| 🔁 **Microsoft updates** | With each new Windows version, Microsoft publishes an updated baseline with the latest protections |
| 🛡️ **Protection against common threats** | Enables Defender Antivirus, configures Firewall, restricts script execution, blocks insecure protocols |

#### What do Security Baselines cover?

Windows Security Baselines cover areas such as:

- **Microsoft Defender Antivirus** — real-time protection, scanning, quarantine
- **Windows Firewall** — rules for domain, private, and public networks
- **BitLocker** — full disk encryption
- **Windows Hello** — passwordless authentication and biometrics
- **Auditing and logs** — tracking logon events, account changes, privilege use
- **Internet Explorer / Microsoft Edge** — browser hardening
- **Credentials and passwords** — complexity, expiration, account lockout
- **User Account Control (UAC)** — privilege elevation
- **Windows Update** — automatic update policies

> 💡 Microsoft recommends starting with Security Baselines and then adjusting settings to your organization's specific needs, rather than building policies from scratch.

---

### 📋 Scripts Overview

This project provides two PowerShell scripts to export and import Intune Security Baselines via the **Microsoft Graph REST API** — without requiring the Microsoft.Graph SDK module. This solves MSAL assembly version conflicts common in environments with multiple PowerShell modules installed.

**Features:**
- ✅ Device Code or username/password (ROPC) authentication
- ✅ Export organized by category (Defender, Firewall, Auditing, etc.)
- ✅ Interactive import selection via `Out-GridView`
- ✅ Merge settings from the same category into a single policy
- ✅ Rename each policy before importing
- ✅ Automatic group assignment support
- ✅ Detailed logs in the script folder

---

### ⚙️ Prerequisites

- PowerShell **7.0 or later**
- Access to **Microsoft Entra ID** to register an application
- Account with **DeviceManagementConfiguration.ReadWrite.All** permission in the tenant
- Windows with internet access

---

### 🔐 App Registration in Entra ID

You must register your own application in Entra ID to obtain a `$clientId` and `$tenantId` to configure in the scripts.

> ⚠️ **Do not use** the generic *Microsoft Graph Command Line Tools* clientId (`14d82eec-...`). Tenant Conditional Access policies may block it on unmanaged devices with error **530033**.

#### Step by step

**1. Create the app registration**
```
Entra ID → App registrations → New registration
  Name: Intune Baselines Script
  Account type: Accounts in this organizational directory only (Single tenant)
  Redirect URI: (leave blank)
  → Register
```

**2. Enable Public Client (required for Device Code flow)**
```
Authentication → Add a platform → Mobile and desktop applications
  ✅ https://login.microsoftonline.com/common/oauth2/nativeclient

Advanced settings:
  Allow public client flows → Yes
  → Save
```

**3. Add delegated permission**
```
API permissions → Add a permission → Microsoft Graph → Delegated permissions
  Search and select: DeviceManagementConfiguration.ReadWrite.All
  → Add permissions

→ Grant admin consent for [your tenant]
→ Confirm: Yes
```

**4. Verify the permission is approved**

| Permission | Type | Status |
|------------|------|--------|
| `DeviceManagementConfiguration.ReadWrite.All` | Delegated | ✅ Granted for [tenant] |

**5. Copy the IDs**
```
App Overview:
  Application (client) ID  →  will be $clientId in the scripts
  Directory (tenant) ID    →  will be $tenantId in the scripts
```

**6. Update the scripts**

Open `Export-SecurityBaselines.ps1` and `Import-SecurityBaselines.ps1` and replace:
```powershell
$clientId = "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx"   # Application (client) ID
$tenantId = "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx"   # Directory (tenant) ID
```

---

### 📤 Export-SecurityBaselines.ps1

Exports **Security Baseline templates** available in the tenant to JSON files organized by category.

#### What it does

1. Lists all Security Baseline templates in the tenant
2. Opens `Out-GridView` for interactive selection
3. Creates a temporary profile to capture Microsoft's recommended default values
4. Saves a full baseline JSON + individual JSONs per category and configuration
5. Automatically removes the temporary profile

#### Generated folder structure

```
Exported-Baselines\
  Security Baseline for Windows 10 and later - Version 25H2\
    Security Baseline for Windows 10 and later - Version 25H2.json   ← full baseline
    Auditing\
      001 - Audit Account Logon\
        Audit Account Logon.json
    Defender\
      001 - Allow Archive Scanning\
        Allow Archive Scanning.json
      002 - Allow Behavior Monitoring\
        Allow Behavior Monitoring.json
    Firewall\
      001 - Enable Domain Network\
        Enable Domain Network.json
      002 - Enable Private Network\
      003 - Enable Public Network\
    ...
```

#### How to use

```powershell
# Export to default folder (.\Exported-Baselines\<timestamp>)
.\Export-SecurityBaselines.ps1

# Export to a specific folder
.\Export-SecurityBaselines.ps1 -OutputPath "C:\Backup\Baselines"

# Provide UPN (skips typing during login)
.\Export-SecurityBaselines.ps1 -AdminUPN "admin@contoso.com"
```

#### Parameters

| Parameter | Default | Description |
|-----------|---------|-------------|
| `-OutputPath` | `.\Exported-Baselines\<timestamp>` | Destination folder for JSON files |
| `-AdminUPN` | *(empty)* | Admin UPN (login hint) |

#### Authentication

When the script starts, it prompts for the authentication method:

```
  Choose the authentication method:
  [1] Device code  (opens browser with code)
  [2] Username and password
  Option (1 or 2):
```

- **[1] Device code** — opens `https://microsoft.com/devicelogin` in the browser with a temporary code. Works with MFA.
- **[2] Username and password** — type UPN and password directly in the terminal (ROPC flow). **Does not work with mandatory MFA.**

---

### 📥 Import-SecurityBaselines.ps1

Imports Security Baselines into the tenant from previously exported JSON files.

#### What it does

1. Recursively searches for `.json` files in the specified folder
2. Shows `Out-GridView` with Category, Configuration, Template, Settings, and File columns
3. **Groups settings from the same category** and asks whether to merge into a single policy
4. Allows **renaming** each policy before importing
5. Checks for duplicates and supports overwriting with `-OverwriteExisting`
6. Supports automatic assignment to an Entra ID group

#### Interactive flow

```
  Category   : Firewall
  Template   : Security Baseline for Windows 10 and later (Version 25H2)
  Settings   : 3 selected:
    - Enable Domain Network
    - Enable Private Network
    - Enable Public Network

  Merge into ONE policy? (Y/N): Y
  Default name: Security Baseline ... - Version 25H2 - Firewall
  Policy name (Enter to keep): Firewall - Network Settings

  Policies to create: 1
  Continue with import? (Y/N): Y
```

#### How to use

```powershell
# Import as Security Baseline (appears under Endpoint Security → Security Baselines)
.\Import-SecurityBaselines.ps1 -KeepAsBaseline

# Import as Settings Catalog (appears under Devices → Configuration)
.\Import-SecurityBaselines.ps1

# Specific folder + assign to group
.\Import-SecurityBaselines.ps1 -SourcePath "C:\Baselines" `
    -GroupAssignmentId "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx" `
    -KeepAsBaseline

# Overwrite existing policies with the same name
.\Import-SecurityBaselines.ps1 -KeepAsBaseline -OverwriteExisting
```

#### Parameters

| Parameter | Default | Description |
|-----------|---------|-------------|
| `-SourcePath` | Current folder | Folder with JSON files (recursive search) |
| `-GroupAssignmentId` | *(empty)* | Group Object ID for automatic assignment |
| `-AdminUPN` | *(empty)* | Admin UPN (login hint) |
| `-KeepAsBaseline` | no | Preserves link to the Security Baseline template |
| `-OverwriteExisting` | no | Deletes and recreates policies with the same name |

> ✅ **Tip:** Use `-KeepAsBaseline` so policies appear under **Endpoint Security → Security Baselines** in Intune, with the version indicator and compliance tracking against the Microsoft template.

---

### 📄 Logs and Debug Files

All log files are saved in the **same folder as the script**:

| File | Description |
|------|-------------|
| `Export-SecurityBaselines.log` | Full export execution log |
| `Import-SecurityBaselines.log` | Full import execution log |
| `DEBUG_settingTemplates_*.json` | Raw API template data (can be deleted after use) |
| `DEBUG_converted_*.json` | Converted settings before POST (can be deleted after use) |

</details>

---

<br/>

<div align="center">

**Made by Marcelo Gonçalves &nbsp;|&nbsp; Microsoft MVP — Intune**

[![MVP](https://img.shields.io/badge/Microsoft%20MVP-Intune-FFB900?style=for-the-badge&logo=microsoft&logoColor=white)](https://mvp.microsoft.com/)

*Microsoft Intune Security Baselines Toolkit*

[![Microsoft Learn](https://img.shields.io/badge/Microsoft%20Learn-Security%20Baselines-0078D4?style=flat-square&logo=microsoft&logoColor=white)](https://learn.microsoft.com/en-us/mem/intune/protect/security-baselines)
[![Graph API Docs](https://img.shields.io/badge/Graph%20API-configurationPolicies-00BCF2?style=flat-square)](https://learn.microsoft.com/en-us/graph/api/resources/intune-deviceconfigv2-devicemanagementconfigurationpolicy)

</div>
