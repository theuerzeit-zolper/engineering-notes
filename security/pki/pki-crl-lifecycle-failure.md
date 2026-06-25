# PKI CRL-Lifecycle-Problem in einer Zwei-Stufen AD CS Umgebung

## Kontext

Dieses Dokument beschreibt ein typisches Betriebs- und Fehlerverhalten in einer Zwei-Stufen PKI bestehend aus:

- Offline Root-CA
- Enterprise Sub-CA
- CRL-Verteilung über HTTP (IIS) und LDAP (Active Directory)

Die Root-CA arbeitet offline und veröffentlicht CRLs manuell.

Die Sub-CA ist für den Betrieb auf gültige CRLs angewiesen.

---

## Architekturübersicht

- Root-CA (Offline)
  - veröffentlicht CRLs manuell
  - Verteilung über Dateisystem, HTTP und LDAP

- Enterprise Sub-CA
  - validiert CRLs der Root-CA
  - stellt Zertifikate aus
  - abhängig von CDP-Erreichbarkeit

---

## Problemstellung

Wenn eine CRL abläuft oder nicht aktualisiert wird:

- CRL-Validierung schlägt fehl
- Sub-CA kann Zertifikate nicht mehr korrekt validieren
- AD CS Dienst startet nicht oder ist instabil
- Zertifikatsausstellung ist blockiert

Im schlimmsten Fall fällt die Sub-CA komplett aus.

---

## Ursache

- CRL-Veröffentlichung erfolgt manuell
- keine automatische Lifecycle-Steuerung
- CDP-Pfade sind kritisch für Validierung
- Sub-CA erzwingt CRL-Prüfung beim Start

Typische Fehler:

- abgelaufene CRL
- falsche oder nicht erreichbare CDP URLs
- verzögerte Veröffentlichung
- LDAP-Replikationsprobleme

---

## Fehlerbehebung

### 1. Root-CA CRL veröffentlichen

Auf der Root-CA:

- Zertifizierungsstelle öffnen
- „Gesperrte Zertifikate“
- „Veröffentlichen“

Pfad der CRL:

C:\Windows\System32\CertSrv\CertEnroll\

---

### 2. CDP aktualisieren

HTTP (IIS):

CRL-Datei auf dem Webserver ersetzen:

http://<ca-server>/CertEnroll/RootCAName.crl

LDAP:

certutil -dspublish -f "RootCAName.crl"

---

### 3. Sub-CA neu starten

net stop certsvc
net start certsvc

---

## Erkenntnisse

- PKI-Verfügbarkeit hängt von CRL-Validität ab
- CRL-Verteilung ist ein Single Point of Failure
- Offline Root-CA erhöht operative Abhängigkeit
- CDP ist kritisch für Systemfunktion

---

## Lessons Learned

- CRL expiry ist ein Fehlerzustand, kein Warnzustand
- manuelle CRL Pflege ist risikobehaftet
- Monitoring muss CRL-Ablauf explizit prüfen
- CDP Design muss redundant sein
- PKI ist kritische Infrastruktur

---

## Empfehlung

- CRL Lifecycle automatisieren
- Ablaufzeiten überwachen
- HTTP + LDAP CDP parallel nutzen
- PKI als kritisches System behandeln
