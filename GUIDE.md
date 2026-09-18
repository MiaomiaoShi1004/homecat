# homecat — full setup guide

Watch your cat from anywhere. Camera is an old Android phone, brain is a Mac mini,
nothing is stored in anyone's cloud. About 45 minutes start to finish.

You need: an Android phone you can leave plugged in, and a Mac mini (or any machine with
Docker). Nothing else, no accounts.

---

## 1. Install Docker on the Mac mini

Install Docker Desktop, open it once, leave it running. Check:

```
docker --version
```

In Docker Desktop → Settings → General, tick **Start Docker Desktop when you log in**.
Otherwise a power cut means no cat.

## 2. Get this repo

```
git clone https://github.com/MiaomiaoShi1004/homecat.git
cd homecat
cp .env.example .env
```

Open `.env` and replace `changeme` with a long random password. Generate one:

```
openssl rand -base64 24
```

This password protects the phone's video stream. Write it down, you need it again in step 3.

## 3. Turn the phone into a camera

Install **IP Webcam** from the Play Store (free). In the app:

- **Video preferences → Video resolution**: 640x480. Yes, really. Detection runs at low
  resolution anyway, and this is what keeps 7 days of clips small.
- **Video preferences → FPS**: 5.
- **Login/password**: username `cat`, password = the one from step 2.
- **Optional → Disable video on battery**: off.
- Scroll to the bottom → **Start server**.

The app shows an address like `http://192.168.1.50:8080`. The RTSP stream is at
`rtsp://192.168.1.50:8080/h264_ulaw.sdp`.

Now make that address permanent. In your router's DHCP settings, reserve that IP for the
phone's MAC address. Every router calls this something different — "DHCP reservation",
"static lease", "bind IP". If you skip this, the camera breaks the next time the phone
reconnects.

Physical setup: plug the phone in permanently, screen face-down or brightness at zero,
propped where it sees the litter box or the food bowl. A phone stand and a long USB cable
is the entire hardware budget.

## 4. Point homecat at the phone

Edit `config/config.yml`. Find this line:

```yaml
      - "rtsp://{FRIGATE_RTSP_PASSWORD}@phone.local:8554/live"
```

Replace it with your real address:

```yaml
      - "rtsp://cat:{FRIGATE_RTSP_PASSWORD}@192.168.1.50:8080/h264_ulaw.sdp"
```

Leave `{FRIGATE_RTSP_PASSWORD}` written exactly like that — it gets filled in from `.env`
at startup so your password never lands in git.

## 5. Start it

```
docker compose up -d
```

Give it two minutes, then open **http://localhost:8080**. You should see live video and an
empty sightings list.

If the video is a black box, check the logs:

```
docker compose logs -f frigate
```

`Connection refused` means the phone address is wrong or IP Webcam stopped.
`401 Unauthorized` means the password in `.env` and in the app don't match.

## 6. Wait for a cat

Detection is not motion detection — a curtain moving does nothing. The cat has to actually
walk into frame and be recognised. First sighting usually takes an hour or two of real cat
activity. Clips appear on the page automatically; anything older than 7 days deletes itself.

Too many false alarms, or missing your cat? One number in `config/config.yml`:

```yaml
        cat:
          min_score: 0.55
```

Raise toward 0.7 for fewer false alarms, lower toward 0.4 to catch a shy cat.

## 7. Watch it from your phone at home

Any device on the same wifi: `http://mac-mini.local:8080`. If `.local` does not resolve,
use the Mac mini's IP — `ipconfig getifaddr en0` prints it.

That covers the normal case. Watching from outside the house needs a VPN or tunnel back to
your home network, which is its own project and not covered here. If you do set one up:
**do not port-forward 8080 on your router.** A camera inside your home on the open internet
gets found by scanners within hours — that is worse than the cloud service you left.

### Letting a friend use it

They clone the repo and follow this guide from step 1 on their own machine. Nothing is
shared, no accounts, no server of yours involved. That is the point of self-hosting.

---

## What this does and doesn't do

Does: live view, cat-only detection, automatic clips, 7-day history, works from anywhere,
no cloud, no subscription.

Doesn't: feeding, portions, weight tracking (those need hardware, not video). No two-way
audio. No phone notifications yet — Frigate can fire a webhook at
[ntfy](https://ntfy.sh) when a cat appears, roughly ten lines of config, add it when the
silence starts bothering you.

## Cost

Zero, past the phone you already own. Storage runs about 2–5 GB for a rolling week at these
settings.
