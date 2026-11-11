# CLAUDE.md

st3-Import: Konverter für Zusi-3-Streckendateien (`.st3`) in ein Knoten-Kanten-Modell als GeoPackage. Einstieg `python -m core.cli` (interaktiv oder mit `-i`/`-o`), Bibliothek `st3_converter.St3Converter`. Überblick und Befehle: README.md, Bedienung: docs/USER_GUIDE.md, Algorithmus und Zielmodell: docs/TECHNICAL_DOCUMENTATION.md.

## Aufbau

- `St3Converter.convert()` läuft in 8 Schritten (XML lesen → Topologie per Union-Find über Elementseiten `(nr, "g"|"b")` → Modulgrenzen → Breakpoints/Knoten → Validierung → Kanten-Walk → Transformation → Features). Ein Vertex ist eine Union-Find-Klasse; Breakpoints sind Vertices mit Grad ≠ 2 oder Modulverweis.
- `core/cli.py` schreibt das GeoPackage (geopandas). `St3Converter` liefert nur Feature-Dicts (`geometry`, `attrs`) und braucht nur lxml/pyproj.
- Aufruf immer aus der Wurzel (`python -m core.cli`). `cli.py` entfernt `core/` aus `sys.path`, das Wurzel-`__init__.py` ergänzt den Ordner für eingebettete Aufrufe (deshalb N999-Ausnahme in `pyproject.toml`).

## Regeln

- Englische Bezeichner für Dateinamen, Funktionen, Methoden, Klassen und Variablen (Eigennamen ausgenommen). Als Eigennamen gelten Zusi-Bezeichnungen (XML-Elemente und -Attribute wie `StrElement`, `NachNorm`, `Fkt`, Richtungen `NORM`/`GEGEN`) und die Namen und Werte der GeoPackage-Ausgabe (`Gleisknoten`, `knotenname`, `typ`, `Weiche` …) – sie sind Datenschema und bleiben deutsch.
- Branches: `dev` ist der Entwicklungszweig, `main` erhält nur Releases. Nach `dev` wird nur per PR gemergt; direkte Commits nur im Ausnahmefall.
- Grundsätzlich in Git-Worktrees arbeiten. Ablage unter `.claude/worktrees/<name>` (ignoriert): `git worktree add .claude/worktrees/<name> -b <branch> origin/dev`. `EnterWorktree` scheitert hier an der Schreibweise des Laufwerks (`e:` vs. `E:`).
  Aufräumen nach dem Merge aus dem Hauptverzeichnis: `git worktree remove .claude/worktrees/<name>`, dann `git branch -D <branch>` (nach Squash-/Merge-Commit meldet `-d` „not fully merged“) und `git push origin --delete <branch>`.
- Claude wird weder in Kommentaren erwähnt noch als Co-Autor eingetragen.
- Issues und PRs dürfen in Quelltext-Kommentaren genannt werden.
- Projektsprache Deutsch (Kommentare, Docstrings, Doku, Konsolenausgaben) mit korrekten Umlauten.

## Hinweise

- Abhängigkeiten: `requirements.txt` (geopandas, lxml, pyproj, shapely). Das System-Python hat sie nicht; das OSGeo4W-Python schon, braucht aber `PYTHONHOME=C:\OSGeo4W\apps\Python312` und `C:\OSGeo4W\bin` im `PATH` (nicht mit `-I` starten – der isolierte Modus ignoriert `PYTHONHOME`).
- Ausgaben über `print` aus `core.core` (ersetzt das eingebaute `print`, leitet ins Logging): Präfix `[WARNUNG] `/`[FEHLER] `/`[DEBUG] ` bestimmt den Level, `file=sys.stderr` wird zu error.
- Ruff 0.17 prüft ohne Konfiguration bereits u. a. BLE, S, PERF, N und RUF – breite `except Exception` nur an der Programmgrenze mit `# noqa: BLE001` und Begründung.
- Linting/Formatierung/Typen: Ruff für Python, Pyright (`standard`, `pythonVersion = "3.12"` wegen QGIS; beides in `pyproject.toml`), Biome (`biome.json`, Standardregeln und -formatierung) für Markdown/JSON; `.vscode/` ist lokal und nicht versioniert. Ziel: `ruff check .`, `ruff format --check .`, `pyright` und `npx @biomejs/biome@2.6.6 check .` (Version wie `$schema` in `biome.json`) ohne Befunde; dieselben Prüfungen laufen in `.github/workflows/checks.yml` bei jeder PR. Ausnahmen nur mit Begründung im Kommentar, bei Pyright als `# pyright: ignore[<regel>]` (kein pauschales `# type: ignore`). Reine Formatierungsänderungen getrennt von inhaltlichen committen.
- Ruff, Pyright und Typpakete (`lxml-stubs`) stehen in `requirements-dev.txt`, nicht in `requirements.txt`: Der Code läuft unverändert in QGIS (übernommen nach infraqgis-external) und darf zur Laufzeit nichts davon voraussetzen. Aufruf aus der `.venv` des jeweiligen Arbeitsverzeichnisses (Worktree: eigene `.venv` anlegen): `.venv/Scripts/ruff.exe check .`, `.venv/Scripts/pyright.exe`.
- Keine automatisierten Tests. Prüfen durch Aufruf mit einer kleinen, von Hand erstellten `.st3` im Scratchpad und Vergleich der `convert()`-Ausgabe vor und nach der Änderung. Die Ausgabe ist deterministisch (Breakpoints werden sortiert durchlaufen) und lässt sich direkt vergleichen. Minimalgerüst: `<Zusi><Strecke><UTM UTM_WE="500" UTM_NS="5400" UTM_Zone="32"/><StrElement Nr="1" Anschluss="0"><g X="0" Y="0"/><b X="10" Y="0"/><NachNorm Nr="2"/></StrElement>…</Strecke></Zusi>`; Weiche = zwei `NachNorm`, Modulgrenze = `<NachNormModul><Datei Dateiname="…st3"/></NachNormModul>`, Hüllkurve = `<Huellkurve><PunktXYZ X="…" Y="…"/>…</Huellkurve>`.
- Zeilenenden LF (`.gitattributes`, `.bat`/`.cmd` CRLF). Python-Code, der Dateien schreibt, mit `newline="\n"`; `Config.save()` schreibt die JSON mit Tabs wie Biome.
- `config/st3_converter_config.json` ist versioniert und wird vom Einstellungsmenü überschrieben – lokale Änderungen nicht versehentlich committen.
- Bei Änderungen an Aufruf, Optionen oder Ausgabeattributen README.md und die Dokumente in `docs/` nachziehen.
- Markdown-Zeilenumbrüche mit zwei Leerzeichen am Zeilenende beim Bearbeiten erhalten.
