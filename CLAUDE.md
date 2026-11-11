# CLAUDE.md

st3-Import: Konverter für Zusi-3-Streckendateien (`.st3`) in ein Knoten-Kanten-Modell als GeoPackage. Einstieg `python -m core.cli` (interaktiv oder mit `-i`/`-o`), Bibliothek `st3_converter.St3Converter`. Überblick und Befehle: README.md, Bedienung: docs/USER_GUIDE.md, Algorithmus und Zielmodell: docs/TECHNICAL_DOCUMENTATION.md.

## Regeln

- Englische Bezeichner für Dateinamen, Funktionen, Methoden, Klassen und Variablen (Eigennamen ausgenommen). Als Eigennamen gelten Zusi-Bezeichnungen (XML-Elemente und -Attribute wie `StrElement`, `NachNorm`, `Fkt`, Richtungen `NORM`/`GEGEN`) und die Namen und Werte der GeoPackage-Ausgabe (`Gleisknoten`, `knotenname`, `typ`, `Weiche` …) – sie sind Datenschema und bleiben deutsch.
- Branches: `dev` ist der Entwicklungszweig, `main` erhält nur Releases. Nach `dev` wird nur per PR gemergt; direkte Commits nur im Ausnahmefall.
- Grundsätzlich in Git-Worktrees arbeiten. Ablage unter `.claude/worktrees/<name>` (ignoriert): `git worktree add .claude/worktrees/<name> -b <branch> origin/dev`. `EnterWorktree` scheitert hier an der Schreibweise des Laufwerks (`e:` vs. `E:`).
  Aufräumen nach dem Merge aus dem Hauptverzeichnis: `git worktree remove .claude/worktrees/<name>`, dann `git branch -d <branch>`.
- Claude wird weder in Kommentaren erwähnt noch als Co-Autor eingetragen.
- Issues und PRs dürfen in Quelltext-Kommentaren genannt werden.
- Projektsprache Deutsch (Kommentare, Docstrings, Doku, Konsolenausgaben) mit korrekten Umlauten.

## Hinweise

- Abhängigkeiten: `requirements.txt` (geopandas, lxml, pyproj, shapely). Das System-Python hat sie nicht; das OSGeo4W-Python schon, braucht aber `PYTHONHOME=C:\OSGeo4W\apps\Python312` und `C:\OSGeo4W\bin` im `PATH`.
- Linting/Formatierung: Ruff für Python (Einstellungen in `pyproject.toml`), Biome (`biome.json`, Standardregeln und -formatierung) für Markdown/JSON; `.vscode/` ist lokal und nicht versioniert. Ziel: `ruff check .`, `ruff format --check .` und `npx @biomejs/biome@2.6.6 check .` (Version wie `$schema` in `biome.json`) ohne Befunde (kein CI, lokal vor dem Commit prüfen). Ausnahmen nur mit Begründung im Kommentar. Reine Formatierungsänderungen getrennt von inhaltlichen committen.
- Ruff ist nicht in `requirements.txt` und nicht im `PATH`; aufrufen über die VS-Code-Erweiterung: `$(ls ~/.vscode/extensions/charliermarsh.ruff-*/bundled/libs/bin/ruff.exe | tail -1) check .`.
- Keine automatisierten Tests. Prüfen durch Aufruf mit einer kleinen, von Hand erstellten `.st3` im Scratchpad und Vergleich der `convert()`-Ausgabe vor und nach der Änderung. Die Ausgabe ist deterministisch (Breakpoints werden sortiert durchlaufen) und lässt sich direkt vergleichen.
- Zeilenenden LF (`.gitattributes`, `.bat`/`.cmd` CRLF). Python-Code, der Dateien schreibt, mit `newline="\n"`; `Config.save()` schreibt die JSON mit Tabs wie Biome.
- `config/st3_converter_config.json` ist versioniert und wird vom Einstellungsmenü überschrieben – lokale Änderungen nicht versehentlich committen.
- Bei Änderungen an Aufruf, Optionen oder Ausgabeattributen README.md und die Dokumente in `docs/` nachziehen.
- Markdown-Zeilenumbrüche mit zwei Leerzeichen am Zeilenende beim Bearbeiten erhalten.
