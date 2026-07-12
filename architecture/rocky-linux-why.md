# Warum Rocky Linux?

 **Architekturentscheidung**  

## Hintergrund

Viele Jahre war CentOS das Standardbetriebssystem für meine Linux-Server. Es war eine stabile, Enterprise-fähige Plattform ohne die Lizenzkosten von Red Hat Enterprise Linux (RHEL).

Mit der Umstellung von CentOS Linux auf CentOS Stream änderte sich die Ausrichtung des Projekts. Aus einer nachgelagerten, stabilen RHEL-kompatiblen Distribution wurde CentOS zu einer Upstream-Plattform für die Entwicklung von RHEL.

Dieses Modell hat durchaus seine Berechtigung, entsprach jedoch nicht meinen Anforderungen an ein Serverbetriebssystem. Für meine Infrastruktur stehen Stabilität und eine langfristig wartbare Plattform im Vordergrund.

Daher musste eine neue Standardplattform gewählt werden.

## Anforderungen

Die neue Distribution sollte folgende Anforderungen erfüllen:

- Langfristige Stabilität
- Hohe Kompatibilität zu RHEL
- Planbare Release- und Supportzyklen
- Enterprise-fähig
- Große Community und umfangreiche Dokumentation
- Kostenfrei
- Geeignet für private und berufliche Infrastruktur

Die jeweils neuesten Softwareversionen waren kein entscheidendes Kriterium. Für Infrastrukturserver sind Stabilität und Reproduzierbarkeit wichtiger als aktuelle Features.

## Betrachtete Alternativen

### Oracle Linux

Oracle Linux bot bereits eine vollständige RHEL-kompatible Enterprise-Distribution.

Technisch sprach wenig dagegen. Aus strategischer Sicht wollte ich jedoch eine Lösung, die stärker von einer Community getragen wird und weniger von einem Unternehmen abhängig ist.

### AlmaLinux

AlmaLinux entstand nahezu zeitgleich mit Rocky Linux und verfolgte ein ähnliches Ziel.

Zum Zeitpunkt meiner Entscheidung befanden sich beide Projekte noch am Anfang. Letztlich fiel meine Wahl auf Rocky Linux, da mich die Ausrichtung und Organisation des Projekts mehr überzeugte.

## Warum Rocky Linux?

Rocky Linux wurde mit dem Ziel gegründet, wieder eine freie, stabile und RHEL-kompatible Enterprise-Distribution bereitzustellen.

Besonders überzeugt hat mich, dass Rocky Linux von Gregory Kurtzer, einem Mitgründer von CentOS, ins Leben gerufen wurde. Dadurch entstand für mich der Eindruck, dass die ursprüngliche Idee von CentOS in einem unabhängigen Community-Projekt fortgeführt werden sollte.

Für meine Anforderungen erfüllte Rocky Linux alle wichtigen Kriterien:

- Enterprise-fähige Distribution
- Hohe Kompatibilität zu RHEL
- Community-getragene Entwicklung
- Langfristige Stabilität
- Umfangreiche Dokumentation
- Gute Grundlage für den Aufbau von Enterprise-Know-how
- Einheitliche Plattform für sämtliche Infrastrukturdienste

Damit übernimmt Rocky Linux für meine Infrastruktur die Rolle, die zuvor CentOS hatte.

## Nachteile

Keine Distribution ist für jeden Einsatzzweck die beste und einzige Wahl.

Rocky Linux bringt bewusst Kompromisse mit sich:

- Softwarepakete sind häufig älter als bei anderen Distributionen.
- Neue Funktionen stehen teilweise später zur Verfügung.
- Das Ökosystem ist kleiner als beispielsweise bei Ubuntu.

Für klassische Infrastrukturserver sind diese Nachteile jedoch akzeptabel, da Stabilität und Vorhersehbarkeit wichtiger sind als die neuesten Softwareversionen.

## Entscheidung

Rocky Linux ist das Standardbetriebssystem für meine Infrastruktur.

Sofern ein Dienst keine besonderen Anforderungen stellt, basieren neue virtuelle Maschinen und Server auf Rocky Linux.

## Überprüfung der Entscheidung

Architekturentscheidungen sind keine unumstößlichen Wahrheiten.

Diese Entscheidung sollte überprüft werden, insbesondere wenn:

- sich die Ausrichtung von Rocky Linux grundlegend verändert,
- die RHEL-Kompatibilität nicht mehr gewährleistet ist,
- eine andere Distribution die definierten Anforderungen besser erfüllt.

Bis dahin bleibt Rocky Linux die Standardplattform für meine Projekte.

---

## Persönliche Anmerkung

Diese Entscheidung war nie die Suche nach der *besten* Linux-Distribution.

Es ging darum, eine Plattform zu finden, die meinen Anforderungen an Stabilität, Wartbarkeit, Reproduzierbarkeit und Enterprise-Nähe am besten entspricht.

Rocky Linux erfüllt diese Anforderungen bis heute und bildet deshalb die Grundlage meines Homelabs sowie der technischen Beispiele und Dokumentationen in diesem Repository.