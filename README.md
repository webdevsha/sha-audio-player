# Sha's Audio Library — streaming audio player

Static, manifest-driven audio player with Sha's **Qalb-Quran branding** (paper/lavender/tangerine/pucuk palette, Source Serif 4 + Inter + JetBrains Mono, corner cat). Built for big files (hosted on Cloudflare R2, streamed via range requests) and future-proof: adding a new track never touches the code.

## Files
- `index.html` — the player (branded, no build step, no dependencies).
- `manifest.json` — the track list + optional reflection prompt + per-track notes. Edit this to add music.
- `README.md` — this file.

## Live
- Player: https://sha-audio.pages.dev/
- Repo (public): https://github.com/webdevsha/sha-audio-player
- Audio base: `https://media.shafiranoh.com` (R2 public custom subdomain — **not** the apex, so the Hostinger site stays on `shafiranoh.com`).

## Config notes (verified 2026-10-22)
- Audio is stored on R2 at the **root** key. Current object: `DeepSunnah_22Oct_aac.m4a` →
  `https://media.shafiranoh.com/DeepSunnah_22Oct_aac.m4a` (returns 200, `audio/x-m4a`, ~48MB, `Accept-Ranges: bytes` → streams/seek works).
- The original was ALAC (`DeepSunnah_22Oct.m4a`, 68MB) which **no browser plays** in an `<audio>` tag. It was transcoded to AAC (`.m4a`, 48MB) for the player. **Always upload AAC/MP3, not ALAC.**
- `baseUrl` in `manifest.json` is `https://media.shafiranoh.com`.
- Do **not** put the R2 bucket on the apex `shafiranoh.com` — that shadows the Hostinger site (causes the "404 object not found" you hit). Keep audio on the `media.` subdomain.

## Add a future track (no code changes)
1. Upload to the R2 bucket at root (or any key — just match `file` below). Your R2 token, not Hannachan's:
   `wrangler r2 object put <bucket>/NewSong.m4a --file ./NewSong.m4a --content-type audio/mp4`
   (Use the Cloudflare Dashboard → R2 → bucket → upload, or `wrangler` with your own token.)
2. Add one entry to `manifest.json` → `tracks`:
   ```json
   { "title": "New Song", "file": "NewSong.m4a", "date": "2026-11-01" }
   ```
3. Commit + `wrangler pages deploy . --project-name sha-audio` (or set up GitHub Actions — see bottom).

## Reflection prompt (optional, shows on top)
Set at the **root** of `manifest.json`:
```json
"reflectionPrompt": "What is the 'opponent' in my life …?",
"reflectionPromptBy": "Shaykh Adeyinka"
```
Renders as a highlighted gradient card above the player.

## Per-track notes (optional)
Add a `notes` block to any track. Renders under a "Show notes & reflections" toggle:
```json
"notes": {
  "mainPoints": ["…", "…"],
  "reflections": ["…"],
  "insightfulPoints": ["…"]
}
```
All three keys are optional; omit any you don't need.

## Run locally
```
cd audio-player
python3 -m http.server 8080
# open http://localhost:8080
```

## Deploy
```
cd /Users/nshfnoh/audio-player
wrangler pages deploy . --project-name sha-audio
```
Pages serves only the tiny player; audio stays on R2, so the ~25MB Pages asset limit never applies.

## Optional: GitHub Actions auto-deploy
So pushing `main` redeploys without the manual command. Add `.github/workflows/deploy.yml`:
```yaml
name: Deploy to Cloudflare Pages
on:
  push:
    branches: [main]
jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: cloudflare/wrangler-action@v3
        with:
          apiToken: ${{ secrets.CLOUDFLARE_API_TOKEN }}
          accountId: ${{ secrets.CLOUDFLARE_ACCOUNT_ID }}
          command: pages deploy . --project-name sha-audio
```
Store `CLOUDFLARE_API_TOKEN` + `CLOUDFLARE_ACCOUNT_ID` as repo secrets.
