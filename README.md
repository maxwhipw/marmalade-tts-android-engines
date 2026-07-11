# marmalade-tts-android-engines

CDN-style host for the TTS engine bundles consumed by
[marmalade-tts-android](https://github.com/maxwhipw/marmalade-tts-android).
The app downloads engine model + phonemizer data from this repo's
[GitHub Releases](https://github.com/maxwhipw/marmalade-tts-android-engines/releases)
at runtime, when the user opts in via the in-app onboarding wizard or
the Settings → Engines screen.

## Why a separate repo

1. **License hygiene.** The models are Apache-2.0/MIT, but the
   accompanying `espeak-ng-data/` is GPL-3.0. Hosting the bundles here,
   in a repo licensed GPL-3.0 as a whole, keeps the main app repo's MIT
   source licensing honest. The GPL'd content only lands on a user's
   device when they explicitly install an engine.
2. **Stability.** An earlier Hugging Face mirror got moved/renamed and
   started returning HTTP 401 under old app versions. Hosting on
   infrastructure we control means we set the URL contract. Release
   assets are **immutable once an app version pins them** — the app
   verifies a SHA-256 per archive, so replacing an asset in place would
   break installs. New content always means a new release tag.

## Current bundles (per-release tar archives)

Each release attaches one archive per engine, extracted by the app's
`EngineInstaller` (SHA-256 verified, zip-slip guarded). The app's
[`EngineCatalog.kt`](https://github.com/maxwhipw/marmalade-tts-android/blob/main/app/src/main/java/app/marmalade/tts/install/EngineCatalog.kt)
pins the exact release URL + hash it consumes — that file is the source
of truth for which release each app version uses, and a tampered mirror
would be rejected at install time.

| Bundle | Contents |
|---|---|
| `kitten-direct-v0_8` | KittenML nano 15M ONNX + 8 voice embeddings + `phonemizer/espeak-ng-data` |
| `kitten-direct-mini-v0_8` | KittenML mini 80M ONNX + the same 8 voices + espeak data |
| `kokoro-direct-v1_0` | Kokoro v1.0 53-voice multi-lang ONNX + `voices.bin` + espeak data + `lexicon-zh.txt` (Mandarin) + `openjtalk_dic/` (Japanese, BSD-3 naist-jdic) |
| `pocket-tts-en-v2026_04-mixed` | Kyutai Pocket TTS 5-graph ONNX set (int8 transformer / fp32 mimi) + 6 CC0/CC-BY voices |

**No executable code at rest** (v22+): bundles carry models and data
only. espeak-ng itself is compiled from source into the app's APK;
releases up to v21 carried a legacy `libttsespeak.so` the app never
loaded, removed in the v22 re-spin.

## GPL-3.0 §6 provenance (espeak-ng-data)

The `phonemizer/espeak-ng-data/` tree in the Kitten and Kokoro bundles
is built from:

> **https://github.com/espeak-ng/espeak-ng** at tag **1.52.0**
> (commit pinned as the app repo's `third_party/espeak-ng` submodule —
> the same tag whose library is compiled into the app APK)

Build: CMake `data` target (`cmake <src> && make data`) on x86_64
Linux; the compiled dictionaries are architecture-independent. To
reproduce, check out the tag and run that target — the output
`espeak-ng-data/` directory is what ships, unmodified.

Bundles up to v21 instead carried data derived from Debian's
`espeak-ng 1.51+dfsg` package (runtime-compatible with the 1.52.0
library; disclosed in the app's NOTICE.md at the time).

## Model provenance

- **Kitten** models: [KittenML](https://github.com/KittenML) (Apache-2.0).
- **Kokoro** v1.0: [hexgrad/Kokoro-82M](https://huggingface.co/hexgrad/Kokoro-82M)
  (Apache-2.0); `openjtalk_dic` is Open JTalk's naist-jdic (BSD-3);
  `lexicon-zh.txt` is sherpa-onnx's pre-baked Han→IPA table (Apache-2.0).
- **Pocket TTS**: [Kyutai Labs](https://github.com/kyutai-labs/pocket-tts)
  (MIT model code; voices CC0/CC-BY-4.0 — the two CC-BY-NC-4.0 upstream
  voices are excluded from v21+).

Per-component license texts ship inside the app (Settings → About →
Open-source licenses) and in the app repo's `LICENSES/`.

## License

This repository as a whole is licensed **GPL-3.0** (see
[LICENSE](LICENSE)) because the espeak-ng-data it redistributes is
GPL-3.0. Individual model files remain under their upstream licenses
listed above.
