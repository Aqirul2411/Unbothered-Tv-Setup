# What changed, build by build

Only the APK names and sizes are kept: the builds themselves stay on the owner's PC.
Sizes are the release APK (`assembleRelease`, under the 400 KB the project's CI allows).

## v9 — hujan lebat dibezakan · 341,037 bytes · current

* Rain used to have one look: codes 61–67 and 80–82 all drew the same 80 drops, so **heavy
  rain looked exactly like light rain**. Heavy (65, 67, 82) is now its own scene: 128 drops,
  longer (21–37 dp), faster (620–930 dp/s), thicker (2.25 dp) than rain's 80.
* Storm raised to 160 drops (was 104) so it still leads; `MAX` in `WeatherFxView` went
  110 → 170 to hold them. `reduced_motion` still halves everything (heavy → 64, storm → 80).
* New config key **`weather_preview`** (`drizzle|rain|heavy|storm|snow`): forces one scene
  whatever the forecast says, so a theme can be checked — and screenshots taken — without
  waiting for the sky. Unknown values are ignored; `""` means the real forecast.
* Verified on the TV by logcat: `rain: 40 drops`, `heavy rain: 64 drops`,
  `thunderstorm: 80 drops` (reduced motion on).
* Screenshots added: hujan biasa, hujan lebat, ribut + kilat, and three wallpapers after
  D-pad moves only.

## v8 — dim 35 % over every app · 340,637 bytes

* The dim sheet went from 25 % to **35 % black**; still its own `APPLICATION_OVERLAY`
  window, so other apps dim too.
* Measured: white `255` → `166` (255 × 0.65), both on Home and over the TV's Settings app.

## v7 — dim as an overlay window · 340,633 bytes

* The dim stopped touching `screen_brightness` (this TV ignores it — the button looked
  dead even though it ran) and became a translucent window above everything.
* `SYSTEM_ALERT_WINDOW` added; the install scripts grant it with `appops` and turn
  `WRITE_SETTINGS` back off.
* Verified: window type `APPLICATION_OVERLAY`, white `255` → `191` at 25 %.

## v6 — dim by system brightness · 340,601 bytes

* First attempt at Dim 25 %: lowered `Settings.System.SCREEN_BRIGHTNESS` to 75 % of its
  value and remembered the original to restore it.
* Looked fine in `dumpsys`, changed nothing on the panel. Replaced in v7.

## v5 — dissolve fixed · 339,049 bytes

* The dissolve no longer uses `ViewPropertyAnimator`/`TransitionDrawable` (every framework
  animation on this TV ends instantly because the animation scales are `0.0`); it is a
  `Handler` at 33 ms ticking `setImageAlpha` over 900 ms (500 ms with `reduced_motion`).
* A change asked for during a dissolve is **queued** rather than dropped, so fast D-pad
  presses still end on a new picture.
* Verified by sampling pixels mid-fade: `520D2B` → `3F085B` → `8AB6E8`.

## v4 — clean wallpaper, no tray, no app names · 338,793 bytes

* The glass tray behind the dock tiles is gone (`DockDecoration` now only paints the focus
  shape and keeps the 6 dp/12 dp offsets); the wallpaper shows through.
* The caption that printed the focused app's name ("SmartTube") is hidden
  (`heroShelf` → `GONE`): apps only, no labels.
* Wallpaper changed on focus, but the change was still a hard cut here.

## v3 — calendar empty state · 338,917 bytes

* The calendar box no longer disappears when there is nothing to show: it stays, with
  **`TIADA JADUAL`** underneath, so the layout never jumps.
* First round of the wallpaper-on-focus work.

---

## Also done on the TV (no APK change)

| Change | Detail |
|---|---|
| Debloat | Stock Xiaomi/MediaTek packages **disabled for user 0**, never uninstalled; full package and settings backup taken first, `restore-all` list written. |
| Wallpaper folder | Own pictures copied to `/sdcard/Android/data/com.gathin.tvhome/files/wallpapers/` (no storage permission needed); 8 pictures, 2752 × 1536. |
| Screensaver | Pointed at the launcher's clock before the stock screensaver was disabled. |
| High contrast text | Turned off by the install script (it draws a dark outline round every glyph); the owner confirmed it was not needed for reading. |
| AI chat app | *AI Chat* (`com.arcelik.chatbot`) installed from Play Store for chatting with Gemini/ChatGPT/DeepSeek on the TV; typed only, because the TV shares no microphone with apps. |

## Backups kept (on the owner's PC, not in this folder)

```text
GlassTV-v3-calendar-glass-wallpaper.apk   338,917
GlassTV-v4-clean-dissolve.apk             338,793
GlassTV-v5-dissolve-fixed.apk             339,049
GlassTV-v6-dim25-system.apk               340,601   (dim did not work)
GlassTV-v7-dim25-overlay.apk              340,633
GlassTV-v8-dim35-overlay.apk              340,637
GlassTV-v9-weather-heavy.apk              341,037   (current)
```

Plus the pre-change safety net: full package list, disabled list, `getprop`, `settings list
system/secure/global`, the Home claimants, and a restore list.
