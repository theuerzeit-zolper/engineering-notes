# Engineering Notes – Style Guide

> Style Guide für alle Artikel im Repository **engineering-notes**.

Dieser Style Guide definiert den Aufbau und die Formatierung aller Dokumente im Repository. Ziel ist eine konsistente, leicht lesbare und langfristig wartbare Dokumentation.

---

# Grundprinzipien

Alle Artikel orientieren sich an folgenden Grundsätzen:

- Dokumentation statt Blogbeitrag
- Architektur und Zusammenhänge vor einzelnen Befehlen
- Praxisorientierte Beispiele
- Konsistente Formatierung
- Möglichst wenige, aber gezielt eingesetzte Hervorhebungen
- Nachvollziehbare Entscheidungen statt reiner Schritt-für-Schritt-Anleitungen

---

# Dokumentstruktur

Ein Artikel sollte nach Möglichkeit folgende Reihenfolge verwenden:

1. Titel
2. Kurzbeschreibung
3. Zielgruppe (optional)
4. Einleitung
5. Hauptinhalt
6. Best Practices (optional)
7. Troubleshooting (optional)
8. Weiterführende Informationen (optional)

Nicht jeder Artikel benötigt alle Abschnitte.

---

# Überschriften

Die Überschriften folgen einer festen Hierarchie.

```text
# Hauptkapitel

## Thema

### Unterthema
```

## Regeln

- `#` ausschließlich für Hauptkapitel verwenden.
- `##` für Themen innerhalb eines Hauptkapitels.
- `###` für Unterthemen wie Beschreibung, Eigenschaften oder Beispiele.
- Keine Überschriftenebene überspringen.
- Überschriften werden nicht nummeriert.

---

# Trennlinien

Trennlinien dienen ausschließlich der optischen Trennung größerer Abschnitte.

```markdown
---
```

## Regeln

- Zwischen Hauptkapiteln verwenden.
- Nicht zwischen jeder Überschrift einsetzen.
- Innerhalb kleiner Unterabschnitte möglichst vermeiden.

---

# Hinweise

```markdown
> **Hinweis**
>
> Hinweistext
```

Verwendung:

- ergänzende Informationen
- Hintergrundwissen
- weiterführende Erläuterungen

---

# Best Practice

```markdown
> **Best Practice**
>
> Empfehlung
```

---

# Achtung

```markdown
> **Achtung**
>
> Warnung
```

---

# Wichtig

```markdown
> **Wichtig**
>
> Wichtige Information
```

---

# Listen

Ungeordnete Listen:

```markdown
- Punkt 1
- Punkt 2
- Punkt 3
```

Nummerierte Listen:

```markdown
1. Schritt
2. Schritt
3. Schritt
```

---

# Tabellen

```markdown
| Eigenschaft | Wert |
|-------------|------|
| Beispiel    | Inhalt |
```

---

# Codeblöcke

```bash
openssl version
```

```powershell
Get-ChildItem Cert:\
```

```ini
[req]
prompt = no
```

Bevorzugte Sprachen:

- bash
- powershell
- ini
- yaml
- json
- xml
- text

---

# ASCII-Diagramme

```text
Root CA
    │
    ▼
Intermediate CA
    │
    ▼
Server Certificate
```

Diagramme sollten einfach aufgebaut und ohne zusätzliche Werkzeuge lesbar sein.

---

# Beispiele

Ein technischer Sachverhalt sollte möglichst nach folgendem Schema beschrieben werden:

1. Beschreibung
2. Beispiel
3. Erklärung (falls erforderlich)

---

# Best Practices und Troubleshooting

Empfehlungen und typische Fehler werden möglichst am Ende eines Kapitels zusammengefasst.

---

# Ziel

Alle Dokumente sollen wie Kapitel einer gemeinsamen technischen Referenz wirken.

- Klarheit
- Nachvollziehbarkeit
- Konsistenz
- Praxisbezug
- Wartbarkeit

Das Repository soll nicht wie eine Sammlung einzelner Blogbeiträge wirken, sondern wie ein zusammenhängendes Engineering-Nachschlagewerk.
