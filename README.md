# st3-Import

Konverter für Zusi-3-Streckendateien (`.st3`) in ein topologisches Knoten-Kanten-Modell.  
Ausgabe als GeoPackage (`.gpkg`).

## Funktionen

- Konvertierung von `.st3`-Dateien in ein Knoten-Kanten-Modell (GeoPackage)
- Automatische KBS-Erkennung aus dem `<UTM UTM_Zone="…">`-Element (EPSG: 32600 + Zone)
- Optionale hochpräzise DHDN↔WGS84-Transformation via BETA2007-NTv2-Gitter (`de_adv_BETA2007.tif`)
- Topologie-basierte Knotenklassifikation: `Gleisende` (Grad 1), `Weiche` (Grad ≥ 3), `Modulgrenze` (Modulverweis)
- Erkennung und Normalisierung von Weichen-Signalnamen (EW, DKW, EKW)
- Propagation von Weichenbauform-Beschriftungen über BoundingR-Radius
- Kilometrierungsinterpolation entlang Kanten aus StrElement-km-Attributen
- Import der Hüllkurve (`<Huellkurve>`) als Polygon-Layer in das GeoPackage
- Validierungswarnungen: Nulllängen-Elemente, hochgradige Vertices (Grad ≥ 5), Koordinatenabweichungen (> 5 cm), isolierte Komponenten, geschlossene Schleifen
- Interaktiver Modus und direkter CLI-Modus mit Parametern
- Persistente Konfiguration über JSON-Datei (`config/st3_converter_config.json`)

**Ausgabe pro importierter `.st3`-Datei:**

| Layer         | Geometrietyp | Inhalt                                            |
| ------------- | ------------ | ------------------------------------------------- |
| `Gleiskante`  | LineString   | Gleisabschnitte zwischen Knoten                   |
| `Gleisknoten` | Point        | Weichen, Modulgrenzen, Gleisenden                 |
| `Hüllkurve`   | Polygon      | Streckenbegrenzung (optional, aus `<Huellkurve>`) |

Die Ausgabe erfolgt als GeoPackage (`.gpkg`) in einem frei wählbaren Verzeichnis. Ist das Eingabeverzeichnis schreibgeschützt, wird automatisch das aktuelle Arbeitsverzeichnis als Standard vorgeschlagen.

## Aufbau

| Pfad / Datei                       | Inhalt                                                                                      |
| ---------------------------------- | ------------------------------------------------------------------------------------------- |
| `st3_converter.py`                 | Hauptklasse `St3Converter` (Konvertierungslogik)                                            |
| `core/cli.py`                      | CLI-Einstiegspunkt, interaktiver Modus, GeoPackage-Export                                   |
| `core/core.py`                     | Logging-System, Versionskonstante, `Config`-Klasse                                          |
| `convert/convert_envelope.py`      | Hüllkurven-Import aus dem `<Huellkurve>`-Element                                            |
| `config/st3_converter_config.json` | Persistente Einstellungen                                                                   |
| `docs/`                            | [Handbuch](docs/USER_GUIDE.md), [Technische Dokumentation](docs/TECHNICAL_DOCUMENTATION.md) |
| `requirements.txt`                 | Python-Abhängigkeiten                                                                       |
| `pyproject.toml`                   | Einstellungen für Ruff (kein installierbares Paket)                                         |
| `biome.json`                       | Linter und Formatierer für Markdown/JSON (Biome, Standardregeln)                            |

## Einrichtung

Voraussetzung: Python 3.10 oder neuer. Abhängigkeiten (`geopandas`, `lxml`, `pyproj`, `shapely`) in einer lokalen virtuellen Umgebung:

```powershell
py -m venv .venv
.venv\Scripts\python.exe -m pip install -r requirements.txt
```

Alternativ läuft der Konverter mit dem Python aus OSGeo4W, das die Abhängigkeiten bereits mitbringt. Liegt `C:\OSGeo4W\share\proj\de_adv_BETA2007.tif` vor, wird das BETA2007-Gitter automatisch verwendet.

