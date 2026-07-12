# Warum Proxmox VE?

**Architekturentscheidung**

## Hintergrund

Beruflich arbeite ich täglich mit einer VMware-Infrastruktur bestehend aus mehreren Hosts, zentralem Management und Funktionen wie Live Migration. Diese Betriebsweise hat meine Erwartungen an eine Virtualisierungsplattform geprägt.

Auch im Homelab sollten zentrale Infrastrukturdienste nicht von einem einzelnen Host abhängig sein. Gleichzeitig sollte die Plattform wirtschaftlich betrieben werden können und keine laufenden Lizenzkosten verursachen.

Ziel war daher nicht, eine Enterprise-Infrastruktur im Maßstab nachzubauen. Ziel war es, dieselben Architekturprinzipien auf handelsüblicher Hardware umzusetzen.

## Anforderungen

Die Virtualisierungsplattform sollte folgende Anforderungen erfüllen:

- Keine proprietären Lizenzkosten
- Hochverfügbarkeit zentraler Infrastrukturdienste
- Live Migration zwischen mehreren Hosts
- Zentrale Administration
- Automatisierte Backups
- Betrieb auf handelsüblicher Hardware
- Klare Trennung zwischen Hypervisor und Workloads
- Geeignet für produktive Dienste und Testumgebungen

Neben produktiven Diensten sollte die Infrastruktur ausreichend Ressourcen für Experimente und neue Technologien bereitstellen.

## Betrachtete Alternativen

### VMware vSphere

Technisch entsprach VMware bereits allen Anforderungen. Durch meine berufliche Tätigkeit war die Plattform vertraut und etabliert.

Für eine private Infrastruktur standen die erforderlichen Lizenzkosten jedoch in keinem angemessenen Verhältnis zum Nutzen. Aus wirtschaftlicher Sicht kam VMware daher nicht in Betracht.

### Einzelner Virtualisierungshost

Ein einzelner leistungsfähiger Server wäre die einfachste Lösung gewesen.

Diese Architektur hätte jedoch einen zentralen Single Point of Failure geschaffen. Wartungsarbeiten oder Hardwareausfälle hätten sämtliche produktiven Dienste gleichzeitig betroffen.

Für Testsysteme wäre dieser Kompromiss akzeptabel gewesen, für zentrale Infrastrukturkomponenten jedoch nicht.

## Warum Proxmox VE?

Proxmox VE erfüllt für meine Anforderungen die wesentlichen Eigenschaften einer modernen Virtualisierungsplattform, ohne an proprietäre Lizenzmodelle gebunden zu sein.

Entscheidend war dabei nicht die Anzahl der verfügbaren Funktionen, sondern die Möglichkeit, bewährte Architekturprinzipien aus Enterprise-Umgebungen auch im Homelab umzusetzen.

Die Infrastruktur besteht aus einem Zwei-Node-Cluster mit lokalem ZFS-Storage sowie einer regelmäßigen Replikation der virtuellen Maschinen. Ergänzt wird die Plattform durch einen separaten Proxmox Backup Server auf der QNAP.

Besonders wichtig ist dabei die konsequente Trennung der Verantwortlichkeiten.

Proxmox stellt ausschließlich die Virtualisierungsschicht bereit. Anwendungen werden nicht auf dem Hypervisor betrieben, sondern grundsätzlich in eigenen virtuellen Maschinen.

Als Gastbetriebssystem verwende ich standardmäßig Rocky Linux. Dadurch bleibt die Betriebsplattform meiner Dienste unabhängig vom eingesetzten Hypervisor. Aktualisierungen des Proxmox-Hosts beeinflussen weder die Gastbetriebssysteme noch deren Lebenszyklus.

Ich verzichte bewusst auf Linux-Container. Virtuelle Maschinen benötigen zwar mehr Ressourcen, bieten dafür jedoch eine vollständige Trennung vom Hostsystem und eine einheitliche Plattform für sämtliche Server.

## Nachteile

Auch diese Architektur bringt bewusste Kompromisse mit sich.

- Zwei Hosts verursachen mehr Betriebs- und Wartungsaufwand als ein einzelner Server.
- Virtuelle Maschinen benötigen mehr Ressourcen als Linux-Container.
- Ein Zwei-Node-Cluster erreicht nicht die Ausfallsicherheit größerer Enterprise-Cluster.
- Hochverfügbarkeit ersetzt keine Datensicherung und macht ein separates Backup-Konzept weiterhin erforderlich.

Diese Nachteile sind für meine Anforderungen akzeptabel, da Wartbarkeit, Stabilität und Reproduzierbarkeit höher bewertet werden als eine möglichst ressourcensparende Infrastruktur.

## Entscheidung

Proxmox VE bildet die Standardplattform für die Virtualisierung meiner Infrastruktur.

Produktive Dienste werden ausschließlich als virtuelle Maschinen betrieben. Linux-Container verwende ich bewusst nicht, um eine einheitliche Betriebsplattform auf Basis von Rocky Linux beizubehalten und den Hypervisor klar von den Workloads zu trennen.

Der Cluster stellt sicher, dass zentrale Infrastrukturdienste auch während Wartungsarbeiten oder beim Ausfall eines Hosts möglichst ohne längere Unterbrechung verfügbar bleiben.

Dadurch lassen sich Dienste wie Pi-hole oder andere zentrale Infrastrukturkomponenten unabhängig von der zugrunde liegenden Hardware betreiben und Wartungsarbeiten planbar durchführen.

## Überprüfung der Entscheidung

Architekturentscheidungen gelten immer im jeweiligen Kontext.

Diese Entscheidung sollte überprüft werden, wenn

- sich die Lizenz- oder Produktstrategie von Proxmox grundlegend verändert,
- die Plattform die definierten Anforderungen nicht mehr erfüllt,
- oder eine alternative Virtualisierungslösung dieselben Architekturprinzipien besser unterstützt.

Bis dahin bleibt Proxmox VE die Standardplattform meiner Virtualisierungsinfrastruktur.

---

## Persönliche Anmerkung

Diese Entscheidung entstand nicht aus dem Wunsch, eine Enterprise-Umgebung möglichst originalgetreu nachzubauen.

Für mich beginnt Enterprise nicht bei der Hardware, sondern bei den Architekturentscheidungen.

Redundanz, klare Verantwortlichkeiten, planbare Wartung und reproduzierbare Betriebsprozesse lassen sich ebenso mit zwei gebrauchten Business-PCs umsetzen wie mit deutlich größerer Hardware. Die Skalierung unterscheidet sich – die zugrunde liegenden Prinzipien nicht.