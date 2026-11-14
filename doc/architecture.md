# DiegeticAmbience - architecture and method

DiegeticAmbience is the atmosphere soundscape for S.T.A.L.K.E.R. Anomaly. Every map, every weather state, and every hour plays a living, audible ambience.
It is a curated work. The config is the source of truth, authored by hand: the channels, their pools, the presets, the level bindings.
Every sound here is mastered by a private build tool. This repo holds its output, the record and the ogg tree, beside the authored config.
`da_sound_metadata.script` is the corpus record AND the runtime trace table, one file. Each row carries the sound's measured facts and keeps no source pack or path.
Intake brings net-new sounds in. Refresh levels them from measurement, reads no author value, and regenerates the record. The machine never chooses content.

The soundscape carries its own dread layer. Distant mutant cries, far gunfire, spooks, and dark ambience play standalone.
Ownership splits on what a sound pretends to be (ruled 2026-11-14). A fake sound, a gunshot with no shooter, a monster with no monster, a scream or a spook, is horror.
Real fauna and nature are this mod's own: small birds, crows, owls, gulls, crickets, insects, frogs and rodents, with wind and weather.
Dogs and cats are horror, since each is a Zone mutant (blind dog, pseudodog, cat). A sound processed to sound eerie is horror too.
Horror stays in this mod too, in its own spook channels. The dread layer here holds only the fake sounds.
A directed horror layer is optional on top. Its director places dynamic horror cues, and its static exclusion takes the captured sounds out of the base channels at load, so the two never double.

## Content model

Dark Signal Amplified Soundscape is the spine, structure and content.
It is the only pack with a complete level/weather config (33 levels, all Atmospherics states, plus the extended maps), and it is the content default for every voice slot.
The spine is curated whole. Every ambience-scope file in the pack is kept, re-homed to a better channel, or excluded with a written reason.
That includes content its other sound systems own, such as the thunder corpus of its thunderbolt configs.
"The pack" means all of its sound systems, not only the ambient channels. The narrower definition is how an earlier build lost the entire thunder category to the prune stage without a trace.

Other packs add what the spine lacks or does weakly, always as channels, never as pool filler:

- Soundscape Overhaul: wind. The spine's own wind is muffled, its known weak spot, and SO's measured strongest.
- Audio Expansion: insects and frogs, content the spine does not have.
- Immersive Ambience: helicopter and wind reinforcement.
- Thunder, rebuilt from the spine's thunderbolt corpus plus SO's and AE's thunder folders.
  The distant rumble plays as storm-state ambient channels, and the close claps play through the lightning-strike system at the vanilla strike paths.
- The old era builds are mined for specific missing voices only. An original-game recording beats a re-edit at equal quality.
- Every other pack in the corpus (the other packs) is a bench. A family enters only by clearly beating the current holder of its slot, by measurement and by ear.

`doc/licensing.md` holds the licence basis per source. Sources are pulled locally by hand, and the pipeline never downloads.

## The channel model

The channel is the unit of curation.
One channel is one voice: one coherent recording family with its own placement (`min_distance`, `max_distance`, `height`) and its own cadence (`period0..3`).

- Pools are single-source by default: one pack's folder, files mastered together.
  Coherence is a verified property, established per pool.
  The packs' own folders are internally inconsistent, which is why the audibility pass exists.
  Every pool therefore passes a spread check (crest, spectral shape, noise floor) and the ear regardless of origin.
  Cross-pack pooling is a flagged exception, allowed only when measurement says the families are compatible and the ear confirms.
