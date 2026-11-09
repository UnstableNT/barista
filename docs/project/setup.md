---
for: the user, before the first run
project: Barista
repository: UnstableNT/barista
tasks: 8 (+ je Task mindestens ein Review-Thread)
---

# Setup — Barista

Stand der Klickpfade: Claude Code Projects, Public Beta (Oktober 2026). Weicht
etwas ab, schau unter https://code.claude.com/docs/en/claude-projects nach.

## Einmalig (falls noch nicht erledigt)

- [ ] GitHub-Konto unter claude.ai/code verbunden.
- [ ] Claude GitHub App installiert und für `UnstableNT/barista` freigegeben
      (https://github.com/apps/claude).

## Repository

- [x] Repository `UnstableNT/barista` angelegt, öffentlich, Default-Branch
      `main`. Es ist das einzige Repo im Projekt.
- [x] Handoff-Dateien als `docs: handoff` auf `main` committet (vom Planer
      erledigt).
- [ ] **Settings > Actions > General:** "Allow all actions and reusable
      workflows" aktiv. Unter "Workflow permissions" darf der
      Release-Workflow (T008) per `permissions: contents: write` Schreibrechte
      anfordern. Die Voreinstellung "Read repository contents" genügt dafür,
      solange es keine Organisationsrichtlinie gibt, die das verbietet.

## Externer Zugang

Das ist die einzige Liste dafür. Fehlt hier etwas, wird der betroffene Task
geparkt.

- [ ] **Secrets / Umgebungsvariablen:** keine. Die App wird ad-hoc signiert;
      der Release-Workflow nutzt nur das automatische `GITHUB_TOKEN`.
- [ ] **Konten / Domains / Dienste:** keine außer GitHub. Kein Apple-Konto
      nötig.

## Cloud-Umgebung

Unter **Project settings > Environment** eine Umgebung wählen oder anlegen mit:

- [ ] **Setup-Skript**, das Swift 6 unter Linux installiert. Anforderung: Nach
      dem Skript liefert `swift --version` in einer neuen Shell Swift 6.x, und
      `swift test` läuft. Vorschlag:

      ```bash
      #!/usr/bin/env bash
      set -euo pipefail
      if ! command -v swift >/dev/null 2>&1; then
        export DEBIAN_FRONTEND=noninteractive
        sudo apt-get update
        sudo apt-get install -y binutils git gnupg2 libc6-dev libcurl4-openssl-dev \
          libedit2 libgcc-13-dev libncurses-dev libpython3-dev libsqlite3-0 \
          libstdc++-13-dev libxml2-dev libz3-dev pkg-config tzdata unzip zlib1g-dev
        cd /tmp
        curl -fsSLO "https://download.swift.org/swiftly/linux/swiftly-$(uname -m).tar.gz"
        tar zxf "swiftly-$(uname -m).tar.gz"
        ./swiftly init --assume-yes --quiet-shell-followup
        . "${SWIFTLY_HOME_DIR:-$HOME/.local/share/swiftly}/env.sh"
        sudo ln -sf "$(dirname "$(command -v swift)")"/* /usr/local/bin/
      fi
      swift --version
      ```

      Ich konnte das Skript nicht selbst ausführen, weil der Planer-Sandbox der
      Zugriff auf swift.org fehlt. Der empfohlene Start mit T001 allein prüft
      es. Scheitert es, endet T001 als *blocked*, und du passt nur dieses
      Skript an. Läuft die Umgebung ohne `sudo` als root, lass `sudo` weg.
- [ ] **Netzwerkzugriff** zusätzlich zu den Standard-Registries:
      `download.swift.org`, `swift.org`.
- [ ] Secrets und Variablen: keine.

## Kosten außerhalb von Claude

Keine. Das Repo ist öffentlich; Standard-Runner von GitHub, auch macOS, sind
dafür kostenlos.

## Projekt anlegen

- [ ] **Projects > New project**
  - Name: `Barista`
  - Goal: `v1 nach docs/plan.md fertigstellen`
  - Context: Repository `UnstableNT/barista`
- [ ] **Project settings > Memory > Project instructions:** Inhalt von
      `docs/project/instructions.md` ab der zweiten `---`-Zeile einfügen.
- [ ] **Project settings > General:** Thread-Modell und -Effort prüfen. Für T001
      und T002 (je `review: deep`) lohnt das stärkste Modell, weil jeder
      spätere Task auf ihnen aufbaut.
- [ ] Desktop-Benachrichtigungen an lassen. Darüber erfährst du von geparkten
      Tasks.

## Start

**Empfohlen — erst das Skelett, dann der Rest:**

1. Startnachricht:

       Starte das Projekt nach den Projektanweisungen. Führe T001 inklusive
       Review aus und halte danach an, damit ich das Ergebnis prüfe.

2. Pull Request von T001 ansehen:
   - Laufen beide CI-Jobs?
   - Zeigt `core-linux` die `core_*`-Tests als erwartete Fehlschläge?
   - Zeigt `macos` die Tabelle aus `scripts/integration.sh` mit
     `it_app_launches` als PASS und dem Rest als XFAIL?
   - Hat der Thread das `macos-<N>`-Label und die Swift-Version im `Result`
     von `docs/runs/T001.md` vermerkt?
3. Im Repository unter **Settings > Branches** (bzw. **Rules > Rulesets**) für
   `main` einstellen:
   - Status Checks `core-linux` und `macos` müssen grün sein. Die Namen gibt es
     erst nach dem ersten CI-Lauf.
   - Direktes Pushen ist verboten.
4. Freigabe:

       T001 passt. Arbeite ab jetzt alle Tasks bis zur Definition of Done ab,
       ohne auf mich zu warten — außer in den Fällen aus den Projektanweisungen.

**Alternative — alles auf einmal:** nur die Nachricht aus Schritt 4 senden, ohne
"T001 passt.". Schritt 3 trotzdem erledigen, sobald der erste CI-Lauf existiert.

## Verbrauch

8 Tasks → mindestens 16 Threads. Dazu kommen Fix-Runden, Entscheidungen, vier
Meilenstein-Prüfungen und die Abschlussprüfung, realistisch also 25–30
Threads. Ein Projekt zieht deutlich schneller an den Plan-Limits als einzelne
Sessions. Erreicht ein Thread das Limit, wartet er und macht nach dem Reset
selbst weiter. Den Verbrauch zeigt **Project settings > Usage**; anhalten
kannst du über **Project settings > General > Pause**.

## Nach `status: done`

- DMG aus dem letzten CI-Lauf auf `main` laden und auf deinem Mac
  ausprobieren. Was im `Result` der Run-Records unter "manuell zu prüfen"
  steht, ist dort gesammelt.
- Release erstellen: `git tag v1.0.0 && git push origin v1.0.0`. Der
  Release-Workflow baut die DMG und hängt sie an.

## Was du während des Laufs siehst

- **Waiting on you** in der Projektübersicht: nur geparkte Tasks.
- **`docs/status.md`**: Überblick, nach jedem Meilenstein und jedem Parken neu
  aufgebaut.
- **`docs/runs/`**: Ergebnis und Log jedes Tasks. **`docs/reviews/`**: jede
  Review-Runde.
- **Pull Requests**: einer pro Task, einer pro Entscheidung.
