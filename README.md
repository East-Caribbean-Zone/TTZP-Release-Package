# TTZP PiarcoFIR

EuroScope sector package for the Piarco FIR (TTZP).

## Installing through EuroScope

EuroScope can download and update this package itself.

1. In EuroScope, open **Open SCT → Download Sector Files…**.
2. Fill in the fields below the provider list, then click **New**:
   - **Name:** `TTZP Piarco FIR - VATCAR`
   - **URL:** `https://github.com/East-Caribbean-Zone/TTZP-Release-Package/releases/latest/download/TTZP_Sector_Provider.txt`
   - **Filename:** a file in the folder where you keep this package, e.g. `C:\EuroScope\TTZP\TTZP_Sector_Provider.txt`. The package is unpacked into that folder.
3. Tick **A** on the new provider and click **Download**.
4. In **Provided Sector Files**, tick **A** on the package and click **Download**.

From then on EuroScope checks for a new version at start-up. When it finds one, it asks whether to switch your profiles and ASRs to the new sector file. Each update overwrites the package files in that folder, so keep personal changes in your own copies.

## Releasing an update

Releases are published by the **EuroScope release** GitHub Action whenever a tag is pushed. This repo is private, so the release goes to the public download repo [TTZP-Release-Package](https://github.com/East-Caribbean-Zone/TTZP-Release-Package) (set as `repository` in `tools/euroscope-release.json`), using the `RELEASE_TOKEN` secret. That token needs Contents read/write on the public repo and has to be renewed before it expires. The workflow builds the package from the tagged commit and attaches `TTZP-Package.7z` and `TTZP_Sector_Provider.txt` to the release. EuroScope only sees a new version if the sector file name changes, so every release needs a new one:

1. Commit your changes.
2. Raise the version and commit the renames:
   ```powershell
   powershell -ExecutionPolicy Bypass -File tools\EuroScopeRelease.ps1 -Bump
   ```
   This raises the last counter (`…-261101-0001` → `…-261101-0002`), renames the `.sct`/`.ese`/`.rwy` and updates every profile and ASR. It prints the commands for the next step.
3. Tag the commit and push the tag, e.g. `git tag 261101-0002` then `git push origin main 261101-0002`.

To try a build locally without publishing, run the script with `-Build`. It writes to `dist\`, which is not committed. The archive is packed with plain LZMA because EuroScope unpacks it with its bundled 7-Zip 4.65, which can't read newer formats. `.rwy` files are left out so controllers keep their runway selections.

## Updating to a new AIRAC

The provider's package is kept as its own commit, so a new AIRAC can be merged like any other branch and our changes carry over.

1. **Add the new package on a branch.** Start from the last unmodified package commit (`git log --grep "(unmodified)"`), copy the new package over it and commit it as `AIRAC XXXX package (unmodified)`.
2. **Rename on `main` first.** Rename the `.sct`, `.ese` and `.rwy` files to the new AIRAC names. Update `Settings sector` in every `.prf` and `SECTORTITLE` in every ASR that has one. Commit.
3. **Merge the package branch into `main`.** In `.ese` conflicts, take the provider's `BORDER`/`SECTORLINE` numbers (they get renumbered between AIRACs) and keep our `OWNER` lines.
4. **Re-apply our changes to provider data** if the new package still lacks them:
   - `[AIRPORT]` and `[RUNWAY]`: TGPZ, TVSB and TVSU were dropped in 2611 and re-added from 2510. They're used by `Piarco Radar.asr` and `Maurice Bishop Terminal.asr`.
5. **Check before committing:** run `tools\EuroScopeRelease.ps1 -Check`, which flags any profile or ASR still pointing at an old sector file name. Also check:
   - No conflict markers are left.
   - No broken characters (U+FFFD) in the `.sct`, `.ese` or settings files. They are windows-1252, so `·`, `©` and `ç` must survive.
   - Every ASR item still exists in the sector file, including runways. Some ASRs have an empty `SECTORTITLE`, so check those too.
   - Airports and points named in the TopSky files still exist. TAPT, for example, was removed in 2611.

## Editing

Open `.sct`, `.ese` and the `TTZP/Settings` files as **windows-1252**. Saving them as UTF-8 breaks `·`, `©` and accented characters.
