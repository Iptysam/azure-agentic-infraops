# Azure Infrastructure Agent Workflow

> **Version**: 1.2
> **Last Updated**: 2025-12-02

This document describes the 5-step agent workflow for Azure infrastructure development in this repository.

---

## Overview

```mermaid
%%{init: {'theme':'neutral'}}%%
graph LR
    subgraph Step 1
        P["@plan<br/>(built-in)"]
    end
    subgraph Step 2
        A["azure-principal-architect<br/>(NO CODE)"]
    end
    subgraph Step 3
        D["diagram-generator<br/>(visualization)"]
    end
    subgraph Step 4
        B["bicep-plan<br/>(plan only)"]
    end
    subgraph Step 5
        I["bicep-implement<br/>(code generation)"]
    end
    subgraph Optional
        ADR["adr-generator"]
    end
    subgraph MCP Integration
        MCP["azure-pricing<br/>(real-time pricing)"]
    end

    P -->|"requirements"| A
    A -->|"architecture"| D
    D -->|"diagram"| B
    B -->|"plan"| I

    A -.->|"optional"| ADR
    I -.->|"optional"| ADR
    
    MCP -.->|"pricing data"| A
    MCP -.->|"cost estimates"| B

    style P fill:#e1f5fe
    style A fill:#fff3e0
    style D fill:#f3e5f5
    style B fill:#e8f5e9
    style I fill:#fce4ec
    style ADR fill:#e8eaf6
    style MCP fill:#fff9c4
```

## Workflow Steps

| Step | Agent                       | Purpose                 | Creates                      | Approval Required |
| ---- | --------------------------- | ----------------------- | ---------------------------- | ----------------- |
| 1    | `@plan` (built-in)          | Gather requirements     | Requirements plan            | ✅ Yes            |
| 2    | `azure-principal-architect` | WAF assessment          | Architecture recommendations | ✅ Yes            |
| 3    | `diagram-generator`         | Architecture diagram    | Python diagram + PNG         | ✅ Yes            |
| 4    | `bicep-plan`                | Implementation planning | `INFRA.md` plan file         | ✅ Yes            |
| 5    | `bicep-implement`           | Code generation         | Bicep templates              | ✅ Yes            |

### Optional Steps

| Agent           | When to Use    | Creates                      |
| --------------- | -------------- | ---------------------------- |
| `adr-generator` | After any step | Architecture Decision Record |

---

## How to Use

### Starting the Workflow

1. Open GitHub Copilot Chat (`Ctrl+Alt+I`)
2. Click the **Agent** button or press `Ctrl+Shift+A`
3. Select `@plan` to start with requirements
4. Follow the agent handoffs through each step

### Approval Gates

Each step requires your explicit approval before proceeding:

| Response                   | Action                             |
| -------------------------- | ---------------------------------- |
| **"yes"** or **"approve"** | Continue to next step              |
| **Feedback text**          | Agent refines output               |
| **"no"**                   | Return to previous step or restart |

### Example Conversation Flow

```
You: @plan Create a HIPAA-compliant patient portal with Azure App Service and SQL Database

Plan Agent: [Generates requirements plan]
           Do you approve this plan?

You: yes

[Handoff to azure-principal-architect]

Architect: [Provides WAF assessment with scores]
           Do you approve this architecture assessment?

You: Add Azure Front Door for global load balancing

Architect: [Updates assessment with Front Door]
           Do you approve this architecture assessment?

You: approve

[Handoff to diagram-generator]

Diagram: [Generates architecture visualization]
         ✅ Created docs/diagrams/patient-portal/architecture.py
         ✅ Generated architecture.png
         Continue to implementation planning?

You: yes

[Handoff to bicep-plan]

Planner: [Creates implementation plan in .bicep-planning-files/]
         Do you approve this implementation plan?

You: yes

[Handoff to bicep-implement]

Implementer: [Generates Bicep templates]
             Validation: bicep build ✅ | bicep lint ✅
             Do you approve this implementation?

You: deploy

Implementer: [Executes deployment to Azure]
```

---

## Agent Responsibilities

### @plan (Built-in VS Code Feature)

- **Input**: Natural language requirements
- **Output**: Structured requirements plan with cost estimates
- **Limitations**: Cannot access workspace files

### azure-principal-architect

- **Input**: Requirements from @plan or user
- **Output**: WAF pillar assessment, SKU recommendations, cost estimates, pricing documentation
- **Limitations**: ❌ Cannot create or edit Bicep/Terraform code files
- **Focus**: Architectural guidance and cost analysis
- **MCP Integration**: Uses `azure-pricing/*` tools for real-time Azure pricing data
- **Documentation**: Can create `docs/{project}-waf-assessment.md` and `docs/{project}-cost-estimate.md`

### diagram-generator

- **Input**: Architecture context from azure-principal-architect
- **Output**: Python diagram file in `docs/diagrams/{goal}/` + PNG image
- **Limitations**: Requires `diagrams` Python library
- **Focus**: Visual architecture documentation

