# Identity & Access Management für Matrix (OIDC mit Dex und Active Directory)

## Ausgangssituation

Die Einführung einer zentralen Kommunikationsplattform auf Basis von Matrix erforderte eine saubere Integration in die bestehende Identity-Infrastruktur.

Im Unternehmen existiert bereits ein zentrales Active Directory als führendes Benutzerverzeichnis. Gleichzeitig sollte die neue Plattform keine zusätzlichen Benutzerkonten oder parallele Identity-Systeme einführen.

Ziel war es, eine Authentifizierungsarchitektur zu schaffen, die:

- bestehende Benutzer- und Gruppenstrukturen weiter nutzt
- keine Cloud-Identity voraussetzt
- moderne Authentifizierungsstandards unterstützt
- langfristig wartbar bleibt
- und sich sauber in Matrix Synapse integrieren lässt

Dabei zeigte sich früh, dass Identity in diesem Setup kein Nebenaspekt ist, sondern ein zentraler Architekturbaustein zwischen Anwendung, Infrastruktur und bestehender Unternehmens-IT.

---

## Problemstellung

Matrix selbst bringt nativ keine LDAP-Integration mit.

Historisch existierten verschiedene LDAP-Plugins für Synapse, diese sind jedoch inzwischen offiziell EOL (End of Life) und in modernen Synapse-Versionen teilweise nicht mehr kompatibel.

Parallel dazu entwickelt sich das Matrix-Ökosystem zunehmend in Richtung moderner Authentifizierungsarchitekturen. Der Fokus liegt klar auf dem Matrix Authentication Service (MAS), was die generelle Richtung hin zu standardisierten, OIDC-basierten Identity-Flows unterstreicht.

Damit wird deutlich: LDAP-basierte Integrationen sind langfristig keine stabile strategische Basis mehr für neue Implementierungen.

---

## Architekturentscheidung: Entkopplung der Identity-Schicht

Da Synapse kein LDAP spricht und Active Directory kein OIDC bereitstellt, wurde eine zusätzliche Identity-Schicht eingeführt.

Die resultierende Architektur trennt klar zwischen:

- Identity Source: Active Directory
- Identity Provider: Dex
- Service Layer: Matrix Synapse

Diese Trennung ermöglicht es, Active Directory als führende Benutzerquelle beizubehalten, während gleichzeitig moderne Authentifizierungsprotokolle verwendet werden.

Dex fungiert dabei als OIDC-Bridge zwischen klassischer Unternehmensverzeichnisstruktur und der Matrix-Plattform.

---

## Systemarchitektur

Die Komponenten sind vollständig containerisiert auf einer gemeinsamen Docker-Host-Plattform betrieben.

Dex läuft dabei ohne direkt exponierte Ports im internen Docker-Netzwerk. Der externe Zugriff erfolgt ausschließlich über einen Reverse Proxy (Nginx Proxy Manager), welcher TLS-Terminierung und Zertifikatsverwaltung übernimmt.

Externer Endpoint:
dex.firma.de → Reverse Proxy → Dex Container

---

## Authentifizierungsfluss

Der Login-Prozess basiert auf OIDC und folgt einem Redirect-basierten Flow.

Der lokale Matrix-Login wurde deaktiviert, sodass Benutzer ausschließlich über den zentralen Identity Provider authentifizieren.

Der Ablauf:

User
→ Element Client
→ Synapse
→ Redirect zu Dex (OIDC Login)
→ LDAP Bind gegen Active Directory
→ Token Issuance durch Dex
→ Callback zu Synapse
→ Session Establishment

Der Login ist für den Benutzer sichtbar als:

„Weiter mit Active Directory“

Dieser Button startet den vollständigen OIDC-Flow über Dex.

---

## Active Directory Integration

Dex ist direkt per LDAPs an das Active Directory angebunden.

Dabei wird kein einzelner Domain Controller verwendet, sondern die generische AD-Endpoint-Adresse:

- ad.firma.de

Die Kommunikation erfolgt ausschließlich über LDAPs mit Zertifikatsvalidierung (inkl. SAN-Zertifikat).

Für die Authentifizierung wird ein dedizierter Service Account verwendet, der bewusst stark eingeschränkt ist (keine interaktive Anmeldung, keine lokale oder RDP-Logins über GPO).

Zusätzlich wird der Login über eine dedizierte AD-Gruppe gesteuert. Nur Mitglieder dieser Gruppe sind berechtigt, sich an der Matrix-Plattform anzumelden.

---

## Identity Mapping

Eine der zentralen Herausforderungen bestand im Mapping zwischen Active Directory und Matrix User IDs.

