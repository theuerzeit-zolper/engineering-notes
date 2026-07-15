# Warum Docker?

**Architekturentscheidung**

## Hintergrund

Das Homelab orientiert sich bewusst an Arbeitsweisen aus dem professionellen Infrastrukturumfeld. Ziel ist nicht, Anwendungen möglichst schnell bereitzustellen, sondern eine Infrastruktur aufzubauen, die auch nach Jahren noch nachvollziehbar, dokumentiert und reproduzierbar betrieben werden kann.

Moderne Anwendungen bestehen häufig aus mehreren Komponenten wie Webservern, Laufzeitumgebungen oder Datenbanken. Werden diese direkt auf dem Betriebssystem installiert, entstehen Abhängigkeiten zwischen Anwendungen und Hostsystem, die Wartung und Updates erschweren können.

Für jede Anwendung eine eigene virtuelle Maschine bereitzustellen, würde diese Abhängigkeiten zwar reduzieren, gleichzeitig jedoch den Ressourcenverbrauch und den administrativen Aufwand deutlich erhöhen.

Docker bietet die Möglichkeit, Anwendungen einschließlich ihrer Abhängigkeiten in Containern auszuführen. Dadurch bleibt das Betriebssystem auf seine Rolle als Plattform für den Containerbetrieb beschränkt, während Anwendungen unabhängig voneinander betrieben werden können.

## Anforderungen

Die gewählte Plattform sollte folgende Anforderungen erfüllen:

- Standardisierte Bereitstellung von Anwendungen
- Klare Trennung zwischen Betriebssystem und Anwendungen
- Reproduzierbare Deployments
- Einfache Wartung
- Fehlerisolierung
- Unabhängige Updates einzelner Dienste
- Einfache Backup- und Restore-Prozesse
- Geringer Ressourcenverbrauch gegenüber mehreren virtuellen Maschinen

## Betrachtete Alternativen

### Klassische Installation auf dem Betriebssystem

Die direkte Installation von Anwendungen auf dem Betriebssystem ist einfach umzusetzen und benötigt keine zusätzliche Laufzeitumgebung.

Mit zunehmender Anzahl installierter Anwendungen entstehen jedoch Abhängigkeiten zwischen Diensten und Hostsystem. Updates, Konfigurationsänderungen oder Paketabhängigkeiten können sich auf andere Anwendungen auswirken und erschweren einen reproduzierbaren Betrieb.

### Eigene virtuelle Maschine pro Dienst

Eine eigene virtuelle Maschine pro Anwendung ermöglicht eine klare Trennung der einzelnen Dienste.

Dem stehen jedoch ein höherer Ressourcenverbrauch sowie ein größerer Aufwand für Betrieb, Wartung und Verwaltung gegenüber. Für ein Homelab mit zahlreichen Diensten ist dieser Ansatz daher nur eingeschränkt geeignet.

### Kubernetes / k3s

Kubernetes beziehungsweise k3s bietet umfangreiche Möglichkeiten zur Orchestrierung containerisierter Anwendungen und ist für größere oder hochskalierbare Umgebungen konzipiert.

Für die Anforderungen eines kleinen Homelabs würde diese Plattform jedoch zusätzliche Komplexität einführen, ohne einen entsprechenden Mehrwert für den vorgesehenen Einsatzzweck zu bieten.

### Docker

Docker ermöglicht den Betrieb containerisierter Anwendungen mit vergleichsweise geringem Ressourcenbedarf.

In Verbindung mit Docker Compose lassen sich Anwendungen deklarativ beschreiben und reproduzierbar bereitstellen. Gleichzeitig bleibt der Betriebsaufwand überschaubar und die einzelnen Dienste können unabhängig voneinander verwaltet werden.

## Warum Docker?

Docker erfüllt die definierten Anforderungen an eine standardisierte und langfristig wartbare Anwendungsplattform.

