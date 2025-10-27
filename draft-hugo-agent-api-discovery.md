---
title: "API Discovery and Authentication for AI Agents"
abbrev: "API Discovery for AI Agents"
category: info

docname: draft-hugo-agent-api-discovery-latest
date: 2025-10-27
ipr: trust200902
area: Applications and Real-Time
workgroup: Individual Submission

keyword:
 - AI
 - Agent
 - API
 - Discovery
 - OAuth
 - OpenAPI

author:
 - ins: J. Hugo
   name: Jasper Hugo
   organization: Independent
   email: jasper@jasperhugo.com

---
# Abstract

This document describes a method for AI agents to discover, understand, and authenticate to APIs by composing existing Internet standards. It proposes that the combination of api-catalog for discovery, OpenAPI for capability description and schema discovery, and OAuth 2.0 with its associated metadata specifications for authentication and authorization provides a robust, decentralized alternative to bespoke single-purpose protocols.

This approach avoids vendor lock-in, reduces complexity by reusing well-understood technologies, and promotes an open and interoperable ecosystem for AI agent interactions with the existing Web.

# Introduction

The proliferation of AI agents necessitates a standardized mechanism for them to interact with the vast landscape of existing APIs. These agents require methods to discover available APIs, understand their functionality, and securely authenticate to them on behalf of a user or system.

## Problem Statement

Agent-API interaction faces several challenges:

- **Discovery Fragmentation**: No standardized way for agents to discover available APIs across different domains
- **Integration Complexity**: Each API requires custom integration code and documentation parsing
- **Authentication Inconsistency**: Different APIs use various authentication mechanisms without standardized discovery
- **Vendor Lock-in**: Proprietary solutions create dependencies on specific platforms or protocols

While solutions like the Model Context Protocol (MCP) have gained popularity, they introduce new protocols and create additional complexity in the ecosystem. These challenges make it difficult for AI agents to dynamically interact with the diverse ecosystem of existing APIs, limiting their effectiveness and creating barriers to interoperability.

## Solution Approach

This document proposes a composable approach that leverages existing Internet standards rather than introducing new protocols. By combining api-catalog [[RFC9727]], OpenAPI [[OAS]], and OAuth 2.0 [[RFC6749]] with its discovery mechanisms [[RFC8414]], this approach provides a robust, decentralized solution that avoids vendor lock-in and reduces complexity.

The core capabilities required by an agent can be broken down into three stages:

1. **Discovery**: Finding the right API for a given task
2. **Description**: Understanding the API's endpoints, parameters, and data structures
3. **Delegated Authorization**: Gaining secure, scoped access to the API on behalf of a user

This three-stage approach fully addresses the requirements for AI agent-API interaction using well-established Internet standards.

# Terminology

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD", "SHOULD NOT", "RECOMMENDED", "NOT RECOMMENDED", "MAY", and "OPTIONAL" in this document are to be interpreted as described in BCP 14 [[RFC2119]] [[RFC8174]] when, and only when, they appear in all capitals, as shown here.

# The Composable Approach

## 1. Discovery with api-catalog

The first stage of agent-API interaction requires the agent to discover which APIs are available for a given task. RFC 9727 "api-catalog: A Well-Known URI and Link Relation to Help Discovery of APIs" [[RFC9727]] provides a standardized mechanism for this discovery process.

### Agent Workflow

An agent implementing this discovery mechanism would:

1. **Direct Discovery**: Query `/.well-known/api-catalog` or use link relations to discover APIs available on target domains
2. **Catalog Processing**: Parse the API catalog (in Linkset format [[RFC9264]]) to extract API endpoints and metadata
3. **Capability Matching**: Match discovered APIs against the agent's task requirements

This approach enables agents to systematically discover APIs across the Web without requiring custom discovery protocols or vendor-specific mechanisms.

## 2. Description with OpenAPI Specification

Once an agent has discovered an API through the catalog mechanism described in Section 1, it needs to understand the API's capabilities, parameters, and data structures. The OpenAPI Specification [[OAS]] provides a standardized, machine-readable format for this purpose.

### OpenAPI Document Discovery

While OpenAPI documents are typically served at conventional endpoints (such as `/openapi.json`, `/swagger.json`, or `/api-docs`), the api-catalog mechanism described in Section 1 can include direct links to OpenAPI specifications. This creates a seamless flow from discovery to description:

1. **Catalog Reference**: The api-catalog includes links to OpenAPI documents for each discovered API
2. **Direct Access**: Agents can fetch OpenAPI documents directly from the URLs provided in the catalog
3. **Conventional Discovery**: As a fallback, agents MAY attempt to discover OpenAPI documents at common endpoints

### Agent Workflow

An agent implementing this description mechanism would:

1. **Fetch OpenAPI Document**: Retrieve the OpenAPI specification from the URL provided in the api-catalog
2. **Parse Specification**: Process the document to extract API metadata
3. **Analyze Capabilities**: Match API operations against the agent's task requirements
4. **Plan Integration**: Determine which endpoints and parameters are needed for the task
5. **Prepare Requests**: Structure API calls according to the OpenAPI schema

This standardized approach enables agents to dynamically understand and integrate with APIs without requiring custom integration code or vendor-specific documentation formats.

### Scalability Considerations

This approach scales well across different deployment scenarios:

- **Distributed Discovery**: Agents can discover APIs across multiple domains without centralized coordination
- **Caching**: OpenAPI specifications and api-catalog responses can be cached to reduce discovery overhead
- **Incremental Updates**: Agents can refresh their understanding of APIs as specifications evolve
- **Load Distribution**: No single point of failure in the discovery process

## 3. Delegated Authorization with OAuth 2.0

