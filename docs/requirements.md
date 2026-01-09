# Requirements Document

## Introduction

This document specifies the requirements for modernizing the integration between Gluu Server/Jans and Radiator RADIUS server. The current implementation uses OAuth 2.0 Resource Owner Password Grant (ROPG) flow, which will be replaced with a more secure, policy-driven approach using the /authorization_challenge endpoint, Agama flows, and Cedarling for authorization decisions.

## Glossary

- **Jans_Server**: The Janssen Project authorization server (successor to Gluu Server)
- **Radiator_RADIUS**: The RADIUS server that handles network access authentication
- **Authorization_Challenge_Endpoint**: Jans endpoint that supports non-browser authentication flows
- **Agama_Flow**: Jans authentication flow engine for custom business logic
- **Cedarling**: Policy-based authorization engine using Cedar policy language
- **SSA**: Software Statement Assertion for dynamic OAuth client registration
- **ROPG**: Resource Owner Password Grant (deprecated OAuth flow being replaced)
- **RADIUS_Client**: Network access server (NAS) that forwards authentication requests to RADIUS server

## Requirements

### Requirement 1: Authentication Flow Modernization

**User Story:** As a network administrator, I want to replace the insecure ROPG flow with a modern authentication mechanism, so that user credentials are handled more securely and authentication logic is more flexible.

#### Acceptance Criteria

1. WHEN a RADIUS authentication request is received, THE Radiator_RADIUS SHALL initiate authentication via the Jans /authorization_challenge endpoint instead of the /token endpoint
2. WHEN the authorization challenge is processed, THE Jans_Server SHALL execute the appropriate Agama_Flow for authentication logic
3. WHEN authentication succeeds, THE Jans_Server SHALL return an authorization response that Radiator_RADIUS can process for RADIUS Access-Accept
4. WHEN authentication fails, THE Jans_Server SHALL return appropriate error codes that map to RADIUS Access-Reject responses
5. THE system SHALL maintain backward compatibility with existing RADIUS client configurations during the transition period

### Requirement 2: Agama Flow Integration

**User Story:** As a system architect, I want to use Agama flows for authentication business logic, so that authentication processes can be customized and extended without code changes.

#### Acceptance Criteria

1. WHEN a user authentication is required, THE Jans_Server SHALL invoke the configured Agama_Flow for RADIUS authentication
2. WHEN the Agama flow executes, THE system SHALL support multi-step authentication processes including MFA scenarios
3. WHEN the Agama flow completes successfully, THE system SHALL provide user identity and authentication context to the authorization layer
4. WHEN the Agama flow encounters errors, THE system SHALL return standardized error responses with appropriate RADIUS error codes
5. THE Agama_Flow SHALL be configurable per RADIUS client or authentication policy without system restart

### Requirement 3: Policy-Based Authorization with Cedarling

**User Story:** As a security administrator, I want authorization decisions to be made by Cedarling using Cedar policies, so that access control is centralized, auditable, and easily modified.

#### Acceptance Criteria

1. WHEN authentication succeeds, THE Jans_Server SHALL invoke Cedarling for authorization evaluation before returning the final response
2. WHEN Cedarling evaluates policies, THE system SHALL provide complete authentication context including user identity, RADIUS client information, and request attributes
3. WHEN Cedarling permits access, THE system SHALL include any policy-determined attributes in the RADIUS response
4. WHEN Cedarling denies access, THE system SHALL return RADIUS Access-Reject with appropriate reason codes
5. THE system SHALL support dynamic policy updates without requiring system restart or authentication flow interruption

### Requirement 4: SSA-Based Dynamic Client Registration

**User Story:** As a deployment engineer, I want RADIUS clients to register dynamically using Software Statement Assertions, so that client onboarding is automated and secure.

#### Acceptance Criteria

1. WHEN a new RADIUS client needs registration, THE system SHALL accept SSA-based registration requests through the OAuth dynamic client registration endpoint
2. WHEN processing SSA registration, THE Jans_Server SHALL validate the SSA signature and claims before creating the OAuth client
3. WHEN SSA registration succeeds, THE system SHALL provision the RADIUS client with appropriate OAuth client credentials and RADIUS shared secrets
4. WHEN SSA registration fails, THE system SHALL return detailed error information for troubleshooting
5. THE system SHALL support SSA revocation and client deregistration workflows

### Requirement 5: RADIUS Protocol Compliance

**User Story:** As a network engineer, I want the modernized system to maintain full RADIUS protocol compliance, so that existing network infrastructure continues to work without modification.

#### Acceptance Criteria

