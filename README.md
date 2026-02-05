# XLIFF Regex Tool

Desktop-verktøy for Find & Replace med regex direkte på XLIFF-filer.

## Funksjonalitet

- **XLIFF-parsing**: Støtte for XLIFF, MQXLIFF, SDLXLIFF
- **Regex Find & Replace**: Full regex-støtte med tag-preservering
- **Automatisk backup**: Sikkerhetskopi før endringer
- **Xbench-integrasjon**: Les checklist-filer (.xbckl XML)
- **GUI**: Desktop-grensesnitt (planlagt: Tauri/Electron)

## Arkitektur

```
RegEx_tool/
├── src/
│   ├── parsers/        # XLIFF og Xbench parsers
│   ├── regex_engine/   # Regex motor med tag-preservering
│   ├── backup/         # Backup-funksjonalitet
│   └── cli.py          # CLI for testing
├── tests/              # Unit tests
└── samples/            # Sample XLIFF-filer for testing
```

## Installasjon

```bash
# Opprett virtuelt miljø
python3 -m venv venv
source venv/bin/activate  # På Windows: venv\Scripts\activate

# Installer avhengigheter
pip install -r requirements.txt
```

## Bruk

### Vis XLIFF-statistikk

```bash
python src/cli.py stats samples/sample.xliff
```

### Søk etter pattern

```bash
# Søk i target-segmenter (standard)
python src/cli.py find samples/sample.xliff "pattern"

# Søk i source-segmenter
python src/cli.py find samples/sample.xliff "pattern" --source

# Case-sensitive søk
python src/cli.py find samples/sample.xliff "Pattern" --case-sensitive

# Søk etter e-postadresser
python src/cli.py find samples/sample.xliff "\w+@\w+\.\w+"

# Søk etter flere mellomrom
python src/cli.py find samples/sample.xliff "\s{2,}"

# Lagre søket til biblioteket hvis det er nyttig
python src/cli.py find samples/sample.xliff "\s{2,}" --save
```

### Replace med regex

```bash
# Erstatt pattern i target-segmenter
python src/cli.py replace samples/sample.xliff "gammelt" "nytt"

# Normaliser flere mellomrom til ett
python src/cli.py replace samples/sample.xliff "\s{2,}" " "

# Lagre til ny fil
python src/cli.py replace input.xliff "pattern" "replacement" --output output.xliff

# Uten backup (ikke anbefalt)
python src/cli.py replace input.xliff "pattern" "replacement" --no-backup

# Maksimum antall erstatninger per segment
python src/cli.py replace input.xliff "pattern" "replacement" --max-replacements 1

# Bruk backreferences
python src/cli.py replace input.xliff "(\w+)@(\w+)" "\\1 at \\2"

# Lagre replacement til biblioteket hvis nyttig
python src/cli.py replace input.xliff "pattern" "replacement" --save
```

### Håndter backups

```bash
# List backups
python src/cli.py backup list samples/sample.xliff

# Restore backup
python src/cli.py backup restore samples/sample.xliff --backup samples/.backups/sample_20231224_120000.xliff

# Cleanup gamle backups (behold siste 10)
python src/cli.py backup cleanup samples/sample.xliff --keep 10
```

### Parse Xbench checklist

```bash
# Parse og vis statistikk
python src/cli.py xbench checklist.xbckl

# Export regex patterns
python src/cli.py xbench checklist.xbckl --export
```

### Pattern Library

Verktøyet kommer med et innebygd bibliotek av vanlige regex-mønstre for oversettelse/lokalisering.

```bash
# List alle tilgjengelige patterns
python src/cli.py patterns list

# List kun aktive patterns
python src/cli.py patterns list --enabled

# Vis kategorier
python src/cli.py patterns categories

# Søk etter patterns
python src/cli.py patterns search --query "typo"

# Vis detaljer for et pattern
python src/cli.py patterns show --name "Multiple spaces"

# Bruk et pattern fra biblioteket
python src/cli.py patterns apply --name "Multiple spaces" --file input.xliff

# Legg til egendefinert pattern
python src/cli.py patterns add \
  --name "My Pattern" \
  --pattern "regex_here" \
  --replacement "replacement_here" \
  --description "What it does" \
  --category "Custom"

# Fjern pattern
python src/cli.py patterns remove --name "My Pattern"
```

**Innebygde kategorier:**
- **Whitespace**: Multiple spaces, leading/trailing spaces, space before punctuation
- **Punctuation**: Double periods, double commas, etc.
- **Typos**: Common typos (teh→the, recieve→receive, etc.)
- **Norwegian**: Norwegian-specific patterns (quotes, dates, etc.)
- **URLs & Emails**: Find email addresses and URLs
- **Tags & Markup**: Find unmatched brackets/parentheses
- **Consistency**: Ensure consistent terminology

## Eksempler

### Normaliser mellomrom

```bash
python src/cli.py replace input.xliff "\s{2,}" " "
```

### Rett vanlige skrivefeil

```bash
python src/cli.py replace input.xliff "teh\b" "the"
```

### Konverter datoformat (DD.MM.YYYY til MM/DD/YYYY)

```bash
python src/cli.py replace input.xliff "(\d{2})\.(\d{2})\.(\d{4})" "\\2/\\1/\\3"
```

### Fjern unødvendige punktum

```bash
python src/cli.py replace input.xliff "\.\." "."
```

## Neste steg

- [ ] GUI med Tauri eller Electron
- [ ] Batch-prosessering av flere filer
- [ ] Preview av endringer før apply
- [x] Regex pattern library med built-in patterns
- [ ] Unit tests
- [ ] Import patterns fra Xbench til library
- [ ] Export library til andre formater
