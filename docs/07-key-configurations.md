# Key configurations to enable SSO

Checklists of **settings and artifacts** that must be known or configured before SSO works. Each item is a concrete noun — tenant URL, certificate, scope name, trust identifier — not a portal procedure. Pattern context and pitfalls live in the pattern docs; use this page as the consolidated reference they point to.

## Entra ID — browser app (OIDC or SAML)

- **Tenant ID** — GUID of your Entra tenant
- **Issuer URL** — `https://login.microsoftonline.com/{tenant-id}/v2.0` (OIDC) or SAML IdP entity ID from Entra federation metadata
- **Application (client) ID** — OIDC app registration client ID
- **SP Identifier (Entity ID)** — SAML audience / trust identifier the IdP and SP agree on (not the OIDC client ID)
- **Redirect URIs** (OIDC) — exact callback URLs the RP validates on return (scheme, host, path, trailing slash)
- **Assertion Consumer Service (ACS) URL** (SAML) — reply URL where the SP receives the SAML response POST
- **Logout URL** — federated sign-out endpoint registered on both sides when single logout is required
- **Protocol choice** — OpenID Connect or SAML 2.0; must match what the vendor or app supports
- **Client authentication method** (OIDC confidential clients) — client secret, certificate credential, or federated credential
- **SAML IdP signing certificate** — current signing cert from Entra IdP metadata; rollover schedule before expiry
- **Required outbound claims** — UPN, email, display name, and any vendor-mandated attribute names
- **Group → role strategy** — app roles assignment, security-group claims, filtered group claims, or Microsoft Graph group lookup (high level — pick one approach per app)
- **Enterprise application assignment** — users and groups permitted to sign in
- **Conditional Access / MFA requirement flag** — whether tenant policies require step-up beyond federation alone

## Entra ID — API / OAuth

- **API app registration** — resource server registration that exposes permissions
- **Application ID URI** — stable identifier (`api://{app-id}` or custom URI) used in scope names
- **Access token version** — `accessTokenAcceptedVersion` / `requestedAccessTokenVersion` on the API registration (determines whether `aud` is the Application ID URI or client ID GUID)
- **Exposed scopes (delegated permissions)** — named permissions clients request for user-delegated access
- **App roles (application permissions)** — roles for client-credentials and admin-consented app-only access
- **Authorized client applications** — optional preauthorization list for trusted calling apps (suppresses consent prompts; not the sole permission gate)
- **Admin consent grant** — tenant-wide consent record for delegated or application permissions
- **`.default` scope** — `https://graph.microsoft.com/.default` or `api://{api-app-id}/.default` for client credentials; for OBO exchange, prefer `.default` to obtain all consented delegated permissions (specific downstream scopes work only if already consented)
- **Audience validation rules** — API accepts tokens whose `aud`, `iss`, signature, `scp`/`roles`, and lifetime match its registration and access token version
- **Calling app delegated permissions** — scopes granted to the client app registration for Pattern A (user → API)
- **Calling app application permissions** — app roles assigned to the client for Pattern B (daemon / service)
- **OBO middle-tier registration** — confidential client representing the middle-tier API
- **OBO middle-tier delegated permissions** — consented delegated permissions on the **downstream** API registration (no permission named "OBO"; exchange typically requests downstream `/.default`, or specific scopes if already consented)
- **OBO downstream Application ID URI and scopes** — downstream resource identifier and permissions the exchanged token must carry
- **Client credential** — certificate (production) or secret (non-production) for confidential clients and OBO exchange

## Entra ID — cross-federation (Company A → Company B portal)

Primary scenario: **Company A employees** access **Company B’s portal**, sign in only at **Company A Entra** (credentials never through B), and **Company A manages RBAC**. No separate partner-user login. Detail: [05](./05-cross-federation.md).

### Pattern 1 — Multi-tenant app (A owns day-to-day RBAC)

- **Company B:** multi-tenant app registration; redirect URIs / ACS; optional app-role definitions  
- **Company A:** admin consent to B’s app; **user/group assignment** on the enterprise app in Tenant A  
- **Sign-in:** authorize routed to **Company A Entra** — passwords/MFA stay at A  

### Pattern 2 — B2B + A→B group sync (A owns group membership)

