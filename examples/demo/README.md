# Demo: drei kleine Workspaces, einer pro Skill

Fiktive, bewusst kleine Beispiele aus einem ICE-Werk. Sie zeigen den
Mechanismus, nicht die Domäne: Der Agent liest das Modell, hängt seine
Befunde als Notizen dorthin, wo sie hingehören, und legt Alternativen als
Vorschläge zur Abstimmung darunter. Menschen stimmen ab und entscheiden.

Alle drei Workspaces enthalten im Ausgangszustand **keine Notizen**. Was
nach dem Lauf dasteht, kommt vom Agenten (erste Zeile
`Agent · <skill> · <art>`, in Studio als „via Agent“ markiert).

| Workspace | Skill | Was drin ist |
|---|---|---|
| **Demo 1 · Big Picture (Schnitt finden)**, id `5da35318-40fd-4788-bc16-9bd24c955b5a` | `propose-context-cut` | Ein Board „Big Picture: Eine Nacht im ICE-Werk“: 14 Ereignisse, 7 Rollen, 1 Fremdsystem, 1 Regel, 1 Read Model, alle verbunden. Kein Modell. |
| **Demo 2 · Context Map (Schnitt prüfen)**, id `c1643f40-ec82-461b-9e51-6c110bf1e0b4` | `challenge-context-cut` | Zwei Kontexte (`instandhaltung`, `werkdisposition`) mit Glossar, ein Fremdsystem, drei Mappings, eine Policy, ein Read Model. |
| **Demo 3 · Flow (Daten prüfen)**, id `69ed10f6-855d-4301-bace-bebe80c9a959` | `check-flow-data` | Ein Kontext, drei Aggregate, ein Flow „Nacht von Triebzug 4711“ mit 6 Schritten. `validate_model` meldet: vollständig, 0 Verstöße. |

## Vor der Demo zurücksetzen

Gespeicherte Versionen in Studio enthalten **keine Notizen**. Deshalb in
dieser Reihenfolge:

1. `/ddd-studio:clear-agent-notes` für jeden der drei Workspaces. Der Skill
   löscht nur Notizen mit dem Agenten-Marker.
2. Falls am Modell oder Board etwas verändert wurde: in Studio auf die
   Version `demo-start` zurückgehen.

## Ablauf in der Demo

1. Workspace in Studio öffnen und zeigen: leer, keine Notizen.
2. In Claude Code den Skill starten, zum Beispiel:
   „/ddd-studio:check-flow-data Prüf den Flow ‚Nacht von Triebzug 4711‘ im
   Workspace ‚Demo 3 · Flow (Daten prüfen)‘.“
3. Während der Agent arbeitet, erzählen, was er prüft.
4. In Studio zeigen: Die Notizen erscheinen live, mit Vorschlägen darunter.
   Einen Vorschlag mit 👍 bewerten und die Notiz als „übernommen“
   entscheiden. Das kann nur ein Mensch.

## Was eingebaut ist

Nur für die Vorbereitung. Der Agent bekommt diese Datei in der Demo nicht
zu sehen.

**Demo 1 (Board):**
- Takt und Zuständigkeit wechseln: „Gleisbelegungsplan für die Nacht
  erstellt“ (Vortag, Werksplanung) gegen „Gleis kurzfristig umdisponiert“
  (nachts, Disposition).
- „verfügbar“ mit zwei Bedeutungen: im Lager „liegt im Regal“, in der
  Disposition „einsatzbereit für den Umlauf“.
- Geteilte Regel: „Material fehlt kurz vor Bereitstellung → gefährdet“. Die
  Schwelle setzt die Disposition, beurteilen kann es nur die Instandhaltung.

**Demo 2 (Context Map):**
- Zyklus: `instandhaltung` → `werkdisposition` (Freigabe) und zurück
  (Arbeitsgleis), beides customer-supplier.
- Sprachleck: „verfügbar“ in der Disposition ist mit Freigabe, Fristarbeit
  und Lagerbestand definiert, also mit Wörtern der Instandhaltung und des
  Lagers.
- Geteilte Regel: Die Policy `material-fehlt-kurz-vor-bereitstellung`
  reagiert auf ein Ereignis der Instandhaltung und löst einen Befehl der
  Disposition aus; die Schwelle hat die Disposition gesetzt.
- Was hält: die Übersetzung zum Materialwirtschaftssystem.

**Demo 3 (Flow):**

| Schritt | Was nicht stimmt |
|---|---|
| Arbeit abgeschlossen | 00:20, begonnen wurde erst um 00:30 (Invariante `abschluss-nach-beginn`) |
| Schadmeldung erledigt | Triebzug 4717 statt 4711 |
| Ingenieurin prüft offene Schadmeldungen | SM-2211 steht noch auf „offen“, obwohl sie erledigt ist |
| Technische Freigabe erteilt | 1 offene Fristarbeit, die Invariante verlangt 0 |

## Testläufe (09.11.2026)

Je ein frischer Agent ohne diese Datei, mit Veröffentlichen:

| Skill | Ergebnis | Dauer |
|---|---|---|
| `propose-context-cut` | 5 Notizen, 3 Schnitte als Vorschläge; alle drei Signale gefunden | ca. 3 Min., 15 Aufrufe |
| `challenge-context-cut` | 6 Notizen (3 blockierend, 2 sollte behoben werden, 1 was hält), 10 Vorschläge | ca. 4 Min., 40 Aufrufe |
| `check-flow-data` | 4 Notizen, 7 Vorschläge; alle vier Fehler gefunden | ca. 3 Min., 28 Aufrufe |
| `clear-agent-notes` | im Workspace „ICE-Werk (Demo)“: 2 Agenten-Notizen gelöscht, 5 andere behalten | 7 Aufrufe |

Die Notizen dieser Läufe stehen noch in den drei Demo-Workspaces. Vor der
Demo mit `clear-agent-notes` entfernen.
