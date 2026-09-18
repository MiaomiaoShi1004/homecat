# homecat

Self-hosted cat monitoring. Samsung phone = camera, Mac mini = NVR. Nothing leaves the house.

## PetKit feature parity

| PetKit feature | Here |
| --- | --- |
| Live view | the dashboard at `:8080`, or Frigate's own UI |
| Motion-triggered clips | Frigate event recording |
| Pet detection (not motion) | Frigate object detection, `track: [cat]` |
| Timeline of visits | dashboard sightings grid, from `/api/events` |
| Snapshots / thumbnails | Frigate snapshots |
| 7-day history | `retain.days: 7` |
| Push notifications | Frigate webhook -> ntfy (add when wanted) |
| Two-way audio | not done. Needs a phone app that accepts audio back |
| Night vision | phone LED or an IR cam |
| Feeding schedule / portions | hardware, not video. Out of scope |
| Weight tracking | needs a scale. Out of scope |

## Setup

**[Full step-by-step guide → GUIDE.md](GUIDE.md)** — start there if you want it to work.

Short version:

```
cp .env.example .env   # set a password
# point config/config.yml at your phone's RTSP address
docker compose up -d
open http://localhost:8080
```

## Over the internet

Do not port-forward. `tailscale serve --bg 8080` gives you an HTTPS URL that only your own
devices can reach. Friends get access via Tailscale device sharing, or just run their own
copy. See [GUIDE.md](GUIDE.md#7-watch-it-from-outside-the-house).

## Live-streaming notes

go2rtc (bundled in Frigate) is the piece worth reading: RTSP in, WebRTC/HLS/MSE out, zero re-encode when codecs match. That restream is why one phone stream can feed detection, recording and the browser at once.
