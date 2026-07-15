# Enterprise SSO decision reference

This reference helps enterprise architects and application developers map an SSO requirement to the right pattern, understand the protocols involved, and collect the key configurations needed to enable sign-in. It covers the **modern Entra ID** path for cloud and SaaS integrations and the **legacy ADFS / Active Directory** path for on-prem in-house apps—conceptual guidance and essential knobs only, not portal walkthroughs or SDK code.

## Got an SSO requirement? Start here

1. Skim the [decision table](./01-sso-landscape.md#decision-table) to match your requirement signal to a pattern.
2. Open the matching pattern doc linked from that row.
3. Use [components & topology](./02-components-and-topology.md) with stakeholders to align on identity planes and network boundaries.
4. Collect settings from [key configurations](./07-key-configurations.md) before implementation or vendor calls.

## Reading paths

- **Architect:** README → [01 — SSO landscape](./01-sso-landscape.md) → [02 — Components and topology](./02-components-and-topology.md) → [05 — Cross-federation](./05-cross-federation.md) → [06 — Legacy ADFS and AD](./06-legacy-adfs-ad.md) (if on-prem) → [07 — Key configurations](./07-key-configurations.md)
- **Developer:** README → [01 — SSO landscape](./01-sso-landscape.md) (skim) → [03 — Browser SSO (SAML / OIDC)](./03-browser-sso-saml-oidc.md) or [04 — API OAuth and OBO](./04-api-oauth-obo.md) (or [06 — Legacy ADFS and AD](./06-legacy-adfs-ad.md)) → [07 — Key configurations](./07-key-configurations.md) → [05 — Cross-federation](./05-cross-federation.md) if partner users

## Document index

- [01 — SSO landscape](./01-sso-landscape.md) — Protocol map, decision table, and pattern catalog for mapping requirements to SSO approaches.
- [02 — Components and topology](./02-components-and-topology.md) — Entra, ADFS/AD, and partner federation components with Mermaid topology diagrams.
- [03 — Browser SSO (SAML / OIDC)](./03-browser-sso-saml-oidc.md) — Entra enterprise-app sign-in for SaaS and websites via SAML 2.0 or OpenID Connect.
- [04 — API OAuth and OBO](./04-api-oauth-obo.md) — Delegated access, client credentials, and On-Behalf-Of chains for protected APIs.
- [05 — Cross-federation](./05-cross-federation.md) — B2B guests, Entra-to-Entra collaboration, and federated partner IdPs (Okta, Ping, ADFS).
- [06 — Legacy ADFS and AD](./06-legacy-adfs-ad.md) — On-prem ADFS relying-party SSO for in-house apps backed by Active Directory.
- [07 — Key configurations](./07-key-configurations.md) — Consolidated must-configure checklists (tenant URLs, certs, scopes, trust identifiers).
- [Glossary](./glossary.md) — Definitions for IdP, SP/RP, claims, B2B, OBO, and other terms used across the reference.

## Design spec

Source requirements and document structure: [Enterprise SSO Reference Documentation — Design Spec](./superpowers/specs/2026-07-15-enterprise-sso-reference-design.md).

## Glossary

Term definitions used throughout this reference: [glossary.md](./glossary.md).
