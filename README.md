# Glass TV on a Xiaomi Mi TV — the Malaysian setup

A home-screen replacement for an **Android TV** set, tuned for a Malaysian living room:
prayer times that turn through the five daily prayers, an azan reminder at each one,
weather for the hour now plus the next six, rain falling over the wallpaper when it
rains outside, a calendar box that never disappears, and wallpapers that dissolve into
the next one as you move around the screen.

This folder is the write-up of that setup: what was done, why, and what to watch out
for on this exact TV. It contains **no private data** — no account, no API key, no
calendar address, no home coordinates, no device address.

| | |
|---|---|
| TV | Xiaomi Mi TV `MiTV-MOOQ0` |
| OS | Android TV 10 (API 29) |
| Launcher | Glass TV `com.gathin.tvhome` (this project), release APK ≈ 340 KB |
| Screen | 1920 × 1080 |

The launcher itself is [Glass TV / Clean Your TV](https://github.com/Gathin23/cleanyourtv)
by Gathin (MIT). This folder only describes one TV's installation of it.

---

## What is on the screen

```
┌──────────────────────────────────────────────────────────────┐
│  10:42   Isnin 27 Oktober         [ Inputs ][ ☀ ][ Wi-Fi ][⚙]│
│                                                              │
│  ┌─ Waktu Solat ────────┐   ┌─ Cuaca ──────────────────────┐ │
│  │ Zohor        13:12   │   │   Sekarang  +3j    +6j       │ │
│  └──────────────────────┘   │   29°       30°    27°       │ │
│                             └──────────────────────────────┘ │
│  ┌─ Jadual ──────────────────────────────┐                   │
│  │ 8:30  Mesyuarat                        │                  │
│  └────────────────────────────────────────┘                  │
│                                                              │
│        (wallpaper: a place, rain over it when it rains)      │
│                                                              │
│   ▢▢▢▢▢  ◄ dock: apps, focused tile lifts                    │
└──────────────────────────────────────────────────────────────┘
```

---

## Features, one by one

### 1. Waktu Solat — five prayers, turning

* Times come from **JAKIM e-Solat** (`e-solat.gov.my`), zone `PNG01` for this TV,
  fetched once a day and cached on the TV, so the box still shows today's times with
  no internet.
* The box **turns through the five prayers every 4.2 s** — Subuh, Zohor, Asar, Maghrib,
  Isyak — then starts again, so all five are readable without pressing anything.
* The prayer whose time is **running now** is shown warm (gold) with a lit rim;
  the others are plain white.
* Caption says `Waktu Solat` when the times are today's, and names the zone when they
  are from the cache (TV has been off since yesterday).
* Hijri date beside the clock.
* Config: `"prayer_zone": "PNG01"`. Leave empty to hide the box.

### 2. Azan reminder

When a prayer's time arrives:

* A **banner** slides in: `Azan <prayer>` and the time and zone,
* the **azan recording plays for a fixed number of seconds** and is then cut, so the
  TV does not keep singing for four minutes,
* a notification is posted in the `azan` channel (so it also appears if something else
  is on screen).

Config:

```json
"azan": true,
"azan_seconds": 15
```

`azan_seconds` is how much of the recording to play — `5` gives a short reminder,
`15` the default here, `0` plays the whole azan.

### 3. Cuaca — weather for now, +3 h, +6 h

* **Open-Meteo**, free and keyless: `temperature_2m` + `weather_code`, hourly, two days.
* Three columns: the **hour now**, **three hours on**, **six hours on**; rewritten when
  the clock reaches a new hour, so it always starts at "now" and never turns by itself.
* Icons follow Google Weather's shapes (sun, partly cloudy, cloud, rain, snow, storm).
* Fetched every 30 minutes on a background thread; a failure waits 15 minutes and keeps
  the cached hours on screen (offline-friendly).
* Config: `"weather_location": "<lat>,<lon>"` — **your** coordinates, kept on the TV only.

### 4. Rain over the wallpaper

When the WMO code for the hour says it is raining, the picture outside gets weather:

| Code | Scene | Drops |
|---|---|---|
| 51–57 | drizzle | 40 |
| 61–67, 80–82 | rain | 80 |
| 95+ | thunderstorm (rain + lightning) | 104 |
| 71–77, 85–86 | snow | 54 |

* Drops are short slanted lines moved by one loop at **30 fps**, drawn only while Home is
  in front — nothing runs behind another app.
* `"reduced_motion": true` halves the count (and makes every animation short), for a
  weaker GPU. That is what this TV uses.
* Config: `"weather_effect": true`.

### 5. Jadual — calendar

* Reads **iCalendar feeds** (a Google Calendar "secret iCal address", or any `.ics`
  link) configured **on the TV**, merges them, and shows the next few events with time
  and title.
* When there is nothing: the box stays, caption shows **`TIADA JADUAL`** — it never
  disappears or leaves a hole in the layout.
* 🔒 The feed address is a private secret. It lives only in the TV's own settings and in
  **no** file in this folder.

### 6. Wallpapers that dissolve

* Two sources: the built-in **Places** pictures (Bing's weekly home-page photo of
  somewhere in the world, with its name and country — the launcher's only download) and
  **your own pictures** in the app's folder
  `/sdcard/Android/data/com.gathin.tvhome/files/wallpapers/`.
* The picture **changes when you move focus** between apps, and the change is a real
  dissolve: the next picture is revealed behind and the old one fades out over 500 ms
  (`reduced_motion`) or 900 ms — no instant jump.
* Asking for a change while one is still dissolving **queues** it, so fast D-pad presses
  still end on a new picture instead of being dropped.
* `"places": false` keeps everything offline and uses only your own pictures.
* Config: `"wallpaper_seconds": 15` (time between changes), `"wallpaper_minutes"`,
  `"wallpapers": [...]`, `"wallpaper": true`.

### 7. Dim — 35 %, over every app

A sun pill beside **Inputs** dims the whole screen **35 %** with a translucent sheet in
its own window, so it covers **YouTube and everything else**, not just Home.

* 35 % was chosen after 25 % felt like nothing: white `255` → `166` on screen.
* The window is `APPLICATION_OVERLAY`, takes no focus and no touches — the remote works
  straight through it.
* The state is remembered across restarts.
* This TV **ignores Android's brightness setting** (`screen_brightness` changes, the panel
  does not), which is why the sheet is used — see `docs/TV-QUIRKS.md`.

### 8. Self-cleaning caches

Glass TV clears other apps' caches **every 15 days** on its own (`TvHome.Clean` in
logcat). It needs a one-time "Allow USB debugging?" confirmation on the TV, and network
debugging left on; turn debugging off and only the cleaning stops.

