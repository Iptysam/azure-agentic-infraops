# Step 4: Bicep Implementation Specialist

> **Agent Used:** `bicep-implement` > **Purpose:** Generate production-ready Bicep templates from implementation plan

---

## 💬 Prompt

```text
Generate Bicep templates from the Contoso patient portal implementation plan.

Key requirements:
- Subscription scope for main.bicep (creates resource group)
- Generate uniqueSuffix in main.bicep using uniqueString()
- Key Vault names ≤24 chars
- App Service Plan P1v3 (Linux)
- Azure AD authentication for SQL
- Private endpoints for SQL and Key Vault
- Output to infra/bicep/contoso-patient-portal/

Include deploy.ps1 with:
- What-if support
- Bicep validation
- Parameter prompts for SQL credentials
```

---

## ✅ Code Generation Strategy

### Module Structure

```text
infra/bicep/contoso-patient-portal/
├── main.bicep                   # Subscription-scoped orchestrator
├── main.bicepparam              # Parameter file
├── deploy.ps1                   # Deployment script
├── README.md                    # Documentation
├── modules/
│   ├── networking.bicep         # VNet, subnets, NSGs
│   ├── monitoring.bicep         # Log Analytics, App Insights
│   ├── app-service-plan.bicep   # Hosting plan
│   ├── sql-server.bicep         # SQL Server
│   ├── sql-database.bicep       # SQL Database
│   ├── key-vault.bicep          # Key Vault
│   ├── private-endpoints.bicep  # Private connectivity
│   ├── app-service.bicep        # Web application
│   ├── key-vault-secrets.bicep  # Secrets population
│   └── rbac-assignments.bicep   # Role assignments
└── parameters/
    └── (optional env-specific params)
```

---

## Generated Files Summary

### main.bicep

- **Scope**: Subscription
- **Resource Group**: Created dynamically
- **Unique Suffix**: `uniqueString(subscription().subscriptionId, resourceGroupName)`
- **Deployment Names**: Prefixed with project-environment
- **4-Phase Deployment**: Respects module dependencies

### Module Count: 10

| Module                  | Lines   | Resources |
| ----------------------- | ------- | --------- |
| networking.bicep        | 120     | 4         |
| monitoring.bicep        | 65      | 2         |
| app-service-plan.bicep  | 40      | 1         |
| sql-server.bicep        | 85      | 2         |
| sql-database.bicep      | 50      | 1         |
| key-vault.bicep         | 75      | 1         |
| private-endpoints.bicep | 150     | 4         |
| app-service.bicep       | 110     | 1         |
| key-vault-secrets.bicep | 35      | 2         |
| rbac-assignments.bicep  | 45      | 1         |
| **Total**               | **775** | **19**    |

---

## Key Implementation Decisions

### 1. Unique Suffix Generation

```bicep
// In main.bicep - generates consistent 5-6 char suffix
var uniqueSuffix = uniqueString(subscription().subscriptionId, resourceGroupName)

// Key Vault name construction (≤24 chars)
// Pattern: kv-{project}-{env}-{suffix}
// Example: kv-contoso-prod-abc123 (22 chars) ✅
```

### 2. Azure AD Authentication for SQL

```bicep
// sql-server.bicep
resource sqlServerAdAdmin 'Microsoft.Sql/servers/administrators@2022-05-01-preview' = {
  parent: sqlServer
  name: 'ActiveDirectory'
  properties: {
    administratorType: 'ActiveDirectory'
    login: azureAdAdminLogin
    sid: azureAdAdminObjectId
    tenantId: tenant().tenantId
  }
}
```

### 3. App Service VNet Integration

```bicep
// app-service.bicep
resource appService 'Microsoft.Web/sites@2023-12-01' = {
  properties: {
    virtualNetworkSubnetId: webSubnetId
    httpsOnly: true
    siteConfig: {
      ftpsState: 'Disabled'
      minTlsVersion: '1.2'
      vnetRouteAllEnabled: true
    }
  }
  identity: {
    type: 'SystemAssigned'
  }
}
```

### 4. Private Endpoints with DNS

```bicep
// private-endpoints.bicep
resource sqlPrivateEndpoint 'Microsoft.Network/privateEndpoints@2023-11-01' = {
  name: 'pe-sql-${uniqueSuffix}'
  properties: {
    subnet: { id: privateEndpointSubnetId }
    privateLinkServiceConnections: [{
      name: 'sqlConnection'
      properties: {
        privateLinkServiceId: sqlServerId
        groupIds: ['sqlServer']
      }
    }]
  }
}

// Private DNS Zone
resource sqlPrivateDnsZone 'Microsoft.Network/privateDnsZones@2020-06-01' = {
  name: 'privatelink.database.windows.net'
  location: 'global'
}
```

---

## Deployment Script Features

### deploy.ps1

```powershell
# Key features:
# 1. Parameter validation
# 2. Bicep build + lint
# 3. What-if analysis
# 4. Confirmation prompt
# 5. Auto-detect current user for SQL admin

[CmdletBinding(SupportsShouldProcess)]
param(
    [string]$Environment = 'prod',
    [string]$Location = 'swedencentral',
    [string]$SqlAdminUsername = 'sqladmin',
    [SecureString]$SqlAdminPassword
)
```

---

## Validation Commands

```powershell
# Build all modules
bicep build main.bicep --stdout --no-restore

# Lint for best practices
bicep lint main.bicep

# What-if deployment
.\deploy.ps1 -WhatIf

# Actual deployment
.\deploy.ps1 -Environment prod -Location swedencentral
```

---

## Expected Outputs

After deployment:

```text
Outputs:
  resourceGroupName: 'rg-contoso-patient-portal-prod'
  appServiceUrl: 'https://app-contoso-portal-prod-abc123.azurewebsites.net'
  sqlServerFqdn: 'sql-contoso-portal-prod-abc123.database.windows.net'
  keyVaultUri: 'https://kv-contoso-prod-abc123.vault.azure.net/'
```

---

## ➡️ Next Steps

1. Run `bicep build` to validate syntax
2. Run `bicep lint` to check best practices
3. Run `.\deploy.ps1 -WhatIf` to preview changes
4. Run `.\deploy.ps1` to deploy

See [`infra/bicep/contoso-patient-portal/README.md`](../../../infra/bicep/contoso-patient-portal/README.md) for full documentation.