- Pool size: target 15-25 files (the ecosystem's own median-to-average band, measured across 7 packs), hard cap 40.
  A channel earns 26-40 only for a genuinely deep, uniform family.
  Past 40, overflow that forms a distinct character cluster becomes its own channel, and more-of-the-same is cut.
- The mechanical ceiling: one `sounds =` value is a single line read into a fixed 4096-byte buffer (`Xr_ini.cpp`).
  At the corpus average of 43 chars per token that is about 95 files. The `LINE_CAP = 3900` guard fails the build before the buffer can.
  A pool that outgrows the line continues in a sibling channel, `<name>__2`, with the same placement and cadence.
  Every preset state that lists the base also lists its sibling (`birds__2`, `crows__2`, `insects__2`, `bugs_day__2`, `bugs_night__2`, `bugs_swamp__2`, `frogs__2`, `ugrnd_ambient_new__2`).
- Channels fill ROLES, and roles are a fixed menu per state class (clear day, clear night, rain, storm, fog, underground).
  The menu holds a bed, a wind, birds or their night counterpart, insects, foliage, a rare-treasure slot, and weather-specific extras.
  A new channel exists only by taking an empty role in some state or by beating the role's current holder.
  The scale lands in the spine's own neighbourhood (its 94 channels rationalized plus the gap additions), not in the hundreds.
- When one voice needs different cadence per state (common in storms, rare in clear weather), a twin definition carries the same pool with different periods.
  Twins are used sparingly, and their pool lines are kept identical by hand.

## Density

Vanilla is the reference, and the invariant is hard (ruled 2026-11-14).
On every level and weather state, the Lua channels start 1.0x to 1.3x as many sounds per minute as vanilla's presets start there: never fewer, and at most 30% more.
Richness comes from the variety of the pools, not from playing more often.
An extra map from an addon, which vanilla lacks, takes the median vanilla rate of the levels that share its preset.
The rule measures this mod alone. A horror layer that switches channels off is a separate mod, and those channels do not count against it.

The rate follows vanilla's Lua channel loop in `sound_ambient.script`:

- `reset_settings()` builds the round from the state's distinct defined `sound_channels_dynamic` channels. `update_ambient()` checks one of them per second, and each channel gets one turn per round.
- A channel re-arms with one of its four periods at random and plays at its first turn after that. Its gap is the round length times the turns the period needs.
  Its rate is 60 over the mean of its four gaps.
- A placeholder channel (`ambient\no_sound`) takes a turn and plays nothing.
- Beds play outside the round and do not count. xray arms a bed's first play from `period0..1` and each later one from the sound's length plus `period2..3`.
  The timers live in `CEnvAmbient::SSndChannel` (`Environment.h`) and `CGamePersistent::WeathersUpdate()`.

Three further rules keep many channels from becoming a wall:

- No fixed interval: a channel whose four periods sit within one round plays every round, in the same order as its neighbours, a pulse the ear hears.
  Its longest period is at least 3x its shortest, and the gaps then differ. Vanilla's `wind_normal` runs 1/20/20/40 s.
- Distribution over stacking: a channel does not join every state its category appears in.
  Variety spreads across time and place (wind-howl in fog and night, wind-gust in day, the metal groan only in the deep underground presets), and concurrency per state stays flat.
- Rare-voice channels carry the treasures: periods of 120000-300000 ms put a one-in-a-hundred find in the world at almost no budget cost.

## Curation method

Division of labor:

- The curator (assistant) works from evidence: the measurement tables (LUFS, crest, duration, native channel count, rate), the source packs' own wiring, and the dedup maps.
  What channel the pack's author bound a file to is evidence. Ignoring it is how thunderclaps ended up in a wind channel.
  The curator drafts family-level A/B proposals, the channel vocabulary, and the wiring plans.
- The ear (the user) closes every contested call through the player (`ui_da_player.script`): family vs family for a slot, the contested middle of a ranking, the density ceiling over vanilla.
- The pipeline masters and proves. It never chooses.

Every rejected sound carries a written reason in the private build history. Nothing is deleted by inference.

## Audibility method

Community ambient corpora sound fine at-ear yet die in-game, for engine reasons (`SoundRender_Core.cpp`, `SoundRender_Emitter_FSM.cpp`):

1. A STEREO ogg force-plays 2D at-ear, escaping distance rolloff and occlusion. Only MONO spatialises.
2. Every 3D voice attenuates twice, through X-Ray's linear fade AND OpenAL's inverse model keyed on the ogg blob's `min_distance`.
   An unset min of 1-2 costs -26..-31 dB at a 25-50 m placement before the linear fade applies.
3. Content loudness is a separate floor the distance fixes cannot reach.

Both Diegetic mods share one mastering model, and it reads NO author value in any blob field.
The min is a crest-keyed floor. base_volume starts at 1.0 and moves into the floor/ceiling loudness band off the measured LUFS. The max is the content's reach.
This mod takes the reach from its own hand-authored channel config, which also places the sound (below).

The mastering rules, scoped by playback path:

- Fold to mono applies to ALL 3D-played audio, ambient channels and strike files alike.
  The fold is the one lossy step. Off-rate files are resampled to 44100, because the engine hard-rejects any other rate (`SoundRender_Source_loader.cpp`).
  The source blob is kept as an audit column only, never read by the master.
- Two schedulers play a channel, and the master evaluates each sound at the farthest place either one puts it.
  System B, vanilla's `sound_ambient.script`, places a dynamic channel at `random(min', max) / 2` plus the channel height.
  System A, the engine bed, places a preset's `sound_channels` at `randF(min, max)` and 10 m up (`GamePersistent.cpp`).
- The max is the farthest referencing channel's `max_distance`. A sound any bed plays gets a 300 m max instead, vanilla's own value on its trx beds and the engine default.
  The min is a crest-keyed ratio (0.40-0.60) of the farthest placement, capped below the max.
  Strike files are excluded. The engine overrides their range per strike (see Thunder below), and blob distances do nothing there.
- Every bed-played file is typed `WORLD_AMBIENT` in its blob, like vanilla's own beds. Beds take the type from the blob (`Environment_misc.cpp`).
  Only that type skips the occlusion ray (`SoundRender_Emitter_FSM.cpp`), and a wall then never muffles the weather.
- The loudness band is per class, both arms lossless and partial. A bed is levelled toward -30 to -24 dB delivered, a single cue toward -36 to -28.
  The class comes from the file's path. Wind, storm, rain, thunder, fog, background, drone and rumble names count as beds.
  A floor lifts a file whose delivered loudness at its placement sits below the band, and a ceiling lowers a file above it. A file is only ever in one arm.
- Dead audio is reported, never auto-deleted. A file at or under -60 LUFS leaves only by a hand reject.

All blob edits are lossless (audio pages byte-identical). The fold and the off-rate resample are the only re-encodes. They exist because the engine positions mono only and loads 44100 only.

Blob contract (the engine-read fields).
The comment is a `0x0003` X-Ray struct of five fields: `min`, `max`, `base_volume`, `game_type`, `max_ai_dist` (`SoundRender_Source_loader.cpp`, the `0x0003` branch).
The rewrite sets all five, and the engine reads exactly these. Nothing written is ignored, and nothing read is left unset.
`min`, `max`, and `base_volume` carry the leveling above (linear rolloff plus the OpenAL inverse keyed on `min`, and the `base_volume` gain multiplier, `SoundRender_Emitter_FSM.cpp`).
`game_type` is `WORLD_AMBIENT` on every bed-played file and keeps the file's own value elsewhere. `max_ai_dist` is written equal to `max`.
A script play overrides the blob `game_type` (`SoundRender_Core.cpp`), and System B plays are script plays.
`max_ai_dist` only sets NPC hearing range, and a bed has no owner to make NPCs hear it (`SoundRender_Emitter.cpp`).
`max_ai_dist` is set to `max` only to satisfy the loader's `>= 0.1` assert.

## Thunder

Thunder has two homes, matching the engine's two systems:

1. Distant rumble: ambient channels in the rain/storm states (`thunder_far`), curated and mastered like any voice.
   The `pre_storm` sections carry it too, but no stock or Atmospherics keyframe emits that state (see The five-link binding chain).
2. Strike claps: the engine thunderbolt system.
   The weather mod (Atmospherics) drives timing per weather cycle (`thunderbolt_collection`, `thunderbolt_period`, `thunderbolt_duration` in `weathers/w_*.ltx`).
   The collections resolve to sections in `thunderbolts.ltx`, and each section's `sound =` names a path under `sounds\nature\`.
   DiegeticAmbience ships its curated strike recordings AT those vanilla paths, under `sounds\nature\`.
   The best claps play with no config, and the weather mod's tuning stays intact.
   The engine plays each strike positioned at the bolt with a speed-of-sound delay and a per-strike attenuation range
   (`thunderbolt.cpp`: `snd.play_no_feedback(0, 0, dist / 300.f, &pos, 0, 0, &Fvector2().set(dist / 2, dist * 2.f))`).
   That is why strike files need selection and loudness only, and no placement engineering.

## The effect layer

System A carries a second output besides the bed: ambient EFFECTS, a particle burst plus a wind blast plus one recording, fired outdoors on the preset's `min/max_effect_period` timer.
`CEnvAmbient::load` reads the `effects =` key, and the play block in `CGamePersistent::WeathersUpdate` is gated on outdoor luminocity.
Vanilla wires `effect_0..9` into every outdoor state: fog wisps and gust particles with the trx `wind_gust` recordings.

The layer is dead across the AtmosFear-lineage packs (found 2026-09-03).
The Dark Signal configs strip the `effects` keys from every preset, and Atmospherics' `effects.ltx` re-points all ten sounds at `nature\wind_01..10`, files that exist in no db archive and no mod.
Where the keys survive (the 304 field preset), the sounds play as silence.

DiegeticAmbience restores both halves.
Every outdoor state of every preset wires `effect_0..9`.
The DLTX overlay `mod_effects_diegeticambience.ltx` points the ten `sound =` keys at vanilla's proven trx recordings, winning over whichever `effects.ltx` is active.
The tree ships the nine `ambient\trx\nature\wind_gust` files the override names. No channel lists them, and verify accepts them because the override names them.
Underground presets keep empty `effects` (the play block never fires indoors, vanilla parity).

The surge beds are the same class of repair.
`blowout_channels.ltx` inherited `blowout_impacts`, `blowout_rumble`, `blowout_ambient` and `blowout_flare` muted, because their only source was vanilla itself.
The tree now ships the 21 vanilla `ambient\trx\blowout` recordings plus later surge recordings, and the four pools play again during emissions.

## Deduplication

Deduplication runs twice, both on our side, never against the target install. A byte hash collapses identical reships.
fpcalc (Chromaprint) fingerprints the deployed audio and reports same-recording aliases the byte hash cannot see. Short clips fall back to the byte hash.
In the authored-config model both run as warners. A duplicate pick is reported for the curator to resolve by hand.

Those two are build-time.
Runtime deduplication is separate and lives in `da_dedup.script`, a listener on the xlibs script-sound seam.
System B picks a channel's sound uniform-random with replacement, so it can replay the same file back to back.
On a repeat within 20s the listener vetoes that play and reissues a fresh sibling from the same channel at the same position.
The seam fires before the caller sets its volume.
The sibling carries the volume the base layer would have given it: the channel's indoor table times the ambient slider, as `sound_ambient.script` computes it.
It stays silent when the channel has no unplayed sibling or that volume is 0 (an indoor channel heard outdoors).
It filters to our ambient files through the channel index (non-ambient script sounds pass untouched) and is inert on an exe without the seam.
Unlike `da_diag`, which observes and returns nil, this consumer returns a veto.

## The five-link binding chain

An ambient sound reaches the player through five links, and the verifier proves each resolves:

1. Weather state: the active weather set's `weathers/w_*.ltx` sets `ambient = <state>` per time frame.
   The vocabulary is the AtmosFear set: day, morning, evening, night, rain, rain_day, rain_night, storm_day, storm_night, tuman, tuman_night, and indoor_underground.
   Stock Anomaly 1.5.3 and Atmospherics emit exactly this set (both weather trees swept 2026-09-03), so the mod runs on either with no variant.
   The presets also carry `pre_storm` and `tuman_day` sections, the vanilla preset shape. No stock or Atmospherics keyframe emits them, so they stay inert until a weather mod uses those states.
2. Level to preset: `ambients/<level>.ltx` is a one-line `#include` binding each map to a preset.
3. Preset to channels, per state: one section per weather state with `sound_channels` (the bed) and `sound_channels_dynamic` (the layers).
4. Channel to pool: `ambient_channels/backgrounds.ltx` and `sound_channels.ltx`.
5. Pool to ogg: the mastered audio files.

