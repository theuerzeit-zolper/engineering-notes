# Design einer internen Kommunikationsplattform mit Matrix (Self-Hosted, DMZ, OIDC)

## Ausgangssituation

Die interne Kommunikation erfolgte über mehrere Kanäle mit unterschiedlichen Anforderungen und Einschränkungen.

Klassische Messenger-Lösungen boten zwar eine hohe Benutzerfreundlichkeit, waren jedoch hinsichtlich Datenschutz, Datenhoheit und Compliance nur eingeschränkt geeignet. Gleichzeitig erwiesen sich bestehende Unternehmenswerkzeuge für viele Anwendungsfälle als zu komplex oder wurden nicht flächendeckend genutzt.

Gesucht wurde eine Lösung, die sowohl auf mobilen Endgeräten als auch auf Desktop-Systemen einfach nutzbar ist und gleichzeitig die Anforderungen an Datenschutz, Betriebssicherheit und zentrale Benutzerverwaltung erfüllt.

---

## Anforderungen

Die neue Plattform sollte folgende Anforderungen erfüllen:

- Self-Hosted Betrieb mit vollständiger Datenhoheit
- Keine Abhängigkeit von US-Cloud-Diensten
- Zentrale Benutzerverwaltung über das bestehende Active Directory
- Unterstützung für Desktop-, Web- und Mobile-Clients
- Einfache Nutzung für technische und nicht-technische Anwender
- Wirtschaftlicher Betrieb für ca. 200 Benutzer
- Betrieb innerhalb bestehender Sicherheits- und Netzwerkzonen
- Zukunftsfähige Authentifizierungsarchitektur

---

## Warum Matrix?

Matrix wurde als technische Grundlage ausgewählt, da das Protokoll offen, etabliert und unabhängig von einzelnen Herstellern ist.

Neben der Möglichkeit des vollständigen Eigenbetriebs bietet Matrix eine große Auswahl an Clients sowie eine flexible Integration in bestehende Unternehmensumgebungen.

Besonders relevant war die Möglichkeit, Identitäts- und Authentifizierungsprozesse an bestehende Verzeichnisdienste anzubinden, ohne dabei von proprietären Cloud-Plattformen abhängig zu sein.

---

## Architekturüberblick

[Architekturdiagramm folgt]

Die Plattform wurde als eigenständige Kommunikationslösung innerhalb der DMZ betrieben.

Die einzelnen Komponenten laufen containerisiert auf einer dedizierten Rocky-Linux-Plattform und kommunizieren über ein internes Docker-Netzwerk. Externe Zugriffe erfolgen ausschließlich über einen Reverse Proxy.

---

## Zentrale Architekturentscheidungen

### Self-Hosted statt Cloud

[Inhalt folgt]

### Docker auf einer dedizierten Plattform

[Inhalt folgt]

### OIDC mit Dex statt LDAP-Plugin

[Inhalt folgt]

### Betrieb in einer DMZ

[Inhalt folgt]

---

## Technische Umsetzung

### Komponenten

- Rocky Linux
- Docker
- Matrix Synapse
- PostgreSQL
- Dex
- Nginx Proxy Manager
- Element Clients

### Datenflüsse

[Inhalt folgt]

### Netzwerkdesign

Die detaillierte Beschreibung der Netzwerk- und Sicherheitsarchitektur befindet sich unter:

→ infrastructure/001-matrix-network-design.md

### Identity & Access Management

Die detaillierte Beschreibung der Authentifizierungsarchitektur befindet sich unter:

→ identity/001-matrix-oidc-authentication.md

---

## Ergebnis

Die Plattform erfüllt die definierten Anforderungen hinsichtlich Datenhoheit, Benutzerfreundlichkeit und Integration in die bestehende Unternehmensumgebung.

Durch die Nutzung offener Standards und den vollständigen Eigenbetrieb konnte eine langfristig wartbare Kommunikationsplattform geschaffen werden.

---

## Rückblick

[Inhalt folgt]

---

## Fazit

[Inhalt folgt]