Geprüft und formatiert wird mit [Ruff](https://docs.astral.sh/ruff/) für Python und [Biome](https://biomejs.dev/) für Markdown/JSON (in VS Code über die jeweilige Erweiterung). Von der Kommandozeile:

```powershell
ruff check .                                  # Python prüfen
ruff format --check .                         # Python-Formatierung prüfen
npx @biomejs/biome@2.6.6 check .              # Markdown/JSON prüfen
npx @biomejs/biome@2.6.6 check --write .      # sichere Korrekturen und Formatierung anwenden
```

---

## Verwendung (interaktiver Modus)

```bash
python -m core.cli
```

Das Programm führt durch die Auswahl der Eingabedatei, des Ziel-KBS und der Ausgabedatei.  
Einstellungen können im Menü dauerhaft gespeichert werden.

## Verwendung (CLI-Modus)

```bash
# Einfache Konvertierung
python -m core.cli -i Streckenmodul.st3 -o Ausgabe.gpkg

# Mit explizitem Ziel-KBS
python -m core.cli -i Streckenmodul.st3 -o Ausgabe.gpkg -e 25832

# Automatische CRS-Erkennung deaktivieren
python -m core.cli -i Streckenmodul.st3 -o Ausgabe.gpkg --no-auto-detect --fallback-epsg 31467

# Version anzeigen
python -m core.cli --version
```

**CLI-Parameter:**

| Parameter                 | Beschreibung                                                      |
| ------------------------- | ----------------------------------------------------------------- |
| `-i`, `--input`           | Pfad zur `.st3`-Eingabedatei                                      |
| `-o`, `--output`          | Pfad zur Ausgabe-GeoPackage-Datei                                 |
| `-e`, `--epsg`            | EPSG-Code des Ziel-KBS (Standard: `31467`)                        |
| `--no-auto-detect`        | Automatische CRS-Erkennung deaktivieren                           |
| `--fallback-epsg`         | Rückfall-EPSG wenn Auto-Erkennung fehlschlägt (Standard: `32632`) |
| `--no-normalize-switches` | Normalisierung von Weichenknoten-Signalnamen deaktivieren         |
| `--import-envelope`       | Hüllkurven-Polygon importieren und als Layer speichern            |
| `--no-import-envelope`    | Hüllkurven-Import deaktivieren                                    |
| `--open-log`              | Protokolldatei nach Abschluss automatisch öffnen                  |
| `-v`, `--version`         | Versionsnummer anzeigen                                           |

## Verwendung als Python-Bibliothek

```python
from st3_converter import St3Converter

converter = St3Converter(
    input_path="Strecke.st3",
    target_epsg=31467,  # Ziel-KBS: DHDN/GK Zone 3
    auto_detect_crs=True,  # UTM-Zone automatisch aus Datei lesen
    normalize_switch_names=True,  # Weichenknoten-Namen normalisieren
)
nodes_features, edges_features = converter.convert()
```

## Konfiguration (`config/st3_converter_config.json`)

| Schlüssel                | Standard | Beschreibung                                      |
| ------------------------ | -------- | ------------------------------------------------- |
| `auto_detect_crs`        | `true`   | UTM-Zone automatisch aus Datei lesen              |
| `fallback_epsg`          | `32632`  | Rückfall-KBS wenn Auto-Erkennung fehlschlägt      |
| `target_epsg`            | `31467`  | Ziel-KBS der Ausgabelayer                         |
| `normalize_switch_names` | `true`   | Weichen-Signalnamen normalisieren                 |
| `import_envelope`        | `true`   | Hüllkurven-Polygon aus `<Huellkurve>` importieren |
| `create_log_file`        | `true`   | Protokolldatei (`.log`) erstellen                 |
| `open_log_file`          | `false`  | Protokolldatei nach Abschluss öffnen              |

---

## Arbeitsweise

- `dev` ist der Entwicklungszweig, `main` erhält nur Releases.
- Änderungen entstehen in Git-Worktrees und gehen per Pull Request nach `dev`.
- Englische Bezeichner für Dateinamen, Funktionen, Klassen und Variablen (Eigennamen ausgenommen).

## Lizenz

MIT, siehe [LICENSE](LICENSE).

Copyright © 2026 Fabian Schöpflin  
Autor: Fabian Schöpflin  
Kontakt: Fabian.Schoepflin@infrageo.de
