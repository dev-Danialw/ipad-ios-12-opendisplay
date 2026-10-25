# 0001. Compile OpenDisplay Legacy Receiver for iOS 12.5.8 via Standalone Toolchain

## Context
The official OpenDisplay iOS receiver requires iOS 15+ due to its use of SwiftUI and Combine. An iPad Air 1st generation (`iPad4,1`) is capped at iOS 12.5.8. Additionally, modern Xcode releases refuse deployment targets below iOS 15.

## Decision
We build the receiver from the `Francesco149/opendisplay` (`ios12-legacy-receiver`) fork, which ports the official stream receiver and video pipeline to UIKit with Combine shims. We use the custom `iOSLegacy/build.sh` script to invoke `swiftc`, `actool`, and `ibtool` directly against the iOS 12 deployment target, packaging an unsigned `.ipa` for deployment via Sideloadly. The Mac host runs the official release of `OpenDisplay.app`.

## Consequences
- Bypasses Xcode IDE deployment target restrictions without requiring binary reverse-engineering or patching of the iOS 15 App Store IPA.
- Enables native Retina (2048×1536 @ ~42 fps) streaming over USB via `usbmuxd`.
- Allows signing and refreshing using a standard free Apple ID through Sideloadly.
