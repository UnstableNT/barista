---
copy_to: Project settings > Memory > Project instructions
copy: everything below the second '---' line
---
# Projektanweisungen — Barista

## Ziel
Barista hält einen Mac per Klick in der Menüleiste wach — unbegrenzt, für
eine Dauer, bis zu einer Uhrzeit oder bis ein Prozess endet. Ziel ist v1 gemäß
der Definition of Done in `docs/plan.md` im Repository `UnstableNT/barista`,
Branch `main`.

## Quelle der Wahrheit
- `docs/plan.md` — Meilensteine, Tasks, Spezifikation, Entscheidungen,
  Entscheidungsspielraum, Definition of Done, Merge-Methode.
- `CLAUDE.md` — wie jeder Thread arbeitet, welche Dateien er schreiben darf und
  wie er endet.
- `docs/runs/TNNN.md` — Stand und Ergebnis jedes Tasks.

Diese Anweisungen regeln nur den Ablauf. Bei einem Widerspruch gilt das
Repository auf dem Default-Branch.

## Thread-Arten
Jeder Thread bekommt seine Art, die Task-ID und das unten genannte Material. Den
Rest liest er selbst aus dem Repository.

| Art | Auftrag | Schreibt |
|---|---|---|
| Task | Task TNNN umsetzen | Code, Tests, `docs/runs/TNNN.md` |
| Retry | Task TNNN mit anderem Ansatz, neuer Branch (alter Pull Request wird geschlossen); bekommt das `Result` des Vorversuchs | wie Task |
| Review | Pull Request prüfen, mit dem Skill `task-review` | Datei in `docs/reviews/`, Log-Zeile im Run-Record |
| Fix | Must-fix-Punkte eines Reviews, auf demselben Branch | wie Task |
| Entscheidung | eine Designfrage nach dem Entscheidungsspielraum festhalten | Entscheidung, betroffene offene Tasks |
| Buchhaltung | Exit-Kriterien oder Definition of Done prüfen, Stand nachziehen | `docs/status.md`, Frontmatter von `docs/plan.md` |

Nie prüft ein Thread seine eigene Arbeit: Review-Threads sind immer frisch.

## Ablauf
1. **Strikt sequentiell.** Höchstens ein arbeitender Thread gleichzeitig. Der
   nächste Task laut Reihenfolge in `docs/plan.md` startet, wenn der vorige
   gemergt oder geparkt ist und alles aus seinem `depends_on` gemergt ist.
2. **Pro Task:** Task-Thread → Review-Thread → bei *must-fix* Fix-Thread → wieder
   Review. Punkte unter "Notes" werden nicht umgesetzt, außer der Review verlangt
   es ausdrücklich.
3. **Merge** nur bei Verdict *pass* für den aktuellen Head des Pull Requests und
   grüner CI, mit der `merge_method` aus dem Plan. Einzige Ausnahme: Pull Requests
   von Buchhaltungs-Threads, die nur `docs/status.md` und das Frontmatter von
   `docs/plan.md` ändern, werden bei grüner CI ohne Review gemergt. Sonst wird
   nichts gemergt.
4. **Meilenstein-Ende:** Nach dem letzten gemergten Task eines Meilensteins prüft
   ein Buchhaltungs-Thread die Exit-Kriterien auf dem Default-Branch, setzt
   `current_milestone` weiter und baut `docs/status.md` neu auf. Ein Fehlschlag
   wird ein Fix-Task `TNNN-fK` im selben Meilenstein — eine Korrektur, kein neuer
   Umfang.
5. **Abschluss:** Nach dem letzten Meilenstein prüft ein Buchhaltungs-Thread jeden
   Punkt der Definition of Done. Alle erfüllt → `status: done`, eine
   Abschlussnachricht an mich, Ende. Erfinde keine Folgearbeit; alles Weitere ist
   v2 und wird neu geplant.

## Wenn ein Thread *blocked* endet
- **Designfrage im Rahmen des Zwecks:** Ein Entscheidungs-Thread entscheidet nach
  "Decision latitude" in `docs/plan.md` und hält die Entscheidung als neue
  Zeile unter "Decisions" mit `(coordinator, TNNN)` fest. Im selben Pull
  Request passt er jeden noch nicht gemergten Task an, der die ersetzte
  Entscheidung nennt oder der neuen widerspricht. Gemergte Tasks und Code bleiben
  unberührt. Review, Merge, dann setzt der blockierte Task auf seinem Branch fort.
- **Fehlender Zugang** (Secret, Konto, Domain, Repository) **oder** eine Lösung,
  die Zweck, Non-goals oder Definition of Done ändern würde: Task parken.

## Wenn ein Task nicht fertig wird
- Task-Thread *over budget* → ein Retry-Thread. Endet auch er *over budget* →
  parken.
- Drei Review-Runden ohne *pass* → parken.
- Zwei erfolglose Fix-Runden nach einer Meilenstein- oder DoD-Prüfung → parken.

## Parken ist der einzige Halt
- Der Run-Record bekommt `status: parked`; ein Buchhaltungs-Thread trägt unter
  "Waiting on you" ein, was gebraucht wird.
- Einmal an mich: Task, Grund, was ich konkret tun oder entscheiden soll.
- Danach weiter mit dem nächsten Task, der nicht vom geparkten abhängt. Gibt es
  keinen: kurze Zusammenfassung, dann warten.
- In allen anderen Fällen wartest du nicht auf mich: keine Rückfragen, keine
  Bestätigungen, keine Pläne zur Freigabe. Ausnahme: eine Anweisung in meiner
  Startnachricht, nach einem bestimmten Task anzuhalten.

## Nachrichten an mich
Nur wenn ein Task geparkt wird, ein Meilenstein abgeschlossen ist (eine Zeile:
Ergebnis der Exit-Kriterien) oder das Projekt fertig ist. Keine
Fortschrittsmeldungen dazwischen.

## Für alle Threads
Arbeite nach `CLAUDE.md` und deinem Auftrag. Keine Rückfragen an mich: Was dein
Task, die Entscheidungen und `CLAUDE.md` nicht klären, beendest du als *blocked*
mit genauer Angabe, was zu entscheiden ist.

## Projektspezifisch
- Der CI-Job `macos` ist für Task-Threads die einzige Quelle für macOS-Ergebnisse.
  Ein Pull Request ist erst mergebar, wenn **beide** Jobs `core-linux` und
  `macos` auf dem aktuellen Head grün sind.
- Tags und Releases legt niemand im Lauf an; das macht der Nutzer nach `status: done`.
