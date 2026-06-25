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

Die Plattform wurde als eigenständige Kommunikationslösung innerhalb der DMZ betrieben.

Die einzelnen Komponenten laufen containerisiert auf einer dedizierten Rocky-Linux-Plattform und kommunizieren über ein internes Docker-Netzwerk. Externe Zugriffe erfolgen ausschließlich über einen Reverse Proxy.

---

## Zentrale Architekturentscheidungen

### Self-Hosted statt Cloud

Für die Einführung einer neuen Kommunikationsplattform standen grundsätzlich sowohl Cloud-basierte Lösungen als auch selbst betriebene Systeme zur Verfügung.

Microsoft Teams wurde bewusst nicht als primäre Lösung für diesen Anwendungsfall betrachtet. Obwohl Teams bereits viele Funktionen für Zusammenarbeit und Kommunikation bereitstellt, liegt der Schwerpunkt der Plattform auf einer umfassenden Collaboration-Suite und nicht auf einem einfachen Messenger-Erlebnis.

Ein wesentlicher Teil der zukünftigen Nutzer arbeitete außerhalb klassischer Büroumgebungen. Insbesondere auf Baustellen waren viele Kollegen an die direkte und unkomplizierte Kommunikation über Messenger-Dienste wie WhatsApp gewöhnt. Ziel war daher eine Lösung, die den Wechsel möglichst einfach gestaltet und ohne umfangreiche Schulung akzeptiert wird.

Neben der Benutzerfreundlichkeit spielten auch organisatorische und technische Aspekte eine Rolle. Durch den Self-Hosted-Betrieb verbleiben Kommunikationsdaten unter eigener Kontrolle, während gleichzeitig Abhängigkeiten von externen Cloud-Anbietern und nutzerabhängigen Lizenzmodellen reduziert werden.

Die Entscheidung für Matrix wurde daher nicht ausschließlich aus technischen Gründen getroffen. Ausschlaggebend war die Kombination aus Datenhoheit, Integrationsfähigkeit und einer Benutzererfahrung, die den Einstieg für die Anwender möglichst einfach gestaltet.

### Docker auf einer dedizierten Plattform

Die Plattform wurde auf einer dedizierten Rocky-Linux-Instanz betrieben. Die Wahl fiel dabei bewusst auf Rocky Linux, da die Distribution im Unternehmen bereits etabliert war und als langfristig stabiler Nachfolger von CentOS betrachtet wird.

Für den Betrieb der einzelnen Komponenten wurde ein containerbasierter Ansatz gewählt. Synapse, PostgreSQL, Dex sowie weitere Dienste werden jeweils in eigenen Docker-Containern betrieben und über ein separates internes Docker-Netzwerk miteinander verbunden.

Der Einsatz von Containern reduziert die Komplexität auf Betriebssystemebene erheblich. Anstatt zusätzliche Datenbanken, Laufzeitumgebungen oder Abhängigkeiten direkt auf dem Host zu installieren, bleiben die einzelnen Komponenten voneinander getrennt und können unabhängig aktualisiert oder ausgetauscht werden.

Darüber hinaus bietet die Containerisierung eine zusätzliche Trennung zwischen Betriebssystem und Anwendungsdiensten. Sicherheitsprobleme oder Fehlkonfigurationen einzelner Anwendungen bleiben zunächst auf die jeweilige Komponente beschränkt und führen nicht automatisch zu einem direkten Zugriff auf das zugrunde liegende Betriebssystem. Container ersetzen dabei keine Sicherheitsmaßnahmen auf Host-Ebene, tragen jedoch zu einer klareren Trennung von Verantwortlichkeiten und Angriffsflächen bei.

Bewusst wurde auf die Verteilung der Dienste auf mehrere virtuelle Maschinen verzichtet. Für die geplante Benutzerzahl bot eine einzelne Plattform ausreichend Ressourcen, während gleichzeitig Betriebsaufwand, Netzwerkkomplexität und Infrastrukturkosten gering gehalten werden konnten.

Das Ergebnis ist eine kompakte und in sich geschlossene Plattform, bei der Betriebssystem, Netzwerkzugriffe und Anwendungsdienste klar voneinander getrennt sind.

### OIDC mit Dex statt LDAP-Plugin

