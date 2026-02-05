# GitHub Releases og Auto-Oppdatering - Oppsettguide

Denne guiden forklarer hvordan du setter opp GitHub Releases og auto-oppdatering for XLIFF RegEx Tool.

## Del 1: Forberedelser

### 1.1 Opprett GitHub Repository

1. Gå til GitHub.com og opprett et nytt repository
2. Kall det `xliff-regex-tool` (eller annet navn)
3. Sett det som public (må være public for at auto-updater skal fungere gratis)

### 1.2 Push koden til GitHub

```bash
cd /Users/havardnorjordet/Python-prosjekter/RegEx_tool
git init
git add .
git commit -m "Initial commit - version 0.4.2"
git branch -M main
git remote add origin https://github.com/havardnorjordet/xliff-regex-tool.git
git push -u origin main
```

## Del 2: Generer Signing Keys

For sikkerhet bruker Tauri kryptografiske signaturer på oppdateringer. Du må generere et nøkkelpar.

### 2.1 Generer nøkkelpar

```bash
cd gui
npm run tauri signer generate -- -w ~/.tauri/xliff-regex-tool.key
```

Dette vil:
- Opprette en privat nøkkel i `~/.tauri/xliff-regex-tool.key`
- Vise den offentlige nøkkelen i terminalen (ser ut som: `dW50cnVzdGVkIGNvbW1lbnQ6...`)

### 2.2 Oppdater tauri.conf.json

Kopier den offentlige nøkkelen og erstatt `PLACEHOLDER_WILL_BE_GENERATED` i `gui/src-tauri/tauri.conf.json`:

```json
"plugins": {
  "updater": {
    "active": true,
    "endpoints": [
      "https://github.com/havardnorjordet/xliff-regex-tool/releases/latest/download/latest.json"
    ],
    "dialog": true,
    "pubkey": "DIN_OFFENTLIGE_NØKKEL_HER"
  }
}
```

**VIKTIG**: Den private nøkkelen (`~/.tauri/xliff-regex-tool.key`) må holdes hemmelig! Ikke commit den til Git.

## Del 3: Bygg og Publiser Release

### 3.1 Bygg applikasjonen

```bash
cd gui
npm run tauri build
```

Dette vil skape:
- DMG: `src-tauri/target/release/bundle/dmg/XLIFF RegEx Tool_0.4.2_aarch64.dmg`
- App: `src-tauri/target/release/bundle/macos/XLIFF RegEx Tool.app`

### 3.2 Generer signatur for DMG

```bash
npm run tauri signer sign \
  src-tauri/target/release/bundle/dmg/XLIFF\ RegEx\ Tool_0.4.2_aarch64.dmg \
  -k ~/.tauri/xliff-regex-tool.key
```

Dette vil outputte en signatur som ser ut som: `dW50cnVzdGVkIGNvbW1lbnQ6...`

### 3.3 Opprett latest.json

Lag en fil `latest.json` med følgende innhold:

```json
{
  "version": "0.4.2",
  "notes": "Se CHANGELOG.md for full liste over endringer",
  "pub_date": "2026-01-28T12:00:00Z",
  "platforms": {
    "darwin-aarch64": {
      "signature": "SIGNATUREN_FRA_STEG_3.2_HER",
      "url": "https://github.com/havardnorjordet/xliff-regex-tool/releases/download/v0.4.2/XLIFF.RegEx.Tool_0.4.2_aarch64.dmg"
    }
  }
}
```

Erstatt:
- `SIGNATUREN_FRA_STEG_3.2_HER` med signaturen fra forrige steg
- URL-en med riktig GitHub release URL (justeres etter filnavn)

### 3.4 Opprett GitHub Release

1. Gå til GitHub repository
2. Klikk på "Releases" → "Create a new release"
3. Tag version: `v0.4.2`
4. Release title: `v0.4.2 - Standalone Distribution`
5. Description: Kopier fra CHANGELOG.md
6. Last opp filer:
   - `XLIFF RegEx Tool_0.4.2_aarch64.dmg` (omdøp gjerne til `XLIFF.RegEx.Tool_0.4.2_aarch64.dmg` - uten mellomrom)
   - `latest.json`
7. Klikk "Publish release"

## Del 4: Test Auto-Oppdatering

### 4.1 Installer den nåværende versjonen

Send DMG-en til kollega og få ham til å installere den.

### 4.2 Publiser en ny versjon

Når du har gjort endringer:

1. Oppdater versjonsnummer i:
   - `gui/package.json`
   - `gui/src-tauri/tauri.conf.json`
   - `gui/src/App.tsx`
   - `USER_GUIDE.html`
   - `CHANGELOG.md`

2. Commit og push endringene

3. Bygg ny versjon:
   ```bash
   cd gui
   npm run tauri build
   ```

4. Generer ny signatur for DMG

5. Oppdater `latest.json` med ny versjon, ny signatur og ny URL

6. Opprett ny GitHub Release med ny tag (f.eks. `v0.4.3`)

### 4.3 Kollegaen får oppdatering

Når kollegaen åpner appen:
- Den sjekker automatisk GitHub for oppdateringer ved oppstart
- Hvis ny versjon finnes, får han en dialog med spørsmål om han vil oppdatere
- Hvis han klikker "Yes", lastes ny versjon ned og installeres automatisk
- Appen restarter med ny versjon

Kollegaen kan også manuelt sjekke for oppdateringer via menyen: **Help → Check for Updates...**

## Del 5: Automatisering (valgfritt)

For å gjøre prosessen enklere kan du:

1. **Bruke GitHub Actions** til å automatisk bygge og publisere releases når du pusher en ny tag
2. **Lage et script** som oppdaterer versjonsnummer i alle filer automatisk
3. **Bruke Tauri's GitHub Action** som håndterer bygging, signering og publisering

Eksempel på GitHub Actions workflow kommer i neste oppdatering!

## Viktige Notater

- **Private nøkkelen** må aldri deles eller committes til Git
- **Versjonsnummer** må være konsistent i alle filer
- **latest.json** må alltid peke til nyeste versjon
- **GitHub repository** må være public for gratis auto-update
- **Signaturen** må genereres på nytt for hver ny DMG

## Feilsøking

### "Failed to check for updates"
- Sjekk at GitHub repository er public
- Verifiser at `latest.json` URL-en i `tauri.conf.json` er korrekt

### "Invalid signature"
- Sørg for at du bruker samme private nøkkel for alle releases
- Sjekk at signaturen i `latest.json` matcher DMG-filen

### "Update not detected"
- Sørg for at versjonsnummeret i `latest.json` er høyere enn installert versjon
- Sjekk at `latest.json` har riktig format

## Ressurser

- [Tauri Updater Documentation](https://v2.tauri.app/plugin/updater/)
- [Tauri GitHub Actions](https://github.com/tauri-apps/tauri-action)
