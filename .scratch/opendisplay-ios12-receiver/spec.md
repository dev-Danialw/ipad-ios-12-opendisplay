# Spec: OpenDisplay Receiver for iOS 12.5.8 iPad Air

Status: resolved

## Problem Statement

The user has an iPad Air (1st generation) running iOS 12.5.8 connected via USB to a MacBook. The official OpenDisplay receiver app on the App Store enforces a minimum deployment target of iOS 15, and modern versions of Xcode refuse to target iOS 12. Consequently, the user cannot install the official OpenDisplay receiver directly from the App Store or build standard Xcode projects for this older device. The user wants to repurpose this iPad as a functional, low-latency second display for their Mac over USB without having to purchase third-party subscription software or specialized hardware dongles.

## Solution

Build and deliver an iOS 12–compatible build of the OpenDisplay receiver (`OpenDisplayLegacy`) packaged as a sideload-ready `.ipa`. The receiver ports the core video decoding and touch event pipeline to a UIKit application shell while enforcing an Apple A7 hardware decode budget (~42 fps at native 2048×1536 resolution). The application is compiled via a direct compiler pipeline (`swiftc`, `actool`, `ibtool`) that bypasses Xcode IDE version constraints. The user installs the resulting IPA via Sideloadly (or a jailbreak-assisted installer) onto the iPad, and pairs it with the official macOS `OpenDisplay` sender running on the MacBook over a USB Lightning cable using `usbmuxd`.

## User Stories

1. As a Mac user with an older iPad, I want to compile an iOS 12–compatible receiver application, so that I can run OpenDisplay on an iPad Air 1 stuck on iOS 12.5.8.
2. As a user, I want the compiled receiver to be packaged into a standard `.ipa` file, so that I can easily sideload it onto my connected device using Sideloadly or an on-device package installer.
3. As a user connecting over a Lightning cable, I want the receiver to establish a local socket connection via USBMux, so that screen streaming has lower latency and is immune to Wi-Fi jitter.
4. As a user on a Retina iPad panel, I want the sender to stream at native 2048×1536 resolution, so that Mac desktop windows, text, and graphics appear sharp and unblurred.
5. As an owner of an Apple A7 device, I want the receiver to automatically enforce an H.264 decode budget, so that the hardware decoder does not experience buffer overflows, frame drops, or decoder stalls.
6. As a user rotating my iPad between portrait and landscape orientations, I want the receiver to signal orientation changes back to the sender, so that the macOS virtual display adjusts its dimensions and aspect ratio automatically.
7. As a touchscreen user, I want single-finger taps on the iPad display to register as mouse clicks on macOS, so that I can click buttons and select items directly on the secondary display.
8. As a touchscreen user, I want dragging on the iPad display to map to mouse drag events on macOS, so that I can move windows and drag objects smoothly.
9. As a touchscreen user, I want two-finger drag gestures to translate to smooth scroll events on macOS, so that reading documents and web pages feels natural.
10. As a user monitoring connection quality, I want the receiver to report frame stats, decode latency, and drop counts back to the sender, so that the host can adjust bitrate or recover lost keyframes.
11. As a Mac user, I want to download and install the official macOS OpenDisplay sender application, so that I do not need to compile or maintain custom Mac-side virtual display drivers.
12. As a user who frequently plugs and unplugs their device, I want the receiver and sender to automatically reconnect upon cable reconnection, so that I do not have to restart both applications manually.
13. As a user running dark and bright desktop content, I want video compression artifacts to be minimized via real-time VideoToolbox H.264 encoding with immediate keyframe recovery upon packet loss.
14. As an automated test engineer, I want to verify receiver stability and frame throughput using a headless fake-sender script over USB, so that the video and network pipeline can be proven without needing desktop UI interaction.

## Implementation Decisions

- **Receiver Codebase & Architecture**:
  - Adopt the `iOSLegacy` UIKit implementation branch rather than reverse-engineering the official iOS 15 App Store binary or creating a clean-room client.
  - Retain the exact shared stream protocol, socket transport, and `AVSampleBufferDisplayLayer` video renderer from upstream OpenDisplay.
  - Provide minimal, local in-tree shims for Combine reactive primitives and UIKit view controllers to maintain compatibility with iOS 12.2+ runtime environments.

- **Build Toolchain**:
  - Use a standalone shell build driver that calls `swiftc`, `actool`, `ibtool`, and Python plist generators directly.
  - Set the deployment target explicitly to `arm64-apple-ios12.2` to ensure modern Xcode command-line tools emit valid binaries for the A7 architecture while skipping IDE project version checks.
  - Package the compiled `.app` directory into an unsigned `Payload/` zip archive formatted as `OpenDisplayLegacy.ipa`.

- **Device Installation & Provisioning**:
  - Produce an unsigned IPA package optimized for Sideloadly deployment using the user's personal Apple ID.
  - Support direct deployment via `ideviceinstaller` for environments with active signing identities or AppSync Unified jailbreaks.

- **Host Sender Component**:
  - Standardize on the official prebuilt macOS `OpenDisplay.app` binary installed to `/Applications`.
  - Use macOS system virtual display APIs (`CGVirtualDisplayPrivate`) to expose the iPad as a genuine display in macOS System Settings.

- **Stream Protocol & Decode Budget**:
  - Cap H.264 macroblock throughput during the initial hello handshake to match the A7 chip's maximum sustained throughput (~42 fps at 2048×1536).
  - Use H.264 Baseline/Main profile exclusively, as HEVC decoding is not supported at this resolution on A7 hardware.

## Testing Decisions

- **Definition of a Good Test**:
  - Tests must verify external behavioral contracts: socket protocol negotiation, frame ingestion rate, decode success, and statistics reporting.
  - Tests must never assert on internal view controller hierarchies, private variable states, or intermediate buffer representations.

- **Modules Tested**:
  - The compiled package bundle (`OpenDisplayLegacy.app` / `OpenDisplayLegacy.ipa`): verify binary architecture (`arm64`), minimum OS requirement (`12.2`), and plist structure.
  - The receiver network & video pipeline: driven by `tools/fake-sender.swift` over the USB connection.

- **Prior Art**:
  - The upstream `tools/fake-sender.swift` utility provides an existing harness for driving receivers with synthetic VideoToolbox test streams and parsing reported receiver telemetry.

## Out of Scope

- Audio streaming from macOS to the iPad receiver.
- Apple Pencil pressure and tilt integration (not supported on 1st-generation iPad Air hardware).
- Wi-Fi discovery setup via Bonjour for environments without local network connectivity (USB is the required primary transport).
- App Store distribution or automated notarization workflows.
- Reverse engineering or patching compiled Swift binaries from the App Store iOS 15 receiver.

## Further Notes

- The iPad Air 1 (`iPad4,1`) contains an Apple A7 SoC and 1 GB of RAM. Memory consumption of the receiver must remain conservative to prevent iOS jetsam terminations.
- Sideloadly installs using free Apple IDs must be refreshed every 7 days unless installed on a jailbroken environment.
