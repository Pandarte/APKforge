# ⚡ APKforge

**Android APK build interface, driven from your phone.**

> 🇬🇧 English &nbsp;•&nbsp; [🇫🇷 Français](README.fr.md)

APKforge is an Android app (Kotlin / Jetpack Compose, Material 3 Expressive,
dynamic Material You colors) that drives a local build toolchain running inside
Termux. Paste a git repository URL, tap **Build**, watch the logs stream live,
and grab the resulting APK at the end — all without leaving your phone.

<p align="left">
  <img alt="Platform" src="https://img.shields.io/badge/platform-Android%2012%2B-3DDC84">
  <img alt="Language" src="https://img.shields.io/badge/Kotlin-Jetpack%20Compose-7F52FF">
  <img alt="minSdk" src="https://img.shields.io/badge/minSdk-26-blue">
  <img alt="compileSdk" src="https://img.shields.io/badge/compileSdk-36-blue">
</p>

---

## How it works

APKforge is only the **interface**. The entire build happens server-side, where
the `aapt2` + `qemu` toolchain lives. This is a technical necessity: a sandboxed
Android app cannot launch `proot`/`qemu` itself.

```
  APKforge app (Compose)  ──HTTP 127.0.0.1:8765──▶  buildserver.py (Termux / proot)
        │                                                  │
   URL input,                                    runs android-builder.sh
   live logs,                                    (clone, detect, build via qemu)
   install button
```

The companion project that actually runs the builds is
[`android-build-tools`](https://github.com/Pandarte/android-build-tools).

## Requirements

To set up once on the phone:

1. **Termux** installed, with the
   [`android-build-tools`](https://github.com/Pandarte/android-build-tools)
   toolchain and its `buildserver.py` in place inside the Ubuntu proot.
2. **The server running**:
   ```bash
   proot-distro login ubuntu
   python3 ~/buildserver/buildserver.py
   ```
   (or as an automatic service via `start-build-server.sh`.)
3. **APKforge** installed and launched. It checks the connection to the server
   on startup.

On first launch, if the toolchain isn't detected, the app offers an **Install
toolchain** button that triggers the installation server-side and shows its logs.

## Usage

1. Paste an Android git repository URL into the field at the top.
2. Tap **Build**.
3. Follow the logs live in the console.
4. Grab the resulting APK (`APKforge.apk`) at the end.

## Building APKforge

APKforge builds with the very toolchain it drives — or via continuous
integration.

**GitHub Actions** (easiest): a push to `main`/`master` triggers the
[`build.yml`](.github/workflows/build.yml) workflow, which produces the debug
APK and makes it available as an artifact.

**Locally** from Termux:
```bash
bash ~/android-build-tools/build-android-local.sh ~/forge
```

Native Android project: `compileSdk 36`, `minSdk 26`, AGP 8.13, Material 3
Expressive (`material3:1.4.0-alpha10`).

## Technical notes

- **Dynamic Material You**: active on Android 12 and above; falls back to a
  neutral palette below.
- **Localization**: French and English, following the system language, with an
  in-app language picker (System / Français / English) in the top bar.
- **Network security**: cleartext traffic is restricted to `127.0.0.1` (see
  [`network_security_config.xml`](app/src/main/res/xml/network_security_config.xml)).
  Nothing leaves the phone.
- **Alpha dependencies**: Material 3 Expressive relies on fast-moving alpha
  versions. If Gradle rejects a version, use the latest available `material3`
  alpha and the matching `compose-bom`.

## Project structure

```
app/src/main/
├── AndroidManifest.xml
├── assets/
│   ├── forge-install.sh          server-side toolchain installation
│   └── forge-start.sh            build server startup
├── java/fr/buildtool/app/
│   ├── MainActivity.kt           Material You theme + edge-to-edge
│   ├── BuildScreen.kt            UI: URL, button, log console, animation
│   ├── BuildViewModel.kt         state + log polling
│   ├── BuildClient.kt            HTTP client for the local server
│   └── TermuxHelper.kt           Termux interactions
└── res/
    ├── drawable/                 adaptive icon (anvil + Android)
    ├── values/ values-en/        localized strings (FR default, EN)
    ├── xml/network_security_config.xml
    └── xml/locales_config.xml    supported app languages
```

## Related projects

- [`android-build-tools`](https://github.com/Pandarte/android-build-tools) — the
  build toolchain and HTTP server that APKforge drives.
