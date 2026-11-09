---
project: Barista
size: small
version: v1
status: in-progress
current_milestone: M1
default_branch: main
merge_method: squash
---

# Barista — Plan

## Purpose

Barista hält einen Mac per Klick in der Menüleiste wach: unbegrenzt, für eine
feste Dauer, bis zu einer Uhrzeit oder bis ein gewählter Prozess endet. Damit
ersetzt es den manuellen Aufruf von `caffeinate`.

Barista ändert bewusst **keine** Energieeinstellungen des Systems und verhindert
**nicht** den Ruhezustand beim Zuklappen des Deckels.

## Non-goals

- Sleep bei geschlossenem Deckel verhindern (`pmset disablesleep`,
  privilegierter Helper). Stattdessen gibt es einen Hinweis im UI (Spec §7).
- Automatische Auslöser: Zeitplan, externer Monitor, CPU-, Netzwerk- oder
  Festplattenaktivität, Fenster oder Vordergrund-App. Zeitplan und Monitor sind
  v2-Kandidaten.
- Globaler Hotkey, App Intents / Kurzbefehle. Beides sind v2-Kandidaten.
- Mehrere gleichzeitige Wachphasen; mehrere überwachte Prozesse gleichzeitig.
- Frei wählbare Timer-Dauern im Menü (v1 hat eine feste Liste plus "Bis
  Uhrzeit…"; per URL ist jede Minutenzahl von 1–1440 möglich).
- Verlängern-Button in Benachrichtigungen.
- App Store, Notarisierung, Developer-ID-Signatur, Auto-Update.
- Eigenes App-Icon. Menüleiste und App nutzen SF Symbols bzw. das
  Standard-Icon.
- Weitere Sprachen außer Englisch und Deutsch.

## Definition of Done (v1)

Wenn alle Punkte auf `main` erfüllt sind, wird `status` auf `done` gesetzt.
Weitere Arbeit ist v2 und beginnt mit einer neuen Planungsrunde, nicht in
diesem Lauf.

- [ ] CI auf dem letzten Commit von `main` ist grün, in beiden Jobs `core-linux`
      und `macos` (Spec §10).
- [ ] `make core-test` und `make check-invariants` sind unter Linux **und**
      macOS grün, `make lint` unter Linux.
- [ ] `make test` ist auf macOS grün, mit allen `app_*`-Tests aus Spec §11.
- [ ] `make integration` ist auf macOS grün, mit allen `it_*`-Tests aus
      Spec §11. Kein Test ist dabei als erwarteter Fehlschlag (XFAIL) gelaufen.
- [ ] `test ! -s tests/integration/expected-failures.txt` gilt, und
      `grep -rn "withKnownIssue" Tests/` findet nichts.
- [ ] Der letzte CI-Lauf auf `main` hat das Artefakt `Barista.dmg` hochgeladen.
      `it_dmg_contents` ist grün.
- [ ] `repo_release_workflow` ist grün. Der Release-Workflow existiert und
      erzeugt bei einem Tag `v*` ein GitHub-Release mit `Barista.dmg`.

## Milestones

### M1 — Fundament
**Exit-Kriterien:** CI auf `main` grün. Unter Linux: `make lint`,
`make core-test`, `make check-invariants`. Unter macOS: `make test`, und
`make integration` mit `it_app_launches`, `it_url_on_off`, `it_mode_display`,
`it_invalid_url_ignored`, `it_assertion_released_on_quit` als PASS (nicht
XFAIL).
**Tasks:** T001, T002, T003

### M2 — Wachphasen
**Exit-Kriterien:** alles aus M1, zusätzlich als PASS: `it_timer_expires`,
`it_until_time`, `it_process_exit`, `it_process_already_dead`.
**Tasks:** T004, T005

### M3 — Einstellungen & Energie
**Exit-Kriterien:** alles aus M2, zusätzlich als PASS: `it_activate_on_launch`,
`it_power_guard_ac_allows`. Alle `app_*`-Tests aus Spec §11 sind ohne
`withKnownIssue` grün.
**Tasks:** T006, T007

### M4 — Auslieferung
**Exit-Kriterien:** jeder Punkt der Definition of Done.
**Tasks:** T008

## Tasks

Die Tasks laufen strikt sequentiell von oben nach unten. Ein Task startet, wenn
alles aus seinem `depends_on` gemergt ist. Ein geparkter Task blockiert nur die
Tasks, die von ihm abhängen. Fix-Tasks, die während des Laufs entstehen, heißen
`TNNN-fK` und kommen direkt nach `TNNN`.

"CI-only" heißt: Das Ergebnis gibt es nur im macOS-Job der CI. Du pushst,
wartest auf CI und liest die Logs. Ein lokales Ergebnis darfst du dafür nicht
behaupten (siehe `CLAUDE.md`, Build-Umgebung).

### T001 — Skelett, CI, Invarianten, erwartete Fehlschläge
**Milestone:** M1 · **depends_on:** — · **review:** deep
**Reads:** alle Decisions, die gesamte Spezifikation (§1–§11). Die Tests
müssen das Zielverhalten aus §3–§7 prüfen.

**Scope:** Das vollständige Gerüst des Repositorys. Dazu gehören Paketstruktur,
Build-, Test- und Lint-Befehle, CI und die Core-API aus Spec §2 als
kompilierende Stubs. Außerdem alle Tests aus Spec §11: Invarianten grün, alle
anderen als markierte erwartete Fehlschläge. Danach ist CI auf `main` grün, und
jeder spätere Task muss nur Implementierung liefern und Markierungen entfernen.

**Deliverables:**
1. `Package.swift` nach Spec §1. Tools-Version 6.0, Swift-6-Sprachmodus,
   Plattform `.macOS(.v14)`. App-Targets nur unter `#if os(macOS)`.
2. `Sources/BaristaCore/` mit allen öffentlichen Typen und Signaturen aus
   Spec §2. Die Implementierungen sind Stubs, die nicht abstürzen: Sie geben
   leere Effektlisten, `nil` oder Standardwerte zurück, nie `fatalError`.
   `L10nKey` enthält alle Schlüssel aus Spec §8.
3. `Sources/BaristaApp/main.swift` und `Sources/BaristaAppKit/` mit einer
   startfähigen Agent-App (`NSApplication`, Delegate ohne Funktion).
4. `Resources/Info.plist` (Spec §9), `Resources/en.lproj/Localizable.strings`
   und `Resources/de.lproj/Localizable.strings` mit allen Schlüsseln aus
   Spec §8.
5. `scripts/bundle.sh` (Spec §9), `scripts/integration.sh` (Spec §11),
   `Makefile` mit allen Zielen aus Spec §10, `.swift-format`, `.gitignore`.
6. `.github/workflows/ci.yml` mit den Jobs `core-linux` und `macos` (Spec §10).
7. Alle Tests aus Spec §11 unter ihrem Namen:
   - `Tests/RepoTests/` mit den `inv_*`-Tests, alle grün.
     `repo_release_workflow` ist mit `withKnownIssue("T008")` markiert.
   - `Tests/BaristaCoreTests/` mit allen `core_*`-Tests, jeder mit
     `withKnownIssue("T002")` markiert.
   - `Tests/BaristaAppTests/` mit allen `app_*`-Tests, markiert mit der ID des
     Tasks aus Spec §11.
   - `tests/integration/it_*.sh` mit allen Integrationstests.
     `tests/integration/expected-failures.txt` listet jeden `it_*`-Test außer
     `it_app_launches`, jeweils mit seiner Task-ID.
8. `README.md`, höchstens 15 Zeilen: was Barista ist, `make`-Ziele, Verweis auf
   `docs/plan.md`.

**Acceptance tests:**
- Grün: alle `inv_*`, `it_app_launches`. Außerdem `make lint` und `make build`
  auf beiden Plattformen.
- Als erwarteter Fehlschlag markiert und in CI so gemeldet: jeder andere Test
  aus Spec §11.
- CI auf dem Pull Request ist in beiden Jobs grün.

**Hard limits:** Die macOS-Ergebnisse sind CI-only. Du schreibst keine
Implementierung über Stubs hinaus. Ein Test, der nur durch eine Implementierung
grün würde, bleibt markiert. Die Assertions in den Tests müssen die Zielwerte
aus der Spec prüfen und dürfen nicht auf die Stubs zugeschnitten sein.
**Non-goals:** Release-Workflow (T008), DMG (T008), echte Logik (ab T002).

### T002 — Core: Zustandsautomat, URL, Menümodell, Einstellungen
**Milestone:** M1 · **depends_on:** T001 · **review:** deep
**Reads:** D2, D4, D5, D6, D8, D13, Spec §2, §3, §4, §5, §6, §7, §8

**Scope:** Die vollständige plattformfreie Logik in `BaristaCore`:
`WakeEngine` mit allen Ereignissen und Effekten, `BaristaURL.parse`,
`Settings` (Standardwerte, Validierung), `PowerGuard`, `MenuModel.build` und
die Restzeit-Formatierung. Der Task läuft und prüft vollständig unter Linux.

**Deliverables:**
1. Implementierung aller Core-Typen aus Spec §2 nach dem Verhalten in
   Spec §3–§7.
2. Ergänzte Schlüssel in `L10nKey` und in beiden `.strings`-Dateien, falls
   nötig (Spec §8).
3. Entfernte `withKnownIssue("T002")`-Markierungen.

**Acceptance tests:** alle `core_*`-Tests aus Spec §11 grün ohne Markierung.
Alle `inv_*` bleiben grün.
**Hard limits:** Du änderst nichts in `Sources/BaristaApp/` und
`Sources/BaristaAppKit/` und nichts an den öffentlichen Signaturen aus
Spec §2. Neue öffentliche Typen sind erlaubt, wenn sie nötig sind; begründe
sie dann im `Result`.
**Non-goals:** App-Code, Fenster, IOKit.

### T003 — App-Grundgerüst: Menüleiste, Assertions, URL-Scheme
**Milestone:** M1 · **depends_on:** T002 · **review:** light
**Reads:** D1, D3, D4, D5, D8, D9, D14, D16, Spec §2, §4, §5, §8, §9

**Scope:** Die App verbindet `WakeEngine` mit dem System. Dazu gehören ein
Effekt-Ausführer, der `AssertionAdapter` (IOPM), der `StatusItemController`
(rendert `MenuModel`, Klickverhalten, Icon-Zustände), der URL-Handler, der
`Localizer` und ein `SettingsWindowController` mit Platzhalter-Inhalt. Danach
funktionieren manuelles Ein/Aus, Umschalten und der Display-Modus per Klick und
per URL.

**Deliverables:**
1. `AppDelegate`: App läuft als Agent, Lebenszyklus, Ereignisse `launched` und
   Systemaufwachen bzw. Uhrzeitänderung als `tick` (Spec §3).
2. `AssertionAdapter` nach D4.
3. `StatusItemController` nach Spec §5 und D16. "Für Dauer", "Bis Uhrzeit…"
   und "Bis Prozess endet…" sind sichtbar, aber deaktiviert, bis T004/T005 sie
   aktivieren.
4. URL-Handler für `barista://` nach D8.
5. `Localizer` nach D9 mit Sprache aus `Settings.language`.
6. Einstellungsfenster mit Platzhalter. Es öffnet sich nach D14.

**Acceptance tests:** grün und aus `expected-failures.txt` entfernt:
`it_url_on_off`, `it_mode_display`, `it_invalid_url_ignored`,
`it_assertion_released_on_quit`. Ohne Markierung grün:
`app_assertion_adapter_holds_and_releases`, `app_status_item_renders_model`,
`app_url_handler_routes_commands`, `app_localizer_switches_language`.
**Hard limits:** Alle Ergebnisse sind CI-only. Du änderst nichts an
`BaristaCore` außer neuen `L10nKey`-Schlüsseln. Sonst endest du als *blocked*.
**Non-goals:** Timer-Fenster, Prozessauswahl, echte Einstellungen,
Benachrichtigungen, Stromquelle.

### T004 — Timer, Bis-Uhrzeit, Benachrichtigungen
**Milestone:** M2 · **depends_on:** T003 · **review:** light
**Reads:** D5, D6, D9, Spec §3, §4, §5, §6

**Scope:** Die Effekte `scheduleTick`/`cancelTick` und `notify` werden
ausgeführt. Im Menü funktionieren das Untermenü "Für Dauer" und das Fenster
"Bis Uhrzeit…". Die Restzeit erscheint im Menü und optional in der Menüleiste
(Spec §5). Der `Notifier` nutzt `UNUserNotificationCenter`.

**Deliverables:**
1. Tick-Ausführung über einen einzelnen `Timer` auf die Deadline aus dem
   Effekt. Zusätzlich aktualisiert ein UI-Timer im 30-Sekunden-Takt die
   Restzeit-Anzeige, aber nur bei aktiver Deadline-Phase.
2. Fenster "Bis Uhrzeit…": Stunden- und Minutenauswahl, Standard ist die
   aktuelle Zeit plus 1 h, gerundet auf 5 Minuten. OK sendet `onUntil`.
3. `Notifier` hinter einem Protokoll. Er fragt die Berechtigung beim ersten
   `notify` an, nicht beim Start. Ohne Berechtigung tut er still nichts.

**Acceptance tests:** grün und entfernt: `it_timer_expires`, `it_until_time`.
Ohne Markierung grün: `app_notifier_respects_permission`.
**Hard limits:** Alle Ergebnisse sind CI-only. Du änderst die Core-Logik nicht;
ein Core-Fehler heißt *blocked* mit dem fehlschlagenden Szenario.
**Non-goals:** Prozessüberwachung, Verlängern-Button.

### T005 — Bis Prozessende
**Milestone:** M2 · **depends_on:** T004 · **review:** light
**Reads:** D5, D7, D8, Spec §3, §4, §5

**Scope:** Die Effekte `watchProcess`/`stopWatchingProcess` werden ausgeführt.
Dazu kommen `ProcessCatalog` (Prozessliste des Benutzers) und das Fenster
"Bis Prozess endet…" mit Suche. Die URL `barista://on?pid=N` funktioniert, mit
Namensauflösung nach D7.

**Deliverables:**
1. `ProcessWatcher` nach D7, er meldet `processExited` bzw.
   `processWatchFailed`.
2. `ProcessCatalog`: Apps (`NSWorkspace.runningApplications`) zuerst, mit
   Icon, alphabetisch. Dann alle übrigen Prozesse des aktuellen Benutzers
   (libproc) mit Name und PID, alphabetisch. Barista selbst ist ausgeschlossen.
3. Auswahlfenster mit Suchfeld (Name oder PID), Aktualisieren-Button und
   OK/Abbrechen.

**Acceptance tests:** grün und entfernt: `it_process_exit`,
`it_process_already_dead`. Ohne Markierung grün:
`app_process_watcher_reports_exit`, `app_process_watcher_dead_pid`,
`app_process_catalog_lists_own_processes`.
**Hard limits:** Alle Ergebnisse sind CI-only. `inv_no_shellout` gilt auch
hier: Du startest keine Prozesse aus dem App-Code, nur aus Tests.
**Non-goals:** Fremde Prozesse anderer Benutzer, mehrere Prozesse gleichzeitig.

### T006 — Einstellungen, Sprache, Start bei Anmeldung
**Milestone:** M3 · **depends_on:** T005 · **review:** light
**Reads:** D9, D13, D14, D16, Spec §6, §7, §8

**Scope:** Das echte Einstellungsfenster mit allen Einstellungen aus Spec §6.
Dazu gehören der `SettingsStore` (UserDefaults ↔ `Settings`), der
Sprachwechsel ohne Neustart, Start bei Anmeldung über `SMAppService.mainApp`
und `activateOnLaunch`.

**Deliverables:**
1. `SettingsStore` nach D13. Jede Änderung sendet `settingsChanged`.
2. Einstellungsfenster mit den Bereichen und der Reihenfolge aus Spec §6. Der
   Bereich "Energie" enthält die Steuerelemente von T007 noch nicht; dort steht
   ein leerer Bereich.
3. `LoginItemService` hinter einem Protokoll. Scheitert `register` oder
   `unregister`, springt der Schalter zurück, und darunter erscheint eine
   Fehlerzeile.
4. Verhalten beim Start und beim erneuten Öffnen nach D14.

**Acceptance tests:** grün und entfernt: `it_activate_on_launch`. Ohne
Markierung grün: `app_settings_store_roundtrip`,
`app_settings_store_invalid_values`, `app_login_item_failure_reverts`.
**Hard limits:** Alle Ergebnisse sind CI-only. Neue Einstellungen über Spec §6
hinaus sind nicht erlaubt.
**Non-goals:** Netzteil-Schutz-UI und Deckel-Hinweis (T007).

### T007 — Netzteil-Schutz, Akku-Schwelle, Deckel-Hinweis
**Milestone:** M3 · **depends_on:** T006 · **review:** light
**Reads:** D5, D6, Spec §3, §5, §6, §7

**Scope:** Der `PowerSourceMonitor` liest die Stromquelle über IOKit
(`IOPSCopyPowerSourcesInfo`, Änderungsbenachrichtigung über
`IOPSNotificationCreateRunLoopSource`) und sendet `powerChanged`. Dazu kommen
der Bereich "Energie" im Einstellungsfenster mit Netzteil-Schutz,
Akku-Schwelle und Deckel-Hinweis sowie die Darstellung "ausgesetzt" in Icon und
Menü.

**Deliverables:**
1. `PowerSourceMonitor`. Die reine Funktion
   `PowerSourceParser.snapshot(from:)` wandelt die IOPS-Beschreibungen in
   `PowerSnapshot` um (Spec §7).
2. Bereich "Energie" nach Spec §6 und §7.
3. Icon und Statuszeile im Zustand "ausgesetzt" (Spec §5).

**Acceptance tests:** grün und entfernt: `it_power_guard_ac_allows`. Ohne
Markierung grün: `app_power_source_parsing`.
**Hard limits:** Alle Ergebnisse sind CI-only. Ein CI-Runner hat keinen Akku.
Das Akku-Verhalten wird nur über `core_power_guard_rules` und
`app_power_source_parsing` mit festen Beispieldaten geprüft. Simuliere keinen
Akku im laufenden System.
**Non-goals:** Sleep bei geschlossenem Deckel verhindern.

### T008 — DMG und Release
**Milestone:** M4 · **depends_on:** T007 · **review:** light
**Reads:** D10, D15, Spec §9, §10

**Scope:** `scripts/dmg.sh` und `make dmg` bauen `build/Barista.dmg`. Die CI
lädt sie als Artefakt hoch. Ein Release-Workflow erstellt bei einem Tag `v*`
ein GitHub-Release mit der DMG. Die README erhält Installationshinweise.

**Deliverables:**
1. `scripts/dmg.sh`: `hdiutil` erzeugt eine UDZO-DMG mit dem Volume-Namen
   `Barista`, `Barista.app` und einem Symlink auf `/Applications`.
2. Der Job `macos` in `ci.yml` lädt `build/Barista.dmg` als Artefakt
   `Barista.dmg` hoch.
3. `.github/workflows/release.yml`: Trigger `push` auf Tags `v*`, Runner wie
   der Job `macos`, `permissions: contents: write`. Er baut mit
   `make dmg VERSION=${GITHUB_REF_NAME#v}` und erstellt das Release mit
   `gh release create`. Er braucht nur `GITHUB_TOKEN`, kein weiteres Secret.
4. README-Abschnitt "Installation": DMG öffnen, App nach Programme ziehen.
   Beim ersten Start: Da die App nicht notarisiert ist, Systemeinstellungen >
   Datenschutz & Sicherheit > "Dennoch öffnen". Alternativ
   `xattr -dr com.apple.quarantine /Applications/Barista.app`.

**Acceptance tests:** grün und entfernt: `it_dmg_contents`. Ohne Markierung
grün: `repo_release_workflow`.
**Hard limits:** Die macOS-Ergebnisse sind CI-only. Du erstellst kein Tag und
kein Release; der Workflow wird nur angelegt.
**Non-goals:** Notarisierung, Auto-Update, Homebrew-Cask.

## Spezifikation

Verbindlich für alle Tasks. Bezeichner im Code sind englisch.

### §1 Paketstruktur

```
Package.swift
Sources/BaristaCore/        Plattformfrei, nur Foundation (D2)
Sources/BaristaAppKit/      macOS-App-Code (Library, damit testbar)
Sources/BaristaApp/         nur main.swift, ausführbares Produkt "Barista"
Tests/BaristaCoreTests/     core_*  (Linux + macOS)
Tests/RepoTests/            inv_*, repo_*  (Linux + macOS, prüfen Repo-Dateien)
Tests/BaristaAppTests/      app_*   (nur macOS)
tests/integration/          it_*.sh, expected-failures.txt  (nur macOS)
Resources/                  Info.plist, en.lproj/, de.lproj/  (keine SwiftPM-Ressourcen, D3)
scripts/                    bundle.sh, integration.sh, dmg.sh
build/                      Ausgabe, in .gitignore
```

Abhängigkeiten zwischen den Targets: `BaristaApp` → `BaristaAppKit` →
`BaristaCore`, `BaristaAppTests` → `BaristaAppKit`, `RepoTests` →
`BaristaCore` (für `L10nKey.allCases`). `BaristaAppKit`, `BaristaApp` und
`BaristaAppTests` gibt es nur unter macOS.

### §2 Core-API

Diese öffentlichen Signaturen sind fest, damit die Tests aus T001 kompilieren.
Alle Typen sind `Sendable`. Wertetypen sind `Equatable`, `AssertionKind` ist
`Hashable`.

```swift
public enum WakeMode: String, CaseIterable { case system, systemAndDisplay }
public enum WakeSource: String { case manual, timer, until, process }
public enum WakeEnd { case unlimited; case deadline(Date); case processExit(pid: Int32, name: String) }
public struct WakeSession { public var source: WakeSource; public var mode: WakeMode; public var end: WakeEnd; public var startedAt: Date }

public enum Command {
  case on(mode: WakeMode?)                                   // manuell, unbegrenzt
  case onFor(minutes: Int, mode: WakeMode?)                  // 1...1440
  case onUntil(hour: Int, minute: Int, mode: WakeMode?)
  case onUntilProcessExit(pid: Int32, name: String?, mode: WakeMode?)
  case off
  case toggle
  case setDefaultMode(WakeMode)
}

public enum PowerSnapshot { case ac; case battery(percent: Int); case unknown }
public enum SuspendReason { case notOnAC; case batteryBelow(threshold: Int); case powerUnknown }
public enum AssertionKind { case preventSystemSleep, preventDisplaySleep }
public enum NotificationKind { case timerEnded, untilEnded, processEnded(name: String), processNotFound(name: String), assertionFailed }

public enum Event {
  case launched
  case command(Command)
  case tick
  case processExited(pid: Int32)
  case processWatchFailed(pid: Int32)
  case assertionFailed
  case powerChanged(PowerSnapshot)
  case settingsChanged(Settings)
}

public enum Effect {
  case holdAssertions(Set<AssertionKind>, name: String)   // ersetzt die gehaltene Menge vollständig
  case releaseAssertions
  case watchProcess(pid: Int32)
  case stopWatchingProcess
  case scheduleTick(at: Date)                             // ersetzt einen geplanten Tick
  case cancelTick
  case notify(NotificationKind)
  case saveSettings(Settings)                             // App schreibt die Einstellungen
}

public struct WakeState { public var session: WakeSession?; public var suspension: SuspendReason? }

public struct WakeEngine {
  public init(settings: Settings, power: PowerSnapshot)
  public private(set) var state: WakeState
  public private(set) var settings: Settings
  public mutating func handle(_ event: Event, now: Date, calendar: Calendar) -> [Effect]
}

public enum BaristaURL { public static func parse(_ string: String) -> Command? }

public enum LanguageSetting: String, CaseIterable { case system, en, de }
public enum ClickBehavior: String, CaseIterable { case leftOpensMenu, leftToggles }
public struct Settings {
  public var language: LanguageSetting
  public var defaultMode: WakeMode
  public var clickBehavior: ClickBehavior
  public var activateOnLaunch: Bool
  public var launchAtLogin: Bool
  public var showRemainingTimeInMenuBar: Bool
  public var notificationsEnabled: Bool
  public var onACOnly: Bool
  public var batteryThreshold: Int?          // nil oder 5...95 in 5er-Schritten
  public static let defaults: Settings
  public static func validated(from values: [String: Any]) -> Settings   // Schlüssel = Eigenschaftsnamen
}

public enum PowerGuard { public static func evaluate(_ settings: Settings, _ power: PowerSnapshot) -> SuspendReason? }

public struct L10nText { public var key: L10nKey; public var args: [String] }
public enum MenuItemID: String { case status, toggle, duration, d15, d30, d60, d120, d240, untilTime, untilProcess, keepDisplayAwake, settings, quit }
public struct MenuItem { public var id: MenuItemID; public var title: L10nText; public var enabled: Bool; public var checked: Bool; public var children: [MenuItem] }
public struct MenuModel {
  public var items: [MenuItem]
  public var iconState: IconState
  public var menuBarText: String?            // nur bei showRemainingTimeInMenuBar und Deadline-Phase
  public static func build(state: WakeState, settings: Settings, now: Date, calendar: Calendar) -> MenuModel
}
public enum IconState { case inactive, active, suspended }
public enum RemainingTime {
  public static func menuText(seconds: Int) -> L10nText   // Statuszeile
  public static func barText(seconds: Int) -> String      // "H:MM"
}
public enum L10nKey: String, CaseIterable { /* Schlüssel aus §8 */ }
```

### §3 Verhalten der WakeEngine

- **Höchstens eine Wachphase.** Jeder `on*`-Befehl beendet eine laufende Phase
  ohne Benachrichtigung und startet die neue.
- **Modus:** `mode: nil` nimmt `settings.defaultMode`. `setDefaultMode` ändert
  `settings.defaultMode` und erzeugt `saveSettings`. Bei aktiver Phase wechselt
  diese sofort in den neuen Modus (`holdAssertions` mit der neuen Menge).
- **`settingsChanged`** übernimmt die neuen Einstellungen ohne `saveSettings`.
  Weicht `defaultMode` ab, wechselt eine aktive Phase wie bei `setDefaultMode`
  den Modus. Danach wird die Aussetzung neu bewertet.
- **Assertions:** `system` → `{preventSystemSleep}`. `systemAndDisplay` →
  `{preventSystemSleep, preventDisplaySleep}`. Der Name ist
  `"Barista: <source>"`, z. B. `"Barista: timer"`.
- **Start einer Phase:** Ohne Aussetzung kommt `holdAssertions`. Bei Deadline
  zusätzlich `scheduleTick(at: deadline)`, bei Prozess zusätzlich
  `watchProcess(pid)`.
- **Dauer:** Die Deadline ist `now + minutes·60`. Minuten außerhalb von
  1...1440 → keine Änderung, keine Effekte.
- **Bis Uhrzeit:** Die Deadline ist die nächste Uhrzeit `hour:minute` im
  übergebenen `calendar`. Liegt sie heute ≤ `now`, gilt morgen. Ungültige
  Werte → keine Änderung.
- **Prozess:** `name ?? "PID <pid>"` wird als Name gespeichert.
- **`tick`:** Ist `now ≥ deadline`, endet die Phase. Dann kommen
  `releaseAssertions`, `cancelTick` und `notify(.timerEnded)` bzw.
  `notify(.untilEnded)`. Sonst kommt erneut `scheduleTick(at: deadline)`.
- **`processExited(pid)`** der überwachten PID → Ende, `releaseAssertions`,
  `stopWatchingProcess`, `notify(.processEnded(name))`. Eine fremde PID wird
  ignoriert.
- **`processWatchFailed(pid)`** → wie oben, aber `notify(.processNotFound(name))`.
- **`assertionFailed`** → Ende, `releaseAssertions`, `cancelTick` bzw.
  `stopWatchingProcess`, `notify(.assertionFailed)`.
- **`off`** → Ende ohne Benachrichtigung, mit allen Aufräum-Effekten. Ohne
  aktive Phase → keine Effekte.
- **`toggle`:** inaktiv → wie `on(mode: nil)`, aktiv → wie `off`.
- **`launched`:** Bei `settings.activateOnLaunch` → wie `on(mode: nil)`.
- **Aussetzung:** Nach jedem Ereignis gilt
  `suspension = PowerGuard.evaluate(settings, power)`. Wechselt sie während
  einer aktiven Phase von `nil` auf einen Grund, kommt `releaseAssertions`.
  Wechselt sie zurück auf `nil`, kommt `holdAssertions`. Die Phase läuft weiter,
  Deadlines zählen weiter.
- **Benachrichtigungen** kommen nur bei `settings.notificationsEnabled`.
- **Reihenfolge der Effekte:** Bei einem Wechsel zwischen zwei Phasen kommt
  zuerst `holdAssertions` für die neue Phase, die die Menge vollständig
  ersetzt. Ein `releaseAssertions` dazwischen gibt es nicht. Der Adapter legt
  neue Assertions an, bevor er alte freigibt (D4).

### §4 URL-Grammatik (`barista://`)

| URL | Command |
|---|---|
| `barista://on` | `on` |
| `barista://on?minutes=N` | `onFor`, N ganzzahlig 1–1440 |
| `barista://on?until=HH:MM` | `onUntil`, genau zwei Ziffern je Teil, 00–23 / 00–59 |
| `barista://on?pid=N` | `onUntilProcessExit(name: nil)`, N > 0 |
| `…&display=1` bzw. `display=0` | `mode: .systemAndDisplay` bzw. `.system`; fehlt der Parameter → `nil` |
| `barista://off` | `off` |
| `barista://toggle` | `toggle` |

Schema und Host werden ohne Beachtung der Groß-/Kleinschreibung verglichen,
Parameternamen exakt. Ungültig, also `nil`, sind: unbekannter Host,
unbekannter oder doppelter Parameter, mehr als einer von
`minutes`/`until`/`pid`, Parameter an `off`/`toggle`, ungültige Werte, ein Pfad
oder ein Fragment. Die App ignoriert ungültige URLs und protokolliert sie per
`os_log` auf Stufe `.error`.

### §5 Menü und Icon

Die Reihenfolge der Einträge in `MenuModel.items` ist fest:

1. `status`, deaktiviert. Text je nach Zustand:
   - inaktiv → `status.inactive`
   - unbegrenzt → `status.active.unlimited`
   - Timer oder Uhrzeit → `RemainingTime.menuText`
   - Prozess → `status.active.process` mit Name
   - ausgesetzt → `status.suspended.notOnAC`, `status.suspended.battery` (mit
     Schwelle) oder `status.suspended.powerUnknown`
2. `toggle`: `menu.turnOn` bzw. `menu.turnOff`.
3. `duration` (`menu.duration`) mit den Kindern `d15`, `d30`, `d60`, `d120`,
   `d240` und `untilTime` (`menu.untilTime`). Das Kind der aktiven Dauer ist
   nicht markiert, weil die Restzeit in der Statuszeile steht.
4. `untilProcess` (`menu.untilProcess`).
5. `keepDisplayAwake` (`menu.keepDisplayAwake`). Markiert, wenn der Modus der
   aktiven Phase, oder ohne Phase `settings.defaultMode`, `systemAndDisplay`
   ist. Ein Klick sendet `setDefaultMode` mit dem anderen Modus.
6. `settings` (`menu.settings`).
7. `quit` (`menu.quit`).

Trenner zeichnet die App nach 1, nach 5 und nach 6; sie gehören nicht zum
Modell.

**Icon (SF Symbols):** `inactive` → `cup.and.saucer`, `active` →
`cup.and.saucer.fill`, `suspended` → `cup.and.saucer.fill` mit Alpha 0.4. Das
Icon ist ein Template-Bild. `menuBarText` steht rechts neben dem Icon.

**Restzeit:** `menuText` liefert bei ≥ 3600 s `status.active.remainingHours`
mit "H:MM", bei < 3600 s `status.active.remainingMinutes` mit Minuten,
aufgerundet und mindestens 1. `barText` liefert immer "H:MM", die Minuten
aufgerundet.

### §6 Einstellungen

| Schlüssel (UserDefaults) | Werte | Standard |
|---|---|---|
| `language` | `system`, `en`, `de` | `system` |
| `defaultMode` | `system`, `systemAndDisplay` | `system` |
| `clickBehavior` | `leftOpensMenu`, `leftToggles` | `leftOpensMenu` |
| `activateOnLaunch` | Bool | `false` |
| `launchAtLogin` | Bool | `false` |
| `showRemainingTimeInMenuBar` | Bool | `false` |
| `notificationsEnabled` | Bool | `true` |
| `onACOnly` | Bool | `false` |
| `batteryThreshold` | fehlt (aus) oder Int 5–95 in 5er-Schritten | fehlt |

Ein fehlender, falsch typisierter oder ungültiger Wert ergibt den Standardwert
dieses einen Schlüssels (`Settings.validated`).

**Fenster "Barista-Einstellungen"** mit drei Bereichen untereinander, ohne Tabs:

- **Allgemein:** Sprache, Standardmodus ("Display wach halten"),
  Klickverhalten, Beim Start aktivieren, Bei Anmeldung starten, Restzeit in der
  Menüleiste, Benachrichtigungen.
- **Energie:** Nur am Netzteil, Akku-Schwelle (Aus, 5 %, 10 % … 95 %), dazu der
  Deckel-Hinweis (§7).
- **Info:** Version (`CFBundleShortVersionString`), Link zum Repository
  `https://github.com/UnstableNT/barista`.

### §7 Energie

- `PowerGuard.evaluate`: Bei `onACOnly == false` und `batteryThreshold == nil`
  → `nil`. Sonst:
  - `.unknown` → `.powerUnknown`
  - `.ac` → `nil`
  - `.battery(p)` mit `onACOnly` → `.notOnAC`
  - `.battery(p)` mit Schwelle `t` und `p < t` → `.batteryBelow(threshold: t)`
  - sonst `nil`
- `PowerSourceParser`: Keine interne Batterie im IOPS-Ergebnis (Desktop-Mac,
  CI-Runner) → `.ac`. Stromquelle `AC Power` → `.ac`. `Battery Power` mit
  Prozentwert → `.battery`. Alles andere → `.unknown`.
- **Deckel-Hinweis:** Steht dauerhaft im Bereich "Energie", nur wenn eine
  interne Batterie existiert. Text `settings.lidHint`: Zuklappen versetzt den
  Mac trotzdem in den Ruhezustand.

### §8 Lokalisierung

Englisch ist die Basissprache, Deutsch die Übersetzung. Beide Dateien enthalten
genau dieselben Schlüssel, keiner davon leer. Die Schlüssel stehen in `L10nKey`
mit dem Schlüssel als `rawValue`; der Case-Name ist der Schlüssel in
camelCase ohne Punkte (`status.inactive` → `statusInactive`). Platzhalter nutzen `%@`, positionell
(`%1$@`) bei mehreren. Mindestumfang, den T001 anlegt:

`status.inactive`, `status.active.unlimited`, `status.active.remainingHours`,
`status.active.remainingMinutes`, `status.active.process`,
`status.suspended.notOnAC`, `status.suspended.battery`,
`status.suspended.powerUnknown`, `menu.turnOn`, `menu.turnOff`,
`menu.duration`, `menu.d15`, `menu.d30`, `menu.d60`, `menu.d120`, `menu.d240`,
`menu.untilTime`, `menu.untilProcess`, `menu.keepDisplayAwake`,
`menu.settings`, `menu.quit`, `notify.timerEnded`, `notify.untilEnded`,
`notify.processEnded`, `notify.processNotFound`, `notify.assertionFailed`,
`settings.lidHint`.

Spätere Tasks ergänzen Schlüssel für Fenstertexte. Sie tragen sie in
`L10nKey` und in beide Dateien ein.

### §9 Bundle

- `scripts/bundle.sh [VERSION]` baut `swift build -c release --product Barista`
  und erzeugt `build/Barista.app` mit:
  - `Contents/MacOS/Barista`
  - `Contents/Info.plist` (aus `Resources/Info.plist`, Version eingesetzt)
  - `Contents/Resources/en.lproj` und `Contents/Resources/de.lproj`
  - danach `codesign --force --deep --sign -`
- `VERSION` ist das Argument, sonst `git describe --tags --abbrev=0` ohne
  führendes `v`, sonst `0.0.0`.
- `Info.plist`:
  - `CFBundleIdentifier` = `io.github.unstablent.barista`
  - `CFBundleName`/`CFBundleExecutable` = `Barista`
  - `LSUIElement` = true
  - `LSMinimumSystemVersion` = `14.0`
  - `CFBundleDevelopmentRegion` = `en`, `CFBundleLocalizations` = `en`, `de`
  - `CFBundleURLTypes` mit Schema `barista`
  - `NSHumanReadableCopyright`

### §10 Befehle und CI

| Ziel | Linux | macOS |
|---|---|---|
| `make build` | `swift build` | `swift build` |
| `make core-test` | `swift test --filter BaristaCoreTests` | dito |
| `make check-invariants` | `swift test --filter RepoTests` | dito |
| `make test` | wie `core-test` + `check-invariants` | `swift test` (alle Targets) |
| `make lint` | `swift format lint --strict --recursive Sources Tests Package.swift` | dito |
| `make bundle` | Abbruch mit "macOS only" | `scripts/bundle.sh $(VERSION)` |
| `make integration` | Abbruch mit "macOS only" | `make bundle`, dann `scripts/integration.sh` |
| `make dmg` | Abbruch mit "macOS only" | `make bundle`, dann `scripts/dmg.sh` (ab T008) |

**`.github/workflows/ci.yml`:** Trigger `pull_request` und `push` auf `main`,
`concurrency` pro Ref mit `cancel-in-progress`.

- **Job `core-linux`:** `ubuntu-latest` im offiziellen Container `swift:<Version>`.
  Schritte: `make lint`, `make core-test`, `make check-invariants`.
- **Job `macos`:** Runner nach "Decision latitude".
  Schritte: `swift --version`, `make test`, `make integration`. Ab T008
  zusätzlich `make dmg` und Upload-Artefakt `Barista.dmg`.

### §11 Tests

**Erwartete Fehlschläge.**
- Swift-Tests eines späteren Tasks stehen in `withKnownIssue("TNNN") { … }`.
  Swift Testing meldet einen Fehler, wenn darin kein Issue auftritt. Ein Task
  muss die Markierung also entfernen, sobald der Test grün ist.
- `tests/integration/expected-failures.txt` hat pro Zeile `<it_name> <TNNN>`.
- `scripts/integration.sh [name…]` führt die gewählten oder alle
  `tests/integration/it_*.sh` aus, jeden mit 180 s Zeitlimit. Das Ergebnis ist
  je Test PASS, FAIL, XFAIL (gelistet und gescheitert, zählt als OK) oder
  XPASS (gelistet und bestanden, zählt als Fehler mit dem Hinweis "aus
  expected-failures.txt entfernen"). Am Ende steht eine Tabelle, der Exit-Code
  ist ≠ 0 bei FAIL oder XPASS.
- Jeder `it_*`-Test setzt die Defaults zurück
  (`defaults delete io.github.unstablent.barista`), registriert das Bundle bei
  LaunchServices, startet `build/Barista.app`, prüft und beendet die App.
  Assertions prüft er über `pmset -g assertions` auf den Namen `Barista: …`.

**Invarianten (`RepoTests`, T001, grün ab T001):**

| Test | prüft |
|---|---|
| `inv_core_platform_free` | Kein `import` außer `Foundation` in `Sources/BaristaCore/` |
| `inv_no_third_party_deps` | `Package.swift` ohne `.package(`; keine `Package.resolved` |
| `inv_no_shellout` | Kein `Process(`, `NSTask`, `posix_spawn`, `system(`, `popen(` in `Sources/` |
| `inv_l10n_complete` | en und de haben gleiche Schlüsselmengen, keine leeren Werte, jeder `L10nKey.allCases`-Schlüssel ist vorhanden |
| `inv_min_macos_14` | `Package.swift` enthält `.macOS(.v14)`; `Resources/Info.plist` hat `LSMinimumSystemVersion` `14.0` |
| `inv_known_issues_tagged` | Jedes `withKnownIssue(` in `Tests/` nennt eine Task-ID `T\d{3}`; jede Zeile in `expected-failures.txt` hat das Format `it_\w+ T\d{3}` und der Test existiert |

**Weitere Repo-Tests:** `repo_release_workflow` (T008):
`.github/workflows/release.yml` existiert, triggert auf Tags `v*`, ruft
`make dmg` auf und hängt `build/Barista.dmg` an ein Release an.

**Core-Tests (T002):**
`core_manual_on_off`, `core_toggle`, `core_timer_deadline`,
`core_until_time_today_and_tomorrow`, `core_invalid_command_values`,
`core_process_exit_ends_session`, `core_process_exit_foreign_pid_ignored`,
`core_process_watch_failed`, `core_new_session_replaces_old`,
`core_mode_switch_while_active`, `core_assertion_failed_ends_session`,
`core_power_guard_rules`, `core_suspension_releases_and_restores`,
`core_suspension_keeps_deadline`, `core_notifications_respect_setting`,
`core_launch_activates_if_enabled`, `core_url_grammar_valid`,
`core_url_grammar_invalid`, `core_settings_defaults`,
`core_settings_invalid_values`, `core_menu_model_inactive`,
`core_menu_model_active_variants`, `core_menu_model_suspended`,
`core_remaining_time_format`.

**App-Tests (`BaristaAppTests`, Task-ID für die Markierung):**

| Test | Task |
|---|---|
| `app_assertion_adapter_holds_and_releases` (echte IOPM-Assertions, über `IOPMCopyAssertionsByProcess` geprüft) | T003 |
| `app_status_item_renders_model` | T003 |
| `app_url_handler_routes_commands` | T003 |
| `app_localizer_switches_language` | T003 |
| `app_notifier_respects_permission` (Fake-Center) | T004 |
| `app_process_watcher_reports_exit` | T005 |
| `app_process_watcher_dead_pid` | T005 |
| `app_process_catalog_lists_own_processes` | T005 |
| `app_settings_store_roundtrip` | T006 |
| `app_settings_store_invalid_values` | T006 |
| `app_login_item_failure_reverts` (Fake-Service) | T006 |
| `app_power_source_parsing` (feste IOPS-Beispieldaten) | T007 |

**Integrationstests (macOS):**

| Test | prüft | Task |
|---|---|---|
| `it_app_launches` | App läuft nach 5 s noch, keine `Barista:`-Assertion | T001 |
| `it_url_on_off` | `open barista://on` → `Barista: manual` mit PreventUserIdleSystemSleep; `off` → weg | T003 |
| `it_mode_display` | `on?display=1` → zusätzlich PreventUserIdleDisplaySleep | T003 |
| `it_invalid_url_ignored` | `on?minutes=0`, `barista://foo` → keine Assertion, App läuft | T003 |
| `it_assertion_released_on_quit` | an → `kill -TERM` → weg; neu starten, an → `kill -9` → weg | T003 |
| `it_timer_expires` | `on?minutes=1` → `Barista: timer`, nach spätestens 90 s weg | T004 |
| `it_until_time` | `on?until=<jetzt+2 min>` → `Barista: until`, nach spätestens 150 s weg | T004 |
| `it_process_exit` | `sleep 60 &`, `on?pid=$!` → `Barista: process`; Prozess beenden → nach spätestens 5 s weg | T005 |
| `it_process_already_dead` | PID eines beendeten Prozesses → keine Assertion | T005 |
| `it_activate_on_launch` | `defaults write … activateOnLaunch -bool true`, Start → `Barista: manual` | T006 |
| `it_power_guard_ac_allows` | `onACOnly` an, `on` → Assertion vorhanden (Runner hat Netzstrom/keinen Akku) | T007 |
| `it_dmg_contents` | DMG einhängen: `Barista.app` und Symlink `Applications`; `codesign --verify` ok; `LSUIElement`, URL-Schema, en/de-lproj vorhanden | T008 |

## Decisions

- **D1** — Swift 6 mit AppKit-`NSStatusItem` für die Menüleiste und SwiftUI in
  `NSWindow` für die Fenster. Verworfen wurde SwiftUI-`MenuBarExtra`: Es
  unterscheidet Links- und Rechtsklick nicht (D16) und gibt wenig Kontrolle
  über den Text neben dem Icon. Verworfen wurden auch Rust mit ObjC-Bindings,
  Objective-C und Tauri/Electron: Sie arbeiten gegen die Plattform oder sind
  für eine Menüleisten-App zu schwer. *(planner)*
- **D2** — Die gesamte Logik liegt in `BaristaCore`, plattformfrei und nur mit
  Foundation, als reiner Zustandsautomat (Ereignis → Effekte). Die App führt
  nur Effekte aus. Verworfen wurde Logik in AppKit-Controllern: Threads laufen
  unter Linux und könnten sie nicht testen. *(planner)*
- **D3** — Gebaut wird nur mit SwiftPM. `scripts/bundle.sh` erzeugt das
  `.app`-Bundle. Ressourcen liegen in `Resources/` außerhalb der SwiftPM-Targets
  und werden per `Bundle.main` geladen. Verworfen wurden ein Xcode-Projekt bzw.
  XcodeGen (zweites Buildsystem, zusätzliches Werkzeug) und SwiftPM-Ressourcen:
  `Bundle.module` legt ein Bundle ins App-Root und bricht damit die Signatur.
  *(planner)*
- **D4** — Assertions laufen über `IOPMAssertionCreateWithName` und
  `IOPMAssertionRelease` direkt, mit den Typen
  `kIOPMAssertionTypePreventUserIdleSystemSleep` bzw.
  `…PreventUserIdleDisplaySleep` und Namen nach Spec §3. Der Adapter hält nur
  die zuletzt geforderte Menge: erst neue anlegen, dann alte freigeben.
  Scheitert das Anlegen, gibt er alles frei und meldet `assertionFailed`.
  Verworfen wurde `caffeinate` als Kindprozess: verwaiste Prozesse, keine
  Fehlerrückmeldung, verstößt gegen `inv_no_shellout`. *(planner)*
- **D5** — Fail-open: Im Zweifel darf der Mac schlafen. Ein Fehler beendet die
  Phase bzw. setzt sie aus, nie umgekehrt.

  | Fall | Verhalten |
  |---|---|
  | Absturz/Beenden der App | macOS gibt die Assertions frei |
  | Assertion scheitert | Phase endet, Benachrichtigung |
  | Prozess nicht überwachbar | Phase endet, Benachrichtigung |
  | Stromquelle unbekannt bei aktivem Schutz | ausgesetzt |
  | Benachrichtigung nicht erlaubt | still keine |
  | Ungültige URL | ignoriert, `os_log` |
  | Login-Item scheitert | Schalter springt zurück, Fehlerzeile |

  Verworfen wurde fail-closed (im Zweifel wach bleiben): Ein ungewollt wacher
  Mac leert den Akku unbemerkt. *(planner)*
- **D6** — Es gibt höchstens eine Wachphase. Eine neue ersetzt die alte. Phasen
  überleben keinen Neustart der App. Bei Aussetzung zählen Deadlines weiter.
  Verworfen wurden Persistenz der Phase (Wiederaufleben nach Absturz
  widerspricht D5) und angehaltene Timer während der Aussetzung (das Ende würde
  unvorhersehbar). *(planner)*
- **D7** — Das Prozessende wird über
  `DispatchSource.makeProcessSource(identifier:eventMask: .exit)` erkannt.
  Direkt nach dem Anlegen prüft `kill(pid, 0)` die Lebendigkeit; `ESRCH` →
  `processWatchFailed`. Scheitert das Anlegen → `processWatchFailed`. Den Namen
  löst die App vor dem Befehl über `proc_name` auf (libproc). Verworfen wurden
  Polling (Latenz, CPU) und `NSRunningApplication`-Beobachtung (nur Apps, keine
  CLI-Prozesse). *(planner)*
- **D8** — Steuerung von außen nur über das URL-Scheme `barista://` nach
  Spec §4, empfangen über `NSAppleEventManager` (`kAEGetURL`). Es ist
  gleichzeitig die Testschnittstelle der Integrationstests. Verworfen wurde ein
  testeigener Hook (Code, den nur Tests nutzen). *(planner)*
- **D9** — Englisch und Deutsch kommen aus `.strings`-Dateien. Der `Localizer`
  lädt das passende `.lproj` aus `Bundle.main` gemäß `Settings.language`
  (`system` → bevorzugte Systemsprache, falls `de`, sonst `en`). Ein Wechsel
  wirkt sofort auf Menü und offene Fenster (`ObservableObject`). Verworfen wurde
  `AppleLanguages` mit Neustart (umständlich) und String Catalogs (`.xcstrings`
  braucht den Xcode-Build, D3). *(planner)*
- **D10** — Signiert wird ad-hoc (`codesign --sign -`). Die DMG entsteht mit
  `hdiutil`, das Release über einen Tag-Workflow mit `GITHUB_TOKEN`. Verworfen
  wurde Notarisierung: Sie braucht einen bezahlten Developer-Account, den es
  nicht gibt. *(planner)*
- **D11** — Keine Fremdabhängigkeiten, erzwungen durch
  `inv_no_third_party_deps`. Verworfen wurde freie Wahl mit Begründung:
  Apple-Frameworks decken alles in v1 ab, und jede Abhängigkeit wäre Wartung
  ohne Nutzen. *(planner)*
- **D12** — Getestet wird mit Swift Testing. Ausstehende Tests sind mit
  `withKnownIssue("TNNN")` markiert, Integrationstests sind Shell-Skripte mit
  `expected-failures.txt`, und XPASS zählt als Fehler (Spec §11). Verworfen
  wurden XCTest (Testnamen müssen mit `test` beginnen; kein striktes
  "erwarteter Fehlschlag") und XCUITest (braucht ein Xcode-Projekt, D3).
  *(planner)*
- **D13** — Einstellungen liegen in `UserDefaults.standard` unter den Schlüsseln
  aus Spec §6. Ungültige Werte werden pro Schlüssel durch den Standardwert
  ersetzt und nicht zurückgeschrieben. Verworfen wurde eine eigene Datei
  (UserDefaults ist der Plattformstandard, und `defaults` ist für Tests
  nutzbar). *(planner)*
- **D14** — Das Einstellungsfenster öffnet sich beim Start, außer
  `launchAtLogin` ist aktiv. Beim erneuten Öffnen der laufenden App
  (`applicationShouldHandleReopen`) öffnet es sich immer. Verworfen wurde die
  Erkennung "per Login gestartet": Mit `SMAppService` gibt es dafür kein
  verlässliches Signal. *(planner)*
- **D15** — Mindestversion ist macOS 14, der Bundle-Identifier
  `io.github.unstablent.barista`. Verworfen wurde macOS 13 (keine
  Einschränkung nötig, ältere SwiftUI-APIs). *(planner)*
- **D16** — Klickverhalten bei `leftOpensMenu`: Linksklick öffnet das Menü,
  Rechtsklick schaltet um (`toggle`). Bei `leftToggles` ist es umgekehrt.
  Umgesetzt wird es über `button.sendAction(on: [.leftMouseUp, .rightMouseUp])`
  und ein Menü, das nur bei Bedarf gesetzt wird. Verworfen wurde ein fest
  zugewiesenes `statusItem.menu`: Es fängt jeden Klick ab. *(planner)*

Coordinator-Entscheidungen werden als `(coordinator, TNNN)` ergänzt. Eine
Entscheidung wird nie geändert. Eine ersetzende Entscheidung kommt als neue
Zeile dazu, und die alte bekommt `→ replaced by Dn` angehängt. Das ist die
einzige erlaubte Änderung.

## Decision latitude

**Kriterien, in dieser Reihenfolge:** Konsistenz mit den Decisions und der
Spezifikation > fail-open (D5) > Einfachheit > Apple-Framework statt Eigenbau >
Optik.

**Vorab entschiedene Regeln:**
- **API neuer als macOS 14** → nur hinter `#available` mit funktionierendem
  macOS-14-Pfad. Gibt es keinen, wird die API nicht genutzt.
- **Swift-Version Linux** → die neueste stabile Swift-6-Version, die es als
  offizielles Docker-Image `swift:<Version>` gibt. Sie gilt für die CI.
  `Package.swift` bleibt bei Tools-Version 6.0. Weicht die Version in der
  Thread-Umgebung ab, ist das kein Blocker, solange `make core-test` dort läuft.
- **macOS-Runner** → das neueste allgemein verfügbare (nicht Beta-)
  `macos-<N>`-Label von GitHub, nicht `macos-latest`. Steht im `Result` von
  T001. Ein späterer Wechsel ist eine Entscheidung, keine Nebenänderung.
- **Unklarer Text in der Oberfläche** → kurz auf Englisch formulieren, ins
  Deutsche übersetzen, als neuer `L10nKey` in beide Dateien.
- **Unklares Layout eines Fensters** → Standard-SwiftUI-`Form` mit
  `.formStyle(.grouped)`, Reihenfolge wie in der Spezifikation, keine eigenen
  Farben oder Bilder.
- **Unklarer Standardwert** → die Variante, bei der der Mac eher schläft bzw.
  die Funktion aus ist.
- **Timing-instabiler Integrationstest** → Wartezeiten bis höchstens zum
  Doppelten der Spec-Werte erhöhen, nie Prüfungen entfernen. Im `Result`
  vermerken.
- **Verhalten nur am echten Mac prüfbar** (Aussehen einer Benachrichtigung,
  Eintrag unter Anmeldeobjekte, Akku) → wird über Protokoll und Fake bzw. feste
  Beispieldaten getestet. Im `Result` steht es unter "manuell zu prüfen".
- **Logging** → `os_log` mit Subsystem `io.github.unstablent.barista`, Kategorie
  nach Komponente (`engine`, `assertions`, `url`, `process`, `power`,
  `settings`).
- **Datei- und Typnamen**, die die Spezifikation nicht festlegt → frei, ein Typ
  pro Datei, Dateiname = Typname.

**Nie ohne den Nutzer entschieden:** Purpose, Non-goals, Definition of Done.

## External access

Siehe `docs/project/setup.md` — die einzige Liste.
