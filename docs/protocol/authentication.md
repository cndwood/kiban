# Authentifizierungsvertrag

[Dokumentationsindex](../INDEX.md) · [Architekturvertrag](../architecture/overview.md)

POST /api/v1/guest-sessions erstellt serverseitig PrincipalId und Session. display_name ist UI-Text und nicht unique. Native Response enthält session_token genau zur sicheren lokalen Speicherung, Browser Response setzt ausschließlich Cookie und liefert kein lesbares Tokenfeld. Response enthält principal_id und expires_at. Ein Netzwerkfehler bei Guest-Erzeugung kann eine verwaiste Session erzeugen; sie belegt keinen Sitz und läuft ab.

Cookie ist HttpOnly, Secure, SameSite=Lax, Path=/; Host-only ohne breite Domain. Ein CSRF-Token wird getrennt an den berechtigten Browser ausgegeben und für zustandsändernde REST-Requests im Header verlangt. Origin bei Browser-HTTP und WS gegen exakte öffentliche Origin prüfen, nicht gegen Suffix. Native Clients ohne Origin dürfen nur mit Bearer oder WS-Frame-Token authentifizieren; fehlender Origin allein ist keine Berechtigung.

WS authenticate verwendet protocol_version1, type authenticate und payload mit token nativ beziehungsweise leer im Cookie-Modus. Token darf nicht in URL, Log, Analytics oder Fehlermeldung auftauchen. Revocation via POST /api/v1/session/revoke sperrt Tokenhash und benachrichtigt zugehörige WS-Tasks. Absolute Ablaufzeit30Tage; Prüfung beim Upgrade und vor jeder Mutation. Bereits laufende Partie kann nach Ablauf nur nach erlaubter Reauth desselben Principals fortgesetzt werden; ein neuer Gast ist anderer Principal.

Tests: manipuliertes Token401; gültiges Token fremden Sitzes forbidden; abgelaufenes Token401; Logout macht bestehende WS-Mutation unmöglich; Cookie ohne CSRF scheitert; böse Origin scheitert auch mit Cookie. Tokens werden gehasht persistiert und mit konstantzeitgeeigneter Vergleichs-/Lookupstrategie behandelt. Reconnect ist kein Grund, serverseitig Clientprincipal_id zu vertrauen.

Ausnahme für idempotentes Logout: session/revoke erkennt einen bekannten bereits widerrufenen Tokenhash und liefert erneut204 ohne irgendein weiteres Recht zu vergeben. Ein unbekanntes Credential bleibt401. Alle anderen Endpunkte lehnen widerrufene Sessions strikt ab.
