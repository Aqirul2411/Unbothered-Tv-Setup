# TV quirks found on this set (MiTV-MOOQ0, Android TV 10)

Notes written down the hard way, one evening at a time. They matter if you tune this
launcher for a Xiaomi or any Xiaomi-made Android TV.

## The panel ignores Android's brightness setting

`settings put system screen_brightness 20` writes fine, `dumpsys` reports it, and the
picture does not change at all. So a "Dim" button that lowers `screen_brightness` does
nothing visible on this TV.

That is why the dim here is a **translucent overlay window**:

```java
new WindowManager.LayoutParams(
        MATCH_PARENT, MATCH_PARENT,
        WindowManager.LayoutParams.TYPE_APPLICATION_OVERLAY,
        FLAG_NOT_FOCUSABLE | FLAG_NOT_TOUCHABLE | FLAG_LAYOUT_IN_SCREEN,
        PixelFormat.TRANSLUCENT)
```

with `0x59000000` (35 % black). Because it is its own window, it covers other apps too —
verified over the TV's own Settings app, white `255` → `166`.

Needs `SYSTEM_ALERT_WINDOW`, granted once:

```text
adb shell appops set com.gathin.tvhome SYSTEM_ALERT_WINDOW allow
```

Limitation: the window lives while the launcher's process lives. A foreground service
(tiny persistent notification) would make it survive the launcher being killed.

## All framework animations are off

```text
animator_duration_scale      0.0
transition_animation_scale   0.0
window_animation_scale       0.0
```

So `ViewPropertyAnimator`, `TransitionDrawable` and friends finish instantly. A wallpaper
cross-fade built on them looks like a hard cut. Anything that must visibly animate here is
driven by a `Handler` ticking every 33 ms and calling `setImageAlpha`.

## Screenshots do capture overlay windows

`adb shell screencap` includes overlays, so pixel sampling is a usable check: take a
screenshot, read the brightest pixel, and compare before/after. It does **not** capture
panel brightness, which is exactly why the brightness trick looked "worked" in tests and
did nothing in the room.

## The mic is not shared with apps

`pm list features` has no `android.hardware.microphone`. The remote's microphone belongs to
the system Assistant (`com.google.android.katniss`), and third-party apps get a granted
`RECORD_AUDIO` permission with **no input device** behind it:

```text
- Available input devices:      (empty)
```

Consequence: any AI chat app on this TV is **typed**, not spoken. Gemini on the phone
answers "Hey Google"; here the Assistant answers, and it cannot start a Gemini chat.

## Gemini on TV needs Android TV OS 14+

Gemini for Google TV is limited to Android TV OS 14+ devices with 2 GB RAM in selected
countries. An Android TV 10 set will not get it, and a launcher cannot take over the
`ASSIST` intent — it resolves to the system Assistant.

What does work: the Gemini **website** inside a TV app's WebView. Google's sign-in is
allowed there (tested: a bogus address answers "Couldn't find this account", not
"This browser or app may not be secure"), so the owner can sign in once with the remote.

## Injected D-pad OK is not the same as a real remote press

`input keyevent KEYCODE_DPAD_CENTER` sometimes did not trigger a pill's click while
`input tap <x> <y>` on the same bounds did. When testing whether a button responds to the
remote, watch for the side effect in `logcat` or in a setting it writes, rather than
trusting the injected key.

## MediaTek TVs and `exec-out screencap`

`adb exec-out screencap -p` produces broken PNGs on this chipset. Use:

```text
adb shell screencap -p /sdcard/s.png
adb pull /sdcard/s.png
```

Live TV, HDMI and Netflix frames come out black — that is the secure layer, not a bug.

## Home has more than one claimant

After removing the stock launcher, check who else answers Home:

```text
cmd package query-activities --brief -a android.intent.action.MAIN \
    -c android.intent.category.HOME
```

On Xiaomi the factory test app can take it. Disable the extras (for user 0) only after
confirming Home still lands on the new launcher.
