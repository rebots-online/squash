# PRD — squash, the Unreal Engine 5 rebuild (`rebots-online/squash`)

**Inception date:** 2026-11-09
**Owner:** Robin L. M. Cheung
**Status:** inception
**Reference conversation:** Claude Code session `d4da2aa8-23df-44c0-a6f5-0a28d1f0896b`, 2026-11-09, run from `/home/robin/CascadeProjects/squash`. Approved plan: `/home/robin/.claude/plans/this-is-a-new-polymorphic-whistle.md`.
**Drafted by:** Claude (orchestrator seat), from operator decisions in that conversation. Template: `/home/robin/Admin-Manual/DOCS/PRD_TEMPLATE.md`. Sections 0-15 follow the template; section 16 is the product definition, which the template has no place for.

Marking used below: **[OPERATOR]** = only the operator can state this; not filled in. **[PROPOSED]** = drafted by Claude, not yet confirmed. Everything else was decided by the operator in the reference conversation.

---

## 0. Inception Gate — product-or-gtfo

This is a clean rewrite with a stack migration, so the gate re-applies (section 13).

- [ ] **Business case for existence.** [OPERATOR] What the operator has said so far: this is "essentially a POC" whose point is "to emphasize realism as an impactful POC, not mediocrity or ease of implementation or wideness of device support, but to make a splash". That states the intent, not a business case.
- [ ] **Alternative use of the same time/attention.** [OPERATOR]
- [ ] **Deployment vector.** [OPERATOR] The operator referred to "our end users", which suggests something beyond dogfood; not decided.
- [ ] **Sunset criteria.** [OPERATOR] [PROPOSED] candidate: kill or re-scope if the Step 1 feasibility spikes fail with no acceptable fallback (ray tracing, MetaHuman assembly, face photo in a packaged build, each judged on Windows).

---

## 1. Project metadata

- **Stack:** Unreal Engine 5.8.4, C++ gameplay modules, MetaHuman characters. Native. Not a web application.
- **Target platforms:** 1. Windows desktop with an RTX 3090-class GPU (first shipped target; operator decision 2026-11-10). 2. Linux desktop (the development environment on the same dual-boot machine; a Linux build is a by-product, not a release gate). 3. Android (later, reduced visuals). Web: excluded; the engine has no web export.
- **Where the work happens:** day-to-day development, the simulation, parity checking and GLM dispatch run on Linux. A Windows build can only be made on Windows, so Windows sessions are required at four checkpoints: (a) Step 1 feasibility, (b) Step 4 first playable slice, (c) Step 6 roster and face-photo flow, (d) Step 7 final packaging. The two sides stay in step through git only. On Windows the engine is installed on a second drive (the Windows system drive is 256 GB).
- **Bundle ID family:** [PROPOSED] `mba.robin.squashshowcase`.
- **Versioning:** MAJOR.MINOR.BUILD per `~/.claude/BUILD_CONVENTIONS.md`.
- **Repo location:** `https://github.com/rebots-online/squash` (operator decision 2026-11-10), default branch `master`. The repository was created from the template `rebots-online/squash-arcade` and first held the three.js game; that content was retired from the working tree and is kept in history under the tag `donor-threejs-v1`. Local working copy on the original machine: `/home/robin/CascadeProjects/squash`.
- **Predecessor donor projects** (spec donors only, no source copying): the three.js game that this repository held before the rebuild, pinned at commit `180ec040da06c7c444f2ace3e10730b2bcdbe173` (tag `donor-threejs-v1` in this repository; the same game is the template repository `rebots-online/squash-arcade`). Its files are no longer in the working tree; check out the tag in a separate worktree, or clone `rebots-online/squash-arcade`, when the reference is needed. It is a three.js browser game. It is used read-only as (a) the source of tuned numeric values and (b) a reference oracle: its simulation runs headless in Node to produce recordings the rebuild is checked against.

---

## 2. Threat model — solopreneur, not corporate

Template motifs adopted as written.

- Credentials handled by this project: none.
- Personal data: the user's face photo. It is processed and stored on the user's machine only and is never uploaded. No accounts.
- Specific anti-loss hardenings applied: the donor repo is never modified; recordings carry the donor commit hash; MetaHuman source assets are kept under version control (Git LFS).
- Attacker-model hardenings opted into: none.
- Out-of-scope-by-design: no server-side storage of photos or any personal data; no payment data.

