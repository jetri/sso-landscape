# Glossary

Terms used across this SSO reference.

### Access token

An OAuth 2.0 bearer credential issued by the authorization server (typically Entra ID or ADFS) that grants access to a protected resource such as an API. Applications present it in the `Authorization` header; APIs validate issuer, signature, audience, and lifetime before honoring the request. Choose token lifetime and scope breadth to match the integration pattern (browser-delegated, app-only, or OBO chain).

### ACS (Assertion Consumer Service)

The SAML endpoint URL where the IdP POSTs a signed SAML assertion after the user authenticates. The SP registers this URL with the IdP; a mismatch is a common cause of login failure. For browser SSO, the ACS must be reachable over HTTPS and match the value in federation metadata or the enterprise application configuration.

### Active Directory (AD)

Microsoft's on-premises directory service that stores user accounts, groups, and organizational units in a domain forest. In legacy patterns, AD is the authoritative identity store and ADFS reads from it to issue federation tokens. Prefer Entra ID for new cloud and SaaS integrations; retain AD as the source when apps and users remain corp-network-centric.

### ADFS (Active Directory Federation Services)

On-premises Windows Server role that acts as a federation IdP/STS for in-house applications backed by Active Directory. It issues WS-Federation or SAML tokens to relying party trusts and can federate outbound to cloud IdPs. Choose ADFS when the app, users, and trust model are still AD-centric; plan Entra migration when building new cloud-facing workloads.

### Audience (`aud`)

A claim that names the intended recipient of a token—typically an API's Application ID URI or client ID. Resource servers must reject tokens whose `aud` does not match their configured identifier; wrong audience is a frequent integration defect. Align audience between token issuer settings and API validation logic before go-live.

### B2B guest user

An external identity invited into your Entra tenant to access your applications without creating a full member account in your directory. The guest may authenticate via their home IdP (federated) or a one-time passcode, depending on trust configuration. Use B2B when partner org users need access to your apps; distinguish guests from native members for licensing, Conditional Access, and group assignment.

### Claim / assertion

**Claim:** A name–value statement about a subject (user, client, or session) embedded in a token, SAML assertion, or OIDC ID token. IdPs map directory attributes to outbound claims; applications rely on specific claims (UPN, email, groups, roles) for authorization decisions.

**Assertion:** A signed SAML structure that carries one or more claims about a subject, issued by the IdP and consumed by the SP at the ACS. Assertions bind authentication and attribute statements to a specific audience and lifetime—the signed envelope, not a single claim inside it.

### Client credentials

An OAuth 2.0 grant where a confidential client authenticates with its own identity (client ID and secret or certificate) to obtain an access token without a user present. Use this pattern for daemons, batch jobs, and service-to-service calls that do not need delegated user context. Restrict permissions to the minimum app roles or scopes required; prefer certificate-based client authentication over shared secrets in production.

### Federation metadata

Machine-readable XML (SAML/WS-Fed) or JSON (OIDC discovery) published by an IdP describing endpoints, certificates, and supported bindings. Partners import metadata to establish trust without manually copying every URL and signing key. Refresh metadata after certificate rollover and validate that endpoint URLs match your environment (internal vs external, WAP-published vs direct).

### ID token

An OIDC JWT returned to the client alongside (or instead of) an access token, asserting who authenticated and how. It carries identity claims (`sub`, `iss`, `aud`, `exp`) and is intended for the client application, not for calling APIs. Do not send ID tokens to resource APIs; use access tokens with the correct audience and scopes instead.

### IdP (Identity Provider)

The authority that authenticates users (or clients) and issues security tokens or assertions to trusted applications. In this reference, Entra ID is the primary modern IdP; ADFS fills the same role for legacy AD-backed apps. The IdP owns sign-in experience, token signing, and federation outbound to partner identity systems.

### Issuer

The identifier (`iss` claim or SAML `<Issuer>`) that names the token-issuing authority—commonly `https://login.microsoftonline.com/{tenant-id}/v2.0` for Entra or the ADFS federation service URL. Relying parties and APIs must pin validation to the expected issuer to prevent token substitution. Document the issuer URL in runbooks; it changes when migrating tenants or federation endpoints.

### OAuth 2.0

An authorization framework for obtaining limited access to resources on behalf of a resource owner or client. Core grant types are authorization code, client credentials, and refresh token exchange (the implicit grant is legacy and deprecated). Extensions such as device authorization (RFC 8628) address additional client scenarios. OAuth 2.0 separates the authorization server from resource APIs; pair it with OIDC when browser or native apps need both API access and standardized identity claims.

### OIDC (OpenID Connect)

An identity layer on OAuth 2.0 that adds authentication semantics, the ID token, and a standard UserInfo endpoint. Enterprises use OIDC for modern browser SSO, SPAs, and mobile apps against Entra ID and other IdPs. Prefer OIDC over SAML for new SaaS and first-party web apps when both parties support it.