- **Company B:** enterprise app; B2B / cross-tenant access; **one-time** assignment of synced A groups to the portal  
- **Company A:** security groups for portal access; **cross-tenant synchronization** (A → B); MFA/CA for A workforce  
- **Sign-in:** still redirects to **Company A Entra** for the login page  

### Alternate — Company A uses non-Entra IdP (Okta / Ping / ADFS)

- **Federation metadata URL**, **issuer URI**, SAML/WS-Fed inbound claim requirements (persistent NameID + email, or ImmutableID + emailaddress)  
- Inbound **OIDC IdP federation** for workforce tenants is **out of scope** (Entra External ID / CIAM)  
- See [05 — Alternate](./05-cross-federation.md#alternate-non-entra-home-idp-for-company-a)

### Shared settings

- **Reply URL / redirect URI** — ACS or OIDC redirect on B’s portal; exact-match rules still apply  
- **Application outbound claims** — what B’s portal needs after sign-in  
- **Conditional Access** — A for workforce auth; B may apply resource CA and trust A MFA via cross-tenant access  
- **Do not** build a password form on B’s portal for A employees  

## ADFS / Active Directory — in-house RP

**WS-Fed / SAML**

- **ADFS federation service URL** — farm hostname; becomes the token **issuer** relying parties pin
- **Federation metadata URL** — WS-Fed or SAML metadata document (endpoints, token-signing certificates)
- **Relying party trust identifier (realm)** — audience URI the app validates (`wtrealm` / SAML `Audience`)
- **WS-Fed passive sign-in endpoint** — browser redirect target for WS-Federation sign-in
- **SAML Assertion Consumer Service (ACS) URL** — POST endpoint for SAML responses (when the app uses SAML RP semantics)
- **Issuance transform rules** — claim rules mapping AD attributes (UPN, email, display name, group SIDs) to outbound claim types
- **Token-signing certificate** — current ADFS signing cert in metadata; schedule rollover and update all relying parties before expiry
- **Token-signing certificate trust on the app** — app's trusted signing cert store or metadata import aligned with ADFS

**OIDC (ADFS 2016+)**

- **OIDC discovery URL** — `https://{adfs-hostname}/adfs/.well-known/openid-configuration` (issuer, authorize, token, JWKS)
- **Client ID** — application group / OAuth client identifier on ADFS
- **Redirect URI** — exact OIDC callback registered on ADFS and validated by the RP
- **Scopes** — `openid` for ID token; optional resource scopes for ADFS-issued access tokens
- **Client authentication / PKCE** — confidential client secret or certificate, or public-client PKCE as configured
- **JWKS URI** — app validates JWT signature locally using ADFS public keys (cached via discovery)

**Shared**

- **Authentication method assumption** — Windows Integrated Authentication (WIA) on corp network vs. forms-based authentication off-network
- **Active Directory attribute store** — domain controllers supplying credentials and directory attributes to ADFS
- **Web Application Proxy (WAP)** — optional edge publication of ADFS endpoints for external users (DMZ placement)

## Requirement flags (not full guides)

These settings affect whether federation alone is sufficient; full product guides are out of scope.

- **Conditional Access policies** — sign-in conditions, MFA requirements, compliant-device checks
- **Multifactor authentication (MFA)** — per-user, per-app, or policy-driven step-up beyond primary authentication
- **Token lifetime** — access token, refresh token, and SAML assertion `NotOnOrAfter` / JWT `exp` expectations
- **Certificate rollover ownership** — named team responsible for Entra IdP signing cert, partner federation cert, ADFS token-signing cert, and client certificate credential rotation
- **Clock synchronization (NTP)** — server time alignment for SAML and JWT lifetime validation

## Related

- [01 — Enterprise SSO landscape](./01-sso-landscape.md)
- [02 — Components and network topology](./02-components-and-topology.md)
- [03 — Browser SSO (SAML / OIDC)](./03-browser-sso-saml-oidc.md)
- [04 — API OAuth and OBO](./04-api-oauth-obo.md)
- [05 — Cross-federation](./05-cross-federation.md)
- [06 — Legacy ADFS and AD](./06-legacy-adfs-ad.md)
- [Glossary](./glossary.md)