---

## 3. Architectural posture — refuse manufactured asymmetry

- Components / vendors used:
  - **Epic (Unreal Engine and MetaHuman).** The operator accepted Epic's proprietary licence knowingly (5% royalty above US$1M lifetime revenue per product; not expected to apply). This is a deliberate dependency on one vendor for engine and characters. Seam: all game logic lives in `SquashSim`, plain C++ with no engine types, behind a read-only snapshot boundary, so the rules, physics, controls, footwork planning and AI survive an engine change. Honest limit: MetaHuman characters do not transfer to another engine with their rig, hair and shaders intact.
  - **Face-landmark model.** Must carry a licence with no use restrictions. Seam: one interface in `SquashFace`.
  - **Assets.** Free only. Proprietary licences are acceptable; non-commercial-only licences are not.
- Anti-bundle-lock check: distribution = direct download; payment = none; identity = none; entitlement = none; hosting = none; auth = none.
- Earned-asymmetry sources: the operator's squash coaching knowledge, encoded in the control scheme, shot ideals and footwork.

---

## 4. Identity / per-user artifact strategy

- Strategy: neither. The binary is uniform. Per-user value is the user's own face on their player, created locally from a photo. No woven identity.

---

## 5. Entitlement / payment architecture

- No entitlement gate. The proof of concept is free. If that changes, this section is rewritten before any gating is built.

---

## 6. Build / CI / release

- Build commands (Linux, development): `Engine/Build/BatchFiles/Linux/Build.sh SquashShowcaseEditor Linux Development -project=<path>/SquashShowcase.uproject`; package with `Engine/Build/BatchFiles/RunUAT.sh BuildCookRun -project=<path> -platform=Linux -clientconfig=Shipping -build -cook -stage -pak -archive`. The Windows release build uses the same tool from a Windows session: `Engine\Build\BatchFiles\RunUAT.bat BuildCookRun -project=<path> -platform=Win64 -clientconfig=Shipping -build -cook -stage -pak -archive`. All of these are from research and are confirmed in Step 1, not yet run.
- CI workflow file location: none yet. **Gap:** the template requires CI on the self-hosted Forgejo runner; an Unreal build needs the engine (tens of GB) and a GPU on the runner. Not solved for the proof of concept. See section 15.
- Per-task gate: the Build.sh command above exiting 0, plus the parity checker for simulation tasks.

---

## 7. Distribution

- Channels in scope at v1: direct desktop build for Windows (built in a Windows session on the dual-boot machine). A Linux build is produced during development but is not a release channel at v1.
- Channels in scope at v2+: direct desktop for Linux; Android (reduced visuals).
- Excluded: web. Template motif "multi-channel from inception" is knowingly not met; the operator chose realism over device breadth.

---

## 8. File / artifact discipline

Template minimum featureset adopted. `.tmp/` and `.gitignore` are in place. `Content/` is stored with Git LFS. No credentials are handled.

---

## 9. Documentation / decision discipline

Template adopted: PRD, then `DOCS/ARCHITECTURE.md` with an entity table, then `CHECKLIST.md`, then code. The donor repo's architecture document and its ban on automated tests do not bind this repository.

---

## 10. Session continuity / orchestration

- Orchestrator and reviewer: Claude. Fallback if Anthropic quota runs out: Codex.
- Coders: GLM-5.3 with 1M context, dispatched through `zclaude --acp` using `acpx`. The model is confirmed from returned metadata on every run. Each task carries a manifest of the files it may touch. A coder never accepts its own work.
- Continuity store: KStore, per `DOCS/TOOLING_CONVENTIONS/claude-kstore-only.md`.
- **Deviations from the template, by operator instruction or newer convention:** the template names `context-mcp` and `zclaude-opus`; KStore and `zclaude --acp` with GLM-5.3[1m] are used instead. The ACP procedure is marked experimental in `acp-sdlc-semantic-review.md`; this project is its first product use.
- CodeGraph is used first for anything it can answer at no model cost: symbol inventory, call order and dependency order of the donor simulation. The final completeness check maps every donor simulation function to a specification line or to "deliberately dropped".