Eine zentrale Benutzerverwaltung über das bestehende Active Directory war eine grundlegende Anforderung des Projekts. Die Pflege separater Benutzerkonten für die Kommunikationsplattform kam weder aus administrativer Sicht noch hinsichtlich der Benutzerfreundlichkeit in Frage.

Synapse bietet von Haus aus keine direkte LDAP-Anbindung. Historisch existierten hierfür verschiedene Erweiterungen und Plugins. Im Rahmen der Evaluierung zeigte sich jedoch schnell, dass die verfügbaren LDAP-Plugins nicht mehr aktiv gepflegt werden und mit aktuellen Synapse-Versionen nur eingeschränkt oder gar nicht mehr nutzbar sind.

Gleichzeitig entwickelt sich das Matrix-Ökosystem zunehmend in Richtung moderner Authentifizierungsverfahren auf Basis von OIDC. Insbesondere mit Blick auf den Matrix Authentication Service (MAS) wurde deutlich, dass eine direkte LDAP-Integration langfristig keine zukunftsfähige Lösung darstellt.

Stattdessen wurde eine Trennung zwischen Benutzerverzeichnis und Anwendung geschaffen. Das bestehende Active Directory bleibt die zentrale Quelle für Benutzer und Gruppen, während Dex als OIDC-Provider die Authentifizierung gegenüber Matrix übernimmt.

Die Wahl fiel dabei nicht aufgrund eines umfangreichen Produktvergleichs auf Dex. Vielmehr überzeugte die Lösung durch ihre einfache Bereitstellung als Container, die direkte LDAP-Anbindung sowie die unkomplizierte Integration in die bestehende Architektur.

Das Ergebnis ist eine Authentifizierungsarchitektur, die bestehende Verzeichnisdienste weiter nutzt, gleichzeitig aber moderne Standards verwendet und zukünftige Entwicklungen im Matrix-Umfeld berücksichtigt.

### Betrieb in einer DMZ

Da die Plattform sowohl von internen als auch von externen Endgeräten genutzt werden sollte, musste ein sicherer Betrieb außerhalb des internen Unternehmensnetzes gewährleistet werden.

Die Lösung wurde daher innerhalb einer dedizierten DMZ betrieben. Dadurch bleibt die Kommunikationsplattform logisch vom produktiven Unternehmensnetz getrennt, während gleichzeitig ein kontrollierter Zugriff aus dem Internet möglich ist.

Zur Reduzierung der Angriffsfläche werden die eigentlichen Anwendungsdienste nicht direkt veröffentlicht. Stattdessen erfolgt die Bereitstellung der Webdienste ausschließlich über einen vorgeschalteten Reverse Proxy. Dieser übernimmt die TLS-Terminierung sowie die gezielte Veröffentlichung der erforderlichen Dienste.

Die einzelnen Komponenten kommunizieren ausschließlich über interne Docker-Netzwerke. Verbindungen in das Unternehmensnetz werden auf die für den Betrieb notwendigen Dienste beschränkt, insbesondere DNS und LDAPS zur Anbindung an die bestehende Benutzerverwaltung.

Administrative Zugriffe erfolgen ausschließlich über definierte Management-Systeme. Dadurch bleibt die Anzahl möglicher Zugriffswege bewusst gering und die Trennung zwischen Anwendungsbetrieb, Benutzerverkehr und Administration klar nachvollziehbar.

Das Ergebnis ist eine Architektur, die externe Erreichbarkeit ermöglicht, ohne dabei die grundlegenden Sicherheits- und Segmentierungsprinzipien des Unternehmensnetzes aufzugeben.

---

## Technische Umsetzung

### Komponenten

Die Plattform besteht aus mehreren voneinander getrennten Komponenten, die gemeinsam die Kommunikations- und Authentifizierungsdienste bereitstellen.

Als Betriebssystem kommt Rocky Linux zum Einsatz. Die einzelnen Dienste werden containerisiert betrieben und über ein internes Docker-Netzwerk miteinander verbunden.

Die zentrale Kommunikationsplattform bildet Matrix Synapse als Matrix Homeserver. Nachrichten, Räume und Benutzerdaten werden in einer PostgreSQL-Datenbank gespeichert.

Für die Authentifizierung wird Dex als OIDC-Provider eingesetzt. Dex bindet das bestehende Active Directory über LDAP an und stellt die Authentifizierungsinformationen für die Matrix-Plattform bereit.

