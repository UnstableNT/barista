---
project: Barista
size: small
plan: docs/plan.md
---

# CLAUDE.md

Barista ist eine macOS-Menüleisten-App (ab macOS 14), die den Mac wach hält:
unbegrenzt, für eine Dauer, bis zu einer Uhrzeit oder bis ein Prozess endet.
Sie ersetzt `caffeinate`. Die Besonderheit dieses Repos: Die gesamte Logik
steckt in einem plattformfreien Zustandsautomaten (`BaristaCore`), der unter
Linux baut und testet. Die App führt nur dessen Effekte aus.

Dieses Projekt wird unbeaufsichtigt in einem Claude Code Project gebaut.
Niemand liest mit. Alles, was du entscheidest und was nicht in deinem Task
steht, muss ins Repository, sonst geht es verloren.

## Bevor du etwas änderst

| Du willst | Lies zuerst |
|---|---|
| einen Thread beginnen | deinen Task in `docs/plan.md`, die dort unter **Reads** genannten Decisions und Spec-Abschnitte |
| eine Struktur oder Grenze ändern | Spec §1–§2 und D2, D3 in `docs/plan.md` |
| eine festgehaltene Entscheidung umkehren | nicht tun — beende als *blocked* und sag, warum |

## Harte Invarianten

Durch Tests erzwungen (`make check-invariants`). Du schwächst, überspringst
oder löschst sie nie. Blockiert dich eine, endest du als *blocked*.

1. `BaristaCore` importiert nur Foundation (`inv_core_platform_free`). Sonst
   können Threads die Logik unter Linux nicht mehr testen.
2. Keine Fremdabhängigkeiten (`inv_no_third_party_deps`).
3. Der App-Code startet keine Prozesse (`inv_no_shellout`). Assertions laufen
   über IOKit, nie über `caffeinate` oder `pmset`.
4. Englisch und Deutsch sind vollständig und deckungsgleich
   (`inv_l10n_complete`). Jeder sichtbare Text ist ein `L10nKey`.
5. Die Mindestversion ist überall macOS 14 (`inv_min_macos_14`).
6. Jeder erwartete Fehlschlag trägt eine Task-ID (`inv_known_issues_tagged`).
7. Fail-open: Jedes Ende und jede Aussetzung einer Phase gibt alle Assertions
   frei (`core_*`-Tests zu Ende und Aussetzung,
   `it_assertion_released_on_quit`).

## Sicherheitskritischer Kern

Keiner.

## Konventionen

- Swift 6 im Sprachmodus 6, Swift Testing für alle Swift-Tests.
- Abhängigkeiten zwischen Targets: `BaristaApp` → `BaristaAppKit` →
  `BaristaCore`. Core kennt die App nicht.
- Die App enthält keine Entscheidungslogik. Was wann passiert, entscheidet
  `WakeEngine`. Die App übersetzt Systemereignisse in `Event` und `Effect` in
  Systemaufrufe.
- Systemzugriffe (IOKit, Benachrichtigungen, Login-Item, Prozesse) stehen
  hinter einem Protokoll, damit `app_*`-Tests Fakes nutzen können.
- Fehler werden nie still verschluckt: entweder als `Event` an die Engine oder
  per `os_log` (Subsystem `io.github.unstablent.barista`).

## Befehle

    make build
    make core-test         # Linux + macOS, muss grün sein
    make check-invariants  # Linux + macOS, muss grün sein
    make test              # macOS: alle Swift-Tests
    make lint
    make integration       # nur macOS
    make dmg               # nur macOS, ab T008

## Build-Umgebung

Threads laufen unter Linux. Dort laufen `make build` (nur Core), `make
core-test`, `make check-invariants` und `make lint`, und du führst sie vor
jedem Push aus. Alles mit AppKit, SwiftUI, IOKit, Benachrichtigungen, Bundle,
DMG und Integrationstests prüft **nur** der CI-Job `macos`. Dafür pushst du,
wartest auf die CI des Pull Requests und liest ihre Logs. Behaupte nie ein
macOS-Ergebnis, das nicht aus der CI stammt. Das Repo ist öffentlich,
macOS-Minuten kosten nichts. Lass trotzdem keine CI-Läufe ohne Änderung
dazwischen laufen.

## Abhängigkeiten

Keine Fremdabhängigkeiten (D11). Nur Swift-Standardbibliothek, Foundation und
Apple-Frameworks.

## Dateien, die du schreiben darfst

- Task-, Retry- und Fix-Threads: Code, Tests, `tests/integration/`,
  Build-Skripte, CI-Workflows und `docs/runs/TNNN.md` deines Tasks.
  `README.md` nur in T001 und T008.
- Review-Threads: ihre Datei in `docs/reviews/` und ihre Zeile im Run-Record.
- Nie den Run-Record eines anderen Tasks, und nie `CLAUDE.md`, `.claude/`,
  `docs/plan.md`, `docs/status.md` oder `docs/project/`. Diese Dateien ändern
  sich nur über Entscheidungs- und Buchhaltungs-Threads.

## Wie jeder Thread arbeitet

- **Früh pushen.** Committe und pushe nach jedem sinnvollen Schritt. Eine
  verlorene Sandbox verliert alles, was nicht gepusht ist.
- **Ein Task, ein Branch, ein Pull Request.** Bleib in deinem Task.
- **Rate nicht beim Design.** Klären dein Task, die Decisions, die
  Spezifikation und "Decision latitude" eine Frage nicht, endest du als
  *blocked* und sagst genau, was zu entscheiden ist.
- **Tests sind der Vertrag.** Erscheint dir ein Test falsch, schreib es in den
  Run-Record und ändere ihn nicht. Erlaubt und verlangt ist nur: die
  `withKnownIssue("TNNN")`-Markierung bzw. die Zeile in
  `tests/integration/expected-failures.txt` deines eigenen Tasks entfernen,
  sobald der Test grün ist.
- **Ideen** gehören in den Abschnitt `Ideas` deines Run-Records, nie in den
  Code.

## Wie ein Thread endet

Genau eines von:
- **done** — die Akzeptanztests sind grün (macOS-Teile: grün in der CI). Stopp.
  Kein Polieren, keine Extras.
- **blocked** — du brauchst eine Entscheidung, einen Zugang oder einen Umfang,
  den du nicht hast. Stopp.
- **over budget** — derselbe Fehler nach drei wirklich verschiedenen Ansätzen,
  oder die Arbeit ist klar größer als der Task. Stopp.

Vor dem Stopp, in jedem Fall: Run-Record aktualisieren (Status, `Result`,
deine Log-Zeile), pushen und sicherstellen, dass der Pull Request existiert.

## Non-goals

Kein Deckel-Sleep verhindern, keine automatischen Auslöser (Zeitplan, Monitor),
kein Hotkey, keine App Intents, keine Notarisierung, kein Auto-Update, keine
weiteren Sprachen. Vollständige Liste: `docs/plan.md`, Abschnitt "Non-goals".
