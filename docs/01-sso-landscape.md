# Enterprise SSO landscape

## Who this is for

Enterprise architects and application developers who need a **decision reference** for SSO: map a requirement to a pattern, understand which protocols apply, and know where to find topology diagrams and key configuration checklists—without portal walkthroughs or SDK code.

## Protocol map

| Protocol | Typical use | Token / assertion style | Notes |
|---|---|---|---|
| SAML 2.0 | Browser SSO to SaaS / enterprise apps | XML assertion | Common with Entra enterprise apps and ADFS |
| WS-Federation | Legacy browser SSO (esp. ADFS / .NET) | Security token via WS-Fed | Prefer OIDC for new cloud apps |
| OpenID Connect | Modern browser / SPA sign-in | ID token (+ often access token) | Built on OAuth 2.0 |
| OAuth 2.0 | API authorization | Access (and refresh) tokens | Not "SSO" alone; pairs with OIDC for user login |
| OAuth OBO | API → API with user context | New access token for downstream API | Entra-specific extension pattern |

## Decision table

| Requirement signal | Likely pattern | Primary protocol(s) | Jump to |
|---|---|---|---|
| Browser login to SaaS / partner site | Browser SSO (Entra) | SAML 2.0 or OIDC | [03](./03-browser-sso-saml-oidc.md) |
| Modern web/SPA calling your APIs | OIDC + OAuth | Auth code (+ PKCE), tokens to API | [03](./03-browser-sso-saml-oidc.md), [04](./04-api-oauth-obo.md) |
| Service/daemon calling APIs (no user) | App-only | Client credentials | [04](./04-api-oauth-obo.md) |
| API needs user context via another API | Delegated chain | OAuth On-Behalf-Of (OBO) | [04](./04-api-oauth-obo.md) |
| Users from partner org (their IdP or Entra) | Cross-federation | B2B + SAML/OIDC federation | [05](./05-cross-federation.md) |
| In-house app SSO against on-prem AD (legacy) | ADFS + Active Directory | WS-Fed and/or SAML 2.0 | [06](./06-legacy-adfs-ad.md) |

## Pattern catalog (summary)

- **Browser SSO (Entra):** Sign users into SaaS or partner websites via Entra enterprise applications using SAML 2.0 or OIDC—see [03](./03-browser-sso-saml-oidc.md); component and network context in [02](./02-components-and-topology.md); must-configure items in [07](./07-key-configurations.md).
- **OIDC + OAuth (web/SPA + APIs):** Modern first-party apps authenticate users with OIDC and call APIs with delegated OAuth tokens (authorization code + PKCE)—see [03](./03-browser-sso-saml-oidc.md) and [04](./04-api-oauth-obo.md); topology in [02](./02-components-and-topology.md); configs in [07](./07-key-configurations.md).
- **App-only (client credentials):** Daemons and background services obtain access tokens without a signed-in user—see [04](./04-api-oauth-obo.md); topology in [02](./02-components-and-topology.md); configs in [07](./07-key-configurations.md).
- **Delegated chain (OBO):** A middle-tier API exchanges the user's token for a downstream API token that preserves user context—see [04](./04-api-oauth-obo.md); topology in [02](./02-components-and-topology.md); configs in [07](./07-key-configurations.md).
- **Cross-federation:** Partner-organization users access your apps via B2B guests or inbound IdP federation (SAML/OIDC)—see [05](./05-cross-federation.md); federation topology in [02](./02-components-and-topology.md); configs in [07](./07-key-configurations.md).
- **Legacy ADFS + Active Directory:** In-house applications on the corporate identity plane authenticate against on-prem ADFS backed by AD—see [06](./06-legacy-adfs-ad.md); legacy path in [02](./02-components-and-topology.md); ADFS configs in [07](./07-key-configurations.md).

## Terminology

Shared definitions live in the [glossary](./glossary.md). Across protocols, the application that consumes tokens from the IdP is the same role under different names: SAML uses **SP** (service provider); OIDC uses **RP** (relying party); WS-Federation uses **relying party**—same role, different protocol vocabulary.

## What this reference deliberately skips

- Portal or AD FS Management Console step-by-step walkthroughs and screenshots
- SDK or language-specific code samples
- Deep Conditional Access or MFA product guidance (noted only as a requirement flag when SSO alone is insufficient)
- Kerberos and Windows Integrated Authentication internals (WIA is mentioned only as how corp-network users may authenticate to ADFS)
- Full ADFS → Entra migration runbook (decision guidance and "prefer Entra when…" only)
- Deep CIAM / Entra External ID for customer-facing identity (workforce B2B federation remains in scope)
- Runnable SSO demo applications
