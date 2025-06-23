# Architecture Decision Record for Credential Management for Fortinet NVR

In order to streamline API access for CLI management of NVR settings on Fortinet equipment, the decision has been made to utilize the same username and password for API authentication across all services utilized by Dollarama. This approach aims to simplify management, reduce complexity, and enhance interoperability between systems while ensuring users have a seamless experience.

### Submitters

- Ricardo Martinez (Applaudo)

## Change Log

- [Presented] 2025-06-19

## Referenced Use Case(s)

- [API Access Control Use Case](URL)
  
This ADR is primarily addressing the requirement for a unified authentication mechanism for API access while ensuring security and user management efficiency.

## Context

- **Architectural Significance**: The change to use a consistent username and password for API access is architecturally significant as it influences the overall authentication strategy. By centralizing authentication credentials, we minimize the risk of credential management errors and reduce the attack surface that could arise from managing multiple sets of credentials. Also keeping consistent the access centralizes the managing overhead. 

- **High-Level Design Approach**: The proposed design will facilitate easier credential management and audit trails for system access without compromising security protocols. It will require adjustments to current authentication mechanisms to ensure that they securely handle the unified credentials.

## Proposed Design

- **Services/Modules to be Impacted**: 
  - Fortinet NVR API access
  - CLI access services

- **New Services/Modules to be Added**:
  - NVR Healthcheck

- **Model and DTO Impact**: 
  - currently used Models have no impact with this decision.

- **API Impact**:
  - no impact on API Access 

- **General Configuration Impact**: 
  - Configuration files/settings will include sections dedicated to the unified credential setup for all API consumers.

- **DevOps Impact**: 
  - Update deployment scripts and CI/CD pipelines to ensure all environments are configured with the correct, unified credentials securely.

## Considerations

- **Alternatives**: Other options included using unique credentials for each service to enhance security. However, this was determined to increase complexity unnecessarily without significant security benefits in the current architecture.

- **Concerns**: Ensuring that credentials are stored securely during transitions and that appropriate audit logs are kept for compliance purposes. These concerns will be addressed by implementing strict security measures around password storage and access logging.

## Decision

The decision to utilize a unified username and password for API access to Fortinet NVR settings is based on the need for streamlined management and user experience. This decision creates a foundational layer for future enhancements in authentication without imposing significant changes on current implementations. Future considerations include potential implementation of additional security measures, such as two-factor authentication, as the infrastructure matures.

## Other Related ADRs

- None

## References

- [Fortinet API Documentation](URL)
- [Best Practices for Credential Management](URL)

--- 

