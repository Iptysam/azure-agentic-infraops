# ADR-002: Service Bus Premium for Private Endpoint Support

## Status

Accepted

## Date

2025-12-04

## Context

The e-commerce platform uses Azure Service Bus for reliable async order processing. PCI-DSS requires
that all data services be isolated from public internet access using private endpoints. However,
Service Bus only supports private endpoints on the Premium tier.

### Requirements

- Reliable order queue for async processing
- Private endpoint for network isolation (PCI-DSS)
- At least 1 messaging unit capacity
- Integration with VNet-integrated Azure Functions

### Considered Alternatives

1. **Service Bus Standard** - $10/month, no private endpoint support
2. **Service Bus Premium** - $677/month, full private endpoint support
3. **Azure Storage Queues** - $0.05/10K operations, limited features
4. **Azure Event Hubs** - $22/month, but designed for streaming not queuing

## Decision

We selected **Azure Service Bus Premium** despite the significant cost difference:

| Feature            | Standard | Premium | Storage Queue |
| ------------------ | -------- | ------- | ------------- |
| Private Endpoints  | ❌       | ✅      | ✅            |
| Dead-letter Queue  | ✅       | ✅      | ❌            |
| Sessions           | ✅       | ✅      | ❌            |
| Message Size (max) | 256 KB   | 100 MB  | 64 KB         |
| Transactions       | ✅       | ✅      | ❌            |
| FIFO Guarantee     | ✅       | ✅      | ❌            |

### Cost Impact

- **Additional Cost**: $667/month over Standard
- **Annual Cost**: ~$8,000 additional
- **Justification**: PCI-DSS network isolation is mandatory

### Alternatives Evaluated

**Storage Queues** were considered as a cheaper option with private endpoint support, but rejected
because:

- No dead-letter queue for failed order handling
- No transaction support for order processing
- Limited message size (64 KB vs 100 MB for receipts/documents)
- No session support for ordered processing

## Consequences

### Positive

- Full PCI-DSS network isolation compliance
- Enterprise messaging features (sessions, transactions)
- Higher throughput with dedicated capacity
- Future-ready for scaling with additional messaging units

### Negative

- Significant cost increase ($677/month)
- Premium tier requires capacity planning
- May be over-provisioned for initial launch

### Mitigations

- Monitor messaging unit utilization
- Consider downgrading to 0.5 MU if available in future
- For dev/test environments, use Standard tier (no PEs needed)
- Evaluate Storage Queues for non-order workloads

## References

- [Service Bus Pricing](https://azure.microsoft.com/pricing/details/service-bus/)
- [Service Bus Premium Features](https://learn.microsoft.com/azure/service-bus-messaging/service-bus-premium-messaging)
- [Private Endpoint Requirements](https://learn.microsoft.com/azure/service-bus-messaging/private-link-service)
