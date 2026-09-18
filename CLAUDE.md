# Instructions for Claude running on the Mac mini

You are setting up homecat on this machine. Read this whole file before running anything.

Your job: get `http://localhost:8080` showing live video from an Android phone, with cat
detection recording clips. Everything you need is in this repo except three things the
human must give you (listed in step 0).

Work through the steps in order. After each step there is a **Verify** command — run it and
do not move on until it passes.

---

## Step 0 — Ask the human for these two things first

Do not guess these. Ask for both in one message, then wait:

1. **The phone's RTSP URL.** The IP Webcam app on their Android phone shows an IP address
   on its main screen when the server is running. The URL is that IP plus the stream path,
   e.g. `rtsp://192.168.1.50:8080/h264_ulaw.sdp`. If they have not set the phone up yet,
   walk them through **GUIDE.md step 3** before continuing — you cannot do that part, it
   happens on the phone.
2. **The username and password** they set in IP Webcam's Login/password screen.
3. Nothing else. Two answers and you can do the rest.

If they do not answer, stop and wait. Do not invent an IP address.

---

## Step 1 — Check what is already installed

```
docker --version; docker compose version; git --version; brew --version
```

Anything that errors, install below. Anything that prints a version, skip.

### Installing Docker

This is a Mac, so Docker means Docker Desktop:

```
brew install --cask docker
open -a Docker
```

**Docker Desktop must be launched once from the GUI and the license accepted before the
`docker` CLI works.** After `open -a Docker`, the human may need to click through a
permissions prompt and enter their Mac password. Ask them to do that, then wait.

If `brew` itself is missing:

```
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
```

That installer asks for a password and needs a human at the keyboard. Hand it over.

**Verify:**

```
docker run --rm hello-world
```

Must print "Hello from Docker!". If it says `Cannot connect to the Docker daemon`, Docker
Desktop is not running yet — `open -a Docker`, wait 30 seconds, try again.

---

## Step 2 — Get the repo onto this machine

If you are already inside the homecat directory (a `docker-compose.yml` is next to this
file), skip this.

```
cd ~
git clone https://github.com/MiaomiaoShi1004/homecat.git
cd homecat
```

**Verify:** `ls` shows `docker-compose.yml`, `config/`, `site/`.

---

## Step 3 — Create the .env file

`.env` is deliberately not in git — it holds the stream password. Create it:

```
cp .env.example .env
```

Now put the password from step 0 into it. Use the human's actual password:

```
printf 'FRIGATE_RTSP_PASSWORD=THEIR_PASSWORD_HERE\n' > .env
```

If they never set a password in the app, generate one and tell them to set the same value
in IP Webcam:

```
openssl rand -base64 24
```

**Verify:** `grep -q '^FRIGATE_RTSP_PASSWORD=.\{8,\}' .env && echo ok` prints `ok`.
Never print the file's contents into the chat.

---

## Step 4 — Point the config at the phone

Open `config/config.yml`. Under `go2rtc.streams.litterbox` there is a placeholder line:

```yaml
      - "rtsp://{FRIGATE_RTSP_PASSWORD}@phone.local:8554/live"
```

Replace it with the real URL from step 0, keeping the password placeholder intact:

```yaml
      - "rtsp://USERNAME:{FRIGATE_RTSP_PASSWORD}@192.168.1.50:8080/h264_ulaw.sdp"
```

Rules:
- `{FRIGATE_RTSP_PASSWORD}` stays written exactly like that, in curly braces. Frigate
  substitutes it at startup. **Never write the real password into this file** — it is
  tracked by git and would end up published.
- Replace `USERNAME` and the IP and path with what the human gave you.
- Keep the quotes and the leading `- `.

**Verify:**

```
grep -c 'phone.local' config/config.yml
```

Must print `0`. If it prints `1` you did not edit it.

---

## Step 5 — Start it

```
docker compose up -d
```

First run pulls about 1 GB of images. Two to five minutes is normal.

**Verify:**

```
docker compose ps
```

Both `frigate` and `site` must show `running`. Then:

```
sleep 30 && curl -sf http://localhost:8080 >/dev/null && echo "site ok"
curl -sf http://localhost:8080/api/version && echo
```

The second prints a Frigate version number. If it does not, Frigate is crash-looping.

### When it does not work

Read the logs first, always:

```
docker compose logs --tail 50 frigate
```

Match the error:

- **`Connection refused` / `No route to host`** — the phone's IP is wrong, the phone is
  asleep, IP Webcam is not running, or the phone is on a different network (guest wifi,
  or mobile data instead of wifi). Ask the human to check the app is showing "Server
  running" and confirm the IP.
- **`401 Unauthorized`** — the password in `.env` and the one in the IP Webcam app do not
  match. Fix one to match the other, then `docker compose restart frigate`.
- **`Unsupported codec` / ffmpeg exits immediately** — in IP Webcam set Video preferences →
  Video codec to **H.264**.
- **`address already in use`** — something else holds port 8080. Change the `site` port in
  `docker-compose.yml` from `"127.0.0.1:8080:80"` to `"127.0.0.1:8090:80"` and use 8090
  everywhere after this.
- **Config error on startup** — you broke the YAML in step 4. Check indentation: the stream
  line has six leading spaces.

After any fix: `docker compose restart` then re-run the verify commands.

---

## Step 6 — Tell them how to view it

On the same wifi, any device opens `http://mac-mini.local:8080`. If `.local` does not
resolve on their phone, give them the IP instead:

```
ipconfig getifaddr en0
```

If they ask about watching from outside the house: that needs a VPN or tunnel back to their
home network and is out of scope for this repo. **Do not set up port forwarding, ngrok,
cloudflared, or any public tunnel, and do not change the `127.0.0.1:` prefix on the port in
`docker-compose.yml`.** This is a camera inside someone's home; exposing it publicly is not
a call you make. Tell them it is possible and let them decide.

## Step 7 — Make it survive a reboot

Docker Desktop → Settings → General → tick **Start Docker Desktop when you log in**. This
is a GUI checkbox; ask the human to click it. The containers have
`restart: unless-stopped`, so once Docker starts they come back on their own.

**Verify:** `docker compose ps` shows `restart` policy in `docker inspect`, or just trust
the compose file — it is already set.

---

## Step 8 — Report back

Tell the human, in this order:

1. The local URL: `http://localhost:8080`, and `http://mac-mini.local:8080` from their
   phone on the same wifi
2. That the sightings grid will be **empty until a real cat walks into frame** — this is
   object detection, not motion, so it can take an hour or two. Not a bug.
3. Anything you could not finish and why

---

## Things you should not do

- Do not edit `site/index.html`, `site/nginx.conf`, or `docker-compose.yml` beyond the port
  change described in step 5. They work as-is.
- Do not commit `.env`. It is in `.gitignore`; keep it that way.
- Do not `git push`. This machine is running the thing, not developing it.
- Do not raise the resolution or fps "for better quality". 640x480 at 5fps is deliberate —
  it is what keeps CPU usage and a week of storage reasonable on a Mac mini.
- Do not add a Coral TPU, a GPU detector, MQTT, Home Assistant, or a VPN/tunnel unless asked. The CPU
  detector handles one low-res camera fine.