---

## 11. Communication style

Template adopted.

---

## 12. Deployment vector decision

[OPERATOR] Not chosen. See section 0.

---

## 13. Reboot / sunset policy

Template adopted. This project is itself a reboot of squash-arcade (clean rewrite, stack migration, legacy drop), which is why section 0 must be completed by the operator.

---

## 14. Reference conversations & documents

- **Inception conversation:** see header.
- **Global rules spine:** `~/.claude/CLAUDE.md`; `/home/robin/Admin-Manual/CLAUDE.md` (Architecture Law).
- **Routed conventions used:** `model-seat-ladder.md`, `acp-sdlc-semantic-review.md`, `claude-kstore-only.md`, `commit-and-push-every-task-and-handback.md`.
- **Related projects:** the donor repo's `DOCS/ARCHITECTURE.md` v2.7.0 (frozen coordinate system, court dimensions, ball-physics invariants).

---

## 15. Open questions / explicit gaps

Operator decisions:
1. Section 0 inception gate and section 12 deployment vector.
2. Repository name and remote.
3. Ball-hits-player check: the donor code never fires it. [PROPOSED] enable.
4. Hit-stop: in the donor it pauses the simulation. [PROPOSED] make it visual only.
5. Points marked [PROPOSED] in section 16.

Technical gaps, closed in Step 1 (feasibility spikes) before architecture is written:
6. Hardware ray tracing: confirmed on Windows at the first Windows checkpoint. On Linux (NVIDIA driver 595.99) it is unproven; a failure there no longer blocks the release, only how much look-development can be done on Linux.
7. MetaHuman assembly: on Linux, 31 GB RAM may be exhausted (30 GB swap already exists). Fallback is to author on Windows.
8. Running a face-landmark model inside a packaged Windows build (and, for development, a Linux one).
9. How much of an Unreal project can be authored without the editor GUI.
10. Disk: Linux system drive has 196 GB free after clearing build output on 2026-11-10 (estimated need 110 GB or more). Windows: the system drive is 256 GB, so the engine goes on a second drive; free space there is not yet measured.
11. CI for Unreal builds (section 6).
12. GLM dispatch through ACP proven end to end with the model confirmed.

