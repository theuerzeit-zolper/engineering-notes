# OpenSSL for PKI Administrators

> Eine praxisorientierte Referenz für System Engineers und PKI-Administratoren zur Erstellung, Analyse und Konvertierung von X.509-Zertifikaten mit OpenSSL.

---

## Zielgruppe

Dieses Dokument richtet sich an System Engineers, Infrastruktur-Administratoren und PKI-Verantwortliche, die OpenSSL im täglichen Betrieb einsetzen.

Der Schwerpunkt liegt auf wiederkehrenden Aufgaben rund um TLS- und X.509-Zertifikate, beispielsweise:

- Erstellen von Certificate Signing Requests (CSR)
- Erzeugen und Verwalten privater Schlüssel
- Arbeiten mit Subject Alternative Names (SAN)
- Analysieren bestehender Zertifikate
- Konvertieren zwischen unterschiedlichen Zertifikatsformaten
- Validieren von Zertifikaten und Certificate Chains

Das Dokument versteht sich nicht als vollständige OpenSSL-Dokumentation, sondern als technische Referenz für typische Administrationsaufgaben.

---

# Inhaltsverzeichnis

1. Einleitung
2. Zertifikatsgrundlagen
3. Zertifikatsformate
4. Private Keys
5. Certificate Signing Requests (CSR)
6. Subject Alternative Names (SAN)
7. Zertifikate analysieren
8. Zertifikate konvertieren
9. Zertifikate validieren
10. Troubleshooting
11. Nützliche OpenSSL-Kommandos

---

# Einleitung

## Warum OpenSSL?

OpenSSL gehört zu den am weitesten verbreiteten Werkzeugen für die Arbeit mit X.509-Zertifikaten und TLS.

Während Windows-Produkte wie der Internet Information Services (IIS) oder Microsoft Active Directory Certificate Services (AD CS) viele Aufgaben grafisch unterstützen, ist OpenSSL insbesondere in folgenden Szenarien das Standardwerkzeug:

- Linux-Server
- Reverse Proxys (HAProxy, NGINX, Apache)
- Netzwerkkomponenten und Appliances
- Container-Umgebungen
- Zertifikatsmigrationen
- Zertifikatskonvertierungen

Auch in Windows-Umgebungen ist OpenSSL häufig unverzichtbar, beispielsweise um Zertifikate für unterschiedliche Dienste vorzubereiten oder zwischen verschiedenen Dateiformaten zu konvertieren.

## Typische Einsatzszenarien

OpenSSL wird in der Praxis unter anderem verwendet für:

- Erstellen von Certificate Signing Requests (CSR)
- Erzeugen privater Schlüssel
- Erstellen von SAN-Zertifikatsanfragen
- Anzeigen von Zertifikatsinformationen
- Prüfen von Certificate Chains
- Konvertieren zwischen PEM, DER, PKCS#7 und PKCS#12
- Export einzelner Zertifikatsbestandteile
- Validieren von Zertifikaten

Die folgenden Kapitel behandeln diese Aufgaben anhand praxisnaher Beispiele für Windows- und Linux-Umgebungen.

---

# Zertifikatsgrundlagen

## Public Key Infrastructure (PKI)

Eine Public Key Infrastructure (PKI) stellt die technischen und organisatorischen Komponenten bereit, die für die Ausstellung, Verwaltung und Überprüfung digitaler Zertifikate erforderlich sind.

Zu den zentralen Bestandteilen einer PKI gehören:

* Certificate Authorities (CA)
* Zertifikate
* Private und Public Keys
* Certificate Revocation Lists (CRL)
* Certificate Chains

OpenSSL arbeitet dabei nicht als Certificate Authority, sondern als Werkzeug zur Erstellung, Analyse und Verwaltung dieser Komponenten.

---

## Private Key

Der Private Key ist der wichtigste Bestandteil eines Zertifikats.

Er wird lokal auf dem Zielsystem erzeugt und **verlässt dieses idealerweise niemals**. Aus dem Private Key wird der zugehörige Public Key abgeleitet.

Der Private Key wird unter anderem verwendet für:

* Authentifizierung eines Servers
* Entschlüsselung eingehender TLS-Verbindungen
* Digitale Signaturen
* Erstellung eines Certificate Signing Requests (CSR)

> **Best Practice**
>
> Private Keys sollten ausschließlich auf dem Zielsystem erzeugt und sicher gespeichert werden. Eine Weitergabe des Private Keys an Dritte sollte grundsätzlich vermieden werden.

---

## Public Key

Der Public Key wird aus dem Private Key abgeleitet.