### OBO (On-Behalf-Of)

An Entra ID and Microsoft identity platform extension pattern—not a core OAuth 2.0 grant type—where a middle-tier API uses token exchange or assertion mechanisms to obtain a downstream access token that preserves the signed-in user's context. Requires explicit permission grants and a confidential middle-tier registration. Use OBO when a web or API tier must call another API as the signed-in user without storing refresh tokens in the browser.

### Redirect URI

The pre-registered callback URL where the authorization server sends the user (and authorization code or tokens) after login. Entra and other IdPs reject redirects that do not exactly match registered values, including trailing slashes and scheme. Register separate redirect URIs per environment and client type (web, SPA, native) to avoid cross-environment token leakage.

### Refresh token

An OAuth 2.0 credential optionally issued by the authorization server that a client presents only to the token endpoint—never to resource APIs as a Bearer credential—to obtain new access tokens (and optionally a new refresh token) without re-prompting the user. Confidential and public clients, including native and mobile apps, may receive refresh tokens when authorization server policy allows, typically with controls such as PKCE, rotation, and secure storage. Prefer a backend or token broker to hold refresh tokens for browser-only SPAs when possible.

### Relying party (WS-Fed) / Relying party trust

In WS-Federation and ADFS, the application or trust object that consumes tokens issued by the federation service. A relying party trust defines the realm identifier, token issuance rules, and signing certificate requirements. Misconfigured RP identifiers or claim rules are the most common ADFS integration failures.

### RP (Relying Party / OIDC)

In OpenID Connect, the client application that receives tokens from the IdP and relies on them to establish a session or call APIs. Registration includes client ID, redirect URIs, and requested scopes. Treat the RP as the OIDC equivalent of a SAML service provider.

> SAML uses **SP**; OIDC uses **RP**; WS-Fed uses **relying party** — same role, different protocol vocabulary.

### SAML 2.0

An XML-based federation protocol where the IdP sends a signed assertion to the SP, typically via browser POST to an ACS URL. Common for enterprise SaaS integrations and legacy ADFS relying parties. Choose SAML when a vendor or partner requires it; otherwise evaluate OIDC for simpler token handling in modern stacks.

### Scope

An OAuth 2.0 parameter that limits access token permissions to specific resources or actions (e.g., `api://app-id/read`). Delegated scopes require user consent; application permissions (`.default` or app roles) apply in client-credentials flows. Request the narrowest scope set that satisfies the integration to reduce blast radius if a token is exposed.

### SP (Service Provider)

In SAML 2.0, the application that consumes assertions from the IdP and creates a local session for the user. The SP exposes an ACS URL and publishes metadata or manual settings to the IdP. When evaluating federation options, map your application to the SP role regardless of whether the vendor calls it an "enterprise application" or "SAML integration."

> SAML uses **SP**; OIDC uses **RP**; WS-Fed uses **relying party** — same role, different protocol vocabulary.

### STS (Security Token Service)

A component that issues, validates, and brokers security tokens—Entra ID, ADFS, and partner IdPs all function as STS instances in federation topologies. The STS performs authentication, applies issuance policy, and signs tokens. Understanding which STS sits in the trust path clarifies where to configure certificates, claim rules, and federation metadata.

### Tenant (Entra)

An isolated Entra ID (Azure AD) directory boundary representing an organization, with its own users, groups, app registrations, and federation settings. Cross-tenant B2B and org-to-org federation require explicit trust between tenants. Always record tenant ID and primary domain when documenting integrations—they anchor issuer URLs and token validation.

### Token-signing certificate

The X.509 certificate an IdP publishes in federation metadata whose public key relying parties use to verify SAML assertion or JWT signatures; the IdP holds the matching private key separately in protected storage. Applications and API gateways must trust the IdP's signing certificate (or metadata rollover schedule) before accepting tokens. Plan for certificate expiry and rollover; stale trust stores cause sudden, organization-wide authentication outages.

### WAP (Web Application Proxy)

A reverse-proxy role that publishes internal ADFS (and other) endpoints to the internet without placing the ADFS farm directly in a DMZ. External users hit WAP-published URLs while corp users may reach ADFS directly. Include WAP in topology diagrams when partner or remote users authenticate against on-prem ADFS.

### WS-Federation (WS-Fed)

A Microsoft-centric passive browser federation protocol where ADFS issues WS-Federation security tokens to relying parties via redirects and form posts. Still common for legacy .NET in-house applications integrated with ADFS relying party trusts. Prefer SAML 2.0 or OIDC for new integrations unless the application stack requires WS-Fed.

### WIA (Windows Integrated Authentication)

On the corporate network, users may authenticate to ADFS silently via Windows Integrated Authentication (Kerberos/NTLM) without re-entering credentials, subject to intranet zone and SPN configuration.
