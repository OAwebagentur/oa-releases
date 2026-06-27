# OA — Release / Update host (public)

This **public** repo hosts the OA auto-update manifest (`latest.json`) and the
installer artifacts. The source code lives in the **private** repo
`OAwebagentur/oa`.

The OA desktop app calls `check_for_update_command`, which fetches
`https://raw.githubusercontent.com/OAwebagentur/oa-releases/main/latest.json`.
If `version` there is newer (semver) than the running app, About shows
**"Update verfügbar: vX → Download"** (the Download button opens `url`). If not,
it shows "Du hast die neueste Version." It never errors the app.

## Eine neue Version veröffentlichen

1. Im **privaten `oa`-Repo** die Version an DREI Stellen auf denselben Wert bumpen:
   - `src-tauri/src/version.rs` → `local_version()`
   - `src-tauri/tauri.conf.json` → `version`
   - `package.json` → `version`
2. Bauen (Windows, in der MSVC-Umgebung / `vcvars64.bat`):
   ```
   npm run tauri build -- --bundles nsis
   ```
   → Installer: `src-tauri/target/release/bundle/nsis/OA_<ver>_x64-setup.exe`
3. GitHub-Release in **diesem** Repo anlegen, Installer als Asset `OA_Setup.exe`
   (stabiler, versionsunabhängiger Name) hochladen:
   ```
   gh release create v<ver> --repo OAwebagentur/oa-releases -t "OA <ver>" -n "<notes>" \
     "path\to\OA_<ver>_x64-setup.exe#OA_Setup.exe"
   ```
4. `latest.json` hier aktualisieren (version + url auf das neue Asset) und pushen:
   ```json
   { "version": "<ver>", "notes": "…",
     "url": "https://github.com/OAwebagentur/oa-releases/releases/download/v<ver>/OA_Setup.exe" }
   ```
5. Fertig — laufende OA-Apps sehen das Update beim "Nach Updates suchen".

## Optional: voll-automatisches Update (Tauri Updater v2)
Für Download+Installation im Hintergrund (statt Download-Link) zusätzlich nötig:
- Signing-Keypair: `npm run tauri signer generate` (Private Key sicher verwahren),
- `plugins.updater` in `tauri.conf.json` (Endpoint = diese `latest.json`, `pubkey`),
- signierte Builds (`TAURI_SIGNING_PRIVATE_KEY` beim `tauri build`),
- `latest.json` im Tauri-Updater-Format (mit `platforms` + Signaturen).
