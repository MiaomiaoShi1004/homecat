# homecat

Self-hosted cat monitoring. Samsung phone = camera, Mac mini = NVR. Nothing leaves the house.

## PetKit feature parity

| PetKit feature | Here |
| --- | --- |
| Live view | Frigate UI / go2rtc WebRTC |
| Motion-triggered clips | Frigate event recording |
| Pet detection (not motion) | Frigate object detection, `track: [cat]` |
| Timeline of visits | Frigate Events UI + `/api/events` |
| Snapshots / thumbnails | Frigate snapshots |
| 7-day history | `retain.days: 7` |
| Push notifications | Frigate webhook -> ntfy (add when wanted) |
| Two-way audio | not done. Needs a phone app that accepts audio back |
| Night vision | phone LED or an IR cam |
| Feeding schedule / portions | hardware, not video. Out of scope |
| Weight tracking | needs a scale. Out of scope |

## Setup

1. Phone: install an RTSP server app (IP Webcam, or [unal-ai/android-rtsp](https://github.com/unal-ai/android-rtsp)). Set 640x480, 5fps, low bitrate. Give the phone a static DHCP lease.
2. Mac mini: `cp .env.example .env`, edit, `docker compose up -d`.
3. Open https://mac-mini.local:8971.

## Over the internet

Do not port-forward. Put both devices on Tailscale and point `phone.local` at the tailnet name.

## Live-streaming notes

go2rtc (bundled in Frigate) is the piece worth reading: RTSP in, WebRTC/HLS/MSE out, zero re-encode when codecs match. That restream is why one phone stream can feed detection, recording and the browser at once.