Die Veröffentlichung der Webdienste erfolgt über Nginx Proxy Manager. Der Reverse Proxy übernimmt die TLS-Terminierung sowie die kontrollierte Bereitstellung der erforderlichen Dienste gegenüber dem Internet.

Für den produktiven Betrieb wurde Nginx Proxy Manager nicht mit der standardmäßig vorgesehenen SQLite-Datenbank betrieben. Stattdessen kommt eine dedizierte MariaDB-Instanz innerhalb des Docker-Netzwerks zum Einsatz. Dadurch werden Datenhaltung und Anwendung voneinander getrennt und die Plattform besser auf einen langfristigen und stabilen Betrieb ausgerichtet.

Als Benutzeroberfläche kommen Element Desktop, Element Mobile sowie Element Web zum Einsatz.

Während die nativen Clients direkt durch die Anwender genutzt werden, wird Element Web ebenfalls innerhalb der Plattform selbst betrieben und als Container im internen Docker-Netzwerk bereitgestellt. Dadurch verbleibt auch der Zugriff auf die Weboberfläche vollständig innerhalb der eigenen Infrastruktur und erfordert keine Einbindung externer Cloud-Dienste.

Die Trennung der einzelnen Komponenten ermöglicht eine klare Zuordnung von Verantwortlichkeiten und erleichtert Wartung, Updates und zukünftige Erweiterungen der Plattform.

### Datenflüsse

Die Plattform trennt bewusst zwischen Kommunikation, Authentifizierung und Datenhaltung. Dadurch bleiben die einzelnen Komponenten unabhängig voneinander und können bei Bedarf getrennt aktualisiert oder erweitert werden.

Für den Anwender beginnt die Nutzung über einen Element Client auf Desktop-, Mobil- oder Web-Basis. Die eigentliche Kommunikation erfolgt anschließend über den Matrix Homeserver Synapse.

Bei der Anmeldung werden Benutzer nicht direkt durch Synapse authentifiziert. Stattdessen wird die Authentifizierung an Dex als zentralen OIDC-Provider delegiert. Dex wiederum greift auf das bestehende Active Directory als führendes Benutzerverzeichnis zu.

Der Authentifizierungsablauf lässt sich vereinfacht wie folgt darstellen:

Benutzer → Element Client → Synapse → Dex → Active Directory

Nach erfolgreicher Anmeldung erhält Synapse die erforderlichen Authentifizierungsinformationen und stellt dem Benutzer die Matrix-Dienste zur Verfügung.

Die eigentlichen Kommunikationsdaten werden durch Synapse verarbeitet und in PostgreSQL gespeichert. Dadurch bleiben Authentifizierung und Datenhaltung voneinander getrennt.

Der Kommunikationsfluss kann vereinfacht wie folgt dargestellt werden:

Benutzer → Element Client → Synapse → PostgreSQL

Externe Zugriffe erfolgen ausschließlich über den Reverse Proxy. Dieser übernimmt die Veröffentlichung der Webdienste sowie die TLS-Terminierung und bildet den zentralen Einstiegspunkt für Web-, Desktop- und Mobile-Clients.

## Weiterführende Dokumentation

Dieses Dokument beschreibt die Gesamtarchitektur und die wesentlichen Designentscheidungen der Plattform.

Weitere technische Details werden in separaten Dokumenten behandelt.

### Infrastruktur und Netzwerkdesign

Beschreibung der Netzwerkarchitektur, DMZ-Integration, Docker-Netzwerke, Reverse Proxy Routing sowie der Kommunikationspfade zwischen den Komponenten.

→ [001-matrix-network-design.md](../infrastructure/001-matrix-network-design.md)

### Identity & Access Management

Beschreibung der Active-Directory-Integration, OIDC-Authentifizierung über Dex sowie des Identity-Mappings zwischen Active Directory und Matrix.

→ [001-matrix-oidc-authentication.md](../identity/001-matrix-oidc-authentication.md)

### Lessons Learned

Erfahrungen, Beobachtungen und technische Erkenntnisse aus Implementierung, Pilotbetrieb und Betriebskonzept.

→ [001-matrix-lessons-learned.md](../notes/001-matrix-lessons-learned.md)

---

## Ergebnis

