# Sha's Audio Library — streaming player

Static, manifest-driven audio player. Built for big files (hosted on Cloudflare R2, streamed via range requests) and future-proof: adding a new track never touches the code.

## Files
- `index.html` — the player (light theme, no dependencies).
- `manifest.json` — the track list. Edit this to add music.
- `README.md` — this file.

## Verified config (2026-10-22)
- R2 public bucket: `https://pub-84285bbf9ed84ecfb9a74fc62ac51140.r2.dev`
- Objects are stored at the **root** key (not under `audio/`). Example existing object:
  `DeepSunnah_22Oct.m4a` → `https://pub-84285bbf9ed84ecfb9a74fc62ac51140.r2.dev/DeepSunnah_22Oct.m4a`
  (returns 200, `audio/x-m4a`, ~68MB, `Accept-Ranges: bytes` → streams fine.)

## One-time setup
`baseUrl` in `manifest.json` is already set. Nothing else needed unless you switch buckets.

## Add a future track (no code changes)
1. Upload to the R2 bucket at root (or any key — just match `file` below):
   `wrangler r2 object put <bucket>/NewSong.m4a --file ./NewSong.m4a --content-type audio/mp4`
   (For this account the bucket is addressed via the `pub-…r2.dev` public URL; use the Dashboard or `wrangler r2 object put` with your bucket name.)
2. Add one entry to `manifest.json` → `tracks`:
   `{ "title": "New Song", "file": "NewSong.m4a", "date": "2026-11-01" }`
3. Reload the page. Done.

## Run locally
```
cd audio-player
python3 -m http.server 8080
# open http://localhost:8080
```

## Deploy (recommended)
Serve `index.html` + `manifest.json` from Cloudflare Pages (free). The audio stays on R2 — Pages only serves the tiny player, so the ~25MB Pages asset limit is never hit.
```
cd /Users/nshfnoh/audio-player
wrangler pages deploy . --project-name sha-audio
```
The `webdevsha/personal` repo is **private** — a Pages deploy from it is a private deploy (you can listen; others need auth). If you want it public, deploy from a public repo or just host the two files anywhere static.
