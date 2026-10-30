# tenth-man

Ein Plugin für Beratungen mit mehreren Modellen (ein Skill plus Agenten für die Sitze) für [Codex](https://developers.openai.com/codex) und [Claude Code](https://claude.com/claude-code). Es eignet sich für Designentscheidungen, die Überprüfung von Richtlinien oder andere Entscheidungen, die nicht von der Antwort eines einzelnen Modells abhängen sollten. Mitglieder, die unterschiedliche Modelle nutzen, stimmen ab, und ein fest vorgesehener Sitz übernimmt immer die Gegenposition. So sinkt die Gefahr, dass sich der Rat einigt, während alle denselben blinden Fleck teilen.

[English](README.md) | [日本語](README.ja.md) | [简体中文](README.zh-CN.md) | [繁體中文](README.zh-TW.md) | [한국어](README.ko.md) | [Español](README.es.md) | [Português (Brasil)](README.pt-BR.md) | [Deutsch](README.de.md) | [Français](README.fr.md)

Die stimmberechtigten Mitglieder bewerten jeden Vorschlag. Ein separater **zehnter Mann** (tenth man) muss unabhängig von seiner eigenen Einschätzung gegen jeden Vorschlag argumentieren und angeben, welche Belege jeden seiner Einwände widerlegen würden. Die Hauptsession führt den Vorsitz über drei Runden, klärt ernste Bedenken und erstellt ein Protokoll, das festhält, worauf sich die Mitglieder geeinigt haben, warum und was offen bleibt.

Der Skill legt zuerst seinen Plan für den Rat vor und wartet auf Ihre Freigabe. Wenn Sie nicht warten möchten, setzen Sie `fast` vor die Frage (zum Beispiel `/tenth-man:council fast Sollten wir ...?`): Der Vorsitz zeigt den Plan und beginnt sofort, trifft die Standardauswahl selbst und führt diese Entscheidungen im Protokoll auf. Die Schlussfolgerungen eines Rats werden nie automatisch auf Ihr Projekt angewendet.

## So funktioniert es

1. Der Vorsitz schlägt die Punkte, Modelle, Regeln, Kosten und den Ausgabeordner vor.
2. Runde 1: Die Mitglieder stimmen ab; der zehnte Mann widerspricht jedem Punkt.
3. Runde 2: Die Mitglieder überdenken ernste Bedenken und beantworten die gewichtigsten Einwände.
4. Runde 3: Die Mitglieder stimmen über den Protokollentwurf ab; der zehnte Mann prüft das Ergebnis anhand der Belege.
5. Der Vorsitz schreibt `.tenth-man/<date>-<slug>/minutes.md`.

Mögliche Stimmen sind **Zustimmung**, **Zustimmung unter Bedingungen**, **Ablehnung** und **ernste Bedenken**. Eine Einigung erfordert die Mehrheit der stimmberechtigten Mitglieder, und es dürfen keine ernsten Bedenken mehr bestehen. Der zehnte Mann hat kein Stimmrecht.

## Installation für Codex

### Als Plugin

```sh
codex plugin marketplace add Shintaro-My/tenth-man
codex plugin add tenth-man@tenth-man
```

Für einen lokalen Checkout geben Sie stattdessen dessen Pfad an:

```sh
codex plugin marketplace add /path/to/tenth-man
codex plugin add tenth-man@tenth-man
```

Rufen Sie den Plugin-Skill auf, indem Sie `$tenth-man:council` gefolgt von der Frage eingeben, oder wählen Sie ihn unter `/skills` aus:

```text
$tenth-man:council Sollten wir den Sitzungszustand von Redis in signierte Cookies verlagern?
```

### Als eigenständiger Skill

Kopieren Sie `skills/council` in ein Codex-Skill-Verzeichnis:

```sh
cp -r skills/council ~/.agents/skills/
```

Bei dieser Installation rufen Sie ihn mit `$council` auf.

Für ein einzelnes Repository kopieren Sie ihn stattdessen nach `<repo>/.agents/skills/`. Dieses Repository enthält außerdem rollenneutrale Codex-Agentenprofile für die Sitze unter `.codex/agents/`; in einem Checkout erkennt Codex sie automatisch. So verwenden Sie die Profile in allen Repositories:

```sh
cp .codex/agents/seat-*.toml ~/.codex/agents/
```

Der Skill funktioniert auch ohne diese Kopie: Er kann Subagenten mit denselben expliziten Modellzuweisungen starten und ihre Threads über die Runden hinweg fortführen. Das Plugin-Paket installiert TOML-Dateien für benutzerdefinierte Agenten nicht automatisch.

## Installation für Claude Code

### Als Plugin

```text
/plugin marketplace add Shintaro-My/tenth-man
/plugin install tenth-man@tenth-man
```

So testen Sie einen lokalen Checkout:

```sh
claude --plugin-dir /path/to/tenth-man
```

Rufen Sie ihn so auf:

```text
/tenth-man:council Sollten wir den Sitzungszustand von Redis in signierte Cookies verlagern?
```

### Manuell

```sh
cp -r skills/council ~/.claude/skills/
cp agents/seat-*.md ~/.claude/agents/
```

Der manuell installierte Befehl lautet `/council`; bei den Namen der Agenten für die Sitze entfällt das Präfix `tenth-man:`.

## Modell-Presets

Der Vorsitz schlägt für eine kleine Frage `light` vor, sonst `standard`. Beim Freigeben des Plans können Sie ein Preset wählen oder die Besetzung jedes Sitzes ändern.

Der zehnte Mann verwendet dasselbe Modell wie das Mitglied mit dem stärksten Schlussfolgerungsvermögen. Die Rangfolge beruht auf gemessenen Ergebnissen, nicht auf Versionsnummern; die Spalten „Zehnter Mann“ unten geben den Stand vom Oktober 2026 wieder. Ist der Rang eines neuen Modells unbekannt, fragt der Vorsitz Sie. Wenn Sie ein anderes Modell verwenden möchten, sagen Sie das bei der Freigabe.

### Codex

| Preset | Stimmberechtigte Mitglieder | Zehnter Mann | Ausführungen pro Runde |
|---|---|---|---|
| `light` | GPT-6 Astra, GPT-6.1 Sol, GPT-6.1 Terra | GPT-6 Astra | 4 |
| `standard` | GPT-6 Astra, GPT-6.1 Sol, GPT-6.1 Terra, GPT-6 Sol | GPT-6 Astra | 5 |
| `no-astra` | GPT-6.1 Sol, GPT-6.1 Terra, GPT-6 Sol | GPT-6.1 Sol | 4 |

Diese Standardwerte verwenden nur Modelle, die der aktuelle Codex-Host und das Konto bereitstellen. Der Vorsitz meldet nicht verfügbare Modelle und ersetzt nie stillschweigend eines durch ein anderes. Erlaubt der Host weniger gleichzeitige Ausführungen, als der Rat Sitze hat, werden die Sitze in Wellen mit demselben unveränderten Briefing ausgeführt; ausgezählt wird erst, wenn alle Sitze geantwortet haben.

Die passenden benutzerdefinierten Codex-Agenten sind `seat_astra`, `seat_sol`, `seat_terra`, `seat_previous_sol` und der optionale, günstigere `seat_luna`. Ihre TOML-Definitionen legen Modell und Reasoning-Aufwand fest und beschränken die Arbeit im Rat auf Lesezugriffe.

### Claude Code

| Preset | Stimmberechtigte Mitglieder | Zehnter Mann | Ausführungen pro Runde |
|---|---|---|---|
| `light` | Fable, Opus, Sonnet | Fable | 4 |
| `standard` | Fable, Opus, Sonnet, Opus 5, Opus 4.8 | Fable | 6 |
| `no-fable` | Opus, Sonnet, Opus 5 | Opus | 4 |

Jedes Preset enthält einen Opus-Sitz, auch wenn Fable höher eingestuft ist, weil Opus manchmal Probleme findet, die Fable übersieht.

Das Claude-Plugin enthält in `agents/` fünf rollenneutrale Agenten für die Sitze. Ein Briefing weist jeder Ausführung ihre Rolle als Mitglied oder zehnter Mann zu.

### Ein Claude-Modell hinzufügen

Ein älteres Claude-Modell lässt sich über das Feld `model:` einer Agentendefinition auswählen. Kopieren Sie die Agentendatei eines Sitzes, ändern Sie `name`, `description` und `model` und speichern Sie sie unter `~/.claude/agents/` für alle Projekte oder unter `.claude/agents/` für ein einzelnes Projekt:

```markdown
---
name: seat-opus-4-7
description: A seat in a tenth-man council, on Opus 4.7.
model: claude-opus-4-7
effort: high
tools: Read, Grep, Glob, Bash, Write
---
```

Behalten Sie denselben rollenneutralen Inhalt wie bei den mitgelieferten Agenten für die Sitze bei. Der Vorsitz kann `seat-opus-4-7` dann als Mitglied oder als zehnten Mann einsetzen.

### Eigene Claude-Agenten verwenden

Jede vorhandene Claude-Agentendefinition kann einen Sitz einnehmen. Der Vorsitz beginnt den Prompt des Agenten mit dem Hinweis, dass der Agent bei dieser Aufgabe einen Sitz im Rat einnimmt und nicht seine übliche Aufgabe erledigt, und weist ihm dann die Rolle und das gemeinsame Briefing zu. Das ist nützlich für Review-Agenten, die auf ein älteres Modell festgelegt sind.

### Wenn ein festgelegtes Claude-Modell eingestellt wird

Wird ein festgelegtes Modell nicht mehr bereitgestellt, startet der Agent für diesen Sitz nicht. Der Vorsitz lässt den Sitz weg und informiert Sie; er setzt nie stillschweigend ein anderes Modell ein. Löschen oder aktualisieren Sie die Agentendatei des Sitzes passend zu den Modellen, die Ihrem Konto zur Verfügung stehen.

## Kosten und Voraussetzungen

Jeder Sitz kostet pro Runde eine vollständige Subagenten-Ausführung. Über drei Runden verbraucht das Codex-Preset `standard` 15 Ausführungen und das Claude-Code-Preset `standard` 18.

- Codex: eine aktuelle Version mit aktivierten Subagenten und Zugriff auf die ausgewählten Modelle.
- Claude Code: die Tools Agent und SendMessage, Benachrichtigungen über abgeschlossene Hintergrund-Agenten und Zugriff auf die ausgewählten Modelle.

Der ChatGPT-Chat wird derzeit nicht unterstützt, weil er nicht für jeden Sitz Subagenten auf einem anderen Modell ausführen kann.

## Warum „zehnter Mann“?

Der Skill ist von „dem zehnten Mann“ inspiriert, einer fiktiven israelischen Regel aus dem Film *World War Z* von 2013. Wenn neun Personen aus denselben Informationen zum selben Schluss kommen, geht der Zehnte davon aus, dass sie sich irren, und untersucht das Gegenteil. Die Regel ist erfunden, aber es gibt ähnliche Praktiken:

- Nachdem der israelische Militärgeheimdienst vor dem Jom-Kippur-Krieg 1973 Warnzeichen falsch gedeutet hatte, richtete er eine Stelle ein, die seine eigenen Einschätzungen anfechten soll. Sie wird meist *ipcha mistabra* genannt, aramäisch für „das Gegenteil ist wahrscheinlich“.
- Der Talmud (Sanhedrin 17a) hält fest, dass der Angeklagte in einem Kapitalprozess freikommt, wenn alle Richter für eine Verurteilung stimmen; ein einstimmiges Ergebnis deutet darauf hin, dass niemand die Verteidigung ordentlich vertreten hat.
- In katholischen Heiligsprechungsverfahren argumentierte der „Advocatus Diaboli“ gegen den Kandidaten und prüfte Schwachstellen in der Darstellung seines Lebens und seiner Wunder. Das Amt wurde 1587 formalisiert und 1983 stark eingeschränkt.

## Lizenz

MIT. Siehe [LICENSE](LICENSE).
