# Steel Trap — Iteration 2 (v3) Build Log

Base: `~/workspace/steel-trap/build/steel-trap.html` (merged v2, 176,502 bytes) copied to
`~/workspace/steel-trap/build-v3/steel-trap.html` and modified in place.
Final: **268,643 bytes**, single file, exactly 1 `<script>`, zero external references (fully offline).
Spec: `~/workspace/steel-trap/ITERATION-2-SPEC.md` + three follow-up requirement sets from Patrick
(anatomical correctness / beneath-callout / peel; hidden-area readout, all-areas, per-area supports,
nutrition rework, building-blocks; full key rebinding).
Method: three anchored Python patch rounds against the v2 base (HTML/CSS → engine/settings/records →
brain module insert), then a round-2 patch for the follow-up requirements. Every patch asserted exact
anchor counts before replacing; nothing in v2 was removed.

## What shipped, per spec section

### 1. Brain component (shared: Anatomy tab + Living Brain Profile)
- One component (`ensureBrainMounted/brainRender/brainPaint`), two instances ('anatomy', 'profile').
- Anatomically-laid-out schematic SVG brain, viewBox 520×400, **4 views**: left lateral, superior,
  right lateral (mirrored render of left), medial. Mouse-drag horizontally snaps between views
  (order left → top → right → medial); wheel + buttons zoom (scale transform about center, regions
  stay clickable); Rotate/Zoom/Reset buttons on the component.
- **23 named regions** (spec asked ~20 + the follow-up added untargeted ones): dlPFC, rlPFC/frontal
  pole, ACC, Broca/IFG, premotor/motor strip, **somatosensory/postcentral** (new), SPL/IPS, inferior
  parietal, precuneus, occipital visual, fusiform, auditory cortex, Wernicke, insula, hippocampus,
  parahippocampal, entorhinal, retrosplenial, basal ganglia/caudate, thalamus, cerebellum, amygdala,
  **brainstem** (new). Placement follows standard atlas layout per view (Broca inferior-frontal,
  auditory on superior temporal, fusiform inferior temporal, ACC above the corpus callosum on the
  medial view, precuneus medial parietal, retrosplenial behind the cingulate, hippocampus/PHC/
  entorhinal in medial temporal). Sulci suggestion lines (lateral sulcus, central sulcus, cingulate
  sulcus), corpus callosum arc on medial view, cerebellum striping + brainstem shapes. Labeled in UI
  as a schematic illustration, not a medical image.
- Every region: description, **real history** (Gage→frontal; H.M./O'Keefe/Maguire/R.B.→hippocampus;
  Broca's "Tan"; Wernicke 1874; Bálint→parietal; Gerstmann→inferior parietal; Hubel & Wiesel→visual;
  Kanwisher→fusiform/PPA; Klüver–Bucy + S.M.→amygdala; Penfield→motor/sensory; Papez/Botvinick→ACC;
  Naqvi→insula; Parkinson/Milner→basal ganglia; Korsakoff→thalamus; Flourens/Schmahmann→cerebellum;
  grid cells→entorhinal), and a training-relevance line.
- **Functional group chips**: Visual / Spatial / Working Memory & Control / Auditory / Language /
  Memory & Medial Temporal — toggle highlighting of member regions.
- **"What's underneath" callout** (follow-up req): fixed panel at the top of the brain panel; on
  hover OR click of a surface region it shows the region name + "Beneath:" buttons (per-region
  `REGION_BENEATH` map, e.g. auditory cortex → insula, hippocampus, amygdala). Beneath buttons are
  clickable and open that deep region's info (auto-peeling or switching view as needed).
- **Peel Back Cortex toggle** (follow-up req): surface layer fades to 13% opacity and slides
  slightly; the deep layer drawn underneath in place (insula, hippocampus, parahippocampal,
  entorhinal, amygdala, thalamus, caudate) becomes fully visible and directly clickable, and takes
  part in the same emphasis coloring + info panels. Medial view keeps its native deep structures
  (peel button explains this instead of acting). CSS transition animates the change.
- **All areas stay available** (follow-up req): regions with zero n-back weight (amygdala,
  somatosensory, brainstem, insula) are fully clickable with complete info; their training line
  explicitly says "Not directly targeted by n-back modes."
