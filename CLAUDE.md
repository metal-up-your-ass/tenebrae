# Tenebrae — high-gain rhythm distortion (guitar)

Per-repo working memory for Claude Code sessions on this plugin. Part of the **Metal up your ass** symphonic-metal plugin suite (`github.com/metal-up-your-ass`).

## What this is
Tenebrae is the "high-gain rhythm distortion (guitar)" member of the suite. AU / VST3 / Standalone, JUCE 8.

## Status (v0.1 — bootstrap complete)
Core DSP working, **26 Catch2 tests green**, CI (macOS + Windows, pluginval strictness 10 + auval) green. GUI is a functional v0.1 slider editor (custom LookAndFeel is roadmap M3). No signing yet (roadmap M4). Open work is tracked in this repo's GitHub **milestones/issues**.

## DSP
Signal chain: input -> Tight HPF (20-300 Hz, default 90 Hz, 2nd-order Butterworth IIR) -> Gain (0-40 dB pre-gain) -> 8x oversampled 3-stage waveshaper cascade (each stage: fixed internal drive -> AsymmetricClipper tanh nonlinearity with progressively larger asymmetry -> fixed interstage HP/LP pair, tightening/darkening stage by stage: 80/9000 Hz, 120/6500 Hz, 150/5000 Hz) -> passive-style 3-band ToneStack (Bass low-shelf @150 Hz, Mid peak @800 Hz, Treble high-shelf @3000 Hz, all +/-15 dB) -> Level (-24..+24 dB) -> DryWetMixer (0-100%, delay-compensated). Oversampling is 8x (2^3) half-band polyphase IIR with useIntegerLatency=true; TenebraeEngine::getLatencySamples() is reported via setLatencySamples() in prepareToPlay(), and the dry path is time-aligned via DryWetMixer::setWetLatency(), primed with setWetMixProportion() before reset() per the documented JUCE 8.0.14 DryWetMixer gotcha. Ported/completed from the partial gain-forge prototype (AsymmetricClipper, CascadeStage, ToneStack, ParameterLayout — verified correct, ID/range values already matched the Tenebrae spec) by adding the missing TenebraeEngine orchestration class plus PluginProcessor/PluginEditor/tests.</dspSummary>

## Build & test
```sh
export CPM_SOURCE_CACHE="$HOME/.cache/CPM"      # shared JUCE 8.0.14 + Catch2 cache
cmake -B build -G Ninja -DCMAKE_BUILD_TYPE=Debug
cmake --build build --target Tests Tenebrae_Standalone --parallel 4
ctest --test-dir build --output-on-failure
```
Release/universal + pluginval + auval run in CI, not locally.

## Conventions & guardrails
- JUCE 8.0.14 via CPM · C++20 · AGPLv3 · Pamplejuce `SharedCode` pattern · manufacturer `Yvsv`, plugin code `Tnbr`, `com.yvesvogl.tenebrae`.
- **Real-time safety:** no alloc/lock/file-IO/logging on the audio thread; allocate in `prepareToPlay`; `reset()` clears all state; `ScopedNoDenormals`; smoothed params; report latency via `setLatencySamples` where the chain adds any.
- **DryWetMixer gotcha (JUCE 8.0.14):** prime `setWetMixProportion(mix)` before `reset()` in `prepare()` (else it ramps from 100% wet). See sibling `overture`.
- **`main` is protected** — no direct commits; feature branch + PR, green CI required (Conventional Commits). New DSP needs tests (null/reference, NaN/Inf sweep, state round-trip, latency).

## Roadmap
GitHub milestones (M1 DSP & tests · M2 presets/state · M3 GUI & a11y · M4 release/signing/v1.0.0) + issues. Read with `gh issue list --repo metal-up-your-ass/tenebrae`.

## Suite context
Style references: sibling `metal-up-your-ass/overture` and `metal-up-your-ass/twist-your-guts`. The suite: overture, tenebrae, nave, silentium, requiem, seraph, aureate, firmament, triptych, apotheosis, twist-your-guts.
