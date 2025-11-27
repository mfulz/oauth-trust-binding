# RFC DRAFT — OAuth Trust Binding Extension (OTBE)
**Category:** Standards Track (Internet-Draft)  
**Status:** Draft — Work in Progress  
**Author:** Matthias Fulz  
**Date:** 2025-11-27

---

# Table of Contents
1. Introduction
2. Problem Statement
3. Terminology
4. Protocol Overview
5. Trust-Binding Registration
6. Authorization Flow Modifications
7. Security Considerations
8. Privacy Considerations
9. Backward Compatibility
10. Deployment Considerations
11. IANA Considerations
12. Appendix A — Examples
13. Appendix B — Threat Model

---

# 1. Introduction

OAuth 2.0 and OpenID Connect are widely deployed for delegated authorization and federated identity. A fundamental, often implicit assumption is that the Authorization Server (AS) and the Resource Owner (RO) share an established trust relationship before any authorization request is made.

In practice, the current ecosystem assigns implicit trust based solely on possession of an identifier, typically an email address. This creates critical vulnerabilities where:

- any Authorization Server hosting an identity namespace (e.g., *@gmail.com*) can impersonate users who never created an account with that AS;
- Relying Parties (RPs, i.e., clients) accept identities without any ability to verify whether the RO intentionally approved OAuth-based login for that AS;
- large Identity Providers (IdPs) can unilaterally gain access to RP accounts linked by email identifiers alone.

This document introduces the OAuth Trust Binding Extension (OTBE), a mechanism enabling explicit user-controlled trust binding between an identity namespace and an Authorization Server.

OTBE does not replace OAuth 2.0 or OpenID Connect. Instead, it adds a minimal, backward-compatible extension that allows RPs to verify that the RO has explicitly authorized a given AS to authenticate on their behalf.

---

# 2. Problem Statement

Current OAuth deployments allow the AS to assert identity ownership solely on the basis of email-domain control (or similar namespace ownership). This leads to several systemic risks.

## 2.1 Silent Impersonation Risk

A user who owns `user@example.com` may have no account at Google. However, if an RP accepts “Sign in with Google” and treats `user@example.com` from Google as equivalent to its local user `user@example.com`, Google can legitimately assert that identity even though:

- the user never created a Google account,
- the user never consented to Google acting as an IdP for that RP.

This creates:

- account takeover potential,
- unobservable impersonation,
- unilateral trust decisions by third parties.

## 2.2 No User-Driven Consent to Provider-Level Trust

There is currently no mechanism for a user to declare:

> “I trust AS X to authenticate me for RP Y (or globally).”

Instead, trust is assumed, not expressed. RPs often rely solely on a matching email address, combined with the reputation of a large IdP.

## 2.3 Centralization and Surveillance Risks

This implicit trust model enables concentrated identity control in dominant AS entities. Under legal compulsion (for example, national security letters or equivalent mechanisms), such providers could impersonate users across the entire ecosystem, logging into third-party RPs in a way that is:

- technically valid according to the protocol,
- opaque to the RP and the user,
- difficult or impossible to distinguish from a legitimate login.

## 2.4 Namespace Capture

Email domain providers become de facto identity authorities even when users have no intention of delegating authentication to them. This is a form of namespace capture: owning the domain implies being the global IdP for all accounts in that namespace, unless explicit counter-mechanisms are introduced.

OTBE resolves these issues by introducing explicit Trust Binding Records controlled by the Resource Owner.

---

# 3. Terminology

- **AS** — Authorization Server (or Identity Provider).
- **RO** — Resource Owner (end-user).
- **RP** — Relying Party (client) that consumes OAuth / OIDC assertions.
- **Trust Binding (TB)** — A record indicating that the RO has explicitly authorized a specific AS to authenticate their identifier towards an RP.
- **TB-Record** — The stored representation of a Trust Binding at an RP.
- **TB-Token** — A secret or cryptographic token associated with a Trust Binding.

---

# 4. Protocol Overview

OTBE extends OAuth deployments with the following principles:

1. Before an RP accepts a login from an AS for identifier `user@example.com`, it MUST verify that a Trust Binding exists between the RO and that AS (for this identifier).
2. Trust Bindings are simple, RO-controlled artifacts that bind:
   - a subject identifier (e.g. email),
   - an AS identity,
   - a TB-Token.
3. Trust Bindings are created explicitly by the RO, either:
   - manually (copy/paste or confirmation click), or
   - via a controlled enrollment flow.
4. Without a valid Trust Binding, authentication MUST fail with a dedicated error.

This design is agnostic to a specific OAuth/OIDC profile. It can be layered on top of:

- OAuth 2.0 with OIDC,
- OAuth 2.1 when available,
- proprietary extensions, as long as AS and RP can exchange TB-related data.

---

# 5. Trust-Binding Registration

## 5.1 Creating a Trust Binding

