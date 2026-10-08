# DiegeticAmbience: a curated, audible ambient soundscape for STALKER Anomaly

An ambient soundscape built from the original GSC games, the standalone builds that reworked them (Solyanka, OGSR, OGSE, Dead Air, Lost Alpha) and the major soundscape packs, with every file corrected to be audible at the distance the engine places it.
The channels and their pools are curated by hand, one recording family per voice, and a mastering pipeline folds every file to mono, sets its distance and loudness from ffmpeg measurements, and proves every map plays in every weather state.

[Releases](https://github.com/damiansirbu-stalker/DiegeticAmbience/releases) | [Bugs, suggestions](https://github.com/damiansirbu-stalker/DiegeticAmbience/issues)

[![ci](https://github.com/damiansirbu-stalker/DiegeticAmbience/actions/workflows/ci.yml/badge.svg)](https://github.com/damiansirbu-stalker/DiegeticAmbience/actions/workflows/ci.yml) [![Project Health](https://img.shields.io/badge/project_health-dashboard-00ced1)](https://damiansirbu-stalker.github.io/DiegeticAmbience/)

Requires: Anomaly 1.5.3, modded exes (themrdemonized or AOEngine), [xlibs](https://www.moddb.com/mods/stalker-anomaly/addons/xlibs-1001). No weather mod is required: stock Anomaly weather and Atmospherics emit the same ambient states, both covered. Exact versions in [readme.txt](doc/readme.txt).

## My work

- Alife mods: [AlifePlus](https://www.moddb.com/mods/stalker-anomaly/addons/alifeplus-v1-0-01) · [AlifeTactics](https://www.moddb.com/mods/stalker-anomaly/addons/alifetactics) · [AlifeBalance](https://www.moddb.com/mods/stalker-anomaly/addons/alifebalance) · [AlifeGuard](https://www.moddb.com/mods/stalker-anomaly/addons/alifeguard-1001)
- Diegetic mods: [DiegeticControl](https://www.moddb.com/mods/stalker-anomaly/addons/diegeticcontrol) · DiegeticAmbience · DiegeticDread
- Tools: [JitProfiler](https://www.moddb.com/mods/stalker-anomaly/addons/jitprofiler)
- Libraries: [xlibs](https://www.moddb.com/mods/stalker-anomaly/addons/xlibs-1001)
- Engines: [X-Ray Monolith](https://github.com/themrdemonized/xray-monolith/pulls?q=is%3Apr+author%3Adamiansirbu+is%3Amerged) · [OpenXRay](https://github.com/OpenXRay/xray-16/pulls?q=is%3Apr+author%3Adamiansirbu+is%3Amerged)
- Integrations: [Word of Mouth](https://github.com/joshcoppola/word_of_mouth) · [Warfare (erepb)](https://www.moddb.com/mods/stalker-anomaly/addons/warfare-alife-overhaul-new) · [Stealth Overhaul](https://github.com/Alex-leon1594/Stealth_Overhaul_Reworked) · [COMPASS](https://github.com/Crimento/COMPASS)
- Collaborations: [xAGNA](https://www.moddb.com/mods/stalker-anomaly/addons/xagna)

## Documentation

- [readme.txt](doc/readme.txt) - full description, sources, build, credits
- [architecture.md](doc/architecture.md) - method, invariants, build pipeline
- [licensing.md](doc/licensing.md) - per-addon licence and permission

## License

PolyForm Perimeter License. See [LICENSE](LICENSE).
