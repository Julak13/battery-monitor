# Firmware update site

A static site the device reads to find new firmware. Everything here is plain
files — no server, no build step, no account beyond GitHub itself.

## One-time setup

The URLs are already filled in for **github.com/Julak13**.

1. Create a public GitHub repository named **`battery-monitor`** — that exact
   name, lower-case. The repository name is the URL path and *is* case
   sensitive, unlike the user name.
2. Copy the contents of this folder into it, plus the `.bin` you built.
3. In the repo: **Settings → Pages → Source: Deploy from a branch**, branch
   `main`, folder `/ (root)`. Save.
4. After a minute the site is live at
   <https://julak13.github.io/battery-monitor/>
5. Flash the firmware over USB **once**. Every update after that is over Wi-Fi.

Check it worked by opening
<https://julak13.github.io/battery-monitor/firmware.json> in a browser — you
should see the manifest. That is the exact URL the device requests, so if it
loads for you it will load for the device.

## Publishing an update

```powershell
arduino-cli compile --fqbn esp32:esp32:esp32:PartitionScheme=min_spiffs `
  --output-dir build 'C:\Users\jubri\Documents\Claude\BatteryMonitor'
```

Then:

1. Copy `build\BatteryMonitor.ino.bin` here as `BatteryMonitor-<version>.bin`.
2. Update `firmware.json` — `version` must match `FW_VERSION` in `config.h`
   **exactly**, `url` points at the new file, `notes` is one short line.
3. Commit and push.

On the device: **System → Firmware update → Check for updates → Install**.

## Why it is safe to interrupt

The ESP32 writes into the spare OTA slot while the running firmware stays in
the active one. The bootloader only switches over after the whole image is
written and its checksum verified. A dropped Wi-Fi connection mid-download
leaves the device running exactly what it was running before.

The `min_spiffs` partition scheme is what provides the second slot. Do not
switch to `huge_app` — it has only one, and online updates stop working.

## Security

The device fetches over HTTPS but does not verify the certificate
(`setInsecure()`), because pinning a CA on an ESP32 breaks silently whenever
GitHub rotates its chain. Anyone able to intercept your traffic *and* serve a
valid-looking manifest could push firmware to the device. On a home network
that is an acceptable trade; for anything exposed, host the manifest somewhere
you control and pin the certificate.

Keep the repository public. GitHub Pages on a private repository needs a paid
plan, and the device sends no credentials.
