# Architecture Decision Record (ADR) for Upgrading TSL 1.0 to TSL 1.2 on Azure Service Bus for Dev and UAT Environments

This ADR documents the decision to upgrade TSL from 1.0 to 1.2 for Azure Service Bus in the Development and UAT environments.

### Submitters

- Hector Ernesto Amaya (Dollarcity)
- Ricardo Marcel Martinez (Applaudo)

## Change Log

- Azure Service Bus manage

## Referenced Use Case(s)

- [DC - Upgrade TSL version on ASB](https://applaudostudios.atlassian.net/browse/FMS-2852) 

This ADR addresses the requirement for enhanced security in communication protocols.

## Context

The decision to upgrade the TSL version is architecturally significant as it directly relates to the security of our data in transit through the Azure Service Bus. TSL 1.0 has known vulnerabilities that are mitigated in TSL 1.2, ensuring improved data security and compliance with current best practices.

The high-level design approach is to implement TSL 1.2 across all relevant services in the Dev and UAT environments, ensuring comprehensive testing before deployment to production.

## Proposed Design

- **Services/Modules to be Impacted**:
  - Azure Service Bus integrations within the messaging architecture

- **New Services/Modules to be Added**:
  - None

- **Model and DTO Impact**:
  - No changes to existing models or DTOs

- **API Impact**:
  - No changes anticipated, but existing endpoints will require testing for compatibility with TSL 1.2.

- **General Configuration Impact**:
  - Configuration updates required in connection strings to specify TSL 1.2.

- **DevOps Impact**:
  - CI/CD pipelines may need adjustments to incorporate testing for TSL 1.2.

## Considerations

Alternatives considered included maintaining TSL 1.0 and implementing additional security layers, but the decision was made to fully upgrade due to the inherent vulnerabilities of TSL 1.0. Concerns regarding backward compatibility will be addressed during the testing phase.

## Decision

The agreed-upon implementation detail is to enforce TSL 1.2 for all data in transit over Azure Service Bus in the Dev and UAT environments. Future considerations include monitoring for any compatibility issues and planning for a production rollout following successful testing.

The upgrade will satisfy the requirement for secure communications as outlined in the referenced use case, with no outstanding issues expected.

## Other Related ADRs

- None

## References

- [Microsoft TSL Documentation](https://docs.microsoft.com/en-us/security/tls/) 
