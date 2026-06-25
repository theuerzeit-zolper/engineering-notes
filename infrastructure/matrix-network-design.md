# Matrix Infrastruktur- und Netzwerkdesign

## Überblick

Diese Dokumentation beschreibt die tatsächliche Netzwerk- und Systemarchitektur der selbst gehosteten Matrix-Plattform.

Die Architektur basiert auf einer klaren Trennung von Netzwerksegmenten (WAN / LAN / DMZ), einem  Linux-Host in der DMZ sowie einer containerisierten Applikationsschicht.

---

## 1. Netzwerkarchitektur (Perimeter & Firewall)

### 1.1 WAN → Firewall → DMZ

Alle externen Benutzerzugriffe erfolgen ausschließlich über diesen Pfad:

Internet → Firewall → DMZ

In diesem Pfad wird klassisches NAT verwendet sowie ein restriktives Firewall-Regelwerk.

### Erlaubte Ports aus dem WAN:

- TCP 80 (HTTP, ausschließlich für Let’s Encrypt Challenge)
- TCP 443 (HTTPS, gesamte Applikation)

### Exponierte Dienste:

- dex.firma.de
- matrix.firma.de
- chat.firma.de

Alle externen Clients (Web, Mobile, Desktop) nutzen ausschließlich diesen Zugriffspfad.

---

### 1.2 LAN → Firewall → DMZ

Der Zugriff aus dem internen LAN in die DMZ ist erweitert, aber kontrolliert.

Es findet kein NAT statt.

### Erlaubte Verbindungen:

- TCP 443 (HTTPS)
- TCP 80 (HTTP)
- TCP 22 (SSH, administrativ)
- TCP 81 (Nginx Proxy Manager Admin Interface)
- TCP 9090 (Cockpit / System Administration)

Dieser Pfad wird ausschließlich für Administration und Betrieb genutzt.

---

### 1.3 DMZ → Firewall → LAN

Die Rückrichtung vom DMZ-Netz in das LAN ist stark eingeschränkt.

### Erlaubte Verbindungen:

- DNS
- ICMP (Ping)
- LDAPs (LDAPS)

Diese Verbindungen dienen ausschließlich:

- Benutzerverzeichnis (Active Directory)
- Namensauflösung
- Basisdiagnose

---

## 2. Systemebene (Linux Host in der DMZ)

In der DMZ befindet sich ein Linux-Host (VM).

### 2.1 Firewall auf Host-Ebene (firewalld)

Der Host nutzt firewalld mit getrennten Zonen:

#### public zone

- TCP 80
- TCP 443

#### internal zone

Zugriff ausschließlich aus definierten Admin-Netzen:

- TCP 22 (SSH)
- TCP 81 (Nginx Proxy Manager Admin)
- TCP 9090 (Cockpit)

#### docker zone

Standard Docker-Zone ohne zusätzliche externe Freigaben.

---

## 3. Applikationsebene (Docker)

Auf dem Linux Host läuft eine containerisierte Plattform.

Docker ist die interne Applikationsschicht und vollständig vom Netzwerk isoliert.

### 3.1 Docker Netzwerke

#### matrix-network (internal)

Dieses interne Docker-Netzwerk verbindet alle Applikationscontainer.

Eigenschaften:
- vollständige interne Kommunikation zwischen Containern
- nicht vom Host direkt erreichbar
- keine direkte Netzwerk-Routing-Verbindung nach außen

---

## 4. Reverse Proxy (Nginx Proxy Manager)

Der Nginx Proxy Manager (NPM) ist der einzige öffentlich exponierte Container der Plattform.

### 4.1 Netzwerkposition
- im Docker Host installiert
- im Docker Netzwerk verbunden
- gleichzeitig einziger Service mit externen Ports

### 4.2 Exponierte Ports

- TCP 80 (Let’s Encrypt ACME Challenge)
- TCP 443 (HTTPS Traffic)
- TCP 81 (Admin Interface, nur LAN erreichbar)

### 4.3 Funktion

NPM übernimmt:
- Reverse Proxy Routing
- TLS Termination
- Let’s Encrypt Zertifikatsmanagement
- Weiterleitung an interne Docker Services

---

## 5. Applikationsrouting

Alle externen Anfragen laufen über folgenden Pfad:

Internet → Firewall → DMZ Host → Nginx Proxy Manager → Docker Network (matrix-network) → Zielservice

### Beispiele:
- dex.firma.de → Dex Container
- matrix.firma.de → Synapse Container
- chat.firma.de → Element Web Container

---

## 6. Interne Kommunikation

Innerhalb des Docker Hosts erfolgt die Kommunikation ausschließlich über das interne Docker Netzwerk.

Beispiele:
- Synapse → PostgreSQL
- Synapse → Dex (OIDC Login Flow)
- Element Web → Synapse API

Diese Kommunikation ist nicht über die DMZ oder Firewall sichtbar.

---

## 7. DNS- und Namensauflösung

One Way / One Path: Alle Clients nutzen ausschließlich öffentliche DNS-Einträge.

Besonderheit:
Der Synapse Container erhält durch eine interne Host-Override-Konfiguration (`extra_hosts`) direkt die Private DMZ-IP des Hosts
für den Service dex.firma.de. So werden Routing-Schleifen (Hairpin/U-Turn) vermieden.

---

## 8. TLS / Zertifikate

Alle externen Dienste sind ausschließlich über HTTPS erreichbar.

### Let’s Encrypt
- Verwaltung über Nginx Proxy Manager
- HTTP Port 80 ausschließlich für ACME Challenge

### Interne Zertifikate
- LDAPs Zertifikat für Active Directory Kommunikation
- Bereitgestellt durch ADCS
- in Dex Container eingebunden

---

## 9. Sicherheitsmodell

Die Sicherheitsarchitektur basiert auf klaren Trennungen:

- DMZ enthält den Linux Host
- keine direkten Applikations-Ports außer Reverse Proxy
- Docker isoliert alle Services vollständig
- interne Services sind nicht direkt erreichbar
- kein Federation Traffic im Matrix System

---

## 10. Zusammenfassung

Die Plattform basiert auf einem einfachen, klar strukturierten Modell:

- eine DMZ als einzig exponierte Netzwerkzone
- ein Linux Host als zentrale Systeminstanz
- Docker als isolierte Applikationsschicht
- ein Reverse Proxy als einziger Einstiegspunkt

Alle Dienste sind ausschließlich über diesen kontrollierten Pfad erreichbar, während interne Kommunikation vollständig innerhalb des Docker Netzwerks stattfindet.
