# Cross-federation and external identities

## Choose this when

- Users **belong to another organization** and need access to **your** Entra-integrated applications
- Partner users must **authenticate at their home IdP** — partner **Entra** (via B2B/cross-tenant access) or a **SAML 2.0 / WS-Fed** authority like **Okta**, **Ping**, or **ADFS** — rather than receiving credentials in your directory
- You are deciding **which authentication method** a B2B guest population should use to sign in: native Entra-to-Entra B2B, or B2B backed by a federated SAML/WS-Fed identity provider

## Prefer another pattern when

- **All users are members of your tenant only** (no external org identities) → [03 — Browser SSO](./03-browser-sso-saml-oidc.md) or [04 — API OAuth and OBO](./04-api-oauth-obo.md)
- **Pure on-prem ADFS internal** workloads backed by Active Directory, not Entra as the resource-tenant IdP → [06 — Legacy ADFS and AD](./06-legacy-adfs-ad.md)

## Where cross-federation sits

Your Entra tenant remains the **resource-tenant IdP** for your applications: SAML SPs, OIDC RPs, and OAuth APIs still trust **your** issuer and signing keys.

**Important:** in a workforce tenant, SAML/WS-Fed IdP federation is not an alternative *to* B2B — it is an **authentication method used by B2B collaboration**. A partner user still ends up with an identity in **your** tenant (typically a guest, or an external member via cross-tenant sync); federation only changes **where that identity's credentials are verified** (the partner's IdP instead of Entra-managed credentials or a one-time passcode). Self-service sign-up and entitlement management can remove the need for an administrator to send a manual invitation, but they still create and manage a guest lifecycle in your directory — federation does not, by itself, grant domain-wide access with no account in your tenant.

Partner users authenticate at their **home IdP** — the partner's own Entra tenant (via native B2B / cross-tenant access) or a federated SAML/WS-Fed provider such as Okta, Ping, or ADFS. Entra then completes the B2B guest sign-in and issues the same SAML assertions, OIDC tokens, or OAuth access tokens your apps already expect from member users.