Mit der beschriebenen Architektur konnte eine eigenständig betriebene Kommunikationsplattform geschaffen werden, die die ursprünglich definierten Anforderungen erfüllt.

Die Lösung ermöglicht eine zentrale und plattformübergreifende Kommunikation über Desktop-, Web- und Mobile-Clients, ohne dabei auf externe Cloud-Dienste angewiesen zu sein. Gleichzeitig bleibt die Benutzerverwaltung vollständig in die bestehende Active-Directory-Infrastruktur integriert.

Durch die Kombination aus Matrix, OIDC-basierter Authentifizierung und einer containerisierten Betriebsplattform entstand eine Lösung, die sowohl die Anforderungen an Datenhoheit als auch an Benutzerfreundlichkeit berücksichtigt.

Zum Zeitpunkt der Erstellung dieses Artikels befindet sich die Plattform in einer Pilotphase. Bereits in dieser frühen Betriebsphase werden unterschiedliche Nutzungsszenarien abgedeckt, darunter Büroarbeitsplätze, mobile Endgeräte, Home-Office-Arbeitsplätze sowie Benutzer aus angebundenen Außenstellen.

Gerade diese Vielfalt der Testumgebungen erwies sich als wertvoll. Mehrere infrastrukturelle Randbedingungen, darunter DNS- und Routing-Themen im Zusammenspiel von DMZ, VPN-Clients und Standortvernetzungen, wurden erst durch den praktischen Einsatz sichtbar und konnten bereits während der Pilotphase identifiziert und behoben werden.

Die Anzahl der Pilotanwender wächst kontinuierlich. Die bisherigen Erfahrungen bestätigen die grundsätzliche Tragfähigkeit der gewählten Architektur und liefern gleichzeitig wichtige Erkenntnisse für den weiteren Ausbau der Plattform.

---

## Rückblick

Mehrere zentrale Architekturentscheidungen haben sich bereits während der Pilotphase als sinnvoll erwiesen. Insbesondere die Trennung von Authentifizierung, Anwendung und Datenhaltung erleichterte sowohl den Betrieb als auch die Fehlersuche während der Inbetriebnahme.

Die Entscheidung für OIDC und Dex erwies sich als zukunftsfähiger Ansatz. Während ältere LDAP-Integrationen für Synapse nur eingeschränkt nutzbar waren, ließ sich die Authentifizierung über standardisierte Schnittstellen sauber in die bestehende Active-Directory-Umgebung integrieren.

Gleichzeitig zeigte das Projekt erneut, dass viele Herausforderungen nicht durch die eigentliche Anwendung entstehen, sondern durch die Integration in bestehende Infrastrukturen. Insbesondere die unterschiedlichen Zugriffswege über LAN, Internet, VPN und Standortvernetzungen führten zu Anforderungen, die erst im praktischen Einsatz sichtbar wurden.

Die Pilotphase erwies sich dabei als besonders wertvoll. Durch die Einbindung unterschiedlicher Benutzergruppen konnten verschiedene Nutzungsszenarien frühzeitig getestet und infrastrukturelle Randbedingungen identifiziert werden, bevor ein größerer Rollout erfolgte.

Rückblickend bestätigte sich die Entscheidung, zunächst eine tragfähige Architektur aufzubauen und diese anschließend schrittweise unter realen Bedingungen zu validieren.

---

## Fazit

Matrix hat sich für diesen Anwendungsfall als geeignete Grundlage für eine unternehmensinterne Kommunikationsplattform erwiesen. Durch den vollständigen Eigenbetrieb, die Integration in bestehende Verzeichnisdienste und die Nutzung offener Standards konnte eine Lösung geschaffen werden, die sowohl technische als auch organisatorische Anforderungen berücksichtigt.

Besonders wichtig war dabei nicht die Auswahl einzelner Produkte, sondern das Zusammenspiel der Architekturentscheidungen. Self-Hosted-Betrieb, containerisierte Bereitstellung, OIDC-basierte Authentifizierung und die konsequente Netzsegmentierung ergänzen sich zu einer Plattform, die sich mit überschaubarem Aufwand betreiben und weiterentwickeln lässt.

Das Projekt befindet sich weiterhin in Entwicklung. Die bisherigen Erfahrungen aus der Pilotphase zeigen jedoch, dass der gewählte Ansatz tragfähig ist und eine solide Grundlage für den weiteren Ausbau bietet.
