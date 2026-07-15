# API access with Entra ID (OAuth 2.0 and OBO)

## Choose this when

- A **client application** (web app, SPA, daemon, or API) must call **protected APIs** using **OAuth 2.0 access tokens**, not browser federation artifacts alone
- You need to decide between **delegated user access**, **app-only (client credentials)**, or **On-Behalf-Of (OBO)** when a middle tier calls downstream APIs with the signed-in user's context
- The user may already have authenticated via browser SSO ([03 — Browser SSO](./03-browser-sso-saml-oidc.md)); this doc covers how **access tokens** authorize API calls after or without that session

## Prefer another pattern when

- **Browser sign-in only** (SAML/OIDC to establish a SaaS session, no first-party API calls) → [03 — Browser SSO](./03-browser-sso-saml-oidc.md)
- **Partner users authenticate at their home IdP** (B2B guest or inbound federation) → [05 — Cross-federation](./05-cross-federation.md)
- **On-prem AD-backed apps** using ADFS WS-Fed or SAML, not Entra OAuth → [06 — Legacy ADFS and AD](./06-legacy-adfs-ad.md)

## Actors

| Actor | Role |
|---|---|
| Client app | Requests tokens from Entra; presents access tokens to APIs (web/SPA, daemon, or middle tier) |
| Entra ID | Authorization server — authenticates users or clients; issues access (and optionally refresh) tokens |
| API (resource server) | Validates bearer tokens (`iss`, signature, `aud`, scopes/app roles, lifetime) before serving data |
| Middle-tier API (OBO only) | Receives the user's token (`aud` = middle tier), exchanges it at Entra for a downstream token, calls the downstream API |

## Pattern A — Delegated user access (web/SPA → API)

**When:** A signed-in user (or interactive login) should call an API **as themselves**. The client obtains an access token with **delegated permissions** (scopes) that represent what the user is allowed to do.

**Flow:** Authorization code with **PKCE** (public clients such as SPAs) or authorization code with client secret/certificate (confidential web apps). After login, the token endpoint returns an **access token** whose **`aud`** (audience) is the target API's Application ID URI or client ID—not the client app's ID. The client sends `Authorization: Bearer {access_token}` to the API.

**Scopes:** Request the narrowest delegated scopes the API exposes (e.g., `api://{api-app-id}/User.Read`). The API maps scopes (and optional claims) to authorization logic. Sign-in scopes (`openid`, `profile`, `email`) come from OIDC; **resource scopes** authorize API calls.

**Key configs (summary):** Application ID URI on the API registration; exposed scopes; **authorized client applications** (or admin consent) so the calling app may request those scopes; API-side **audience validation** must match the token's `aud`. See [07 — Key configurations](./07-key-configurations.md).

**Relationship to browser SSO:** [03](./03-browser-sso-saml-oidc.md) describes OIDC sign-in; the same app registration often requests both `openid` and API scopes in one authorize request, or a BFF exchanges the code server-side and calls APIs with the access token.

## Pattern B — App-only (daemon / service)

**When:** A **background job, daemon, or service** calls an API **without a user present**—scheduled sync, nightly batch, microservice-to-microservice where no human is signed in.

**Flow:** **Client credentials** grant. The confidential client authenticates to Entra with its client ID plus secret or certificate and requests an access token for the API. No user assertion is involved; authorization is based on **application permissions** (app roles) granted to the client, typically via `https://graph.microsoft.com/.default` or `api://{api-app-id}/.default` depending on the resource.

**Permissions:** Admin consent for **application permissions** (app roles defined on the API registration). The resulting token represents the **application**, not a user. Downstream APIs must enforce app-only authorization explicitly (role checks, separate endpoints, or deny user-context assumptions).

**Key configs (summary):** API exposes app roles; calling app has application permissions assigned; client uses certificate auth in production. See [07 — Key configurations](./07-key-configurations.md).

## Pattern C — On-Behalf-Of (API → API with user)

**When:** A **middle-tier API** receives a user's access token from a client, must call a **downstream API** while preserving **user context** (delegated permissions on the downstream resource), and should not expose downstream credentials or long-lived refresh tokens to the browser.

**Flow:** The client calls the middle tier with a bearer token whose `aud` is the **middle-tier API**. The middle tier uses Entra's **OBO** token exchange: it presents the user's access token (or assertion) plus its own client credential to Entra and receives a **new access token** whose `aud` is the **downstream API**. The middle tier calls the downstream API with that token.

```mermaid
sequenceDiagram
  actor User
  participant Client as Client app
  participant Mid as Middle-tier API
  participant Entra as Entra ID
  participant Down as Downstream API

  User->>Client: Authenticated session
  Client->>Mid: Call with user access token (aud=Mid)
  Mid->>Entra: OBO token request (user assertion + client credential)
  Entra->>Mid: Access token (aud=Downstream)
  Mid->>Down: Call with Bearer token
  Down->>Down: Validate token
  Down->>Mid: Response
```

**Requirements:** The middle-tier app registration must have permission to perform OBO to the downstream API (delegated permission on the downstream resource). The downstream API validates the new token's `aud`, scopes, and user identity claims as usual. OBO is an Entra / Microsoft identity platform pattern—not a generic OAuth grant; other IdPs may use different token-exchange mechanisms.

**Key configs (summary):** Middle-tier confidential registration; **OBO permission** to call downstream; downstream Application ID URI and scopes; authorized client apps as needed. See [07 — Key configurations](./07-key-configurations.md).

## Key configurations

Detailed checklists and Entra field names live in [07 — Key configurations](./07-key-configurations.md). For API access, confirm at minimum:

- **Application ID URI** — stable identifier for the API; appears in token `aud` and scope names
- **Scopes (delegated)** — exposed permissions for user-delegated access (Pattern A and OBO downstream)
- **App roles / application permissions** — for client credentials (Pattern B) and admin-consented app-only access
- **Authorized client applications** — which client apps may request tokens for this API
- **Audience validation** — API rejects tokens whose `aud` or `scp`/`roles` do not match its registration
- **OBO permission on middle tier** — middle-tier app granted delegated access to downstream API for OBO exchange (Pattern C)

## Common pitfalls

- **Wrong `aud`** — API validates a different Application ID URI than Entra puts in the token; every tier in OBO must accept the token issued for its own identifier
- **Using ID token as API bearer** — ID tokens are for the client (`aud` = client app); resource APIs require **access tokens** with API audience and scopes
- **Missing OBO permission** — middle tier receives `invalid_grant` or downstream calls fail because the middle-tier registration lacks permission to exchange for the downstream resource
- **Confusing app-only with delegated** — client credentials tokens carry no user `sub`; do not use Pattern B when audit or authorization requires the signed-in user (use Pattern A or C)
- **Scope vs role mismatch** — delegated flows use `scp`; app-only uses `roles`; APIs must check the claim type that matches the grant

## Related

- [03 — Browser SSO (SAML and OIDC)](./03-browser-sso-saml-oidc.md)
- [05 — Cross-federation](./05-cross-federation.md)
- [07 — Key configurations](./07-key-configurations.md)
- [Glossary](./glossary.md)
