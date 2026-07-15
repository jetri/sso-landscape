# Enterprise SSO Reference Documentation — Design Spec

**Date:** 2026-07-15  
**Status:** Approved for implementation planning (revised: legacy ADFS/AD)  
**Workspace:** `/Users/j3/Documents/sso` (greenfield)

## Purpose

Create a **decision reference** (markdown) that architects and engineers use when they have SSO requirements: map the requirement to a pattern, understand the protocols involved, see Entra ID–centric examples (plus a **legacy ADFS / Active Directory** path for in-house apps), and know the **key configurations** needed to enable SSO.

This is not a portal tutorial or SDK cookbook. Tone is conceptual + diagrams + essential knobs only.

## Audience

| Audience | What they need from the reference |
|---|---|
| Enterprise architects / IAM engineers | Landscape, topology, federation options, trade-offs (including when to retain vs migrate from ADFS) |
| Application developers | Which pattern/protocol to use for websites and APIs, key IdP settings |

Dual **reading paths** in the overview (architect vs developer).

## Goals

1. Document the main **ways** enterprises implement SSO and the **protocols** involved (SAML 2.0, WS-Federation, OpenID Connect, OAuth 2.0 including OBO), including a **legacy ADFS** path for on-prem / in-house apps backed by Active Directory.
2. Use **Microsoft Entra ID** as the primary *modern* IdP example for integrating with external services/websites and APIs.
3. Explain where **cross-federation** fits: Entra B2B guests, federation to a partner IdP (Okta/Ping), and federation to **another organization’s Entra tenant**.
4. Include Mermaid: high-level **component** diagram, **network topology** diagram, and **sequence** diagram(s).
5. Provide **key configuration** checklists (not click-paths or application code).

## Non-goals

- Portal / AD FS Management Console step-by-step walkthroughs or screenshots
- SDK / language-specific code samples
- Deep Conditional Access / MFA product guide (call out as a requirement flag only)
- Deep Kerberos / Windows Integrated Authentication internals (mention only as how users authenticate *to* ADFS when on corporate network)
- Full ADFS → Entra migration runbook (decision guidance and “prefer Entra when…” only)
- Deep CIAM / Entra External ID customer-facing identity (out of scope; B2B workforce federation is in scope)
- Implementing a runnable SSO demo application

## Approach

**Pattern-first cookbook with a shared protocol landscape** (chosen over protocol-first textbook or single-narrative + diagram appendix).

Organize by enterprise pattern; each pattern answers: **when to choose**, **protocol**, **actors**, **diagram pointer**, **key configs**, **common pitfalls**. Shared glossary and consolidated config checklist avoid duplication.

Legacy ADFS is a **first-class pattern** in the decision table, clearly labeled legacy, so readers with in-house AD-integrated apps can choose knowingly—not a footnote.

## Document structure

```
docs/
  README.md                      # Entry: “Got an SSO requirement? Start here”
  01-sso-landscape.md            # Ways to do SSO + protocol map + decision table
  02-components-and-topology.md  # Component + network topology Mermaid (incl. ADFS/AD)
  03-browser-sso-saml-oidc.md    # SaaS / websites via Entra; primary OIDC sequence (+ SAML note)
  04-api-oauth-obo.md            # APIs: delegated, app-only, OBO
  05-cross-federation.md         # B2B; Entra↔Entra; Entra↔third-party IdP
  06-legacy-adfs-ad.md           # ADFS SSO for in-house apps + Active Directory
  07-key-configurations.md       # Consolidated must-configure checklists
  glossary.md                    # IdP, SP/RP, ADFS, RP trust, tenant, claims, etc.
```

### Reading paths

- **Architect:** README → 01 → 02 → 05 → 06 (if on-prem/legacy) → 07  
- **Developer:** README → 01 (skim) → 03 or 04 (or 06 for ADFS apps) → 07 → 05 if external/partner users  

## Decision framework

`README.md` and `01-sso-landscape.md` expose a decision table. Example rows (content to be written precisely during implementation):

| Requirement signal | Likely pattern | Primary protocol(s) | Jump to |
|---|---|---|---|
| Browser login to SaaS / partner site | Browser SSO (Entra) | SAML 2.0 or OIDC | 03 |
| Modern web/SPA calling your APIs | OIDC + OAuth | Auth code (+ PKCE), tokens to API | 03, 04 |
| Service/daemon calling APIs (no user) | App-only | Client credentials | 04 |
| API needs user context via another API | Delegated chain | OAuth On-Behalf-Of (OBO) | 04 |
| Users from partner org (their IdP or Entra) | Cross-federation | B2B + SAML/OIDC federation | 05 |
| In-house app SSO against on-prem AD (legacy) | ADFS + Active Directory | WS-Fed and/or SAML 2.0 | 06 |

