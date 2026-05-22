# marmalade-tts-android-engines

CDN-style host for the TTS engine bundles consumed by
[marmalade-tts-android](https://github.com/maxwhipw/marmalade-tts-android).
The app downloads engine model + phonemizer files from this repo's
[GitHub Releases](https://github.com/maxwhipw/marmalade-tts-android-engines/releases)
at runtime, when the user opts in via the in-app onboarding wizard or
the Settings → Engines screen.

## Why a separate repo

Two reasons.

1. **License hygiene.** The Kitten model itself is Apache-2.0, but the
   accompanying `espeak-ng-data/` is GPL-3.0 (espeak-ng is GPL). Hosting
   the bundle here, in a repo licensed GPL-3.0 as a whole, keeps the
   main app repo's MIT licensing honest. The GPL'd content only lands
   on a user's device when they explicitly install an engine.
2. **Stability.** The previous Hugging Face mirror got moved/renamed
   (v0.1.0 / v0.1.1 of the app shipped against URLs that later started
   returning HTTP 401). Hosting on infrastructure we control means we
   set the URL contract and decide when it changes.

## What's here

- `kitten-nano-en-v0_1-fp16` engine — model + voices + tokens + the
  espeak-ng-data tree, file-by-file, attached as flat-named release
  assets on the [`v1`](https://github.com/maxwhipw/marmalade-tts-android-engines/releases/tag/v1)
  release.
- Path encoding for the espeak data: directory separators (`/`) are
  replaced with double-underscore (`__`) in asset names because GitHub
  release-asset filenames are flat. The app's `EngineInstaller` undoes
  this when writing files to disk, so the on-device layout matches the
  upstream Sherpa-ONNX bundle.

## License

This repository as a whole is licensed under **GNU General Public License
v3.0** (see [LICENSE](LICENSE)) because the espeak-ng-data is GPL-3.0.

The Kitten model files themselves (`model.fp16.onnx`, `voices.bin`,
`tokens.txt`) are Apache-2.0 upstream — but the bundle as distributed
here is treated as a single GPL-3.0 work for licensing purposes.

## Upstream

The engine bundle is mirrored from
[k2-fsa/sherpa-onnx releases](https://github.com/k2-fsa/sherpa-onnx/releases)
(`kitten-nano-en-v0_1-fp16.tar.bz2`). SHA-256 hashes of every file are
pinned in
[`EngineCatalog.kt`](https://github.com/maxwhipw/marmalade-tts-android/blob/main/app/src/main/java/app/marmalade/tts/install/EngineCatalog.kt)
and
[`KittenEspeakDataManifest.kt`](https://github.com/maxwhipw/marmalade-tts-android/blob/main/app/src/main/java/app/marmalade/tts/install/KittenEspeakDataManifest.kt)
so a tampered mirror would be rejected at install time.
