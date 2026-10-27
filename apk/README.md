# apk/

| Fail | Saiz | Nota |
|---|---|---|
| `GlassTV-v9-weather-heavy.apk` | 341,037 bytes | Build terakhir (hujan lebat + `weather_preview`) |

## Pasang

```bash
adb connect <ip-tv>:5555
adb install -r GlassTV-v9-weather-heavy.apk
adb shell appops set com.gathin.tvhome SYSTEM_ALERT_WINDOW allow
adb shell am start -n com.gathin.tvhome/.MainActivity
```

Syarat: Android TV 8+ (API 26+). Diuji pada Xiaomi MiTV-MOOQ0, Android TV 10.

## Tandatangan

Ditandatangani dengan **debug key** projek (`signingConfig signingConfigs.debug` dalam
`build.gradle`). Ini bermaksud:

- boleh sideload pada TV sendiri melalui adb — tiada masalah;
- **tidak** boleh dimuat naik ke Play Store;
- jika anda build sendiri, tandatangan akan berbeza — uninstall versi ini dahulu
  (`adb uninstall com.gathin.tvhome`).

## Theme tidak termasuk

APK ini **tidak** mengandungi waktu solat, lokasi cuaca atau wallpaper. Itu di-push
berasingan sebagai theme (`config.json`), ke TV sahaja — supaya APK ini bebas data
peribadi. Lihat `config.example.json` di pangkal repo.

## Bina dari sumber

```bash
# JDK 17 + Android SDK platform 34, build-tools 34.0.0
cd launcher
./gradlew testReleaseUnitTest assembleRelease
# hasil: app/build/outputs/apk/release/app-release.apk
```

Sumber: [Gathin23/cleanyourtv](https://github.com/Gathin23/cleanyourtv) (MIT).
