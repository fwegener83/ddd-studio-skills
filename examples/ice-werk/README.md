# Beispiel: Eine Nacht im ICE-Werk

Ein fiktives Beispiel für die Demo der Skills. Kein internes Wissen eines
echten Werks; Begriffe und Abläufe sind bewusst vereinfacht.

## Workspace in DDD Studio

- Workspace: **ICE-Werk (Demo)**, id `1e526d29-268b-41fb-b94b-c1832bfacb8a`
- Board: **Big Picture: Eine Nacht im ICE-Werk** (31 Ereignisse, 8 Rollen,
  3 Fremdsysteme, 4 Regeln, 5 Read Models, 5 offene Fragen als Notizen)
- Modell: drei Kontexte (`instandhaltung`, `werkdisposition`,
  `werksplanung`), sechs Mappings, ein Flow
  **Nacht von Triebzug 4711: Material fehlt** (12 Slices)
- Gespeicherte Versionen zum Zurücksetzen vor der Demo:
  `demo-ausgangsstand` (Board und Modell)

Die Skills schreiben nur nach Zustimmung zurück, und dann nur Notizen oder
ein neues Board. Vor einer Demo trotzdem auf `demo-ausgangsstand`
zurücksetzen, damit keine Notizen eines Probelaufs herumliegen.

## Demo-Ablauf (Vorschlag)

1. `/ddd-studio:propose-context-cut` auf dem Board: Der Agent schlägt
   Schnitte vor und zeigt, wo die Heuristiken sich widersprechen.
2. `/ddd-studio:challenge-context-cut` auf dem modellierten Schnitt: ein
   frischer Agent greift ihn an.
3. `/ddd-studio:check-flow-data` auf dem Flow: `validate_model` meldet
   „complete, keine Verstöße“, der Skill findet trotzdem Widersprüche.

Schritt 3 ist der kürzeste und verlässlichste für eine Live-Demo.

## Was eingebaut ist

Nur für die Vorbereitung. Der Agent bekommt diese Datei in der Demo nicht.

### Board (für propose-context-cut)

- **Planung oder Disposition?** Der Gleisbelegungsplan entsteht am Vortag
  in der Werksplanung und wird nachts von der Disposition umdisponiert
  (Takt und Zuständigkeit wechseln, Notiz dazu).
- **Gespaltene Regel:** „Material fehlt kurz vor Bereitstellung →
  gefährdet“: Die 2-Stunden-Grenze setzt die Disposition, ob die Arbeit
  schaffbar ist, weiß nur die Instandhaltung.
- **Ein Wort, drei Bedeutungen:** „verfügbar“ im Lager, in der Disposition
  und in der Instandhaltung.
- **Zug, Triebzug, Fahrzeug:** Zugnummer der Betriebszentrale gegen
  Triebzugnummer im Werk, Doppeltraktion.
- **Wer stellt Fristarbeiten zurück?** Ausnahme mit Folge für die Freigabe.

### Modell (für challenge-context-cut)

- **Abhängigkeitszyklus:** `instandhaltung` → `werkdisposition`
  (Freigabe) und `werkdisposition` → `instandhaltung` (Arbeitsgleis),
  beides customer-supplier.
- **Sprachleck:** Die Disposition definiert „verfügbar“ mit Freigabe,
  Fristarbeit und Lagerbestand, also mit Wörtern zweier anderer Bereiche.
- **Gespaltene Regel:** Die Policy `material-fehlt-kurz-vor-bereitstellung`
  reagiert auf ein Ereignis der Instandhaltung und löst einen Befehl der
  Disposition aus; die Schwelle steht im Glossar der Disposition.
- **Shared Kernel ohne geteilten Kern:** `werksplanung` und
  `werkdisposition` teilen „den Gleisbelegungsplan“, aber niemand besitzt
  ihn.
- **Leerer Kontext:** `werksplanung` hat ein Glossar, aber keine
  Aggregate und keine Entscheidungen im Modell.
- „Fristarbeit“ steht in zwei Glossaren mit verschiedener Bedeutung
  (fällig gegen eingeplant): eher ein gutes Zeichen, wenn gewollt.

### Flow (für check-flow-data)

`validate_model` ist sauber. Eingebaute Widersprüche:

| # | Slice | Was nicht stimmt |
|---|---|---|
| 1 | Material fehlt | `triebzug-nr` ist 4717 statt 4711 |
| 2 | Material fehlt | `fehlmenge` 3, angefordert waren 2 |
| 3 | Bereitstellung gefährdet | `geplant-um` 06:30, sonst überall 05:30 |
| 4 | Bereitstellung gefährdet | Die Policy feuert um 01:11, obwohl bis zur Bereitstellung mehr als 2 Stunden bleiben |
| 5 | Arbeit abgeschlossen | `abgeschlossen-um` 00:20 liegt vor `begonnen-um` 00:30 (Invariante `abschluss-nach-beginn`) |
| 6 | Technische Freigabe erteilt | `offene-fristarbeiten` 1, Invariante verlangt 0 |
| 7 | Mechaniker prüft offene Schadmeldungen zum Schichtende | SM-2211 steht noch auf „offen“, obwohl sie erledigt ist |
| 8 | Disponent prüft die Bereitstellung | Status „gefährdet“, obwohl die Freigabe ihn laut Projektionsregel auf „geplant“ zurücksetzt |

Dazu eine Lücke: Die Beschreibung erzählt vom Kurier mit den Ersatzteilen,
kein Slice zeigt ihn.
