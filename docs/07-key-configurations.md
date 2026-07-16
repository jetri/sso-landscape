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

## Entra ID — cross-federation

Workforce B2B cross-federation uses **SAML 2.0 or WS-Federation** for partner IdPs (Okta, Ping, ADFS, and similar). Inbound **OIDC identity provider federation** is an Entra External ID (CIAM) feature — **out of scope** for workforce tenant direct federation. Federation is an **authentication method for B2B guests**, not a replacement for guest objects in your tenant.

### Option A — native Entra-to-Entra B2B

- **B2B collaboration** — guest or external-member onboarding path (invitation, self-service sign-up, or cross-tenant sync)
- **Cross-tenant access settings** — inbound/outbound B2B trust (for example, acceptance of home-tenant MFA and device claims); does not replace native B2B authentication routing
- **Guest `userType`** — partner users commonly arrive as `Guest` principals in your tenant
- **Application assignment for guests** — enterprise applications and app registrations must permit guest access where partner users need entry
- **RBAC in your (resource) tenant** — security groups, **app roles**, and **Azure RBAC** assignments on guest objects or groups in **your** directory; home IdP does not authorize your apps or subscriptions

### Option B — B2B with federated SAML/WS-Fed partner IdP

- **Federation metadata URL** — partner SAML metadata or WS-Fed federation metadata; refresh after partner certificate rollover
- **Issuer URI** — inbound issuer / entity ID the partner IdP asserts; must match partner configuration exactly
- **Federation routing pattern** — one of:
  - **Verified-domain map** — partner email domain mapped to the federated IdP (domain verified in the **partner home** tenant, **not** in your resource tenant)
  - **Unverified-domain map** — domain association without DNS verification in your tenant (with Microsoft constraints)
  - **Domainless SAML** — issuer association plus `domain_hint` (or equivalent login hint) where applicable
- **Inbound claim requirements (SAML 2.0)** — **persistent NameID** and matchable **email address** from the partner IdP
- **Inbound claim requirements (WS-Fed)** — **ImmutableID** and **emailaddress** from the partner IdP
- **Partner IdP signing certificate** — current signing cert from federation metadata; update trust on partner rollover
- **Redemption order** — federation vs. Microsoft Entra priority during invitation redemption for verified-domain scenarios

Federation trust does **not** offer free-form inbound claim transform rules for groups, roles, or `preferred_username`. Configure **application outbound claims** separately for what Entra emits **after** B2B sign-in completes.

### Shared cross-federation settings

- **Application outbound claims** — UPN, email, display name, group, or role claims in Entra-issued SAML assertions, OIDC ID tokens, or access tokens
- **Conditional Access scoping** — policies targeting `All guest and external users` or specific guest groups (distinct from outbound claim configuration)
- **External identity licensing** — monthly active user billing for B2B guests where applicable
- **Reply URL / redirect URI** — ACS URL or OIDC redirect URI on your apps unchanged by upstream federation; exact-match rules still apply

## ADFS / Active Directory — in-house RP

- **ADFS federation service URL** — farm hostname; becomes the token **issuer** relying parties pin
- **Federation metadata URL** — WS-Fed or SAML metadata document (endpoints, token-signing certificates)
- **Relying party trust identifier (realm)** — audience URI the app validates (`wtrealm` / SAML `Audience`)
- **WS-Fed passive sign-in endpoint** — browser redirect target for WS-Federation sign-in
- **SAML Assertion Consumer Service (ACS) URL** — POST endpoint for SAML responses (when the app uses SAML RP semantics)
- **Issuance transform rules** — claim rules mapping AD attributes (UPN, email, display name, group SIDs) to outbound claim types
- **Token-signing certificate** — current ADFS signing cert in metadata; schedule rollover and update all relying parties before expiry
- **Token-signing certificate trust on the app** — app's trusted signing cert store or metadata import aligned with ADFS
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
