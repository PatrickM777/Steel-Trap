# Steel Trap — Iteration 3 (v4) Build Log

Base: `build-v3/steel-trap.html` (268,643 bytes, validated) copied to `build-v4/steel-trap.html`.
Final: **407,585 bytes / 6,285 lines**, single self-contained HTML file, same `steeltrap_*`
storage keys — all v2/v3 user data (records, presets, binds, stimuli) carries over.

## 1. Setup screen (dedicated, shared config)

- New top-level **SETUP** nav tab (`#sec-setup`): full configuration — all modality
  toggles + distractors, N, interference (classic/group split, see §6), interval
  (slider + typed), flash, duration (typed M.SS + minutes + slider), reverse/mirror
  rule toggles with per-modality reverse opt-ins, stimulus-library shortcut, music toggle.
- **One shared config state**: Setup fields and the Arena panel read/write the same
  `cfg-*` fields through `getFormSettingsObject()` / `applySettingsObject()`.
  Entering Setup calls `syncSetupFromConfig()`; every Setup edit calls
  `syncConfigFromSetup()` → `applySettingsObject()`. Smoke-verified both directions:
  typing `12.30` (M.SS) in Setup sets Arena minutes to 12.5 and the summary line;
  changing N in the Arena reflects into Setup.
- Arena header gained a **config summary line** (modes, N, duration, speed,
  interference systems, audio set, media set, rule, adaptive, override note) that
  refreshes through `updateActiveModeHUD()` → `updateConfigSummary()`.

## 2. SPECT-scan brain aesthetic (Anatomy + Profile)

- Both brain instances restyled: near-black deep-blue panel, **thermal palette**
  (`getThermalColor`: deep indigo → violet → blue → cyan → green → yellow → orange →
  red → white-hot), SVG glow pass (blurred copy under crisp regions, `feGaussianBlur`
  filters), radial sheen overlay, scanline pattern overlay, softened region strokes.
- Emphasis values map onto the thermal scale (low = deep blue/purple, high =
  orange/red/white). Profile **performance mode** colors regions by tier color on
  the same scale. Hover/selected bright outline, drag-rotate views, zoom, peel,
  underneath callout and hidden-areas readout all preserved.
- Caption added under each brain: **"Stylized SPECT-scan look — illustration, not a
  medical scan."**

## 3. Profile performance tiers

- Toggle in Living Brain Profile: **Training emphasis** (unchanged) vs
  **Estimated performance**. Per-region score from records via the existing
  region-weight table: per modality, best N achieved, mean accuracy and derived d′
  at that N, mean interference handled, session count. Region score =
  `78 · skill^0.62 · (1 + 0.10·ln(1+sessions))` — compressive but **non-capping**.
- **Anchoring (also in a code comment)**: typical untrained adult ≈ dual 2-back at
  moderate accuracy ≈ skill 1.0 → score ~78 = **Average** band. Tiers: Below Average /
  Average / Above Average / Slightly Superior / Superior / Gifted Range / Savant.
- **Mode checkboxes** replace the single filter in performance mode: one mode
  isolates, several combine, all/none = General. Plus a **Mirror-Match sessions
  only** filter. Reverse sessions add a small bonus to sequence-order regions
  (spl/precuneus/dlPFC/hippocampus); mirror sessions likewise for flip regions
  (spl/precuneus/dlPFC/occipital) — both also in emphasis view (mirror emphasis
  bonus; a "Mirror sessions" preset filter was added to emphasis too).
- Region click in performance mode shows: tier name + numeric estimate,
  **"Estimated everyday performance"** (per-region templates for all 23 regions,
  three tier bands), **"Compared with the general population"** (percentile-ish
  statement) and an explicit **Gifted range: YES/NO** indicator. Required footnote
  shown verbatim: *"Estimated from your training performance (N level, accuracy,
  interference handled) compared with typical n-back performance. An in-game
  estimate — not a clinical or IQ measurement."*

## 4. Reverse N-Back — corrected to BLOCK REVERSAL (supersedes v3)