### 9. Home, dock and order

* `"columns": 5` — five apps per row, `"rows_below_dock": 0` — no rows under the dock.
* `"dock_carousel": true` — the dock is one long bar that wraps around.
* `"dock_apps"` puts four apps in the dock; anything beyond the dock's width continues as
  the first grid rows, in the same order.
* `"hidden_apps"` hides packages from the grid without uninstalling them.
* The owner can still move apps on the TV (hold OK → Move); their order wins afterwards.

### 10. Debloat, reversibly

The stock Xiaomi/MediaTek packages are **disabled**, never uninstalled:

```text
com.xiaomi.mitv.mediaexplorer   (stock file browser)
com.xiaomi.mitv.smartshare      (Miracast — disabled only after confirming it is unused)
com.mitv.tvhome.mitvplus        (PatchWall extras)
com.mitv.gallery                (stock gallery)
com.mediatek.wwtv.tvcenter      (factory TV app / test menu)
```

Every change was written to a log and `tools/restore.sh` puts the TV back: it re-enables
the packages, re-enables the stock launcher and makes it Home again.

---

## Why it looks the way it does

| Decision | Reason |
|---|---|
| No app-name caption | The picture is the point; app names under the tiles looked like a shop sign. |
| No glass panel behind the tiles | The wallpaper is already the design; the tray cut a hole in it. |
| Wallpaper changes on focus | Static wallpapers get stale on a TV that is on every evening. |
| Dissolve, not a jump | A hard cut on a 55-inch panel is jarring; a 500 ms cross-fade is calm. |
| Prayer box turns by itself | All five times are visible within ~21 s, without the owner pressing anything. |
| `TIADA JADUAL` instead of hiding | A vanishing box makes the whole layout jump; a caption keeps it still. |
| 35 % dim | 25 % was invisible from the sofa; 35 % takes the glare off without dimming the film. |

---

## Reproducing it

1. Android TV 8+ (this TV is 10). Enable developer options → USB debugging, connect over
   the home network.
2. Back up first: package list, disabled list, `getprop`, `settings list system/secure/global`,
   which app owns Home.
3. Build the launcher (JDK 17, Android SDK 34): `./gradlew assembleRelease`, then install
   with the project's `build-and-install` script, which puts the stock launcher back if
   anything fails.
4. Push the theme: `tools/theme.sh themes/mitv-mooq0` (or `tools\theme.ps1` on Windows).
5. Grant the overlay permission once:
   `adb shell appops set com.gathin.tvhome SYSTEM_ALERT_WINDOW allow`.
6. Set the calendar feed **on the TV** (never in the repo) and the wallpaper folder if
   you use your own pictures.

`config.example.json` in this folder is the theme used here, with the coordinates
replaced by a placeholder.

---

## What is deliberately **not** in this folder

* the calendar feed address (a secret iCal link),
* the TV's coordinates (only `<lat>,<lon>` placeholder),
* the TV's IP address and the Wi-Fi name,
* Google account, Play Store details, playback history,
* any screenshot of the calendar or of the app list.

Screenshots are easy to add later: `adb shell screencap -p /sdcard/home.png`, pull it,
and check it for anything personal first.

---

## Credit

Glass TV is built on [Clean Your TV](https://github.com/Gathin23/cleanyourtv) by Gathin
(MIT). Prayer times: JAKIM e-Solat. Weather: Open-Meteo. Places pictures: Bing home page.
