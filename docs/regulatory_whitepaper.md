# OAuth Trust Binding Extension (OTBE) — Regulatory & Privacy Whitepaper

## 1. Executive Summary (English)

This paper describes a structural issue in today’s OAuth/OpenID Connect ecosystem and a minimal fix:

- **Issue:** Authorization Servers (AS) can impersonate users at Relying Parties (RP) solely by asserting control over an identifier (e.g. email domain), even if the user never created an account at that AS.
- **Impact:** This enables silent cross-service impersonation and centralization of identity power in large providers, with direct relevance to data protection, surveillance, and user autonomy.
- **Solution:** Introduce explicit, user-controlled **Trust Binding Records**. An RP only accepts login from an AS for a given subject if such a Trust Binding exists.

OTBE is fully backward-compatible and can be introduced incrementally by service providers without breaking existing authentication flows.

---

## 2. Kurzfassung (Deutsch)

Dieses Papier beschreibt ein strukturelles Problem im aktuellen OAuth/OpenID-Connect-Ökosystem und eine minimale Lösung:

- **Problem:** Authorization Server (AS) können Nutzer bei Relying Parties (RP) impersonieren, allein auf Basis der Kontrolle über einen Identifikator (z. B. Email-Domain). Dies ist auch dann möglich, wenn der Nutzer bei diesem AS nie ein Konto angelegt hat.
- **Auswirkung:** Es entsteht eine unbemerkbare, quer über Dienste hinweg wirksame Impersonation-Möglichkeit sowie eine starke Zentralisierung von Identitätsmacht bei großen Anbietern. Das ist unmittelbar relevant für Datenschutz, Überwachung und Nutzerautonomie.
- **Lösung:** Einführung expliziter, vom Nutzer kontrollierter **Trust-Binding-Records**. Eine RP akzeptiert einen Login eines AS für eine bestimmte Identität nur, wenn ein entsprechendes Trust Binding existiert.

OTBE ist vollständig rückwärtskompatibel und kann von Diensteanbietern schrittweise eingeführt werden.

---

## 3. Legal & Regulatory Angle (English)

### 3.1 Data Protection Principles

Under frameworks such as the GDPR, core principles include:

- **Purpose limitation**
- **Data minimization**
- **Integrity and confidentiality**
- **Fairness and transparency**

Implicit AS impersonation violates the spirit of these principles because:

- RPs cannot reliably determine whether a login was actually authorized by the user.
- Users are unable to control which AS may act as their identity authority.
- Large AS entities can, under legal or policy pressure, perform logins into third-party RPs without user consent.

OTBE restores a clear, auditable signal of user consent at the RP level.

### 3.2 Risk for Critical Sectors

In future scenarios where:

- public services,
- e-government portals,
- banking,
- healthcare

rely on centralized OAuth/OIDC providers, implicit trust based solely on email identifiers becomes a serious systemic risk:

- A single AS compromise, or compelled access, potentially cascades into wide-scale impersonation.
- Citizens may be impersonated without any protocol violation — the current standards permit it.

OTBE provides a simple, protocol-level safeguard to prevent such scenarios.

---

## 4. Rechtliche Perspektive (Deutsch)

### 4.1 Datenschutzrechtliche Bewertung

Die aktuelle Praxis, Identitäten ausschließlich über Übereinstimmung von Email-Adressen und Reputation des Identity Providers zu verknüpfen, führt zu:

- fehlender Transparenz gegenüber Betroffenen,
- potenziell unzulässiger Datenverarbeitung (Logins ohne tatsächliche Einwilligung),
- einer strukturellen Machtverschiebung hin zu wenigen zentralen Anbietern.

Mit OTBE wird eine explizite, dokumentierte Einwilligung auf Ebene der Relying Party eingeführt: Erst wenn der Nutzer ein Trust Binding eingerichtet hat, darf ein externer AS für diese Identität genutzt werden.

### 4.2 Aufsichtliche Relevanz

Für Aufsichtsbehörden (z. B. in der EU):

- OTBE bietet einen konkreten technischen Anknüpfungspunkt, um “Privacy by Design” und “Security by Design” bei Identity-Lösungen einzufordern.
- Anbieter, die großflächig OAuth/OIDC einsetzen, können mit OTBE eine klare Trennung zwischen:
  - legitimen, vom Nutzer gewünschten Single-Sign-On  
  - und strukturell angelegter, stiller Impersonation  
  herstellen.

---

## 5. Recommendation to Regulators (English)

Regulators may consider:

1. Recognizing implicit AS impersonation as a **structural risk** in federated identity systems.
2. Recommending or requiring mechanisms equivalent to OTBE for:
   - critical infrastructures,
   - public digital services,
   - large-scale consumer platforms.
3. Encouraging RPs to document and expose to users:
   - which AS they accept,
   - whether Trust Bindings are enforced.

OTBE is intentionally minimal to ease adoption: it does not ban OAuth or existing IdPs, but adds a missing piece of user agency.

---

## 6. Empfehlung an Anbieter (Deutsch)

Diensteanbieter, die OAuth/OpenID Connect nutzen, sollten:

1. Kurzfristig:
   - Trust-Binding-Mechanismus im eigenen System planen,
   - UI-Flows für “Externe Login-Provider verbinden/trennen” anbieten.
2. Mittelfristig:
   - bestehende OAuth-Verknüpfungen als “unbestätigt” markieren,
   - bei nächsten Logins eine einmalige Bestätigung einholen.
3. Langfristig:
   - strikte Durchsetzung von Trust Bindings aktivieren,
   - in Policies und AGB transparent machen, welche AS akzeptiert werden.

Damit kann das aktuell stillschweigende, implizite Vertrauen in global agierende Identity Provider durch explizite, nachvollziehbare Nutzerentscheidungen ersetzt werden.
