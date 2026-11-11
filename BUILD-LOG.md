# Steel Trap — Merged Build Log

Base: intake/code-v10-steel_trap-v2-logo.html copied to build/steel-trap.html (2026-11-11).
Spec: SUMMARY.md INCLUDED list + decisions D1–D8 (Patrick approved, reversible).

## Plan
1. Text/label pass: D2 honest labels (Session Load Index, training emphasis estimate, trainingHint), Table Rock Lake scrub, challenge same-machine note (D4).
2. Auth pass (D3): per-user random salt + SHA-256(salt::password), legacy unsalted-hash login + transparent upgrade, remember-me = username ONLY (no plaintext password anywhere; also actively deletes any legacy steeltrap_saved_password key), show-password toggle + "show while typing" checkbox + Enter-to-submit (from password patch, minus its plaintext storage).
3. Storage: one-time migration from legacy bw_* / nback_* keys into steeltrap_* keys.
4. Donor merges from v1: 12 default images, 8 default colors, 3rd rotation shape, custom image/sound uploads, background music + per-session music flag, custom colors, mood & notes in daily log. From v2: descriptive pattern analytics + linear trend (labeled descriptive, NOT a forecast/prediction; fluke-odds NOT carried over — dropped per D2).
5. Focus-zoom patch: CSS + quick bar + integrated into the EXISTING keydown handler (no second listener), with typing guard (INPUT/TEXTAREA/SELECT/contentEditable), +/- N keys, F/Z zoom, Esc exit. Live lure +/- in quick bar too. NOTE: while a session is running, N/lure changes update the config + HUD and apply to the NEXT session (the running session keeps its captured parameters so its scoring buffer stays valid).
6. Live-scores patch: live metrics bar, END & SAVE INTERIM SCORE button, unified finishSessionWithStatus() used by normal finish / early end / beforeunload, partial sessions always saved with timestamp + status + goal-folder tag, scoresheet rebuilt with folder filter + group-by-day, CSV updated (Load Index, Audio2, Music, Mood, Notes, Status, Folder).
7. Nutrition patch (D7): Tab [8], guidance framing ("Suggested replenishment ideas — guidance, not a prescription"), safety notes for raw yolks (salmonella — pasteurized/cooked safer) and very high sodium targets (hypertension/kidney/heart caution).
8. Piece 30: stimulus pools per modality with active toggles + relative % weights, custom stimuli persisted per user (steeltrap_stimuli_<user>); second audio stream (aud2, WebAudio tones, 7th modality, rebindable, default key K); experimental distractor channels (audio/image/color, unscored, one rate control); typeable duration M.SS + slider (0–120+ min, cfg-minutes max raised to 240).
9. Relational library: 5 categories × 8 pairs → 8 categories × 10 pairs (80 pairs).

## Decisions made during build
- Stimulus weights are RELATIVE percentages: each stimulus's chance = its weight ÷ total active weight in that pool (shown in the library UI copy). Default weight 10 for all defaults, so a fresh pool is uniform.
- Custom stimuli: uploaded images/sounds are stored as dataURLs in the per-user pool (persist across sessions on the same browser; size-capped ~1.5MB images / ~3MB sounds with an alert when skipped, because localStorage is limited). Background music is session-only (object URL), matching v1's behavior; only the per-session Yes/No music flag is recorded.
- Audio 2 custom stimuli are tones defined by frequency (Hz 40–2000); the 8 defaults are sine tones 220–587 Hz played via WebAudio (no audio files needed, fully offline).
- Distractor design (experimental, unscored): audio = a spoken non-letter word at a low rate offset from the target letter; image = a symbol flashed in a dashed "INTERFERENCE" box under the grid; color = a brief border glow on the grid. One shared rate control (cfg-dist-rate, % of trials). Distractors never enter sequence data or scoring.
- Live N/lure changes while a session is running apply to the NEXT session (running session keeps its captured nLevel/lureRate so its already-generated sequence and scoring stay valid); HUD + config update immediately so the change is visible.
- Folder tagging: challenge sessions → "Peer Challenges"; goal-launched sessions → the goal's domain folder (Reading & Dialogue / Mathematics / Carpentry & Spatial / Relational Reasoning); everything else → "General".
- Analytics (from v2, rewritten descriptive-only): session count, overall average, last-7 vs previous-7 sessions, linear trend slope explicitly labeled "not a forecast," chamber vs non-chamber days, sleep ≥7 vs <7, strongest/weakest modality — with correlation-not-cause wording. The v2 "7-day forecast" framing and fluke-odds p-value were NOT carried over (D2).
- Migration runs once (flag steeltrap_migration_v1_done) and only fills steeltrap_* keys that don't already exist; legacy keys are left in place (not deleted).

## Bug found & fixed during validation
- startGame originally resolved modality checkboxes via the template `cfg-mod-${modKey}`. The Audio 2 mod key is `aud2`, which builds `cfg-mod-aud2` — but the checkbox's real ID is `cfg-mod-audio2`. This would have broken session start for every user. Caught by the jsdom runtime smoke test (after a long detour suspecting a jsdom getElementById quirk — it was a real bug). Fixed with explicit MOD_CHECKBOX / MOD_POOL maps in startGame.

## Validation (2026-11-11)
- `node --check` on the extracted main <script>: SYNTAX OK (re-run after every patch pass, final file clean).
- Grep of final file — zero occurrences of: "Table Rock Lake", "fluke", "metabolic activation", "steeltrap_saved_password", "g-Rating", "encrypt". (The legacy plaintext-password key is removed at startup via a dynamically constructed key string, so the literal never appears.)
- Structure: exactly 1 <script> tag, 0 external URLs/requests (fully offline), div balance 208/208, all section IDs unique.
- jsdom runtime smoke tests (headless, WebCrypto injected — jsdom lacks SubtleCrypto; real browsers on file:// and https:// have it):
  - Auth: salted registration (32-hex salt, 64-hex SHA-256 hash), NO plaintext password anywhere in storage, remember-me stores username only.
  - Legacy: bw_* user DB migrated into steeltrap_*; legacy UNSALTED password record verifies once and is transparently upgraded to salted storage on login.
  - Pools: all 8 pool blocks render in the Stimulus Library; relational library = 8 categories × 10 pairs = 80 pairs.
  - Session (Position + Audio + Audio 2, distractor audio on at 100%): session starts, sequence carries audio2 tone IDs, trials evaluate and log (12 evaluations over ~5 trials), live accuracy ticker updates, END & SAVE saves exactly one record with status "Completed (Early Exit)", folder, duration "0.0m / 6t", Session Load Index, music flag, mood, and per-modality stats including aud2.
  - Scoresheet renders the saved record; descriptive analytics render; Nutrition tab updates to "GUIDANCE FROM LATEST SESSION" from the saved record; Focus Zoom toggles body class on/off; duration parser: "59.30" → 59.5 min.
  - Zero page errors across all flows.
- Deliverable: build/steel-trap.html, 172 KB, single file.

## Partial / deferred (per spec)
- D5/D6: true 3D hovering-camera mode and true Shepard-Metzler 3D rotation — deferred to a later iteration (rotation stays in-plane; no fake CSS orbit).
- D4: challenges remain same-machine only (labeled in UI); cross-device would need an online component.
- Brain graphic: v10's region profile/heatmap retained with honest "training emphasis (estimate)" labeling; a photorealistic brain graphic (piece 30 item 8, second half) was not attempted — the labeled schematic stays.