Ziel war eine konsistente und reproduzierbare Identifier-Strategie.

Mehrere Ansätze wurden getestet:

- employeeID
- extensionAttribute10
- objectGUID

Diese Varianten erwiesen sich jedoch entweder als inkonsistent oder technisch problematisch im OIDC-Claim-Kontext (z. B. binäre Kodierung von objectGUID).

Die finale Lösung basiert auf dem E-Mail-Attribut:

localpart_template: "{{ user.email.split('@')[0] | lower }}"

Dieser Ansatz wurde gewählt, weil:

- E-Mail-Adressen im AD konsistent gepflegt sind
- der lokale Teil stabil und eindeutig ist
- keine binären oder komplexen Typkonvertierungen nötig sind
- eine klare und nachvollziehbare Mapping-Regel entsteht
- die MatrixID trotzdem gut menschenlesbar bleibt.

Damit wird die Matrix User ID deterministisch aus dem AD abgeleitet.

---

## Claim Handling und technische Erkenntnisse

Die Herausforderungen beim Claim- und Attribut-Mapping stammen aus Dex und der OIDC-Claim-Verarbeitung, nicht aus LDAP selbst.

Insbesondere objectGUID führte zu Problemen, da es binär kodiert übertragen wird und in Synapse nicht sinnvoll verarbeitet werden kann.

Die Konfiguration der Claims war einer der zeitintensiveren Teile der Implementierung und erforderte mehrere Iterationen zwischen Dex und Synapse.

Am Ende wurde bewusst eine minimale Claim-Strategie gewählt:

- Username (abgeleitet aus Mail)
- Display Name
- E-Mail Adresse
- Gruppenmitgliedschaft (für Login-Rechte)

Mehr Komplexität wurde bewusst vermieden, um Stabilität und Debuggability zu erhöhen.

---

## Security- und Zugriffskonzept

Der Zugriff auf die Plattform ist strikt an eine Active-Directory-Gruppe gekoppelt.

- Kein Gruppenmitglied → kein Login
- Gruppenmitglied → Zugriff über OIDC Flow

Die Autorisierung erfolgt damit bereits auf Identity Provider Ebene und nicht erst innerhalb von Synapse.

Der eingesetzte Service Account für LDAPs ist zusätzlich durch GPOs abgesichert und für interaktive Nutzung vollständig gesperrt.

---

## Architekturwirkung

Durch die Einführung von Dex als Identity Layer ergibt sich eine klare Entkopplung zwischen:

- Benutzerverzeichnis (AD)
- Authentifizierungslogik (Dex)
- Anwendungsebene (Synapse)

Auswirkungen:

- Synapse bleibt unabhängig von AD-spezifischen Implementierungen
- Identity-Logik kann später ersetzt oder erweitert werden (z. B. MAS)
- Änderungen im AD wirken sich nicht direkt auf die Anwendung aus
- Authentifizierungsprobleme lassen sich isoliert debuggen

---

## Zukunftsperspektive

Der Fokus der Matrix-Entwicklung in Richtung MAS (Matrix Authentication Service) und OIDC zeigt, dass diese Architekturentscheidung langfristig in die richtige Richtung geht.

Dex fungiert als stabile Zwischen- und Integrationsschicht, ohne zukünftige Migrationen zu blockieren.

Aktuell besteht kein funktionaler oder operativer Zwang zur Migration, da die Lösung stabil und wartbar ist.

---

## Rückblick

Die größte Herausforderung lag nicht in der OIDC-Grundintegration, sondern im Mapping zwischen Active Directory und Matrix.

Insbesondere die Auswahl geeigneter Attribute und deren Verhalten im Claim-Kontext führte zu mehreren Iterationen.

Die Entscheidung für ein einfaches string-basiertes Mapping über die E-Mail-Adresse hat sich als stabilste Lösung erwiesen.

Das Risiko dass Änderungen an der E-Mail-Adresse (z.B. Heirat) neue Matrix-Benutzer erstellt wurde dokumentiert und in kauf genommen.

---

## Fazit

Die Einführung einer OIDC-basierten Identity-Schicht über Dex ermöglicht eine saubere Integration von Matrix in eine bestehende Active-Directory-Umgebung.

Entscheidend war nicht die Nutzung möglichst vieler Identity-Attribute, sondern ein stabiles, nachvollziehbares und wartbares Mapping-Modell.

Damit entsteht eine Identity-Architektur, die bewusst zwischen klassischer Unternehmens-IT und moderner Infrastruktur vermittelt, ohne eine der beiden Seiten unnötig zu verkomplizieren.
