# Cybion App

Thin native shells that host the [Cybion](https://cybion.ntnl.io) web app on
platforms whose browser can't run it directly.

Cybion's control surface is the web app at `cybion.ntnl.io`. On a modern
desktop this is just a browser tab. On some platforms it isn't enough: an
Android WebView that is too old for the session check, a phone with no way to
install a PWA, or a user who wants a real app icon and window. This repository
packages the web app into small, auditable native containers:

- **Android** (`android/`) — a minimal Java `WebView` activity.
- **macOS** (`macos/`) — a minimal Swift `WKWebView` app bundle.

Both shells are **viewers**. They render the hosted UI and nothing else: no
model, no worker, no local database, no access token on disk. Execution still
happens on a paired [Worker](https://github.com/zccz14/cybion-worker); see the
Cybion documentation for the Worker model.

## Status

Early development. Built for personal use against the hosted Cybion service;
this is not distributed through an app store.

## What a shell does

- Loads `https://cybion.ntnl.io/` and keeps the session in the platform's own
  persistent website data store (so a sign-in survives restarts).
- Keeps navigation inside the shell for Cybion's own hosts and opens every
  other host in the system browser (Android) or the default browser (macOS).
- Shows a small progress bar and an in-app retry screen when the first page
  fails to load.
- Recovers from a renderer/web-content crash by reloading the page.
- Injects [`shell/ed25519-polyfill.js`](shell/ed25519-polyfill.js) before any
  page script so the session check passes on WebViews without Ed25519 in
  `crypto.subtle` — see [docs/compatibility.md](docs/compatibility.md).

## Requirements

- **Android**: Android 7 (API 24) or newer. The WebView must be updatable so
  the compatibility shim keeps working as browsers improve.
- **macOS**: macOS 12 (Monterey) or newer. On macOS 13.3+ the WebKit engine
  already supports Ed25519 and the shim is a no-op.

## Install

- **Android** — download the latest `cybion-app-android-universal.apk` from
  this repository's Releases and sideload it.
- **macOS** — download the latest `cybion-app-macos-*.dmg`, open it, and drag
  **Cybion** into Applications. See [docs/compatibility.md](docs/compatibility.md)
  if Gatekeeper warns about an unidentified developer.

## Build

### Android

```sh
cd android
export ANDROID_HOME=<sdk>
./gradlew :app:assembleRelease        # or :app:assembleDebug
```

Release signing reads `CYBION_ANDROID_KEYSTORE`, `CYBION_ANDROID_KEYSTORE_PASSWORD`,
`CYBION_ANDROID_KEY_ALIAS`, and `CYBION_ANDROID_KEY_PASSWORD` from the
environment. With no keystore configured the release build is left unsigned,
which is fine for local testing.

### macOS

```sh
cd macos
./build.sh                            # universal bundle in macos/build/Cybion.app
```

`build.sh` compiles a universal (arm64 + x86_64) binary, assembles the `.app`,
copies the icon and the shared polyfill into `Contents/Resources`, and ad-hoc
signs the result. Override the target with `ARCHS="arm64"` or `ARCHS="x86_64"`.

## Security model

A shell is a browser, not a sandbox, and it holds no secrets of its own.

- No model credential, no worker access token, no configuration file.
- The WebView is not granted cleartext traffic; only `https://` is allowed.
- It never overrides certificate errors and never disables Safe Browsing.
- Camera and microphone are surfaced to the page only after the OS asks the
  user, and only because the web app can use them.

The only privileged code is the injected shim. It is limited to the hosts in
`ShellConfig.ALLOWED_HOSTS` (Android) and is a few hundred lines of public
domain [TweetNaCl](https://tweetnacl.js.org/) — read it before trusting it.

## Documentation

- [docs/design.md](docs/design.md) — shell architecture and decisions.
- [docs/compatibility.md](docs/compatibility.md) — browser/OS support matrix,
  the Ed25519 shim, and when a shell is not needed.

## Related repositories

- [zccz14/cybion](https://github.com/zccz14/cybion) — the hosted controller
  and web app.
- [zccz14/cybion-worker](https://github.com/zccz14/cybion-worker) — the desktop
  Worker.
- [zccz14/cybion-worker-for-android](https://github.com/zccz14/cybion-worker-for-android) —
  turn a phone into a Worker.

## License

MIT