- v3's i+N "match ahead" rule is **removed**. New rule per modality with reverse
  enabled: trials stream in blocks of length N; each study block is followed by a
  test block that is either the **exact reverse** (target, ~30%) or a **foil** —
  same items reshuffled into a non-reverse order, or the reverse with one item
  replaced (both foil types used).
- Pressing the mode button during a test block claims "reversed"; scoring at block
  end: HIT / FA / MISS / CR appended to `sessionLogs`, so HUD counts, modStats,
  scoresheet, CSV and d′ treat reverse uniformly. Open test blocks are scored on
  early exit / window close via the unified save path (double-scoring guarded).
- **Per-modality opt-in**: global Reverse master toggle + per-modality checkboxes
  (default all on) in Arena and Setup. Non-reverse modalities in the same session
  keep per-trial matching on the shared stream.
- Works for all 10 modalities (token-based engine). Records carry `reverse:true`
  plus a `reverseMods` list; help text updated in Arena and Setup.
- Anatomy preview: Reverse ON adds spl +0.20, precuneus +0.15, dlPFC +0.15,
  ACC +0.10, hippocampus +0.12 on top of modality weights; hidden-areas readout
  reflects them and labels the rule.

## 5. Arithmetic N-Back (9th tracked modality; 3D Spatial is the 10th)

- Default key **G**, rebindable; own button, HUD entry, scoresheet + CSV columns.
- Problems with operands 0–12, ops + / − / × (× operands 2–6); result always a
  non-negative 1–2 digit number. **Tracked value = result**; targets generate a
  fresh problem with the partner's result. Lures: N±1 result copies (classic) plus
  **same-operands/different-operation near-foils**.
- Brain weights: spl .85, dlPFC .9, inferior parietal .7, ACC .4, Broca .3.
  Included in Setup toggles, settings snapshot, anatomy preview, profile filters
  (own "Arithmetic" preset filter + performance checkbox). Speech of problems was
  optional per brief and was **not** added (visual only) — noted as a partial by
  design choice, not a failure.

## 6. Interference: two systems, independently toggleable, per-modality overrides

- **Classic Interference** = the existing N±1 repeat-lure system; the existing
  interference slider/field is now its rate, plus an on/off toggle.
- **Group Interference** = the similarity-group system (see §7), own toggle + rate.
- Either / both / neither work. A non-target trial rolls classic first, then group,
  so combined lure pressure ≈ classic + (1 − classic) × group of non-target trials
  (per modality). This sequential-fill approach is the documented capping scheme.
- **Per-modality overrides**: Setup has an **Interference Matrix** (rows =
  modalities; Classic on/% and Group on/% per row; Group shows "—" for modalities
  without groups). Global pair = default; a ticked row overrides for that modality.
  Engine reads effective rates per modality (`effectiveIntFor`). Snapshot stores
  global pair + `intOverrides` map; the Arena summary notes which modalities have
  overrides. **Backward compat**: pre-split configs have no `classicOn` — they are
  treated as Classic ON whenever their lure rate was > 0; `intOverrides` defaults
  to empty (all modalities follow the global pair).

## 7. Audio sound sets + similarity groups (audio, image, color)

- **Audio sets** for the main stream (selector in Setup, Arena, and the stimulus
  library; saved in the snapshot): **Letters** (existing speech letters),
  **Instruments** (8 WebAudio-synthesized timbres: piano, bell, plucked string,
  drum, flute, organ, marimba, bass — oscillator stacks + envelopes/filters, no
  samples), **Nature & Effects** (8 synthesized: waterfall, rain, ocean, wind,
  thunder, stream/bubbles, birdsong, fire crackle — noise-buffer + filter based),
  **My Sounds** (uploaded sounds). Per-item toggles/weights/groups apply within a
  set via two extra stimulus pools. **Audio2 stays as tones** (permitted by the
  brief; logged here). Note: with **Letters** selected, the session pool is the
  speech items (letters/words); uploaded sounds play under **My Sounds** — a
  deliberate behavior change from v3's single mixed pool, made to keep sets clean.
