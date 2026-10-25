# OpenDisplay iOS 12 Workspace

A dedicated workspace for building and deploying the OpenDisplay legacy receiver to an iOS 12 iPad and pairing it with a macOS sender host over USB.

## Language

**Receiver**:
The client application (`OpenDisplayLegacy`) running on the iPad that decodes H.264 video frames and renders them to the screen while streaming touch gestures back to the host.
_Avoid_: Client app, display client, viewer

**Sender**:
The macOS host application (`OpenDisplay`) running on the Mac that manages the virtual display, captures desktop frames, encodes them via VideoToolbox, and streams them over USB.
_Avoid_: Server, host app, streamer

**Decode Budget**:
The hardware-enforced limit on macroblock throughput on Apple A7 chips that caps 2048×1536 H.264 decoding to ~42 fps to prevent buffer overrun and decoder stalls.
_Avoid_: Framerate limit, throttler

**USBMux**:
The USB multiplexing protocol (`usbmuxd`) connecting the Mac and iPad over a Lightning cable to provide low-latency socket communication without Wi-Fi jitter.
_Avoid_: Cable link, USB tether