Each pattern doc opens with **“Choose this when…”** / **“Prefer another pattern when…”**.  
For ADFS: prefer Entra (03/04) for new cloud/SaaS work; use/retain 06 when the app and identity plane are still ADFS/AD-centric.

## In-scope patterns & examples

1. **External SaaS / website (Entra)** — Enterprise application in Entra; SAML or OIDC; redirect/ACS; claim mapping (UPN, email, groups/roles).
2. **First-party web + API (Entra)** — App registration(s); exposed API scopes; delegated permissions; audience validation.
3. **Daemon / service (Entra)** — App-only client credentials; app roles or `.default`.
4. **API → API with user (Entra)** — OBO; middle-tier registration; downstream API permission.
5. **Cross-federation (Entra)**  
   - (a) B2B guest users (partner Entra or other)  
   - (b) Inbound federated IdP (SAML/OIDC) for partner Okta/Ping **or** another org’s Entra tenant  
6. **Legacy ADFS + Active Directory (in-house)** — AD as identity store; ADFS as STS/IdP; relying party trusts for internal web apps; WS-Federation and/or SAML; optional note that ADFS can also federate *outbound* to Entra/SaaS (bridge), without making hybrid migration the focus.

## Mermaid diagrams

| Diagram | Location | Content |
|---|---|---|
| High-level components | `02-components-and-topology.md` | Users → Entra (IdP) → apps/SaaS (SP/RP) → APIs; partner IdP / partner Entra via federation & B2B; **parallel legacy path: Users → AD → ADFS → in-house apps** |
| Network topology | `02-components-and-topology.md` | Logical zones (corp/internet), browser, Entra endpoints, app hosting, API tier, partner IdP trust; **corp zone: domain controllers, ADFS (often internal + optional WAP/DMZ)**; TLS and redirect boundaries |
| Browser SSO sequence | `03-browser-sso-saml-oidc.md` | Primary: OIDC Authorization Code via Entra to external website, then token use to API; short callout or secondary sequence for SAML SP-initiated |
| Cross-federation sequence | `05-cross-federation.md` | Partner user → home IdP → assertion/token into your Entra → access to your app |
| Legacy ADFS sequence | `06-legacy-adfs-ad.md` | User → in-house app → ADFS (WS-Fed or SAML) → authenticate to AD (e.g. WIA or forms) → security token → app |

Diagrams must be valid Mermaid that renders in GitHub-flavored Markdown viewers.

## Key configurations (`07` + inline boxes)

Conceptual checklists only, covering:

**Entra / modern**

- Tenant / issuer URL, application (client) ID, redirect URIs / ACS URL, logout URL  
- Protocol choice; signing certificate / client authentication method  
- Required claims; group → role strategy (high level)  
- API: Application ID URI, scopes, authorized client applications, audience  
- Federation: IdP metadata URL, domain federation settings, issuer URI, claim transformation expectations  
- Conditional Access / MFA as a **requirement flag** when SSO alone is insufficient  

**Legacy ADFS / AD**

- ADFS farm / service URL; federation metadata URL  
- Relying party trust: identifier (realm), WS-Fed / SAML endpoints, issuance transform rules (claims)  
- Token-signing certificate trust on the app  
- Auth method to ADFS (WIA vs forms) as an environment assumption  
- Optional: Web Application Proxy (WAP) when publishing externally  

No portal/console click-paths; no code.

## Quality bar

- Consistent terminology; SAML “SP” ↔ OIDC “RP” ↔ WS-Fed “relying party” explained (glossary + first use in landscape).  
- Every pattern section: when, protocol, actors, diagram pointer, key configs, common pitfalls (e.g. wrong audience, missing reply URL, guest vs member; for ADFS: RP identifier mismatch, cert rollover, internal-only endpoints).  
- No placeholder (“TBD”) sections in shipped docs.  
- YAGNI: no extra patterns beyond the **six** listed above; ADFS doc stays decision-oriented, not a migration project plan.

## Delivery / implementation notes

- All content lives under `docs/` as listed; this design file stays under `docs/superpowers/specs/`.  
- Implementation work is **authoring markdown + Mermaid**, not application code.  
- After this spec is user-approved, create an implementation plan via the writing-plans skill (task breakdown for writing each doc file, diagram content outline, glossary terms, cross-links).

## Success criteria

An architect or engineer with an SSO requirement can:

1. Find the matching pattern within a few minutes via the decision table (including **legacy ADFS** for in-house AD apps).  
2. Explain which protocol(s) apply and why.  
3. Point to the relevant Mermaid diagram for stakeholders.  
4. List the key Entra **or** ADFS/AD settings they must obtain or configure.  
5. Know when cross-federation (B2B or IdP federation, including Entra↔Entra) is required.  
6. Know when to **use/retain ADFS** vs prefer modern Entra patterns for new work.
