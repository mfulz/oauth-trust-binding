# OAuth Trust Model Extension — Developer Guide
*(English / Deutsch)*

---

## 1. Introduction (English)
This guide provides a practical implementation blueprint for the **OAuth Trust Binding Extension**, which fixes the structural impersonation vulnerability in today’s OAuth flows by requiring **explicit user-side trust binding (“User-Bound Trust Keys”)**.

Developers integrating OAuth should use this document to:
- Implement trust-binding checks
- Safely migrate existing applications
- Maintain compatibility with existing OAuth provider implementations
- Avoid the implicit-trust vulnerability where a provider can impersonate any user who shares an email address

---

## 1. Einleitung (Deutsch)
Dieses Dokument beschreibt die praktische Entwickler-Implementierung der **OAuth Trust Binding Extension**, welche die strukturelle Impersonation-Schwachstelle aktueller OAuth-Flüsse behebt, indem ein **expliziter, nutzerseitiger Vertrauensanker (“User-Bound Trust Key”)** vorgeschrieben wird.

Entwickler sollen dieses Dokument nutzen, um:
- Trust-Bindings korrekt umzusetzen
- Bestehende Systeme sicher zu migrieren
- Kompatibilität zu OAuth-Anbietern zu gewährleisten
- Die implizite Impersonation-Lücke (“Email-basierte Gleichsetzung”) auszuschließen

---

# 2. Core Concept / Kernkonzept

## English
The vulnerability exists because apps implicitly assume:
> “If Google says the user’s email is alice@example.com, then this must be *the same Alice* who registered with this email.”

This is **false** unless the user has explicitly authorized this provider for this account.

The fix: **Every OAuth login must include a user-generated trust key** that the app stores server-side.

OAuth Provider → claims email `alice@example.com`  
App → checks: “Do I have a trust-binding for this provider + this user?”

If not → reject or request explicit user confirmation.

## Deutsch
Die Schwachstelle entsteht, weil Anwendungen implizit annehmen:
> „Wenn Google sagt, der Nutzer ist alice@example.com, dann ist das dieselbe Alice wie bei uns.“

Das ist **falsch**, solange der Nutzer diesen Provider nicht explizit freigegeben hat.

Die Lösung: **Jeder OAuth-Login muss an ein nutzerseitig erzeugtes Trust-Binding gekoppelt sein.**

OAuth-Provider → liefert Email `alice@example.com`  
App → prüft: „Habe ich ein Trust-Binding für diesen Provider + diesen Nutzer?“

Falls nein → Ablehnen oder explizite Freigabe durch den Nutzer verlangen.

---

# 3. Data Model / Datenmodell

## English
### Minimal required fields
```json
{
  "user_id": "local-internal-id",
  "email": "alice@example.com",
  "provider": "google",
  "provider_subject": "1234567890",
  "trust_key": "random-256bit-key",
  "created_at": "timestamp"
}
```

### Trust Key
- Random 256-bit string
- Generated upon first explicit binding
- Stored **only** by the application, never exposed to OAuth providers
- Used to verify that this provider is actually allowed to bind this user

## Deutsch
### Minimal erforderliche Felder
```json
{
  "user_id": "lokale-id",
  "email": "alice@example.com",
  "provider": "google",
  "provider_subject": "1234567890",
  "trust_key": "random-256bit-key",
  "created_at": "timestamp"
}
```

### Trust Key
- Zufälliger 256-Bit Wert
- Wird bei der ersten expliziten Freigabe erzeugt
- Verbleibt ausschließlich in der Anwendung
- Dient als eindeutiger Vertrauensanker zwischen Nutzer & Provider

---

# 4. Binding Flow / Bindungs-Ablauf

## 4.1 Initial Binding (English)
```
User → Login via provider
App: “No existing trust-binding found”
→ App asks user: “Do you want to enable Google Login for this account?”
User: Yes
→ App generates trust_key
→ App stores (provider, subject, email, trust_key)
→ Next login works automatically
```

## 4.1 Initiale Bindung (Deutsch)
```
Nutzer → Login über Provider
App: „Kein Trust-Binding gefunden“
→ App fragt: „Soll Google-Login für dieses Konto freigegeben werden?“
Nutzer: Ja
→ App generiert trust_key
→ App speichert (provider, subject, email, trust_key)
→ Zukünftige Logins funktionieren automatisch
```

---

# 5. Migration Strategy / Migrationsstrategie

## English
### Goal
Enable the new trust model without disrupting existing users.

### Recommended approach
1. Mark all existing OAuth links as **unverified**.
2. On next login via OAuth:
   - Show a one-time confirmation: “Enable this login provider for your account?”
3. Upon confirmation → set trust_key and mark verified.

Backward compatibility: **unchanged** authentication until user re-confirms.

## Deutsch
### Ziel
Das neue Trust-Modell einführen ohne bestehende Nutzer abzuschneiden.

### Empfohlene Vorgehensweise
1. Alle existierenden OAuth-Verknüpfungen als **unbestätigt** markieren.
2. Beim nächsten OAuth-Login:
   - Einmalige Bestätigung einfordern: „Diesen Login-Provider für Ihr Konto freigeben?“
3. Nach Zustimmung → trust_key setzen & als bestätigt markieren.

Rückwärtskompatibilität: **vollständig**, bis Nutzer bestätigt.

---

# 6. Security Considerations / Sicherheitshinweise

## English
- Trust keys must never be sent to OAuth providers.
- A compromised provider cannot impersonate a user unless a trust-binding exists.
- Removing a provider clears the trust-binding and invalidates future tokens.
- Attackers cannot force trust-binding creation because confirmation is interactive.

## Deutsch
- Trust-Keys dürfen niemals zum Provider gesendet werden.
- Ein kompromittierter Provider kann einen Nutzer nicht impersonieren ohne vorheriges Trust-Binding.
- Entfernen eines Providers löscht das Trust-Binding und macht zukünftige Tokens ungültig.
- Angreifer können keine Trust-Binding-Erstellung erzwingen, da diese interaktiv ist.

---

# 7. Example Code (English)
### Pseudocode for login handler
```python
def oauth_login(provider, id_token):
    email = id_token.email
    subject = id_token.sub

    binding = db.lookup(provider=provider, email=email)

    if not binding:
        return prompt_user_to_enable(provider, email)

    # Valid binding?
    if binding.provider_subject != subject:
        return error("Provider identity mismatch")

    return login_user(binding.user_id)
```

---

# 8. Example Code (Deutsch)
### Pseudocode für Login-Handler
```python
def oauth_login(provider, id_token):
    email = id_token.email
    subject = id_token.sub

    binding = db.lookup(provider=provider, email=email)

    if not binding:
        return frage_nutzer_nach_freigabe(provider, email)

    if binding.provider_subject != subject:
        return fehler("Provider-Identität stimmt nicht überein")

    return login_user(binding.user_id)
```

---

# 9. Conclusion / Fazit

The OAuth Trust Binding Extension provides a **minimal, backward-compatible, user-controlled** fix to the structural impersonation flaw inherent to email-based OAuth identity flows.

Developers implementing this model help ensure:
- Better user safety
- Explicit trust instead of implicit trust
- Compliance with future regulatory frameworks

Das Modell ist so leichtgewichtig, dass es sofort in vorhandene Software integriert werden kann — ohne Kompatibilitätsbrüche.