X-Ray lowercases section names on load (`strlwr(section)`, `Xr_ini.cpp`), so references resolve case-insensitively.
The weather layer couples at exactly three vocabularies: the ambient state names (link 1), the thunderbolt collection names (Thunder, home 2), and the effect ids (The effect layer).
The links below those three are weather-mod-independent. Variants for other weather mods re-cover all three vocabularies.

## Verification

`verify` emits the gate ledger, and the build fails unless every gate is clean:

1. No orphan file: every shipped ogg sits in some channel's pool, the strike set or the effect override. A new sound stays an orphan until it is wired or rejected.
2. No orphan channel: defined but unreferenced (informational, flagged as dead weight for the curator).
3. No dangling channel ref: every channel named in a preset is defined.
4. No missing sound path: every pool entry resolves to audio on disk.
5. Level routing closed: every map binds a preset, underground maps to underground presets.
6. Weather matrix full: every preset defines a section for every state the active weather mod emits.
7. Record closure: every row has its ogg, every ogg has its row, and every ogg's audio hash matches its row. A hand-edited file fails.
8. Density: every level and state starts 1.0x to 1.3x vanilla's sounds per minute, counted the way the Density section describes.
   It runs when the unpacked vanilla tree is on the verify command line, and lists the channels on a fixed interval.
