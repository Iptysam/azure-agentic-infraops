# Example: 120-Char Wrap Preview

This example shows how a long section could be reformatted to respect a 120-character line target after
applying markdownlint MD013=120. It is illustrative only and does not replace any existing documentation.

## Before (typical issues)
- Single paragraphs stretching far beyond 120 characters.
- Tables with unwrapped text that force readers to scroll horizontally.
- Links embedded in long sentences without breaks.

## After (wrapped for readability)
Agentic InfraOps demonstrates a four-step workflow—plan, architect, design, and implement—that reduces
infrastructure delivery time while keeping Azure Well-Architected practices in focus. Default to the
`swedencentral` region, with `germanywestcentral` as a fallback when quotas block progress. Use Azure
Verified Modules (AVM) first to stay aligned with policy and security baselines.

### Quick cross-links (wrapped)
- Workflow: `docs/workflow/WORKFLOW.md`
- Scenarios index: `scenarios/README.md`
- Cost estimate (e-commerce): `docs/cost-estimates/ecommerce-cost-estimate.md`
- Diagrams: `docs/diagrams/ecommerce/`, `docs/diagrams/workflow/`
- Presenter toolkit: `docs/presenter-toolkit/demo-delivery-guide.md`, `roi-calculator.md`,
  `objection-handling.md`, `visual-elements-guide.md`

### Example table (wrapped cells)
| Section        | Purpose                                                          | Link                                     |
| -------------- | ---------------------------------------------------------------- | ---------------------------------------- |
| Workflow       | Explains the four-step agent flow and approval gates             | docs/workflow/WORKFLOW.md                |
| Scenarios      | Entry points for demos; link related diagrams and cost estimates | scenarios/README.md                      |
| Cost Estimates | Pricing references for demos (e-commerce sample provided)        | docs/cost-estimates/ecommerce-cost-estimate.md |
| Presenter Kit  | Demo delivery, ROI, objections, visuals                          | docs/presenter-toolkit/README.md         |

## Notes
- Break lines at natural clauses to keep prose readable.
- Keep bullet items short; split long link lists across lines.
- Wrap table text to avoid exceeding 120 characters per line.
