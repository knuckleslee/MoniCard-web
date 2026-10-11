# MoniCard Web Console

Unofficial Web Bluetooth client for MoniCard devices. No WeChat required.
Single static file: `index.html` (no build step, no dependencies).

## Requirements
- Chrome / Edge on Windows, macOS, Linux or Android (Web Bluetooth).
  Not supported: iOS, Firefox, Safari.
- Served over HTTPS (GitHub Pages is fine) or `http://localhost`.
- Close the WeChat mini program first; a BLE device usually accepts one connection.

## Deploy to GitHub Pages
1. Create a repo and push `index.html` (and this README) to `main`.
2. Repo **Settings → Pages → Build and deployment → Source: Deploy from a branch**,
   branch `main`, folder `/ (root)`.
3. Open `https://<user>.github.io/<repo>/`.

Local test: `python -m http.server 8000` in this folder, then open `http://localhost:8000`.

## Implemented
| Feature | Control command |
|---|---|
| Serial / version / battery / product / storage | 18 / 20 / 22 / 70 / 32 |
| Read / write card text (<= 319 bytes UTF-8) | 16 / 14 |
| Control flags (broadcast, buzzer, vibration, light, proximity, ambient) | 24 |
| Tags (<= 5) | 34 |
| Carousel interval | 44 / 42 |
| Received cards: count / read / delete | 36 / 38 / 40 |
| Raw control command console | any |

Not implemented: image/video transfer (category 4) and firmware OTA (category 1).

## Protocol summary
- GATT service `7369666c-695f-7364-0000-000000000000`,
  data characteristic `7369666c-695f-7364-0002-000000000000` (notify + write).
- Frame (little-endian): `[category][0][len16][payload]`, control category = `31`,
  control payload = `[cmd16][data]`. Responses use the same framing.
- Long frames are split: first `[cat][1][totalLen16][..]`, then `[cat][2][..]`,
  last `[cat][3][..]`. Each write is at most `ATT MTU - 3` bytes
  (the page defaults to 20 because Web Bluetooth cannot read the negotiated MTU).
- Status responses: first `u16` of data, `0` = OK.
- No authentication or encryption was observed in the vendor mini program.

## Notes
- Writes change the state of your device (card text, tags, flags). Delete is permanent.
- Do not commit the decrypted vendor mini program source to a public repo;
  only this client belongs there.