### bicep-plan

- **Input**: Architecture assessment
- **Output**: Implementation plan in `.bicep-planning-files/INFRA.{goal}.md`
- **Limitations**: ❌ Cannot create actual Bicep code
- **Focus**: Detailed planning with AVM modules
- **MCP Integration**: Uses `azure-pricing/*` tools for SKU cost validation

### bicep-implement

- **Input**: Implementation plan from `.bicep-planning-files/`
- **Output**: Bicep templates in `infra/bicep/{goal}/`
- **Limitations**: Must follow the approved plan
- **Focus**: Code generation and validation

### adr-generator (Optional)

- **Input**: Any architectural decision
- **Output**: ADR in `docs/adr/`
- **Focus**: Decision documentation

---

## MCP Cost Estimation Integration

The Azure Pricing MCP Server provides real-time cost data to agents during infrastructure planning.

### Architecture

![Azure Pricing MCP Architecture](diagrams/mcp/azure_pricing_mcp_architecture.png)

### Available Tools

| Tool                     | Purpose                                | Used By              |
| ------------------------ | -------------------------------------- | -------------------- |
| `azure_price_search`     | Query Azure retail prices with filters | architect, bicep-plan |
| `azure_price_compare`    | Compare prices across regions/SKUs     | architect            |
| `azure_cost_estimate`    | Calculate monthly/yearly costs         | architect, bicep-plan |
| `azure_region_recommend` | Find cheapest Azure regions for a SKU  | architect            |
| `azure_discover_skus`    | List available SKUs for a service      | architect, bicep-plan |
| `azure_sku_discovery`    | Fuzzy SKU name matching                | architect            |
| `get_customer_discount`  | Get customer discount (default 10%)    | architect            |

### Setup

The MCP server is automatically configured when you:

1. **Dev Container**: Runs `post-create.sh` which sets up the venv
2. **Manual**: Run the setup commands:

```bash
cd mcp/azure-pricing-mcp
python -m venv .venv
source .venv/bin/activate
pip install -r requirements.txt
```

### Configuration

Pre-configured in `.vscode/mcp.json`:

```json
{
  "servers": {
    "azure-pricing": {
      "type": "stdio",
      "command": "${workspaceFolder}/mcp/azure-pricing-mcp/.venv/bin/python",
      "args": ["-m", "azure_pricing_mcp"],
      "cwd": "${workspaceFolder}/mcp/azure-pricing-mcp/src"
    }
  }
}
```

### Creating Cost Documentation

Ask the `azure-principal-architect` agent:

- **"save costs"** - Creates `docs/{project}-cost-estimate.md`
- **"save"** - Creates `docs/{project}-waf-assessment.md`

### Fallback

If MCP tools are unavailable, agents will recommend [Azure Pricing Calculator](https://azure.microsoft.com/pricing/calculator/).

---

## Alternative: Unified Agent

For simple infrastructure or quick iterations, use `infrastructure-specialist` which combines all steps into a single agent. This is useful for:

- Small projects (< 5 resources)
- Prototyping and experimentation
- Users familiar with Azure who don't need step-by-step guidance

---

## File Locations

| Artifact              | Location                                |
| --------------------- | --------------------------------------- |
| Implementation plans  | `.bicep-planning-files/INFRA.{goal}.md` |
| Bicep templates       | `infra/bicep/{goal}/`                   |
| Architecture diagrams | `docs/diagrams/{goal}/`                 |
| MCP diagrams          | `docs/diagrams/mcp/`                    |
| Cost estimates        | `docs/{project}-cost-estimate.md`       |
| WAF assessments       | `docs/{project}-waf-assessment.md`      |
| ADRs                  | `docs/adr/`                             |
| Agent definitions     | `.github/agents/`                       |
| MCP server            | `mcp/azure-pricing-mcp/`                |
| MCP configuration     | `.vscode/mcp.json`                      |

---

## Troubleshooting

### Agent Not Available

1. Press `Ctrl+Shift+A` to open agent picker
2. Verify agent files exist in `.github/agents/`
3. Restart VS Code if agents don't appear

### Handoff Not Working

1. Look for the handoff button in chat response
2. If no button, manually switch agents with `Ctrl+Shift+A`
3. Copy relevant context to new agent conversation

### Validation Failures

| Error                 | Solution                            |
| --------------------- | ----------------------------------- |
| `bicep build` fails   | Check syntax errors in Bicep files  |
| `bicep lint` warnings | Address best practice violations    |
| Deployment fails      | Verify Azure permissions and quotas |

---

## Related Documentation

- [Demo Prompts](../demos/demo-prompts.md)
- [Copilot Instructions](../.github/copilot-instructions.md)
- [Azure Pricing MCP README](../mcp/azure-pricing-mcp/README.md)
- [README](../README.md)