9. Line cap: no `sounds =` line approaches the 4096-byte ini buffer (`LINE_CAP = 3900`).
10. Collection coverage: every thunderbolt collection name the active weather mod references resolves in the base game's collection set.
11. Exclusion simulation: no channel that the directed horror exclusion touches may end EMPTY at load.
    The intersection itself is designed coexistence (the exclusion exists so base channels do not double the director's captured sounds).
    The hand-authored exclusion overlay carries `>sounds = ambient\no_sound` on every touched channel.
    A fully-excluded channel then plays silence.
    A System A bed with no `sounds` key is a load failure (`Environment_misc.cpp`), which is exactly what that guard prevents.
    The gate FAILs only if a touched channel lacks the guard. It runs when verify is given the horror layer's overlay file.
12. Level coverage: every playable base-game level (`LEVELS_BASE`, the `game_maps_single.ltx` set minus `fake_start`) binds an `ambients/<level>.ltx`.
    An unbound level plays vanilla wiring through the MO2 VFS (the 9 labs, found 2026-09-03) or a bare `ambients.ltx` fallback with no dynamic layers (`y04_pole`).
    Bindings outside the base set (extended maps) are not checked.
13. Bed load asserts: every channel any preset or `ambients.ltx` names as a bed satisfies `SSndChannel::load` (`Environment_misc.cpp`):
    `max_distance > min_distance` strict, `period0 <= period1`, `period2 <= period3`, non-empty `sounds`. A violation is a CTD on level load.
14. Dynamic completeness: every channel named in any `sound_channels_dynamic` defines all 4 periods and both distances, or `sound_ambient.script` nil-errors and breaks that hour's rotation.
15. Indoor routing: no `indoor = true` channel is wired into an outdoor state, where the System B volume table (`sound_ambient.script`) plays it at 0.0.
    Outdoor channels inside underground states (played at 0.3) are reported, not failed.
16. Strike palette: retired with the earlier build. The current tool has no gate 16.
17. Effect vocabulary: every effect id any preset or `ambients.ltx` wires exists in the base effect set (`effect_0..9` + `blowout_effect_01..48`).
    An unknown id is a CTD. `create_effect` reads `life_time` with a throwing `r_float` (`Environment_misc.cpp`).
18. Effect sound paths: every `sound =` in the effect override resolves to a file on disk.
    No channel reads the override, so gate 4 never sees these paths. A dangling one plays silent, since `WeathersUpdate` skips a null handle, with no other trace.

## Coexistence with the directed horror layer

The directed horror layer captures the dark/horror content from the shared source packs into its own `zs/` tree.
It removes its captured sounds with a hand-authored static DLTX overlay: `mod_sound_channels_diegeticdread.ltx`, individual `<sounds` removals per channel plus a `>sounds = ambient\no_sound` guard.
DLTX applies that overlay to OUR resolved `sound_channels.ltx`.
A sound of ours in one of our pools is stripped at load, a fully-captured channel plays silence, and gate 11 proves the composition stays safe.
That director places the horror, and this config plays the living ambience.
Standalone, nothing is excluded and the full dread layer plays. With the horror layer installed, the directed layer replaces the captured subset.

## The mastering mill

The build tool reads the hand-written config as its input and does only what needs a machine.
Every command takes a source tree and a target tree and names no mod. Copying a finished target over this repo is a manual step.

- intake reads a sound pack, and it is the only command that does. It folds stereo to mono and resamples off-rate files, then dedups inside the pack and against the tree by audio alone.
  Every net-new sound goes into the tree at its original relative path, listed by no channel. Verify fails while any such file remains.
  Intake leaves out and reports a path that the tree already holds with different audio. A failed intake writes nothing, and a second intake of the same pack changes nothing.
- wire: the curator lists each new sound in a channel of the authored config, picked from the source pack's own wiring and the analysis report, or rejects it.
  A sound can also join a new channel, and the presets then list that channel in the states it suits.
- refresh never reads a pack and never rewrites the authored config.
  It reports a channel that lists one sound twice, since that doubles the sound's odds, and fails any pool line over the ini buffer cap.
  It recomputes every wired row's blob from measurement under the model above. A constant tunes in both directions, and nothing ratchets.
  Measured values are computed once per audio identity. Strike files and staged files keep their bytes.
  It fills each row's placement columns (`ch_min`/`ch_max`, `indoor`, `h`) from the authored channel config, then regenerates `da_sound_metadata.script`.
  The engine never reads that file. The mod loads it once into `xsound.load_meta` for the delivered-loudness readout.
  It ends with verify (the gate ledger above) and the delivered-loudness audit, and a second refresh writes nothing.
- reject deletes one sound, its ogg and its row. The tool keeps no reject memory, and a later intake can bring the audio back for a new judgement.
- stage and unstage put a candidate family under `sounds/stage/` for an in-game audition. Verify fails while a stage tree is present.
- import is the one-time config baseline from a pack's own sound-routing config, the bootstrap for a fresh variant. It refuses over an existing config.

One `ffmpeg` `astats`+`ebur128` pass reads `lufs` (integrated), `crest` (peak minus RMS), and `peak` (sample peak).
`ffprobe` reads `dur`, `sample_rate`, and `channels`.
Chromaprint reads the fingerprint `fp`, and the fold reads mid/side RMS. Each drives one build step: `lufs`/`crest` the loudness band and the crest-keyed min floor, `sample_rate` the 44100 resample,
`channels` the fold, mid/side RMS the sum-vs-drop split, `fp` the acoustic-duplicate report, and `lufs` also the dead report, while `peak` and `dur` are recorded per row.
Part of that set stays in the record for the game.
At runtime `lufs`/`crest`/`peak`/`bv`/`mn`/`mx` are the `da_sound_metadata` set above, read by the wiring inspector and the sound player through `xsound.compute_delivered_loudness`.

The generator stages of the earlier build (config synthesis, folder-dump grafts, spine path priority, prune-by-inference) are deleted, and the in-place level ratchet with them.
The old tool read its own last output as the author base, so floors could only rise.

## Invariants

- I1 Scope. Nature/weather ambience, the ambient dread layer, storm thunder. Directed cues are the horror layer's. Emission and psi-storm are their own systems. Strike timing is the weather mod's.
- I2 The config is authored. Machines level, report, and prove. They never choose content.
- I3 Spine completeness. Every ambience-scope file of the spine, across all its sound systems, was kept, re-homed or rejected with a reason in the private history. Nothing dies silently.
- I4 One channel, one voice. Single-source pools by default, coherence verified, 15-25 target, 40 cap, roles-menu admission.
- I5 Density is conserved. The per-state budget is a gate, not advice. The entry-burst stagger is held by hand.
- I6 Audible by measurement. Mono fold, crest-inverted min floor, per-category loudness band, placement caps, scoped by playback path.
- I7 Preserve the source except where the engine forbids it. Blob-only edits, audio pages byte-identical, the fold and the resample as the only re-encodes.
- I8 Deduplicate twice, warn, let the curator resolve.
- I9 Config closed. The full gate set passes or the build fails.
- I10 Weather-mod-bound at three vocabularies (ambient states, collection names, effect ids). Stock Anomaly and Atmospherics share all three. Other weather mods need a sweep of all three first.
- I11 Licensed, one record. The record keeps no per-sound origin and is ONE file, `da_sound_metadata.script`, the same file the mod loads.
  `licensing.md` records the basis for every source, and the readme credits every author.
- I12 No audible sound leaves by a rule. Measurement only flags candidates: the dead report, the faint list and the analysis lists.
  Each loads into `ui_da_player` as a playlist, and a file leaves only by a hand reject.
  Intake reports a file whose fold or resample fails and never ships it. That is the one mechanical removal.

## Scripts: dependency gate, MCM, diagnostics, player (no gameplay)

- `_da_manifest.script` holds the identity data (name, version, required xlibs).
- `_da_init.script` holds the xlibs + modded-exes floor asserts, the platform line, and the boot banner. It reads the identity from `_da_manifest`.
- `da_mcm.script` is the informational MCM. There is no master volume slider: the build levels loudness into the blob, and the game's ambient slider sets the overall level.
- `da_debug.script` holds the xlog logger and the level gate.
- `da_diag.script` is the runtime wiring inspector (active level, weather, ambient state, per-state channel counts, live dangling-ref count) AND the runtime sound trace.
  The trace subscribes through the xlibs seam registry to the 6 demonized sound callbacks (bed, script-sound, effect, thunderbolt, rain, level-music).
  It logs each fire with its resolved channel, file, and delivered acoustics (distance to the actor, delivered dB, lufs, crest).
  Those come from the same `get_meta` / `compute_delivered_loudness` calls the player uses, gated on `da_debug.is_on()`.
  The `vd` token grades each sound against the floor the build masters it against: -30 for a bed, which a path holding a bed word marks (wind, storm, rain and the rest), and -36 for every other sound.
  `da_dedup` logs its own decision on the same channel (repeat replaced with which sibling, or silenced). A session log then shows what played, how loud and far, and what the dedup did.
  It is a no-op on a stock exe, where the seam returns false and stays detached. It only observes and returns nil.
  This is the runtime counterpart to the static wiring dump, and the ground truth for the density and repetition rulings.
- `ui_da_player.script` is the curation instrument, mirroring the horror layer's player shape.
  It is a keyboard-owning modal on PageUp (the horror layer keeps PageDown), gated by the MCM sound_player toggle.
  It browses BY CHANNEL from the resolved `sound_channels.ltx` and auditions where the game plays the channel, at-ear, and at fixed distances.
  The level's presets say which scheduler plays a channel: a Lua play at random(min', max) / 2 up the channel height, a bed at randF(min, max) 10 m up, or both.
  The near and far modes stand at the nearest and the farthest of those plays, and the felt mode at the middle of the Lua roll.
  Auditions play through `da_dedup.play_unfiltered`, past the repeat veto, and each plays exactly the sound picked.
  It steps within pools and staged candidate families for A/B rulings.
  Each audition shows a delivered-loudness readout (measured LUFS, base_volume, est. dB at the audition distance) from `xsound.get_meta` / `compute_delivered_loudness`, fed by `da_sound_metadata`.
  Ear verdicts go to `diegeticambience_notes.txt`, an investigation log that nothing parses.
  It is an own copy by family precedent, and it moves to xlibs only when a third consumer exists.

## Deploy

A gamedata overlay. The repo holds the authored config, the mastered audio, the mill, and the docs. Wiring for local sync goes through `stalker-manager`.
Every source carries a free licence or the author's permission before public release (see `licensing.md`), and the readme credits each author.