- **Per-area "What helps this area"** (follow-up req): expandable `<details>` section in both info
  panels with specific supports per region (sleep, aerobic exercise, DHA, meditation, B12, iron,
  hyperbaric, etc.), each tagged **Stronger evidence / Some evidence / Emerging**, claims kept
  modest (e.g. hyperbaric for hippocampus = Emerging/mixed). Generic fallback set for any region
  without a specific entry.

### 2. Anatomy tab — settings → activation preview + hidden readout
- Settings panel beside the brain: all 8 modality toggles + N (1–20) + interference + interval +
  reverse toggle; entering the tab mirrors the current Arena config. Toggling recolors the brain
  live by **estimated** activation: one shared `MOD_REGION_WEIGHTS` table (0–1, per spec examples);
  scale factor `0.7 + 0.06·min(N,10) + 0.003·interference`, ACC additionally gains 0.004/interference
  point, reverse adds dlPFC/ACC bumps — formula documented in a code comment. Legend labeled
  "Estimated training emphasis — schematic, not measured brain activity."
- **Hidden-area readout** (follow-up req): prominent purple panel at the TOP of the brain panel —
  "Also training (below the surface): Hippocampus — strong, Parahippocampal Cortex — moderate…",
  each entry a button opening that region's info. Updates live with the settings; no peeling or
  view-switching needed to discover hidden training targets.
- **Building blocks subsection** (follow-up req): expandable panel "Building blocks: myelin, axons
  & what they're made of" — myelin composition (lipids: cholesterol/phospholipids/
  galactocerebrosides + MBP/PLP), neuron/axon composition (phospholipid membranes, cytoskeleton,
  mitochondria), build/maintain needs (cholesterol, B12, folate, iron, iodine/thyroid, DHA, choline,
  sleep), synapse/neurotransmitter precursors, with an honest "deficiency avoidance is the proven
  part" framing.

### 3. Profile tab — same component, driven by records
- Region intensity = accumulated emphasis from saved sessions: Σ (region weight × session minutes)
  per modality actually present in each record's modStats, saturated `1 − exp(−total/25)`.
- Filter buttons: General / Audio / Visual / Position & Spatial / Image & Color / Relational /
  Rotation — recompute from records per filter.
- **Show untrained areas** button: trainable regions with <2% emphasis under the current filter get
  a pulsing dashed outline on the brain and are listed by full name.
- Clicking a region shows the same info panel content as Anatomy (description/history/training/
  supports) plus the user's emphasis % and their linked session history; "Train This Area" now
  picks the top-2 modalities by that region's weight table.

### 4. Settings (Arena config)
- **N 1–20** (input max, startGame clamp, quick-bar +/− clamp all raised from 9).
- **Duration**: minutes cap raised to 1440; M.SS text entry retained; slider unchanged (0–120).
- **Interval**: typed field now 0.3–30.0s (was a 0.5–15 slider-limited input), clamped at start.
- **Reverse N-Back** toggle with on-screen definition: match = stimulus N trials **ahead** in the
  pre-generated sequence. Implemented via a partner abstraction in generation (targets/lures copy
  from partner i+N instead of i−N), evaluation, and the input gate; the flag is recorded.
- All modality toggles individual, including the new spatial rotation (below).

### 5. Spatial 3D Rotation modality (`srot`, default key T, rebindable)
- 5 Shepard–Metzler-style cube-assembly objects (8–9 cubes each) rendered on a 180×180 canvas:
  yaw (multiples of 45°) + pitch (−30/0/30°) Euler rotation, orthographic projection, painter sort,
  three shaded visible faces per cube. Stimulus key `obj|yaw|pitch|mirror`.
- Match = same object AND same orientation N back (or N ahead in reverse). Lures: **mirrored**
  copies (classic S–M foil — mirror flag never matches) and same-object stimuli copied from the
  partner's neighbours. Own response button, own modStats, scoresheet column ("Spat Rot"), CSV
  column, live HUD entry, nutrition/analytics naming. Full S–M shipped — no reduction needed.

### 6. HUD
- Existing live bar extended: per active modality **✓correct ✗wrong %** (correct = hits + correct
  rejections, wrong = misses + false alarms) plus combined % and Load Index. Small, top-of-arena,
  semi-transparent styling retained; toggle button in the quick bar + rebindable HUD key (default H).

### 7. Focus stage 2
- Focus key (default F) cycles **normal → focus → grid-only → normal**; Esc exits to normal.
  Stage 2 adds `focus-grid-only`: response buttons, HUDs, quick bar, and start/end buttons hidden;
  grid scales to `min(29vmin, 250px)` cells — true fullscreen grid. Works mid-session.