Later, not this version: webcam and phone swing controls; online 1v1; snapping a photo with a camera (available through Unreal's media capture on Windows; Unreal has no webcam capture on Linux); easier camera-view switching and likely removal of the side-wall nick view; tuning lab; jukebox; intro.

---

## 16. Product definition

### 16.1 Purpose

A squash game whose purpose is to **habituate optimal timing, shot choice and technique passively**. The on-screen player is the teacher: whatever it does, the user absorbs. Every control, positioning and animation decision is judged against that. The game deliberately produces the rewarding feel of ideal shots so that the feeling pulls the user toward the same choices on a real court.

It must look like a real squash match between real people, with the user's own face on their player. Realism is the point of the proof of concept. Ease of implementation and breadth of device support are not goals.

### 16.2 Core contract

The user never steers the player. The user chooses the shot and its timing; the game moves the player with correct footwork to the right place for it.

### 16.3 Keyboard controls

Left hand says where. Right hand says how hard and when. The thumb reins the player in.

| Input | Keys | Meaning |
|---|---|---|
| Left hand, held | `Q W E` / `A S D` / `Z X C` | Rows: height on the front wall (high, mid, low). Columns: direction the ball leaves the racket, as seen from behind the court (left, straight, right). |
| Right hand, pressed | `J` `K` `L` | Hit: soft, medium, firm. |
| Thumb, held | `Space` | Hold back. |

Rules:
1. **Direction is literal.** From the right side of the court: left is cross-court, centre is straight down the wall, right is a boast off the right wall. Mirrored on the left side. From the middle: left and right are angled shots to that side, centre goes down the middle. There is no boast key. [PROPOSED] a boast results only when the player is clearly in the outer third of the court.
2. **Holding a grid key puts the racket up**, as early as the user likes, with no commitment to which ball is taken.
3. **The shot can be changed while holding** by rolling to another grid key. The most recently pressed key still held is the selection. The player's body shows no difference, so the opponent learns nothing (holding the shot).
4. **The fire press starts the step-and-swing.** The shot locks at that press.
5. **Each key combination is an intention with exactly one ideal execution** for the situation, computed by the game.
6. Firing with nothing held plays centre-mid.
7. The serve uses the same grid: rows choose the serve type, columns its placement.
8. [PROPOSED] Holding only prepares; the weight of shot comes from the fire key; preparing early slightly improves accuracy.
9. [PROPOSED] Timing sets how closely the ball lands on the ideal. A mistimed hit is a looser version of the same shot (off the wall, short, or sitting up), never a different shot. Early or late is shown after each shot.

### 16.4 Space: the rein

The player is visibly eager. At the opponent's strike he reads the ball and sets off for the earliest ball he can take. That eagerness is what prompts the user's thumb. Space is touched only when needed; releasing it sends the player on at once.

| Moment | Space not held | Space held |
|---|---|---|
| Ball coming | Takes the earliest good ball (a volley when one is on) | Drops back to the later ball (after the bounce, or off the back wall) if it is still reachable; otherwise carries on |
| After the user's shot | Goes straight back to the T | Waits clear of the opponent's lane; goes on release |
| Opponent in the way | Plays through | Stops, which is the appeal for a let |

A late rein costs real time and position. Operator decision: try Space as this all-purpose hold. Whether the incoming-ball default is the earliest or the latest good ball is a setting, starting at earliest.

### 16.5 Shot ideals

Every shot is aimed at the ideal for its type. The game never targets less.

| Shot | Keys | Ideal |
|---|---|---|
| Drop | low + soft | The nick zone in the front corner. A true nick happens with low probability. The zone is the target, never the crack. |
| Kill | low + firm | Low above the tin, dying short. |
| Drive | mid | Deep; first bounce behind the service box; shaped to die in the back corner ([PROPOSED] reading of "as not-flat as possible"). |
| Lob | high + soft | High, peaking past the T, falling steeply; first bounce within about 1 m of the back wall, dying there. |
| Boast | toward the near side wall | Low to the opposite front corner, dying before the far wall. |

Consequences, confirmed by the operator: it is very hard to hit the ball to the T. From either side no column goes to the middle. The one exception is the centre column from the middle of the court, which goes down the middle; this is deliberate and is not re-aimed. Otherwise a loose ball comes only from mistiming. True nicks come only from well-timed drops. The AI opponent is bound by the same rules.

### 16.6 Which ball is taken

At the opponent's strike the game lists every chance to hit the ball (volley, after the bounce, off the back wall), each with a place, a time, a quality rating, and the last moment it can still be reached properly. A ball that will die in the back corner rates poor, so the player cuts it off before the back wall. The game never attempts a ball it cannot reach properly. Optional training markers show each contact spot and fade when it is no longer reachable.

### 16.7 Footwork

Footwork is a main point of the realism. It must be the traditional pattern, shown consistently so the mind's eye always sees the ideal. It does not need motion-capture fidelity.

**The habit this game exists to break is arrive, plant, swing, go.** That pattern makes the player face the front wall and is very hard to unlearn. It is never shown.

Pattern, right-hander from the T to the back forehand corner (backhand mirrors; front and mid-court use the same parts with different step counts):
1. Split step as the opponent strikes. Subtle, "light on the T", not a hop.
2. Turn and push off; one or two long travelling steps.
3. Arrive **one step short** of the hitting position: poised, side-on to the side wall, racket up, weight on the back foot. Any spare time is spent here.
4. **The last step is the swing.** The lead foot goes in as the racket comes down; contact happens as the weight arrives.
5. **The same step finishes as the recovery.** Hips and shoulders open through the ball, the lead foot pushes off, and the body is already moving back toward the T. No pause between hitting and leaving.
6. Two or three steps to the T, into the next split step.

Requirements:
- The body faces the side wall up to and through contact and opens to the front only as part of leaving. A front-facing hit is never shown.
- The fire press triggers the step-and-swing with a constant lead before contact, so the rhythm the user learns is always "step, hit".
- Contact is at correct reach for an adult body (about 1 m from the ball), never on top of it. The donor game stood the player 0.42 m from the ball.
- [PROPOSED] Under time pressure the player performs a compressed version of the same pattern, never a different technique.
- Movement is limited to what the step pattern allows (about 1 s from the T to a back corner). Some balls the donor game let the player reach will be out of reach. The AI opponent uses the same footwork and limits.

### 16.8 Interference, lets and strokes

Two people sharing one box is central to squash and is part of the game.

- Decisions (no let, let, stroke) follow the World Squash rules, written as an explicit table with fixed names for the two roles. The current rules text must be checked before the table is written. In the donor game the live check never ran and awards sometimes appeared inverted.
- Clearing after a shot is automatic and correct: the player's route to the T goes around the opponent's lane to the ball and out of the front-wall triangle.
- **Clearing must be unmistakably visible as intentional and necessary.** On every shot the floor shows the opponent's lane to the ball (the reason) and the player's path bending around it (the action). If the user's ball is loose and there is no clean way out, this shows red at the moment of the hit.
- The user is never penalised for the automatic footwork. Interference against the user can arise only from a loose shot, or from not reining the player in.
- The user is not helpless: while a shot is held, the floor previews that shot's ball path and the exit as clear, tight or blocked, and updates as keys are rolled; Space holds the player out of the lane; Space when blocked is the appeal.
- [PROPOSED] After each call, a short freeze shows why it was given.
- The AI opponent clears, concedes strokes and stops under the same rules.

### 16.9 Courts and visibility

Two courts: a PSA-style all-glass show court in an arena, and a typical club court with a hardwood floor and a glass back wall.

Hard rules, for both:
1. Any wall between the active camera and the play never obscures the players or the ball. Transparency exists for the viewer.
2. See-through never costs playability. Walls the ball is read against give a stable, high-contrast backdrop. Reflections are capped on the wall being looked through.
3. Wall appearance is decided per wall from the active camera's position, not fixed per wall.
4. Ball and floor are paired for contrast on each court (white ball on the dark show-court floor; black ball on pale hardwood). Contrast is measured, not judged by eye.

### 16.10 Cameras

The donor game's views and view cycling are kept as they are for this version. The back-wall ("window") view and the top-down view are the ones played from, and both must meet the visibility rules.

### 16.11 Players and the face photo

- The developers create a small roster of MetaHuman players once (varied build, skin tone, gender), in squash kit. The roster ships with the game. End users never see an editor.
- In the game menu the user picks a body, uploads a photo, and confirms a preview.
- The photo is fitted onto the chosen head as its skin, with skin tone matched to the body. The closest roster head is picked automatically from the photo, and the user can override it. Hair is chosen from a small set.
- Known limits: the head's shape cannot be changed to match the photo while the game runs, so likeness is best from the front and three-quarter view and weaker in profile. Hair cannot come from the photo. Users cannot add new bodies.

### 16.12 Scope of the first version

In: a solo match against the AI with serving, rallies, scoring, interference and replays; both courts; the keyboard controls above; photo upload.

Out, for later: see section 15.

### 16.13 What is taken from the donor game

| Status | Items | How the rebuild is judged |
|---|---|---|
| Keep | Ball physics, the nick model, wall and tin collisions, scoring and serve rules, the strike reach gate, the input buffer, straight and cross drives, back-wall and volley drives, the side-front boast, the basic serve, the principle of never planting and waiting, side-on preparation | Must reproduce recordings of the donor simulation |
| Improve | Lob, drop from deep, kill, back-wall boast, positioning and spacing, arrival timing, footwork, recovery, movement speeds, interference | Specified fresh in this document; donor recordings are a "before" reference only |

### 16.14 Acceptance for the proof of concept

- A full solo match can be played on both courts on the showcase machine.
- Each grid key from each region of the court produces the shot this document states.
- The player is side-on at contact, the last step is the swing, and recovery begins within the swing, on every shot.
- Space behaves as in 16.4.
- Ball-to-background contrast meets its threshold at every tested ball position in the back-wall and top-down views on both courts. [PROPOSED] threshold 3:1.
- A photo becomes a previewed player in the packaged build.
- [PROPOSED] 60 frames per second at 1440p during play.
- The operator judges controls, footwork, look and feel by playing.

### 16.15 Where it is expected to fall short

Stated so the result is judged fairly: the weight shift in the footwork will be less fluid than a filmed athlete, though the pattern will be right; cloth will be simple; likeness is a face on a template head; crowd quality depends on free assets.
