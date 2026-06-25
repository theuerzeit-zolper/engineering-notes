# RSAT Installation via PowerShell – Feature-on-Demand Failure durch WSUS (0x800f0954)

## Kontext

Remote Server Administration Tools (RSAT) unter Windows 10 werden als Feature on Demand (FoD) bereitgestellt und nicht als klassisches MSI installiert.

Die Installation erfolgt über PowerShell und ist abhängig von der FoD-Quelle (Windows Update oder Enterprise-Repository).

## Problem

Die Installation von RSAT schlägt fehl mit dem Fehler:

0x800f0954

Typischer Befehl:

Get-WindowsCapability -Name RSAT* -Online

Ergebnis:
State: NotPresent

## Ursache

In vielen Enterprise-Umgebungen ist Windows über WSUS angebunden.

WSUS (Windows Server Update Services):

- steuert Windows Updates zentral
- stellt keine Feature-on-Demand Inhalte bereit
- blockiert dadurch RSAT Downloads indirekt

Wenn Clients an WSUS gebunden sind, versucht Windows die FoD-Pakete über WSUS zu beziehen und scheitert.

Ergebnis:
Installation bricht mit 0x800f0954 ab

## Diagnose

Get-WindowsCapability -Name RSAT* -Online |
Select-Object DisplayName, State

Wenn RSAT verfügbar, aber nicht installiert ist:

State = NotPresent

## Lösung Option 1 – Gruppenrichtlinie

Pfad:

Computerkonfiguration
Administrative Vorlagen
System
Einstellungen für die Installation optionaler Komponenten und die Reparatur von Komponenten angeben

Einstellung:

- Aktiviert
- Option: Inhalte für optionale Features und Reparatur direkt von Windows Update herunterladen

Wirkung:

- FoD Inhalte werden direkt von Windows Update geladen
- WSUS wird für diese Inhalte umgangen

## Lösung Option 2 – Installation nach Anpassung

Get-WindowsCapability -Name RSAT* -Online |
Where-Object State -eq "NotPresent" |
Add-WindowsCapability -Online

## Typisches Fehlerbild

- RSAT Installation startet
- Download wird über WSUS versucht
- Fehler 0x800f0954
- Kein erfolgreicher Download aus Windows Update

## Hintergrund

RSAT basiert auf dem Windows Feature-on-Demand Modell:

- Features werden dynamisch nachgeladen
- Quelle:
  - Windows Update (öffentlich)
  - Offline FoD Repository (Enterprise Szenarien)

Wenn WSUS FoD nicht unterstützt, wird der Download blockiert.

## Lessons Learned

- RSAT ist kein klassischer Installer, sondern ein FoD Feature
- WSUS ohne FoD Support verursacht Installationsfehler
- 0x800f0954 ist ein Indikator für FoD/WSUS-Blockade
- GPO-Konfiguration beeinflusst Windows Feature Lifecycle direkt
- NotPresent bedeutet oft nicht fehlend, sondern blockierte Quelle

## Einordnung

Dieses Verhalten ist relevant für:

- Windows Client Management
- WSUS Architekturdesign
- Feature Deployment Strategien
- Enterprise AD / Admin Tooling Rollouts

## Ergebnis

Nach korrekter GPO-Konfiguration kann RSAT vollständig per PowerShell installiert werden, ohne ISO, manuelle Pakete oder Workarounds.
