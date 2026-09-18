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

Running this with Claude Code on the host machine? It follows [CLAUDE.md](CLAUDE.md) automatically.

Short version:

```
cp .env.example .env   # set a password
# point config/config.yml at your phone's RTSP address
docker compose up -d
open http://localhost:8080
```

## Viewing it

`http://mac-mini.local:8080` from any device on your wifi. That is the whole setup.

Watching from outside your home is a separate problem and not part of this repo — your Mac
mini is behind NAT, so it needs a VPN or tunnel of your choosing. One warning if you go
there: do not port-forward port 8080 on your router. That puts an unauthenticated view of
your home on the public internet, and it gets scanned and found.

## Live-streaming notes

go2rtc (bundled in Frigate) is the piece worth reading: RTSP in, WebRTC/HLS/MSE out, zero re-encode when codecs match. That restream is why one phone stream can feed detection, recording and the browser at once.