Im OpenSSL-Workflow wird der Public Key **nicht als separate Datei gespeichert**, sondern direkt in den Certificate Signing Request (CSR) eingebettet.

Der CSR enthält somit:
- den Public Key
- die Identitätsinformationen (Subject)
- und eine Signatur, die mit dem Private Key erzeugt wurde

Nach Ausstellung eines Zertifikats wird der Public Key Bestandteil des X.509-Zertifikats.

Während der Private Key geheim bleibt, kann der Public Key ohne Sicherheitsrisiko verteilt und veröffentlicht werden.

---

## Certificate Signing Request (CSR)

Ein Certificate Signing Request (CSR) enthält den Public Key sowie die Identitätsinformationen des Antragstellers.

Der CSR wird an eine Certificate Authority (CA) übermittelt, welche die enthaltenen Informationen überprüft und daraus ein signiertes Zertifikat erstellt.

Ein CSR enthält unter anderem:

* Common Name (CN)
* Subject Alternative Names (SAN)
* Organisationsinformationen
* Den Public Key
* Die Signatur des Antragstellers

Der Private Key ist **nicht** Bestandteil des CSR.

---

## Zertifikat

Nach erfolgreicher Prüfung erstellt die Certificate Authority ein X.509-Zertifikat.

Dieses enthält unter anderem:

* Subject
* Issuer
* Public Key
* Gültigkeitszeitraum
* Erweiterungen (Extensions)
* Digitale Signatur der ausstellenden CA

Das Zertifikat wird zusammen mit dem zuvor erzeugten Private Key auf dem Zielsystem verwendet.

---

## Certificate Chain

Ein Serverzertifikat wird in der Regel nicht direkt von einer Root CA ausgestellt.

Stattdessen besteht eine vollständige Vertrauenskette aus mehreren Zertifikaten:

```text
                Root CA
                    │
                    ▼
          Intermediate CA
                    │
                    ▼
          Server Certificate
```

Der Client überprüft diese Zertifikatskette bis zur vertrauenswürdigen Root CA.

Fehlende Intermediate-Zertifikate gehören zu den häufigsten Ursachen für TLS-Fehler.

---

## Zusammenspiel der Komponenten

Der typische Ablauf bei der Ausstellung eines Zertifikats sieht wie folgt aus:

```text
Private Key
     │
     ▼
Certificate Signing Request (CSR)
     │
     ▼
Certificate Authority (CA)
     │
     ▼
Server Certificate
     │
     ├───────────────┐
     ▼               ▼
Private Key      Certificate Chain
     │               │
     └──────┬────────┘
            ▼
      TLS Service
```

---

## Zertifikatsformate

Zertifikate und Schlüssel können in unterschiedlichen Formaten gespeichert werden. Diese unterscheiden sich hauptsächlich in der Darstellung (Text oder Binär), dem Inhalt (nur Zertifikat oder inkl. Private Key) und der Zielplattform.

Ein korrektes Verständnis der Formate ist entscheidend, da viele Importfehler in Windows-, Linux- oder Appliance-Umgebungen auf ein falsches Format zurückzuführen sind.

---

# Übersicht der wichtigsten Formate

| Format              | Typ           | Inhalt                   | Typische Nutzung              |
| ------------------- | ------------- | ------------------------ | ----------------------------- |
| PEM                 | Text (Base64) | Zertifikat, Key, Chain   | Linux, NGINX, Apache, HAProxy |
| DER                 | Binär         | Zertifikat               | Java, Appliances              |
| PKCS#7 (.p7b)       | Container     | Zertifikat + Chain       | Windows (ohne Private Key)    |
| PKCS#12 (.pfx/.p12) | Container     | Zertifikat + Private Key | Windows IIS, Exchange         |

---

# PEM (Privacy Enhanced Mail)

## Beschreibung

PEM ist das am weitesten verbreitete Zertifikatsformat in Linux- und OpenSSL-Umgebungen.

Es handelt sich um ein Base64-kodiertes Textformat, das typischerweise durch Header und Footer begrenzt ist:

```text id="pem_example"
-----BEGIN CERTIFICATE-----
...
-----END CERTIFICATE-----
```

oder:

```text id="key_example"
-----BEGIN PRIVATE KEY-----
...
-----END PRIVATE KEY-----
```

## Eigenschaften

* Textbasiert (lesbar)
* Kann mehrere Zertifikate enthalten (Chain)
* Kann Private Keys enthalten
* Standardformat für OpenSSL

## Typische Dateiendungen

* .pem
* .crt
* .cer
* .key

---

## PEM und Certificate Chains

Ein häufig übersehener Vorteil des PEM-Formats ist die Möglichkeit, mehrere Zertifikate in einer einzigen Datei zu speichern.

