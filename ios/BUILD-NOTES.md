# Keeping the app bundle in sync with the web build

The iOS target ships the SAME files GitHub Pages serves. Do not fork them.

A **Run Script Phase** named "Stage web bundle" does this automatically, before
"Copy Bundle Resources". It copies `index.html`, `sw.js`, the icons and
`vendor/` from the repo root into `TellaVision/web`, and then **fails the build**
if the staged `APP_VERSION` does not match `MARKETING_VERSION`.

> This paragraph used to describe the phase as something to add by hand, and
> nobody had. `TellaVision/web` was a manual copy that went stale at v2.4.0 while
> the web app reached v3.4.0 — five releases. The simulator quietly ran the old
> bundle, and an archive would have shipped it. The phase is now in the project,
> and the version check is there so this fails loudly instead of silently.

`TellaVision/web` is in the project as a **folder reference** (blue folder, not
yellow group) so the directory structure is preserved — the custom scheme
handler resolves paths relative to it.

`TellaVision/web` is committed even though the build regenerates it, so a fresh
clone builds byte-identically without first running the web build. It is
generated output: never hand-edit it, and expect it in the diff of every
release.

If the build stops with `web bundle is X but MARKETING_VERSION is Y`, rebuild
`index.html` from `tellavision.tsx` or fix the version — do not edit the staged
copy, it is overwritten every build.

## Release checklist

1. Rebuild `index.html` from `tellavision.tsx` (see `PUBLISH-TO-FIELDKIT.md`).
2. Confirm the health badge: self-tests, sweep `0 failing`, audit `0 unbacked`.
3. Bump `APP_VERSION` and `MARKETING_VERSION` together, and increment
   `CURRENT_PROJECT_VERSION`. These drifted four releases once (the project sat
   at 2.4.0 while the web app shipped 2.8.0) because nothing checked. Verify
   from the repo root — it prints nothing when they agree:

   ```bash
   diff <(grep -o 'APP_VERSION = "[0-9.]*"' tellavision.tsx | head -1 | grep -o '[0-9][0-9.]*') <(grep -o 'MARKETING_VERSION = [0-9.]*' ios/TellaVision.xcodeproj/project.pbxproj | head -1 | grep -o '[0-9][0-9.]*')
   ```

   `project.pbxproj` is hand-maintained; run `plutil -lint` on it after any edit.
4. Archive → Distribute → App Store Connect.

## Signing and archiving for the App Store

Team **UP866Y2MP2** (CLIFFORD RYAN BRAVO). `DEVELOPMENT_TEAM` is set on both
Debug and Release; `CODE_SIGN_STYLE` is Automatic.

```bash
# 1. archive (Release, generic iOS device)
xcodebuild -project ios/TellaVision.xcodeproj -scheme TellaVision \
  -configuration Release -destination 'generic/platform=iOS' \
  -archivePath /tmp/tv-build/TellaVision.xcarchive archive -allowProvisioningUpdates

# 2. export a signed App Store .ipa
xcodebuild -exportArchive -archivePath /tmp/tv-build/TellaVision.xcarchive \
  -exportOptionsPlist ios/ExportOptions.plist \
  -exportPath /tmp/tv-build/export -allowProvisioningUpdates
```

The archive is signed with **Apple Development** and that is correct — automatic
signing re-signs with **Apple Distribution** during export. Verify the export,
not the archive:

```bash
codesign -dv --verbose=2 Payload/TellaVision.app   # Authority: Apple Distribution ...
```

### Three traps, all hit once

- **Do not set `CODE_SIGN_IDENTITY = "Apple Distribution"` on Release.** It looks
  like the fix for an archive that signs as Development. It is not: automatic
  signing builds with a development identity by design, and the override fails
  the build with "conflicting provisioning settings". Distribution is an *export*
  concern.
- **The device must be registered in the portal before anything signs.** Even an
  App Store archive needs a development profile during the build step, and a
  development profile needs at least one device on the team. `xcodebuild
  -allowProvisioningUpdates` can only register it for you when Xcode is signed in
  to the account; otherwise add the UDID by hand at
  developer.apple.com/account/resources/devices/add.
- **Developer Mode must be on on the device** (Settings → Privacy & Security →
  Developer Mode → restart). Until then the device reports `connected (no DDI)`
  and every destination lookup times out rather than failing with a clear reason.

Registered device: iPad Pro 11-inch (2nd gen), UDID `00008027-0005041C0C78402E`.