After discovering and understanding an API's capabilities, an agent must obtain authorization to access protected resources on behalf of a user. The OAuth 2.0 Authorization Framework [[RFC6749]] provides a standardized mechanism for this delegated authorization, while RFC 8414 [[RFC8414]] enables agents to discover authorization server capabilities.

### Authorization Server Discovery

RFC 8414 defines the OAuth 2.0 Authorization Server Metadata specification, which provides a well-known URI `/.well-known/oauth-authorization-server` for discovering authorization server configuration. This metadata includes:

- Authorization and token endpoint URLs
- Supported grant types and response types
- Supported scopes and token endpoint authentication methods
- Additional capabilities and extensions

An agent can discover authorization server capabilities by fetching this metadata, enabling it to determine the appropriate OAuth 2.0 flow and required parameters for obtaining access tokens.

### OAuth 2.0 Grant Types for Agents

Different OAuth 2.0 grant types are suitable for different agent deployment scenarios:

- **Authorization Code Grant**: Suitable for agents with a web-based user interface where users can interact with the authorization server
- **Client Credentials Grant**: Appropriate for service-to-service scenarios where the agent acts on behalf of itself rather than a user
- **Device Authorization Grant**: Useful for agents running on devices with limited input capabilities

The choice of grant type depends on the agent's deployment context and the level of user interaction required.

### Security Considerations

Agents implementing OAuth 2.0 MUST adhere to the security best practices outlined in the OAuth 2.0 Security Best Current Practice [[I-D.ietf-oauth-security-topics]]. Key considerations include:

- Using HTTPS for all OAuth 2.0 communications
- Implementing proper state parameter validation
- Securing client credentials and access tokens
- Implementing appropriate token refresh mechanisms

### Agent Workflow

An agent implementing delegated authorization would:

1. **Discover Authorization Server**: Fetch the authorization server metadata from `/.well-known/oauth-authorization-server`
2. **Determine Grant Type**: Select the appropriate OAuth 2.0 grant type based on the agent's capabilities and deployment context
3. **Initiate Authorization Flow**: Follow the standard OAuth 2.0 flow for the selected grant type to obtain an access token
4. **Use Access Token**: Include the access token in API requests to access protected resources
5. **Handle Token Refresh**: Implement token refresh mechanisms to maintain access when tokens expire

This standardized approach enables agents to securely obtain and use access tokens without requiring custom authentication protocols or vendor-specific authorization mechanisms.

### Error Handling

Agents implementing this approach should handle common failure scenarios:

- **Discovery Failures**: If `/.well-known/api-catalog` is unavailable, agents MAY fall back to conventional OpenAPI discovery endpoints
- **OpenAPI Parsing Errors**: Malformed or invalid OpenAPI documents should be handled gracefully with appropriate error reporting
- **OAuth Flow Failures**: Authorization failures should be communicated clearly to users with guidance on resolution
- **Protocol Errors**: Implementations should handle HTTP error responses and invalid protocol responses appropriately

# Security Considerations

This approach relies on the security models of its constituent parts, primarily OAuth 2.0. Implementers MUST adhere to the security best practices outlined in the relevant specifications, including "OAuth 2.0 for Native Apps" [[RFC8252]] and "OAuth 2.0 Security Best Current Practice" [[I-D.ietf-oauth-security-topics]]. By leveraging these well-vetted standards, this approach avoids introducing novel security vulnerabilities that a new protocol might create.

# IANA Considerations

This document has no IANA actions.

# References

## Normative References

[RFC2119] Bradner, S., "Key words for use in RFCs to Indicate Requirement Levels", BCP 14, RFC 2119, DOI 10.17487/RFC2119, March 1997, <https://www.rfc-editor.org/info/rfc2119>.

[RFC6749] Hardt, D., Ed., "The OAuth 2.0 Authorization Framework", RFC 6749, DOI 10.17487/RFC6749, October 2012, <https://www.rfc-editor.org/info/rfc6749>.

[RFC8174] Leiba, B., "Ambiguity of Uppercase vs Lowercase in RFC 2119 Key Words", BCP 14, RFC 8174, DOI 10.17487/RFC8174, May 2017, <https://www.rfc-editor.org/info/rfc8174>.

[RFC8252] Denniss, W. and J. Bradley, "OAuth 2.0 for Native Apps", BCP 212, RFC 8252, DOI 10.17487/RFC8252, October 2017, <https://www.rfc-editor.org/info/rfc8252>.

[RFC8414] Jones, M., Sakimura, H., and J. Bradley, "OAuth 2.0 Authorization Server Metadata", RFC 8414, DOI 10.17487/RFC8414, June 2018, <https://www.rfc-editor.org/info/rfc8414>.

[RFC9264] Nottingham, M., "The Linkset Format for Link Sets", RFC 9264, DOI 10.17487/RFC9264, July 2022, <https://www.rfc-editor.org/info/rfc9264>.

[RFC9727] Nottingham, M., "api-catalog: A Well-Known URI and Link Relation to Help Discovery of APIs", RFC 9727, DOI 10.17487/RFC9727, January 2025, <https://www.rfc-editor.org/info/rfc9727>.

## Informative References

[I-D.ietf-oauth-security-topics] Lodderstedt, T., Bradley, J., Labunets, A., and D. Fett, "OAuth 2.0 Security Best Current Practice", Work in Progress, Internet-Draft, draft-ietf-oauth-security-topics-27, 10 October 2024, <https://datatracker.ietf.org/doc/html/draft-ietf-oauth-security-topics-27>.

[OAS] OpenAPI Initiative, "OpenAPI Specification", Version 3.1.0, 15 February 2021, <https://spec.openapis.org/oas/v3.1.0>.
---
