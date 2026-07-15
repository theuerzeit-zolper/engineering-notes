# Warum Nginx Proxy Manager?

**Architekturentscheidung**

## Hintergrund

Das Homelab orientiert sich bewusst an einer Enterprise-Architektur mit klar getrennten Verantwortlichkeiten.

Anwendungen sollen sich auf ihre eigentliche Aufgabe konzentrieren und keine Verantwortung für TLS-Zertifikate, Portverwaltung oder die Veröffentlichung ins Netzwerk übernehmen.

Werden Anwendungen direkt über Ports veröffentlicht, steigt mit jedem zusätzlichen Dienst die Komplexität der Netzwerkkonfiguration. Gleichzeitig entstehen unnötig viele nach außen erreichbare Endpunkte.

Ein Reverse Proxy übernimmt diese Aufgaben zentral und bildet den definierten Eintrittspunkt für HTTP(S)-Dienste.

## Anforderungen

Die gewählte Lösung sollte folgende Anforderungen erfüllen:

- Zentrale Veröffentlichung von HTTP(S)-Diensten
- Zentrale TLS-Terminierung
- Automatische Verwaltung von Let's Encrypt-Zertifikaten
- Einfache Bereitstellung neuer Dienste
- Klare Trennung zwischen Anwendungen und deren Veröffentlichung
- Geringe Anzahl exponierter Ports
- Unterstützung mehrerer Backend-Systeme
- Einfache Administration

## Betrachtete Alternativen

### Direkte Portfreigaben

Bei diesem Ansatz veröffentlicht jede Anwendung ihre eigenen HTTP(S)-Ports.

Dadurch entstehen mehrere Nachteile:

- Portkonflikte
- Viele Firewall-Regeln
- Jeder Dienst verwaltet seine eigene Erreichbarkeit
- Kein zentraler Einstiegspunkt für HTTP(S)-Dienste

### Apache / nginx

Apache und nginx sind leistungsfähige Webserver und Reverse Proxys.

Für den vorgesehenen Einsatzzweck steigt der Konfigurationsaufwand jedoch mit der Anzahl der bereitgestellten Dienste deutlich an.

### Traefik

Traefik ist eine moderne Reverse-Proxy- und Routinglösung mit umfangreichen Automatisierungsmöglichkeiten.

Für die Anforderungen dieses Homelabs bietet Traefik jedoch mehr Funktionen als benötigt und erhöht die Komplexität gegenüber Nginx Proxy Manager.

### Nginx Proxy Manager

Nginx Proxy Manager stellt eine grafische Verwaltungsoberfläche bereit, integriert die Verwaltung von Let's Encrypt-Zertifikaten und ermöglicht eine einfache Konfiguration von Reverse-Proxy-Regeln.

Die Lösung erfüllt die definierten Anforderungen mit vergleichsweise geringer Komplexität.

## Warum Nginx Proxy Manager?

Die Entscheidung für Nginx Proxy Manager basiert nicht ausschließlich auf der Wahl eines bestimmten Produkts, sondern auf der Architekturentscheidung, HTTP(S)-Dienste grundsätzlich über einen zentralen Reverse Proxy bereitzustellen.

Der Reverse Proxy bildet den einzigen definierten Eintrittspunkt für HTTP(S)-Dienste. Anwendungen müssen ihre eigenen HTTP(S)-Ports nicht direkt veröffentlichen und bleiben unabhängig von ihrer externen Erreichbarkeit.

Die TLS-Terminierung erfolgt zentral am Reverse Proxy. Ebenso werden Let's Encrypt-Zertifikate zentral verwaltet. Dadurch entfällt die Notwendigkeit, Zertifikate für jede Anwendung einzeln bereitzustellen und zu verwalten.

Neue Dienste werden ausschließlich im Reverse Proxy veröffentlicht. Die Anwendungen selbst bleiben unverändert und konzentrieren sich ausschließlich auf ihre eigentliche Aufgabe.

Innerhalb der Docker-Umgebung können Dienste über interne Docker-Netzwerke miteinander kommunizieren, ohne zusätzliche Ports nach außen freizugeben. Die Verwaltung externer Ports erfolgt ausschließlich über den Reverse Proxy und nicht verteilt über die einzelnen Anwendungen.

Diese Trennung der Verantwortlichkeiten vereinfacht die Administration und sorgt für eine klar strukturierte Infrastruktur.

Die Veröffentlichung von HTTP(S)-Diensten ist eine Aufgabe der Infrastruktur und nicht der Anwendung.

## Nachteile

Auch diese Architektur bringt Kompromisse mit sich.

- Der Reverse Proxy wird zu einer zentralen Infrastrukturkomponente.
- Es muss eine zusätzliche Komponente betrieben und verwaltet werden.
- Bei mehreren Reverse Proxys steigt der organisatorische Verwaltungsaufwand.
- In der gewählten Architektur mit einer zentralen Portweiterleitung über die Fritzbox lässt sich die HTTP-01-Challenge von Let's Encrypt für mehrere Internet-Proxys nur eingeschränkt einsetzen.
- Der Reverse Proxy stellt einen zentralen Einstiegspunkt dar. Ein Ausfall beeinträchtigt die Erreichbarkeit aller veröffentlichten HTTP(S)-Dienste.

Diese Nachteile werden zugunsten einer klaren Architektur, einer zentralen Verwaltung und einer besseren Wartbarkeit bewusst akzeptiert.

## Bezug zur Gesamtarchitektur

Aktuell übernimmt ein zentraler Nginx Proxy Manager die Veröffentlichung der aus dem Internet erreichbaren Dienste.

Langfristig ist vorgesehen, pro Docker-Host einen eigenen Nginx Proxy Manager einzusetzen.

Ein zentraler Internet-Proxy verteilt eingehende Anfragen an die jeweiligen internen Reverse Proxys. Diese übernehmen anschließend das Routing innerhalb ihres Docker-Hosts.

Dadurch werden Verantwortlichkeiten klar voneinander getrennt und Docker-Netzwerke nicht unnötig nach außen geöffnet.

## Entscheidung

HTTP(S)-Dienste werden ausschließlich über einen Reverse Proxy veröffentlicht.

Als Implementierung wird Nginx Proxy Manager verwendet, da die Lösung die definierten Anforderungen mit geringem Verwaltungsaufwand erfüllt und sich gut in die bestehende Docker-Architektur integriert.

## Überprüfung der Entscheidung

Architekturentscheidungen sind keine unumstößlichen Wahrheiten.

Diese Entscheidung sollte überprüft werden, insbesondere wenn:

- Nginx Proxy Manager die definierten Anforderungen nicht mehr erfüllt,
- sich die Netzwerkarchitektur grundlegend verändert,
- eine andere Reverse-Proxy-Lösung die Anforderungen besser erfüllt.

Bis dahin bleibt Nginx Proxy Manager die Standardlösung für die Veröffentlichung von HTTP(S)-Diensten innerhalb der Infrastruktur.

---

## Persönliche Anmerkung

Diese Entscheidung ist keine Aussage darüber, dass Nginx Proxy Manager grundsätzlich die beste Reverse-Proxy-Lösung ist.

In meiner Architektur sollen Anwendungen ausschließlich ihre eigentliche Aufgabe erfüllen. Themen wie TLS, externe Erreichbarkeit und Portverwaltung gehören zur Infrastruktur und werden deshalb zentral über den Reverse Proxy umgesetzt. Dadurch bleibt die Architektur auch mit einer wachsenden Anzahl an Diensten nachvollziehbar und wartbar.