Container kapseln die Laufzeitumgebung einer Anwendung. Dadurch können Anwendungen unabhängig voneinander betrieben werden, ohne dass sich deren Laufzeitumgebungen gegenseitig beeinflussen.

Docker Compose beschreibt den gewünschten Sollzustand eines Dienstes in einer deklarativen Konfiguration. Dadurch lassen sich Deployments reproduzierbar durchführen und bei Bedarf auf anderen Systemen erneut bereitstellen.

Jeder Dienst besitzt seinen eigenen Lebenszyklus. Änderungen, Updates oder Fehler wirken sich in der Regel nur auf den jeweiligen Dienst aus.

Persistente Daten werden außerhalb der Container gespeichert. Dadurch können Anwendungen unabhängig von ihren Containern aktualisiert, migriert oder wiederhergestellt werden. Die konkrete Sicherungsstrategie richtet sich weiterhin nach den Anforderungen der jeweiligen Anwendung.

## Architekturentscheidung

Für den Betrieb von Anwendungen gelten folgende Grundsätze:

- Docker dient als Container-Laufzeitumgebung.
- Docker Compose beschreibt die deklarative Bereitstellung der Dienste.
- Jeder Dienst wird als eigenes Compose-Projekt betrieben.
- Jeder Dienst besitzt ein eigenes Datenverzeichnis.
- Persistente Daten werden außerhalb der Container gespeichert.
- Dienste werden logisch voneinander getrennt betrieben.

## Bezug zur Gesamtarchitektur

Docker wird bewusst innerhalb einer virtuellen Maschine mit Rocky Linux betrieben.

Container werden nicht direkt auf dem Proxmox-Host ausgeführt. Dadurch bleiben Virtualisierungsschicht und Anwendungen voneinander getrennt und können unabhängig voneinander gewartet oder migriert werden.

Die Entscheidung für Rocky Linux als Standardbetriebssystem wird im Artikel **„Warum Rocky Linux?“** begründet.

## Nachteile

Die gewählte Struktur bringt einen höheren organisatorischen Aufwand mit sich.

Insbesondere entstehen:

- mehr Compose-Dateien,
- mehr Verzeichnisse,
- höherer initialer Organisationsaufwand,
- die Notwendigkeit einer konsequenten Strukturierung.

Diese Nachteile werden bewusst zugunsten einer besseren Wartbarkeit, Reproduzierbarkeit und klaren Trennung der einzelnen Dienste akzeptiert.

## Entscheidung

Docker ist die Standard-Laufzeitumgebung für Anwendungen im Homelab.

Anwendungen werden grundsätzlich als eigenständige Compose-Projekte betrieben. Persistente Daten werden außerhalb der Container gespeichert, sodass einzelne Dienste unabhängig bereitgestellt, aktualisiert, gesichert und wiederhergestellt werden können.

## Überprüfung der Entscheidung

Architekturentscheidungen sind keine unumstößlichen Wahrheiten.

Diese Entscheidung sollte überprüft werden, insbesondere wenn:

- Docker die definierten Anforderungen nicht mehr erfüllt,
- sich die Anforderungen an die Infrastruktur grundlegend verändern,
- eine andere Plattform die definierten Anforderungen besser erfüllt.

Bis dahin bildet Docker die Standardplattform für den Betrieb von Anwendungen im Homelab.

---

## Persönliche Anmerkung

Ziel dieser Entscheidung ist nicht, Anwendungen möglichst schnell bereitzustellen. Ziel ist eine Infrastruktur, die auch Jahre später anhand von Dokumentation, Konfiguration und Backups reproduzierbar betrieben oder wiederhergestellt werden kann. Der einmal höhere Planungsaufwand reduziert den späteren Betriebs- und Wartungsaufwand erheblich.

Entscheidend sind eine klar strukturierte Infrastruktur, reproduzierbare Bereitstellungen sowie eine langfristig wartbare Betriebsplattform. Docker erfüllt diese Anforderungen und ist deshalb die standardisierte Laufzeitumgebung für Anwendungen innerhalb meines Homelabs.