- **Built-in similarity groups**: letters (letter-name confusables for the
  C,H,K,L,Q,R,S,T set: C/T, K/Q, L/R, H/S), instruments (timbre families), nature
  (texture families), images (shape families over the default set), colors
  (neighboring hues). On a group-lure trial, the lure stimulus is drawn from the
  N-back partner item's group, ≠ the partner. Classic N±1 lures are **kept**
  (Patrick's explicit requirement) and verified in tests.
- **User-assignable groups**: every audio/image/color item (including synth-set
  items) shows its group tags and a group dropdown in the stimulus library **and**
  the media library; users can pick a built-in group or create a named group.
  Assignment is stored on the item (`item.group`), group names are registered in
  `steeltrap_media_<user>`, and both are included in backups.

## 8. Media Library with folders + combined sets (in Setup)

- Images / Sounds / Colors kept separate, each with named **folders**
  (create / rename / delete; deleting a folder moves its items to "My Uploads" —
  nothing is lost). Uploads (image ~1.5 MB, audio ~3 MB caps) and color adds go
  into a chosen folder; items can be moved between folders.
- **Combined sets**: named sets mapping each media type to one or more folders;
  the active-set selector lives in Setup and the set name shows in the Arena
  summary. Session pools for img/audio/col = union of the set's folders, layered
  on the existing stimulus system (toggles + weights + groups still apply).
  Default "Everything" set = Defaults + My Uploads.
- Persistence: `steeltrap_media_<user>` (folders, sets, groups). **Migration**:
  pre-existing v3 stimuli are folder-assigned on login — built-in ids →
  "Defaults", everything else → "My Uploads" (verified by a clean-instance test).

## 9. Mirror Match (third matching rule)

- Hit = current stimulus is the **left–right mirror** of the one N back. Applies
  to **Position** (cell mirrored across the vertical axis), **Image** (flip flag +
  `scaleX(-1)` rendering; 4 new asymmetric default shapes added — arrow, F, flag,
  teardrop — plus the existing wave; mirror targets are only placed on asymmetric
  partners), **Rotation** (flip flag; the 3 default step-polygons are asymmetric),
  **Spatial 3D Rotation** (chirality flag inverted: mirrored = hit, identical =
  lure). Audio, audio2, color, relational, arithmetic keep standard matching.
  Lures: exact-identity copies (the foil), mirror-of-N±1.
- **Exclusivity**: turning Mirror ON turns Reverse OFF and vice versa (Arena,
  Setup, and Anatomy preview), help text states it. Snapshot + record carry
  `mirrorMatch`.
- **Brain representation** (per Patrick's clarification): Anatomy preview treats
  Mirror as a first-class toggle — it adds spl +0.20 / precuneus +0.15 /
  dlPFC +0.15 / occipital +0.10, the active-configuration summary names
  "Mirror Match", the hidden-areas readout is tagged with it, and the
  **Training relevance** texts of spl, precuneus, dlPFC, occipital and premotor
  explicitly describe mirror-match by name. Profile counts mirror sessions toward
  those regions' emphasis and performance; the rules are also described in the
  Arena/Setup help text next to the toggles.

## 10. Final addendum items

- **Patrick Protocol Presets** (built-in, in the preset dropdown, not deletable,
  tweakable after loading): *Fast Visual 1.5s* (image+position, N=2, 1.5s, 87.5%),
  *Audio Only* (audio letters, N=2, 87.5%), *Visualization Hold 5s*
  (image+position, N=2, 5.0s interval, 0.8s flash — labeled his retention variant).
- **Full Backup / Restore** (Setup card + Export button in Profile): export =
  one JSON with **all** `steeltrap_*` keys (users db, records, presets, binds,
  goals, media incl. dataURLs, stimuli). Import validates the file, confirms,
  then **replaces** the covered keys (per-user keys for users in the backup plus
  the users db); data not in the backup is left alone — this behavior is stated
  in the UI help text.
- **Transfer Tests** (new nav tab, labeled *"outside-the-game transfer checks —
  the outcome you actually train for"*): **Photo Recall** — SVG scene with 10
  nameable elements, 10s study, pending test persisted in localStorage with a
  20-minute ready time (survives navigation/sessions), recall checklist =
  10 real + 6 foils, score = hits − false alarms. **Route Recall** — SVG street
  grid route with landmark, study → next-open recall from 3 options (1 real +
  2 altered), score 1/0. Both store dated results in a history list.
- **Adaptive N — opt-in only, default OFF.** The game never changes N unless the
  toggle is on (smoke-verified). When on, at session end: accuracy ≥85% → N+1
  (max 20), ≤55% → N−1 (min 1), applied to the **next** session (the brief's
  "or at session end" option — chosen over mid-session block changes because the
  sequence is pre-generated for the session's N), with an on-screen notice
  ("Adaptive: N 3 → 4") and `adaptive:true` + `adaptiveChange` in the record.
  Manual N changes always work; turning it off freezes N.
- **3D Spatial mode** (10th tracked modality, key V): canvas-rendered 3×3×2 box
  arrangement (18 boxes) with perspective projection; camera orbits continuously
  via requestAnimationFrame (speed adjustable 0–10 in Setup/Arena, slow default);
  match = same **box identity** N back from a different viewpoint; lures =
  neighboring boxes in 3D. Own toggle/button/stats/scoresheet/CSV. Brain weights:
  precuneus .9, retrosplenial .85, hippocampus .8, parahippocampal .8, spl .9,
  occipital .5, dlPFC .5 — hidden-areas readout shows the deep regions for it.
  Continuous camera shipped (no stepped fallback needed).

## Validation

- `node --check` on the extracted main script: **clean**.
- Forbidden-string greps, all **zero**: "Table Rock Lake", "fluke",
  "metabolic activation", "steeltrap_saved_password", "encrypt".
- **v4 jsdom suite: 65/65 checks passed, zero page errors** (two instances —
  main + clean migration instance). Coverage: salted register/login + no
  plaintext; Setup exists; Setup minutes → Arena config + summary (and back);
  SPECT glow hooks + caption on both brains; performance tiers render for a
  seeded user; single-mode isolation changes scores; region click shows tier +
  everyday text + population comparison + gifted indicator; rebinding; arithmetic
  session evaluates + saves; reverse block session (pressed → HIT/FA by target,
  unpressed → MISS/CR); anatomy reverse ON raises parietal + hippocampus;
  media folder create → color into folder → combined set → session pool = union;
  mirror exclusivity + mirrored-cell HIT + identical-cell FA; Instruments session
  runs from synth pool; group lures at 100% (letters, colors, images) and zero
  group lures with group OFF while N±1 classic lures persist; per-modality
  overrides (image group OFF → 0 image lures while audio's appear; position
  classic 0% vs global 87.5% → 0 position lures while audio's remain); custom
  group assignment persists; all four classic/group combinations generate;
  protocol presets load + are delete-protected; backup export + import
  round-trip; photo + route transfer flows score and persist; adaptive OFF never
  changes N, ON steps 3→4 and records it; 3D Spatial session saves with stats;
  regression reverse + spatial-rotation session saves with snapshot + reverseMods;
  migration of a v3 custom stimulus into "My Uploads".
- **v3 regression suites run against the v4 file: 32/33 and 32/33.** The single
  non-pass in each is the v3 assertion that maximal emphasis renders
  `rgb(255, 0, 0)` — the old heatmap's top color. v4's SPECT palette intentionally
  tops out at **white-hot** (`rgb(255,255,255)`) per the new brief, so the region
  is lit *hotter* than the old maximum; this is the requested palette change,
  not a functional regression. All other v3 checks (brain views/peel/hidden
  areas, HUD, focus stages, nutrition layers, conflict-swap rebinding, typing
  guard, unified save with snapshot) pass unchanged.

## Honest partials / notes

- Arithmetic is visual-only (speech optional in the brief, not implemented).
- Audio2 has no similarity groups (tones left ungrouped; matrix shows "—").
- Adaptive N evaluates at session end (for the next session), not mid-session
  blocks — see §10 for the reason.
- Mirror targets for Image are restricted to asymmetric items (symmetric shapes
  are their own mirror image); 4 asymmetric defaults were added to widen coverage.
- Group + classic combined pressure uses sequential fill
  (≈ classic + (1−classic)×group), documented in §6.
- Profile performance scores are estimates from game data only; the UI says so
  wherever they appear.