An RO creates or receives a TB-Token. Two primary models exist:

1. **RP-Generated TB-Token**  
   The RP generates a random secret and stores it in its database:
   ```text
   TB-Token = base64url(random(256 bits))
   ```
   The RO confirms that this TB-Token is associated with a given AS for a given identifier.

2. **AS-Generated TB-Token**  
   The AS issues a token bound to the RO and the AS identity, optionally signed.

The key properties:

- TB-Token MUST be unpredictable and unguessable.
- TB-Token MUST be bound to (identifier, AS) at the RP.

## 5.2 Binding the Token to the RP

A typical registration flow for RP-generated TB-Tokens:

1. RO authenticates with the RP using primary credentials (e.g., password, WebAuthn).
2. RO navigates to “Enable login with AS X”.
3. RP generates TB-Token and stores a TB-Record:
   ```json
   {
     "subject": "user@example.com",
     "as": "https://accounts.example-idp.com",
     "tb_token": "zY9sD2...",
     "created_at": "2025-11-27T21:00:00Z"
   }
   ```
4. RP associates this TB-Record with the RO’s internal user ID.

Alternative flows, where the AS provides an attested TB-Token, are possible but outside the minimal baseline required in this document.

---

# 6. Authorization Flow Modifications

During an OAuth/OIDC login, the following additional checks occur at the RP:

1. AS sends an authorization assertion (code + token, ID token, etc.) to the RP.
2. RP validates the assertion as usual (signatures, audience, nonce, etc.).
3. RP extracts:
   - the RO identifier (e.g. `sub`, `email`),
   - the AS identity (issuer).
4. RP looks up a TB-Record:
   ```text
   TB-Record = lookup(subject, as)
   ```
5. If **no TB-Record** exists, the RP:
   - MUST NOT auto-provision or auto-link this identity, and
   - SHOULD return a dedicated error, e.g. `trust_binding_missing`.
6. If a TB-Record exists, the RP may optionally:
   - verify a TB-Token claim in the assertion,
   - or rely solely on the existence of the TB-Record (baseline model).

This minimal model already forces explicit RO consent before any AS can be used for login at the RP.

---

# 7. Security Considerations

- OTBE prevents silent impersonation by AS, because AS login attempts are rejected unless a TB-Record exists.
- Compromising an AS is no longer sufficient to impersonate all RPs that match only on email; the attacker is limited to identities where a TB-Record has been explicitly created by the RO.
- Removing a provider at the RP (UI: “disconnect Google login”) clears the TB-Record and invalidates future tokens from that AS for that identifier.
- TB-Token guesses are prevented by using high-entropy random values.
- Attackers cannot force TB creation, since it is bound to interactive RO actions (account settings, explicit buttons, etc.).

---

# 8. Privacy Considerations

- OTBE reduces systemic identity leakage by ensuring that RPs do not silently accept unsolicited logins from arbitrary ASes.
- RPs obtain a more accurate model of user intent: which AS the user actually wants to use.
- TB-Tokens must be stored as secrets and handled in accordance with best practices (e.g., at-rest encryption, restricted access).

---

# 9. Backward Compatibility

- RPs may initially operate in a compatibility mode where legacy logins still function, while TB-Records are progressively collected.
- Once sufficient TB coverage is achieved, RPs can enable strict enforcement and reject logins from ASes without TB-Records.
- OTBE requires no changes to OAuth core flows.
- Existing OIDC libraries can be extended by post-processing ID tokens and applying TB checks.

---

# 10. Deployment Considerations

- OTBE can be rolled out incrementally per RP.
- Enterprises can define policy:
  - “Only allow AS from this list.”
  - “Require TB for all external IdPs.”
- IdP vendors may adopt optional TB-related claims (e.g., `trust_binding_token`), but OTBE does not depend on that for the baseline model.

---

# 11. IANA Considerations

This draft suggests, but does not strictly require, registration of:

- `trust_binding_token` — an OAuth / OIDC extension claim identifiying a TB-Token associated with an assertion.
- `trust_binding_missing` — an OAuth error code for missing TB-Records.

---

# 12. Appendix A — Example TB-Record

```json
{
  "subject": "user@example.com",
  "as": "https://accounts.example-idp.com",
  "tb_token": "zY9sD2fh8Kq1H...",
  "created_at": "2025-11-27T21:00:00Z",
  "user_id": "internal-uuid-1234"
}
```

---

# 13. Appendix B — Threat Model

## 13.1 Addressed Threats

- AS-driven impersonation of ROs at RPs that only match on identifiers (e.g. email).
- Government-compelled impersonation where AS is forced to log into RPs without user awareness.
- Namespace hijacking, where domain owners implicitly gain IdP power for all accounts.

## 13.2 Out of Scope

- Phishing of RO credentials at the AS.
- RP credential database compromise.
- Misconfiguration of RPs that ignore TB checks.

---

**End of Draft**
