# ADR-001: Private Endpoints for HIPAA Data Isolation

## Status

Accepted

## Date

2025-12-04

## Context

The Contoso patient portal handles Protected Health Information (PHI) subject to HIPAA regulations.
HIPAA requires administrative, physical, and technical safeguards for electronic PHI (ePHI). Network
isolation is a critical technical safeguard to prevent unauthorized access.

### Requirements

- HIPAA technical safeguard compliance
- No public internet access to data tier
- Private connectivity from App Service to data services
- DNS resolution for private endpoints

### Considered Alternatives

1. **Service Endpoints** - Free, but still allows public access (blocks from other VNets only)
2. **Private Endpoints** - $7.30/month each, full private connectivity
3. **VNet Integration only** - Free, but data services remain public
4. **Azure Firewall** - $300+/month, overkill for this workload

## Decision

We selected **Private Endpoints** for Azure SQL and Key Vault:

| Approach          | Public Access  | Private DNS | Monthly Cost |
| ----------------- | -------------- | ----------- | ------------ |
| Service Endpoints | Yes (filtered) | No          | Free         |
| Private Endpoints | No             | Yes         | $7.30 each   |
| VNet Integration  | Yes            | No          | Free         |

### Implementation

- **Private Endpoint for SQL**: `pe-sql-{uniqueSuffix}`
- **Private Endpoint for Key Vault**: `pe-kv-{uniqueSuffix}`
- **Private DNS Zones**: `privatelink.database.windows.net`, `privatelink.vaultcore.azure.net`
- **Total Cost**: ~$15/month

### HIPAA Alignment

| HIPAA Control              | Implementation                 |
| -------------------------- | ------------------------------ |
| Access Control (164.312a1) | Private endpoints block public |
| Audit Controls (164.312b)  | NSG flow logs track access     |
| Transmission Security      | TLS 1.2 + private connectivity |

## Consequences

### Positive

- Complete network isolation of PHI data stores
- Meets HIPAA technical safeguard requirements
- Private DNS simplifies application configuration
- No changes to connection strings required

### Negative

- Additional cost ($15/month)
- Private DNS zones require management
- More complex networking troubleshooting
- Cannot access data from outside VNet (breaks some dev scenarios)

### Mitigations

- Create separate dev environment without private endpoints
- Document private endpoint troubleshooting in runbook
- Use Azure Bastion for emergency data access
- Enable diagnostic logging on private endpoints

## References

- [HIPAA Technical Safeguards](https://www.hhs.gov/hipaa/for-professionals/security/guidance/cybersecurity/index.html)
- [Private Endpoints Overview](https://learn.microsoft.com/azure/private-link/private-endpoint-overview)
- [Azure HIPAA Compliance](https://learn.microsoft.com/azure/compliance/offerings/offering-hipaa-us)
