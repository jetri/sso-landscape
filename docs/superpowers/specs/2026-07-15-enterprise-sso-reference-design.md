# Enterprise SSO Reference Documentation — Design Spec

**Date:** 2026-07-15  
**Status:** Approved for implementation planning  
**Workspace:** `/Users/j3/Documents/sso` (greenfield)

## Purpose

Create a **decision reference** (markdown) that architects and engineers use when they have SSO requirements: map the requirement to a pattern, understand the protocols involved, see Entra ID–centric examples, and know the **key configurations** needed to enable SSO.

This is not a portal tutorial or SDK cookbook. Tone is conceptual + diagrams + essential knobs only.

## Audience

| Audience | What they need from the reference |
|---|---|
| Enterprise architects / IAM engineers | Landscape, topology, federation options, trade-offs |
| Application developers | Which pattern/protocol to use for websites and APIs, key Entra settings |

Dual **reading paths** in the overview (architect vs developer).

## Goals

1. Document the main **ways** enterprises implement SSO and the **protocols** involved (SAML 2.0, OpenID Connect, OAuth 2.0 including OBO).
2. Use **Microsoft Entra ID** as the primary IdP example for integrating with external services/websites and APIs.
3. Explain where **cross-federation** fits: Entra B2B guests, federation to a partner IdP (Okta/Ping), and federation to **another organization’s Entra tenant**.
4. Include Mermaid: high-level **component** diagram, **network topology** diagram, and **sequence** diagram(s).
5. Provide **key configuration** checklists (not click-paths or application code).

## Non-goals

- Portal step-by-step walkthroughs or screenshots
- SDK / language-specific code samples
- Deep Conditional Access / MFA product guide (call out as a requirement flag only)
- Device SSO, Kerberos, WS-Fed except brief “legacy” mention if needed for protocol map completeness
- Deep CIAM / Entra External ID customer-facing identity (out of scope; B2B workforce federation is in scope)
- Implementing a runnable SSO demo application

## Approach

**Pattern-first cookbook with a shared protocol landscape** (chosen over protocol-first textbook or single-narrative + diagram appendix).

Organize by enterprise pattern; each pattern answers: **when to choose**, **protocol**, **actors**, **diagram pointer**, **key configs**, **common pitfalls**. Shared glossary and consolidated config checklist avoid duplication.

## Document structure

```
docs/
  README.md                      # Entry: “Got an SSO requirement? Start here”
  01-sso-landscape.md            # Ways to do SSO + protocol map + decision table
  02-components-and-topology.md  # Component + network topology Mermaid
  03-browser-sso-saml-oidc.md    # SaaS / websites; primary OIDC sequence (+ SAML note)
  04-api-oauth-obo.md            # APIs: delegated, app-only, OBO
  05-cross-federation.md         # B2B; Entra↔Entra; Entra↔third-party IdP
  06-key-configurations.md       # Consolidated must-configure checklists
  glossary.md                    # IdP, SP/RP, tenant, claims, issuer, etc.
```

### Reading paths

- **Architect:** README → 01 → 02 → 05 → 06  
- **Developer:** README → 01 (skim) → 03 or 04 → 06 → 05 if external/partner users  

## Decision framework

`README.md` and `01-sso-landscape.md` expose a decision table. Example rows (content to be written precisely during implementation):

| Requirement signal | Likely pattern | Primary protocol(s) | Jump to |
|---|---|---|---|
| Browser login to SaaS / partner site | Browser SSO | SAML 2.0 or OIDC | 03 |
| Modern web/SPA calling your APIs | OIDC + OAuth | Auth code (+ PKCE), tokens to API | 03, 04 |
| Service/daemon calling APIs (no user) | App-only | Client credentials | 04 |
| API needs user context via another API | Delegated chain | OAuth On-Behalf-Of (OBO) | 04 |
| Users from partner org (their IdP or Entra) | Cross-federation | B2B + SAML/OIDC federation | 05 |

Each pattern doc opens with **“Choose this when…”** / **“Prefer another pattern when…”**.

## In-scope patterns & Entra examples

1. **External SaaS / website** — Enterprise application in Entra; SAML or OIDC; redirect/ACS; claim mapping (UPN, email, groups/roles).
2. **First-party web + API** — App registration(s); exposed API scopes; delegated permissions; audience validation.
3. **Daemon / service** — App-only client credentials; app roles or `.default`.
4. **API → API with user** — OBO; middle-tier registration; downstream API permission.
5. **Cross-federation**  
   - (a) B2B guest users (partner Entra or other)  
   - (b) Inbound federated IdP (SAML/OIDC) for partner Okta/Ping **or** another org’s Entra tenant  

## Mermaid diagrams

| Diagram | Location | Content |
|---|---|---|
| High-level components | `02-components-and-topology.md` | Users → Entra (IdP) → apps/SaaS (SP/RP) → APIs; partner IdP / partner Entra via federation & B2B |
| Network topology | `02-components-and-topology.md` | Logical zones (corp/internet), browser, Entra endpoints, app hosting, API tier, partner IdP trust; TLS and redirect boundaries |
| Browser SSO sequence | `03-browser-sso-saml-oidc.md` | Primary: OIDC Authorization Code via Entra to external website, then token use to API; short callout or secondary sequence for SAML SP-initiated |
| Cross-federation sequence | `05-cross-federation.md` | Partner user → home IdP → assertion/token into your Entra → access to your app |

Diagrams must be valid Mermaid that renders in GitHub-flavored Markdown viewers.

## Key configurations (`06` + inline boxes)

Conceptual checklists only, covering:

- Tenant / issuer URL, application (client) ID, redirect URIs / ACS URL, logout URL  
- Protocol choice; signing certificate / client authentication method  
- Required claims; group → role strategy (high level)  
- API: Application ID URI, scopes, authorized client applications, audience  
- Federation: IdP metadata URL, domain federation settings, issuer URI, claim transformation expectations  
- Conditional Access / MFA as a **requirement flag** when SSO alone is insufficient  

No portal click-paths; no code.

## Quality bar

- Consistent terminology; SAML “SP” ↔ OIDC “RP” explained once (glossary + first use in landscape).  
- Every pattern section: when, protocol, actors, diagram pointer, key configs, common pitfalls (e.g. wrong audience, missing reply URL, guest vs member).  
- No placeholder (“TBD”) sections in shipped docs.  
- YAGNI: no extra patterns beyond the five listed above.

## Delivery / implementation notes

- All content lives under `docs/` as listed; this design file stays under `docs/superpowers/specs/`.  
- Implementation work is **authoring markdown + Mermaid**, not application code.  
- After this spec is user-approved, create an implementation plan via the writing-plans skill (task breakdown for writing each doc file, diagram content outline, glossary terms, cross-links).

## Success criteria

An architect or engineer with an SSO requirement can:

1. Find the matching pattern within a few minutes via the decision table.  
2. Explain which protocol(s) apply and why.  
3. Point to the relevant Mermaid diagram for stakeholders.  
4. List the key Entra / federation settings they must obtain or configure.  
5. Know when cross-federation (B2B or IdP federation, including Entra↔Entra) is required.
