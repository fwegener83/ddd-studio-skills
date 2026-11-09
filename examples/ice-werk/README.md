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
  `demo-ausgangsstand-v2` (Board) und `demo-ausgangsstand` (Modell)

Die Skills schreiben nur nach Zustimmung zurück, und dann nur Notizen oder
ein neues Board. Vor einer Demo trotzdem auf diese Versionen
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

Nicht eingebaut, aber zu Recht gefunden (Testlauf 2026-11-09): Keine
Projektionsregel der Bereitstellungsübersicht legt eine Zeile an, und kein
Slice plant die Bereitstellung. Spielt man nur den Flow ab, bleibt die
Übersicht leer.

## Testläufe

Am 2026-11-09 hat je ein frischer Agent jeden Skill gegen diesen Workspace
laufen lassen, ohne diese Datei zu kennen:

- `propose-context-cut`: alle fünf Konflikte des Boards gefunden, drei
  Schnitte und ein verworfener; 5 Aufrufe, etwa 4,5 Minuten.
- `challenge-context-cut`: alle eingebauten Schwächen gefunden, dazu
  zwei Folgefehler (die Freigabe hebt „gefährdet“ der Disposition auf;
  Betriebszentrale fehlt auf der Context Map); 21 Aufrufe, etwa 5 Minuten.
- `check-flow-data`: alle acht Fehler und die Lücke gefunden, dazu die
  fehlende Herkunft der Bereitstellung; 41 Aufrufe, knapp 10 Minuten.
  Danach wurde der Skill auf weniger Aufrufe getrimmt.

Für eine Live-Demo ist das zu lang zum Zuschauen. Besser: den Lauf vorher
aufnehmen oder live starten und während der Wartezeit erklären, was er
prüft.