See the **Partner federation** subgraph in [02 — Components and network topology](./02-components-and-topology.md#high-level-components): partner IdP endpoints exchange trust metadata with your Entra tenant; browser redirects cross that boundary before your app receives a session.

## Option A — Entra B2B collaboration, native Entra-to-Entra

**B2B collaboration** brings an external identity into your Entra tenant to access your applications, as a **guest** (most common) or, when the partner org enables it, an **external member** via cross-tenant synchronization. The object lives in your directory; the partner user does not become a full native member of your tenant.

**When the partner also uses Entra ID, prefer native B2B / cross-tenant access over configuring SAML federation between the two tenants.** Entra↔Entra collaboration is handled by **cross-tenant access settings** (inbound/outbound trust of MFA, device, and Conditional Access claims) plus B2B invitation or self-service sign-up — not by setting up the partner tenant as a custom SAML/WS-Fed identity provider. Treating "two Entra tenants" as a SAML-federation problem is a common misconfiguration; the native B2B path already knows how to route authentication to the partner tenant.

**Lifecycle:** An administrator (or automated invitation flow) invites the partner user by email, or the partner self-service signs up (optionally gated by entitlement management) if your tenant allows it for their domain. The guest **redeems** the invitation — typically by signing in at their **home Entra tenant**, or via email one-time passcode when the partner has no Entra tenant and no federation is configured. Self-service and entitlement management reduce *manual* invite effort, but the resulting guest object, its lifecycle, and its access reviews still live in your tenant like any other B2B account.

**Authentication:** For **Entra↔Entra**, the guest signs in at **partner Entra**; cross-tenant access settings route authentication to the home tenant automatically, and your apps see a guest (or external member) principal in **your** tenant with claims issued by your Entra. Partners without their own Entra tenant (Okta, Ping, ADFS, or similar) cannot use this native path — see Option B.

**When it fits:** Named partner individuals or small populations; you want **per-user visibility** in your tenant (assignment, audit, group membership); the partner org also uses Entra ID.

## Option B — B2B with a federated SAML/WS-Fed identity provider

When the partner org does **not** use Entra ID, configure their IdP — **Okta**, **Ping Identity**, **ADFS**, or another **SAML 2.0 or WS-Fed** authority — as a federated identity provider under Entra's external identity providers, then map a verified partner **domain** to it. Guests whose email matches that domain are redirected to the partner IdP to authenticate instead of using an Entra-managed password or email one-time passcode.

This is still **B2B**: Entra still creates and manages a guest object for each partner user in your tenant. Federation only changes the guest's **authentication method** — where their credentials are verified — not whether an account exists or whether an invitation/onboarding step happens.

**Protocol scope:** **SAML 2.0 and WS-Fed** are the protocols for workforce B2B direct federation with an arbitrary partner IdP (Okta, Ping, ADFS, and similar). Google workspace federation is a separate, vendor-specific path. Inbound **OIDC identity provider federation** is an **Entra External ID** (CIAM / external-tenant) feature — **out of scope** for this workforce reference; do not treat it as an available pattern for federating partner workforce IdPs in a corporate tenant.

**Partner is another Entra tenant?** Don't configure it as a SAML/WS-Fed identity provider — use Option A (native B2B / cross-tenant access) instead.

**Key concepts (configuration level):**

- **Federation metadata URL** — partner publishes SAML metadata (or WS-Fed federation metadata); your Entra imports endpoints and signing certificates
- **Issuer URI** — the identifier your Entra expects on inbound assertions (`Issuer` / entity ID); must match partner configuration exactly
- **Domain federation** — maps an email domain (e.g., `partner.com`) to the federated IdP so Entra routes matching guests to the partner sign-in flow instead of OTP
- **Inbound claim requirements** — Entra's federation trust expects fixed inbound claims per Microsoft documentation: a **persistent NameID** (SAML/WS-Fed) and a matchable **email address** so Entra can create or match the guest object. Federation configuration does not offer free-form inbound transform rules for groups, roles, or `preferred_username` the way an app's outbound claims do. Configure **your applications** separately to consume the claims Entra puts in the SAML assertion, OIDC ID token, or access token **after** B2B sign-in completes (see [03 — Browser SSO](./03-browser-sso-saml-oidc.md) and [07 — Key configurations](./07-key-configurations.md))

Federating a domain to a partner IdP still results in guest objects for that population — it changes *how* guests authenticate and can reduce reliance on per-user email OTP, but it does not remove the guest account, its assignment requirements, or its lifecycle review.

## B2B vs federated IdP (decision)

Both rows below are **B2B collaboration**. The decision is which **authentication method** the guest's home-tenant sign-in uses, and separately, which **onboarding path** creates the guest object.

| Aspect | Option A — native Entra home tenant | Option B — federated SAML/WS-Fed home IdP |
|---|---|---|
| **Home IdP / protocol** | Partner's own Entra tenant; cross-tenant access settings | Okta, Ping, ADFS, or other SAML 2.0 / WS-Fed authority configured as a federated identity provider |
| **Applies when** | Partner org uses Entra ID | Partner org does **not** use Entra ID |
| **Who manages credentials** | Partner org, in their Entra tenant | Partner org, in their SAML/WS-Fed IdP |
| **You configure** | Cross-tenant access settings (trust inbound/outbound MFA, device, CA claims) | Federated IdP metadata, issuer, domain federation mapping |
| **Guest object in your tenant** | Yes — created by invitation, self-service sign-up, or cross-tenant sync | Yes — created on first sign-in or invitation; federation just points its auth at the partner IdP |

**Onboarding path (orthogonal to the table above, applies to either option):** a guest object can arrive via administrator invitation, self-service sign-up (optionally gated by entitlement management), or automated provisioning — independent of whether that guest's sign-in ultimately uses native Entra routing or a federated SAML/WS-Fed IdP.

## Sequence: partner user into your Entra app

```mermaid
sequenceDiagram
  actor PartnerUser
  participant Browser
  participant App as Your app (SAML SP or OIDC RP)
  participant YourEntra as Your Entra tenant (B2B guest)
  participant HomeIdP as Home IdP (partner Entra via B2B, or federated SAML/WS-Fed IdP)

  PartnerUser->>Browser: Open your app
  Browser->>App: Request
  App->>Browser: Redirect to Your Entra (SAML AuthnRequest or OIDC /authorize)
  Browser->>YourEntra: Sign-in as guest
  YourEntra->>Browser: Redirect to Home IdP (guest's configured authentication method)
  Browser->>HomeIdP: Authenticate with partner credentials
  HomeIdP->>Browser: Assertion or ID token back toward Your Entra
  Browser->>YourEntra: Complete guest sign-in at Your Entra
  alt App is OIDC / OAuth
    YourEntra->>Browser: Redirect with authorization code
    Browser->>App: Authorization code
    App->>YourEntra: Exchange code for tokens (back channel)
    YourEntra->>App: ID token / access token
  else App is SAML
    YourEntra->>Browser: SAML Response (auto-submit form)
    Browser->>App: POST to Assertion Consumer Service (ACS)
  end
  App->>Browser: Signed-in session for guest
```

Your Entra does not uniformly hand the browser "tokens" for every app: **OIDC/OAuth apps** redeem an authorization code for tokens over the back channel, while **SAML apps** receive a SAML response posted by the browser to the ACS endpoint — the same protocol mechanics as [03 — Browser SSO](./03-browser-sso-saml-oidc.md). Only the **upstream authentication path** (the redirect to the home IdP before Your Entra completes sign-in) is added by cross-federation; the app-facing protocol is unchanged.

## Key configurations

Detailed checklists and Entra field names live in [07 — Key configurations](./07-key-configurations.md). For cross-federation, confirm at minimum:

- **Federation metadata URL** — partner SAML metadata or WS-Fed federation metadata; refresh after partner certificate rollover
- **Issuer URI** — inbound issuer matches partner IdP entity ID (`Issuer` / entity ID); mismatch causes immediate sign-in failure
- **Federated domain** — verified partner domain routed to the federated SAML/WS-Fed IdP (Option B) or covered by cross-tenant access settings (Option A / Entra↔Entra)
- **Inbound claim requirements** — partner IdP must send the persistent NameID and email address Entra expects; validate partner SAML/WS-Fed configuration against Microsoft's federation claim requirements rather than assuming arbitrary inbound remap rules
- **Application outbound claims** — after B2B sign-in, configure each enterprise application or app registration for the UPN, email, display name, group, or role claims your apps and Conditional Access policies consume from **Entra-issued** assertions or tokens
- **`userType`, licensing, and Conditional Access are distinct settings** — B2B collaboration users commonly default to `userType = Guest`, but licensing (e.g., external identity monthly active user billing), Conditional Access scoping (`All guest and external users` vs specific users/groups), and group assignment are each configured independently; don't assume setting `userType` alone determines every policy outcome
- **Application assignment** — enterprise applications and app registrations must allow guest access where partner users need entry; review default member-only assignments

## Common pitfalls

- **Assuming federation removes the guest account** — SAML/WS-Fed IdP federation is an authentication method *for* B2B guests, not a replacement for one; a federated domain still produces guest objects with a lifecycle, assignment, and access reviews in your tenant
- **Configuring SAML federation between two Entra tenants** — when the partner already has Entra ID, use native B2B collaboration and cross-tenant access settings (Option A), not a custom SAML/WS-Fed identity provider; treating Entra↔Entra as a generic SAML federation problem is unnecessary and harder to maintain
- **Expecting OIDC inbound IdP federation in a workforce tenant** — workforce B2B direct federation is **SAML 2.0 / WS-Fed only**. Inbound OIDC IdP federation belongs to **Entra External ID** (CIAM) scenarios and is **out of scope** here; do not plan a partner Okta/Ping federation around generic OIDC in a corporate workforce tenant
- **Treating guests like members** — guest users commonly default to `userType = Guest`, but licensing, Conditional Access, and group limits are separate settings; policies that assume `Member` user type or that assume `userType` alone drives every outcome can block partner access silently or at CA evaluation
- **Issuer mismatch** — partner rotates IdP certificates or changes entity ID without updating your federation trust; validate the issuer/entity ID and signing cert on every partner change
- **Assuming inbound federation claim transforms** — Entra's trust with the partner IdP enforces required inbound claims (persistent NameID, email), not arbitrary remap rules. If your app expects `preferred_username`, a specific SAML NameID format, or group claims, configure **outbound** claims on the Entra enterprise application or app registration for what Entra emits **after** guest sign-in — do not expect the federation trust itself to rewrite partner assertions freely
- **Reply URL and redirect URI unchanged** — federation fixes upstream auth but your app's ACS/redirect URI must still match registration; partner federation does not relax SP/RP URL exact-match rules

## Related

- [02 — Components and network topology](./02-components-and-topology.md)
- [03 — Browser SSO (SAML / OIDC)](./03-browser-sso-saml-oidc.md)
- [07 — Key configurations](./07-key-configurations.md)
- [Glossary](./glossary.md)
