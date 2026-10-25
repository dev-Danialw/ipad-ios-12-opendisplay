# 03: Deploy Receiver to iPad and verify streaming pipeline via fake-sender

**What to build:** Install the compiled receiver package onto the connected iPad Air (iOS 12.5.8), launch the receiver, and drive it through an automated end-to-end test stream using the headless `fake-sender` utility over USB to verify USBMux connectivity, handshake protocol, decode budget enforcement (~42 fps at 2048×1536), and video decoding stability.

**Blocked by:** 02: Compile iOS 12 Legacy Receiver and build unsigned IPA

**Status:** resolved

- [x] The `OpenDisplayLegacy.ipa` package is deployed onto the connected iPad Air (UDID: `4529fd398d9884fe1205fb0104a11790d4b48ae3`) via Impactor.
- [x] The receiver app launches on the iPad and displays its waiting/ready state.
- [x] USB connection established and verified over `usbmuxd`.
- [x] The receiver and sender successfully negotiate the hello handshake at 2048×1536 native Retina resolution.
- [x] Receiver telemetry reports smooth H.264 video decoding within the Apple A7 decode budget (~42 fps) without crashes or stalls.

## Comments

### Verification Summary
1. **Installation & Sideload**:
   - Receiver package `OpenDisplayLegacy.ipa` (`com.peetzweg.opensidecar.ios.legacy.G4LTP8HC5V`) installed on iPad Air 1 (`iPad4,1`, iOS 12.5.8, UDID `4529fd398d9884fe1205fb0104a11790d4b48ae3`).
2. **Protocol Handshake & Decode Budget**:
   - Connected via USBMux on port 9000.
   - Handshake successfully registered: `phone hello: panel 2048x1536 @2.0`.
   - Stream profile selected: `H264 2048x1536 @42fps` from 2048x1536 matching Apple A7 hardware decode budget.
   - Live telemetry verified: round-trip time ~2ms, encode/decode ~15-18ms with zero packet drops.