### 8. Records — full settings snapshot
- Every record (normal finish, early exit, window-close — the unified save path) now includes
  `settings: getFormSettingsObject()` captured at session start (all modality toggles incl. srot +
  distractors + distractor rate, N, interval, flash, lures, duration mode/trials/minutes, reverse)
  plus top-level `reverse`. CSV gained: Spatial Rotation Acc, Reverse, Distractor Rate, and the
  full Settings Snapshot as JSON.

### 9. Nutrition tab rework (follow-up req, progressive disclosure)
- Top level unchanged in spirit: overview from the latest session (foods + electrolytes scaled by
  training load) with guidance framing and the existing safety notes (raw-yolk salmonella,
  high-sodium caution) intact.
- New **per-area breakdown** below it, collapsed by default: one `<details>` per emphasized area
  (from the latest record's modalities × weight table) naming what that area especially uses, with
  per-food suggested amounts tiered by the same training-load score, and a nested **"Why?"**
  `<details>` per food with 1–3 plain mechanism sentences. Carnivore/electrolyte framework kept.

### 10. Full key rebinding (follow-up req)
- Controls table now lists **21 bindings**: 8 modality responses + Start/Pause, End & Save, Focus
  cycle, HUD toggle, Live N +/−, Live Lure +/−, Brain rotate left/right, Brain zoom in/out, Peel.
- Capture uses **KeyboardEvent.code** (any key: F-keys, arrows, Space, punctuation); conflicts are
  detected across all bindings and offer **swap-or-cancel** via confirm; Esc cancels a capture.
- Saved per user in the existing `steeltrap_binds_<user>` storage (merged over new defaults, so old
  users gain the new actions automatically; legacy binds without a code fall back to e.key matching;
  Numpad +/− count as the +/− bindings). **Reset-to-defaults button** added. Typing guard extended:
  no binding fires while typing in any field (v2's start-on-Space now also respects it).
  Gamepad rebinding unchanged from v2.

## Validation (all headless, jsdom + @napi-rs/canvas for a real 2D context)
- `node --check` on the extracted script: **clean**. Exactly 1 script block; zero external URLs.
- Greps on the final file: "Table Rock Lake" 0, "fluke" 0, "metabolic activation" 0,
  "steeltrap_saved_password" 0, "encrypt" 0. Tag balance: div 249/249, button 60/60, svg 18/18.
- **Smoke round 1 — 33/33 passed**: salted registration with no plaintext anywhere; anatomy renders;
  drag switches view; hippocampus (medial) opens info with H.M. history; audio preview lights
  auditory cortex rgb(255,0,0); profile renders + untrained list; session with **reverse ON +
  spatial rotation ON at N=3** ran trials, HUD showed ✓/✗ counts, early exit saved exactly one
  record containing the settings snapshot with `reverse: true` and srot stats; focus stage 2
  toggles; zero page errors.
- **Smoke round 2 — 33/33 passed**: ≥20 clickable brain paths; hidden readout lists Hippocampus +
  Parahippocampal with strength words under Image mode; amygdala opens full info labeled "Not
  directly targeted" with evidence-tagged supports; brainstem + somatosensory info opens;
  building-blocks content present; brain action keys rotate/zoom/peel; nutrition overview renders,
  6 per-area breakdowns all collapsed by default, nested Why expands with mechanism text, safety
  notes intact; **rebinding**: 21 rows listed, End&Save captured to KeyX via KeyboardEvent.code and
  persisted, conflict warning fired with cancel preserving both bindings, swap exchanging them,
  typing guard blocked the rebound key from an input field, the rebound key then ended + saved a
  live session, reset restored defaults; zero page errors.
- Real bug caught by round-1 testing in v2's design and avoided here: none new; one test-side
  artifact (post-drag click suppression at 250 ms) was identified as component behavior working as
  designed, and the test was adjusted to outlast it.

## Deferred / partial
- Nothing from the spec or the three follow-up sets was deferred. Notes on scope choices:
  - Right-lateral view is a mirrored render of the left (labels mirrored positions; region
    anatomy identical) — standard for a schematic.
  - Deep structures on lateral/top views are shown in projected position when peeled (the
    conventional schematic representation); the medial view shows them in true position.
  - Brain action keys act on whichever brain screen (Anatomy or Profile) is currently visible.
  - HUD bar position remains the arena top bar position from v2 (top of the arena, above the
    grid), which is the non-distracting spot the spec described.