Damit kann eine komplette Certificate Chain abgebildet werden.

## Aufbau einer PEM Chain

Eine PEM-Datei kann mehrere Zertifikate hintereinander enthalten:

```text id="pem_chain"
-----BEGIN CERTIFICATE-----
(Server Certificate)
-----END CERTIFICATE-----

-----BEGIN CERTIFICATE-----
(Intermediate CA)
-----END CERTIFICATE-----

-----BEGIN CERTIFICATE-----
(Root CA - optional)
-----END CERTIFICATE-----
```

## Reihenfolge

Die Reihenfolge ist dabei entscheidend:

1. Server Certificate (Leaf)
2. Intermediate CA(s)
3. Root CA (optional)

---

## Unterschied zu PKCS#7

| PEM Chain                     | PKCS#7                    |
| ----------------------------- | ------------------------- |
| Textformat                    | Binärformat               |
| Kann Keys enthalten (separat) | Keine Keys                |
| Sehr flexibel                 | Nur Zertifikate           |
| Standard in Linux-Systemen    | Häufig in Windows-Imports |

---

## Typische Anwendung

PEM Chains werden häufig verwendet für:

* NGINX / Apache TLS-Konfigurationen
* HAProxy SSL Termination
* Container-Umgebungen
* Manuelle Zertifikatsdeployments
* OpenSSL Konvertierungen

---

## Beispiel aus der Praxis

Viele Certificate Authorities liefern Zertifikate getrennt aus:

* server.crt
* intermediate.crt

Diese werden dann zu einer PEM Chain zusammengefügt:

```bash id="pem_concat"
cat server.crt intermediate.crt > fullchain.pem
```

---

## Hinweis

PKCS#12-Dateien (.pfx/.p12) enthalten intern ebenfalls eine Zertifikatskette.

Beim Export nach PEM muss diese Kette jedoch häufig manuell korrekt zusammengesetzt werden.

---

# DER (Distinguished Encoding Rules)

## Beschreibung

DER ist die binäre Variante eines Zertifikats.

Im Gegensatz zu PEM ist DER nicht lesbar und wird häufig in Java- oder Hardware-Systemen verwendet.

## Eigenschaften

* Binärformat
* Enthält nur ein einzelnes Zertifikat
* Kein Private Key
* Nicht direkt lesbar

## Typische Dateiendungen

* .der
* .cer
* .crt (teilweise)

---

# PKCS#7

## Beschreibung

PKCS#7 ist ein Containerformat für Zertifikatsketten.

Es enthält typischerweise:

* Serverzertifikat
* Intermediate-Zertifikate

## Wichtige Einschränkung

👉 PKCS#7 enthält **keinen Private Key**

## Typische Dateiendungen

* .p7b
* .p7c

## Typische Verwendung

* Windows Certificate Import
* CA-Kettenübertragung

---

# PKCS#12

## Beschreibung

PKCS#12 ist ein verschlüsselter Container, der sowohl Zertifikate als auch den zugehörigen Private Key enthalten kann.

Dieses Format ist besonders wichtig für Windows-Umgebungen.

## Eigenschaften

* Enthält Zertifikat + Private Key
* Kann durch Passwort geschützt werden
* Plattformübergreifend nutzbar

## Typische Dateiendungen

* .pfx
* .p12

## Typische Verwendung

* IIS
* Microsoft Exchange
* Windows Certificate Store
* Migration zwischen Systemen

---

# Wichtige Unterschiede

## PEM vs DER

| PEM               | DER            |
| ----------------- | -------------- |
| Text              | Binär          |
| Lesbar            | Nicht lesbar   |
| Flexibel          | Strikt         |
| Standard in Linux | Häufig in Java |

---

## PKCS#7 vs PKCS#12

| PKCS#7               | PKCS#12                  |
| -------------------- | ------------------------ |
| Nur Zertifikate      | Zertifikat + Private Key |
| Keine Keys           | Enthält Private Key      |
| Windows Chain Import | IIS / Export / Migration |

---

# Typische Praxisprobleme

## Problem 1: Private Key fehlt

Ursache:

* PKCS#7 wurde verwendet

Lösung:

* PKCS#12 (.pfx) verwenden

---

## Problem 2: Zertifikat lässt sich nicht importieren

Ursache:

* falsches Format (DER vs PEM)

Lösung:

* Konvertierung mit OpenSSL prüfen

---

## Problem 3: IIS akzeptiert Datei nicht

Ursache:

* PEM statt PFX

Lösung:

* Zertifikat + Key in PFX bündeln

---


