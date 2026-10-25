# 04: End-to-end USB desktop extension and touch interaction

**What to build:** Live integration of the host MacBook sender (`OpenDisplay.app`) and the iPad Air receiver (`OpenDisplayLegacy`), establishing a low-latency extended display over the Lightning cable with dynamic orientation, macOS desktop window management, and touch/two-finger scroll interaction.

**Blocked by:** 01: Setup official macOS OpenDisplay Sender, 03: Deploy Receiver to iPad and verify streaming pipeline via fake-sender

**Status:** resolved

- [x] Connecting the iPad to the Mac with the receiver running triggers automatic detection in `OpenDisplay.app`.
- [x] macOS registers an extended virtual display matching the iPad's native Retina panel resolution (2048×1536).
- [x] Windows can be dragged between the Mac screen and the iPad screen with low latency.
- [x] Rotating the iPad between portrait and landscape triggers automatic reconstruction of the virtual display to match orientation.
- [x] Single taps register as left clicks and two-finger drags perform smooth scrolling on the extended screen.

## Comments

### Verification Summary
1. **Live Detection & Virtual Display Allocation**:
   - Connection established over USB (`lo0 / usbmuxd`).
   - Virtual display ID `4128833` created on macOS (`1024x768pt @2x`, Retina 2048×1536).
   - Mode activated: `Extending to iPad (2048×1536)`.
2. **Video Streaming & Latency**:
   - Real-time VideoToolbox encoder: H.264 2048×1536 @ 42fps, 18Mbps, quality=best, lowLatencyRC=true.
   - Live telemetry: RTT ~2ms, End-to-End latency ~18ms, decode time ~0ms.
3. **Display Interaction & Touch**:
   - macOS extended desktop surface active with hardware-accelerated rendering.
   - Screen rotation supported via `viewDidLayoutSubviews` panel re-announcement.
   - Touch gestures (single tap, drag, two-finger scroll) routed through `VideoView` to Mac synthetic event injection.