1. THE Radiator_RADIUS SHALL continue to accept standard RADIUS Access-Request packets from network access servers
2. WHEN processing RADIUS requests, THE system SHALL preserve all RADIUS attributes and forward relevant ones to the Jans authorization context
3. WHEN returning RADIUS responses, THE system SHALL include appropriate RADIUS attributes based on authorization policy decisions
4. THE system SHALL support RADIUS accounting (Start/Stop/Interim-Update) messages with proper correlation to authentication sessions
5. THE system SHALL maintain RADIUS shared secret authentication between RADIUS clients and Radiator server

### Requirement 6: Security Requirements

**User Story:** As a security officer, I want the modernized authentication system to eliminate credential exposure and provide comprehensive security controls, so that the system meets enterprise security standards.

#### Acceptance Criteria

1. THE system SHALL NOT transmit user passwords in plaintext between Radiator_RADIUS and Jans_Server
2. WHEN handling authentication challenges, THE system SHALL use secure token-based communication with appropriate expiration times
3. WHEN storing client credentials, THE system SHALL encrypt sensitive data at rest using industry-standard encryption
4. THE system SHALL implement rate limiting and brute force protection for authentication attempts
5. THE system SHALL validate all input parameters and sanitize data to prevent injection attacks

### Requirement 7: Audit and Logging

**User Story:** As a compliance officer, I want comprehensive audit logging of all authentication and authorization events, so that security incidents can be investigated and compliance requirements are met.

#### Acceptance Criteria

1. WHEN any authentication attempt occurs, THE system SHALL log the complete authentication flow including timestamps, user identity, RADIUS client, and outcome
2. WHEN authorization decisions are made, THE system SHALL log the Cedarling policy evaluation including which policies were applied and their results
3. WHEN SSA registration events occur, THE system SHALL log client registration, modification, and deregistration activities
4. THE system SHALL provide structured logging output suitable for SIEM integration and automated analysis
5. THE system SHALL support configurable log levels and log rotation to manage storage requirements

### Requirement 8: Performance and Scalability

**User Story:** As a system administrator, I want the modernized system to handle high authentication volumes without performance degradation, so that network access remains responsive during peak usage.

#### Acceptance Criteria

1. THE system SHALL process RADIUS authentication requests with response times under 200ms for 95% of requests under normal load
2. WHEN system load increases, THE system SHALL maintain authentication throughput of at least 1000 requests per second per server instance
3. THE system SHALL support horizontal scaling by adding additional Jans server instances without authentication flow changes
4. THE system SHALL implement connection pooling and caching strategies to optimize performance between Radiator_RADIUS and Jans_Server
5. THE system SHALL provide performance metrics and monitoring endpoints for operational visibility

### Requirement 9: Configuration and Deployment

**User Story:** As a deployment engineer, I want the system configuration to be externalized and deployment to be automated, so that environments can be managed consistently and efficiently.

#### Acceptance Criteria

1. THE system SHALL support configuration through environment variables and configuration files without requiring code changes
2. WHEN deploying to different environments, THE system SHALL use environment-specific configuration for endpoints, credentials, and policies
3. THE system SHALL provide configuration validation and startup health checks to detect misconfigurations early
4. THE system SHALL support rolling updates and blue-green deployments without authentication service interruption
5. THE system SHALL include deployment automation scripts and documentation for common deployment scenarios

### Requirement 10: Backward Compatibility and Migration

**User Story:** As a system administrator, I want to migrate from the current ROPG implementation to the new system without service disruption, so that users experience no authentication outages during the transition.

#### Acceptance Criteria

1. THE system SHALL support a migration mode where both ROPG and authorization_challenge flows can operate simultaneously
2. WHEN in migration mode, THE system SHALL route authentication requests to the appropriate flow based on client configuration
3. THE system SHALL provide migration tools to convert existing OAuth client configurations to the new format
4. THE system SHALL maintain audit trails during migration to track which clients have been successfully migrated
5. THE system SHALL support rollback capabilities in case migration issues are discovered

## Out-of-Scope Items

1. **RADIUS Server Replacement**: This project modernizes the OAuth integration but does not replace Radiator RADIUS server itself
2. **Network Infrastructure Changes**: Existing RADIUS clients (NAS devices) should not require configuration changes
3. **User Directory Integration**: Changes to LDAP/database user stores are not included in this scope
4. **RADIUS Accounting Modernization**: Focus is on authentication; accounting improvements are deferred to future phases
5. **Legacy Protocol Support**: Support for deprecated RADIUS extensions or non-standard attributes is not included

## Assumptions

1. **Jans Server Availability**: Assumes Jans server with authorization_challenge endpoint and Agama flow support is available
2. **Cedarling Integration**: Assumes Cedarling can be integrated with Jans server for policy evaluation
3. **Network Connectivity**: Assumes reliable network connectivity between Radiator RADIUS and Jans server components
4. **Certificate Management**: Assumes existing PKI infrastructure for SSL/TLS certificates and SSA validation
5. **Operational Readiness**: Assumes operations team has capacity to manage the additional complexity of policy-based authorization
