# 02: Compile iOS 12 Legacy Receiver and build unsigned IPA

**What to build:** Build the iOS 12–compatible OpenDisplay legacy receiver from source using direct compiler invocation (`swiftc`, `actool`, `ibtool`) targeting `arm64-apple-ios12.2`, and assemble the resulting application bundle into an unsigned `.ipa` package ready for sideloading.

**Blocked by:** None (can start immediately)

**Status:** resolved

- [x] Source files for the legacy receiver (`iOSLegacy` UIKit fork) are cloned into the workspace.
- [x] Compiler toolchain executes without error against the iOS 12 SDK floor, producing a compiled `OpenDisplayLegacy.app` bundle.
- [x] Asset catalog (`Assets.car`) and launch storyboard (`LaunchScreen.storyboardc`) compile with icons and launch configuration.
- [x] Binary inspection verifies a 64-bit ARM architecture (`arm64`) with `MinimumOSVersion` set to `12.2`.
- [x] The app bundle is packaged into an unsigned archive at `build/legacy/OpenDisplayLegacy.ipa`.

## Comments

### Verification Summary
1. **Source Ingestion**:
   - Cloned `Francesco149/opendisplay` at branch `ios12-legacy-receiver` into workspace.
   - Preserved workspace documentation and agent configuration.

2. **Compilation**:
   - Invoked `./iOSLegacy/build.sh --unsigned` driving `swiftc` targeting `arm64-apple-ios12.2` with SDK `iphoneos` (`26.2`).
   - Swift flags `-O -wmo -D OPENDISPLAY_LEGACY` compiled all UIKit legacy shell files alongside upstream `StreamReceiver`, `Protocol`, and `DecodeBudget`.

3. **Assets & Storyboards**:
   - `actool` compiled `iOS/Assets.xcassets` into `build/legacy/OpenDisplayLegacy.app/Assets.car` and icon assets (`AppIcon76x76@2x~ipad.png`, `AppIcon60x60@2x.png`).
   - `ibtool` compiled `iOSLegacy/LaunchScreen.storyboard` into `build/legacy/OpenDisplayLegacy.app/LaunchScreen.storyboardc`.

4. **Binary & Metadata Inspection**:
   - `file`: `Mach-O 64-bit executable arm64`.
   - `otool -l`: `LC_BUILD_VERSION platform 2 minos 12.2 sdk 26.2`.
   - `Info.plist`: `MinimumOSVersion = 12.2`, `CFBundleIdentifier = com.peetzweg.opensidecar.ios.legacy`.

5. **IPA Packaging**:
   - Enhanced `iOSLegacy/build.sh` to package unsigned IPAs when `--unsigned` is specified.
   - Archive created at `build/legacy/OpenDisplayLegacy.ipa` containing `Payload/OpenDisplayLegacy.app` (371 KB compressed).

