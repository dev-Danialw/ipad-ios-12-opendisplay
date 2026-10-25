# 01: Setup official macOS OpenDisplay Sender

**What to build:** Install and configure the official OpenDisplay macOS sender application on the host MacBook so that the host can allocate virtual Retina displays, capture desktop video frames via VideoToolbox, and listen for incoming USB connections.

**Blocked by:** None (can start immediately)

**Status:** resolved

- [x] The latest official macOS release archive of OpenDisplay is retrieved and installed into `/Applications/OpenDisplay.app`.
- [x] The sender application successfully launches on macOS.
- [x] Required macOS permissions (Screen Recording and Accessibility) are checked and documented for activation.
- [x] The background USB listener (`usbmuxd`) is verified to be active and ready to service connected iOS devices.

## Comments

### Verification Summary
1. **Installed OpenDisplay.app**:
   - Downloaded official release `v1.28.0` (`OpenDisplay.dmg`) from `peetzweg/opendisplay`.
   - Mounted image and copied bundle to `/Applications/OpenDisplay.app`.
   - Verified codesigning and bundle identifier: `com.peetzweg.opensidecar.mac`.

2. **Process Execution**:
   - Launched via `open /Applications/OpenDisplay.app`.
   - Verified process active: PID running `/Applications/OpenDisplay.app/Contents/MacOS/OpenDisplay`.

3. **Required macOS Permissions**:
   - **Screen Recording** (`System Settings > Privacy & Security > Screen & System Audio Recording`):
     - Required for OpenDisplay to stream the virtual display desktop buffers via VideoToolbox/ScreenCaptureKit.
     - OpenDisplay triggers a TCC prompt on initial capture; toggle switch to ON for **OpenDisplay**.
   - **Accessibility** (`System Settings > Privacy & Security > Accessibility`):
     - Required for synthetic input event injection (`CGEventPost`) when receiving touch taps, drag gestures, and scroll events from the iPad receiver.
     - Toggle switch to ON for **OpenDisplay**.

4. **USBMux Listener Verification**:
   - Native daemon `/System/Library/PrivateFrameworks/MobileDevice.framework/Versions/A/Resources/usbmuxd` is active.
   - Socket `/var/run/usbmuxd` verified accessible and receiving connection flows from OpenDisplay.
   - Connected device verified: iPad Air 1 (`iPad4,1`), iOS `12.5.8` (UDID `4529fd398d9884fe1205fb0104a11790d4b48ae3`).

