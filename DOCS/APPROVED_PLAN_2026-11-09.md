> **Note (2026-11-10):** this is the plan as approved on 2026-11-09, copied into the repository so it is not tied to one machine. Where it differs from `DOCS/PRD.md`, the PRD wins. Known difference: the plan names Linux as the showcase target; the operator has since made Windows the first shipped target, with Linux as the development environment.

# Squash: photoreal Unreal Engine 5 rebuild (proof of concept)

STATUS: ready for approval. Approval covers Steps 0-2 (new repo with trimmed
context and the PRD, feasibility spikes, architecture and task list). Product
code starts only after the operator approves the PRD, ARCHITECTURE.md and
CHECKLIST.md. Anthropic quota is about half used: the PRD comes first so
today's decisions are durable.

## Context

**What is being built:** a native, GPU-heavy squash game in Unreal Engine 5,
targeting an RTX 3090 on Linux. Not a web game; no three.js; no browser.
It is an impactful proof of concept whose point is realism: it should look
like a real squash match between real people, with the user's own face on
their player. Ease of implementation and breadth of device support are
explicitly not goals.

**What exists today (reference only, being replaced):** this repo,
`/home/robin/CascadeProjects/squash`, a fork of rebots-online/squash-arcade.
It is a three.js browser game with Mii-style cartoon players. None of its
code is carried over. It is kept read-only as a reference for ball physics
and tuned values.

**Greenfield (operator decision):** the rebuild lives in a NEW repository
beside this one, placeholder name `squash-showcase` (operator to name). It
references this repo pinned to a fixed commit. A new **PRD is written
first**, from the decisions below, then architecture, then tasks, then code.
The old repo's ARCHITECTURE.md, CHECKLIST.md and its ban on automated tests
do not bind the new repo; the Architecture Law in
`/home/robin/Admin-Manual/CLAUDE.md` still does.

**Purpose of the game (operator):** habituate optimal timing, shot choice and
technique passively. The on-screen player is the teacher; every control,
positioning and animation decision is judged against that.

## Settled decisions

1. Clean rebuild; specs kept, code not kept. Old web code stays untouched in
   the repo as a reference oracle. Nothing is deleted.
2. Unreal Engine 5 with MetaHuman players. Epic's licence accepted.
3. Showcase target is this machine (Linux, RTX 3090). No web version;
   Windows/Android later.
4. Assets free only; proprietary licences fine; no non-commercial-only ones.
5. Players: developers author a small MetaHuman roster once; it ships with the
   game. End users pick a body, upload a photo, confirm a preview. Closest
   template auto-picked with override; hair from a small set.
6. Courts: PSA-style glass show court in an arena; club hardwood court with
   glass back wall. Visibility rules: a wall between the active camera and the
   play never obscures players or ball; see-through never hurts playability
   (stable high-contrast backdrop, reflections capped on the looked-through
   wall); wall appearance driven by camera position; ball/floor contrast
   paired per court and measured.
7. Cameras: existing views and cycling kept this pass. Back-wall and top-down
   are the played-from views and both must pass the visibility rules.
8. First version: solo match vs AI, both courts, keyboard, face-photo upload.
   Deferred: webcam/phone swing controls, online 1v1, tuning lab, jukebox,
   intro, view-switcher redesign, "snap a photo" (needs custom Linux capture).

## Control spec (operator-designed, 2026-11-09)

The user never steers. Left hand says where, right hand says how hard and
when, thumb reins the player in.

| Input | Keys | Meaning |
|---|---|---|
| Left hand, HELD | `Q W E / A S D / Z X C` | Rows = height on the front wall (High, Mid, Low). Columns = direction the ball leaves the racket, seen from behind (left, straight, right). Hold = racket up, as early as wanted. Roll between keys to change shot; most recently pressed held key wins. |
| Right hand, PRESSED | `J` `K` `L` | Hit: soft, medium, firm. The press starts the step-and-swing; its timing sets quality. |
| Thumb, HELD | `Space` | Hold back. Releasing sends the player on at once. |

- Direction is literal. From the right side: left = cross-court, centre =
  straight down the wall, right = boast (mirrored on the left). No boast key;
  the game computes the optimal shot in that direction from position and
  timing. Boast only when clearly in the outer third (proposed).
- Each key is an intention with exactly one ideal execution per situation.
- The opponent sees no difference while a shot is held or changed (holding
  the shot). Shot locks at the fire press.
- Firing with nothing held plays centre-mid.
- Serve uses the same grid (rows = serve type, columns = placement).
- Old WASD steering and the Auto-T toggle are removed.

**Space, the all-purpose rein (operator decision: try it).** The player is
visibly eager: at the opponent's strike he reads the ball and sets off for
the earliest ball he can take. That eagerness is what prompts the thumb.
- Ball incoming: held = drop back to the later ball (bounce or back wall) if
  still reachable; otherwise he carries on.
- After own shot: held = wait clear of the opponent's lane; go on release.
- Opponent in the way: held = stop, which is the appeal for a let.
- A late rein costs real time and position; that is the honest price.
- Setting, starting at "earliest": whether the incoming-ball default is the
  earliest or latest good ball.

**Shot ideals (intentional: habituate the rewarding feel of ideal shots).**
Every shot is aimed at the ideal for its type; the game never targets less.
- Drop (low, soft): the nick ZONE in the front corner, with a low chance of a
  true nick. The zone is the target, never the crack.
- Kill (low, firm): low above the tin, dying short.
- Drive (mid): deep, first bounce behind the service box, shaped to die in
  the back corner.
- Lob (high, soft): high, peaking past the T, falling steeply, first bounce
  within about 1 m of the back wall, dying there.
- Boast: low to the opposite front corner, dying before the far wall.
- It is very hard to hit to the T. Only centre column from the middle of the
  court goes down the middle; deliberately not re-aimed. Otherwise a loose
  ball comes only from mistiming. Same rules bind the AI.

**Starting positions for unruled points (changeable after play-testing):**
holding prepares only, power comes from the fire key, early preparation
slightly improves accuracy; a mistimed hit is a looser version of the same
shot, never a different one, and true nicks come only from well-timed drops;
early/late feedback after each shot.

## Positioning and footwork spec

- At the opponent's strike the game enumerates contact chances (volley, after
  bounce, off back wall), each with place, time, quality and the last moment
  it can still be reached properly. A dying back-wall ball rates poor, so the
  player cuts it off.
- The game never attempts a ball it cannot reach properly. Optional training
  markers show each contact spot and fade when it is no longer reachable.
- Spacing at correct reach for a 1.8 m body (about 1 m), not the old 0.42 m.
- **Never arrive-plant-swing-go** (the habit this game exists to break).
  Pattern, right-hander T to back forehand corner; backhand mirrors; front
  and mid-court use the same parts with different step counts:
  1. Split step at the opponent's strike, subtle ("light on the T").
  2. Turn and push off; one or two long travelling steps.
  3. Arrive ONE STEP SHORT, poised side-on, racket up, weight on the back
     foot. Spare time is spent here.
  4. The last step IS the swing: lead foot goes in as the racket comes down,
     contact as the weight arrives.
  5. The same step finishes as the recovery: hips and shoulders open through
     the ball, lead foot pushes off, body already moving to the T. No pause.
  6. Two or three steps to the T into the next split step.
- Body faces the side wall up to and through contact; opens to the front only
  as part of leaving. A front-facing hit is never shown.
- The fire press triggers the step-and-swing with a constant lead before
  contact, so the trained rhythm is always "step, hit".
- Under pressure: a compressed version of the same pattern, never a different
  technique (starting position).
- Footwork is a planned system: the simulation plans each foot placement and
  time and the body is solved onto them. Generic run-cycle animation is not
  used for court movement. Mocap fidelity is not required.
- Movement speed is bounded by what the step pattern allows (about 1 s T to
  back corner), replacing the old 6.8 m/s and 18.5 m/s2. Reachability follows
  from the step plan. Same footwork and limits for the AI.

## Interference, lets and strokes (central: two people sharing one box)

- Explicit decision table (no let / let / stroke) with fixed role names.
  Verify against the current World Squash rules text before writing the spec.
  (Old game: live check never fires, awards sometimes look inverted.)
- Clearing is automatic and correct: recovery routes around the opponent's
  lane to the ball and out of the front-wall triangle.
- Clearing must be unmistakably visible as intentional: on every shot draw
  the opponent's lane (the why) and the recovery path bending round it (the
  what). Turns red at the hit when no clean exit exists.
- The user is never penalised for automatic footwork; interference against
  the user arises only from a loose shot or from not reining in.
- Not helpless: while holding a shot, the floor previews that shot's ball
  path and exit as clear / tight / blocked, updating as keys are rolled;
  Space holds the player out of the lane; Space when blocked is the appeal.
- Short explained freeze after each call showing why (proposed).
- AI bound by the same rules: clears, concedes strokes, stops.

## What is kept from the old game, and how

| Status | Items | How judged |
|---|---|---|
| KEEP | Ball physics, nick model, wall and tin collisions, scoring and serve rules, strike reach gate, input buffer, straight/cross drives, back-wall and volley drives, side-front boast, basic serve, the principle of never planting and waiting, side-on preparation | Must match recordings of the old simulation |
| IMPROVE | Lob, drop from deep, kill, back-wall boast, all positioning and spacing, arrival timing, footwork, recovery, movement speeds, interference | Specified fresh above; old recordings are "before" reference only |
| Operator rulings still needed | Ball-hits-player check [enable]; hit-stop pausing the simulation [make visual only]; ARCHITECTURE.md section 5 ban on automated tests, which parity checking needs [lift for the rebuild] | Recommendation in brackets |

Most tuned numbers behind the KEEP items exist only in code and will be
written into the architecture document. Old tests: `node --test
tests/*.test.mjs` (about 60 cases).

## How the work is done (quota-aware)

- Claude orchestrates and reviews; Anthropic quota is scarce. Codex is the
  fallback orchestrator if it runs out.
- GLM-5.3 with 1M context does the coding, dispatched via `zclaude --acp`
  with `acpx`. Model confirmed from returned metadata on every run; each task
  carries a manifest of owned files; GLM never accepts its own work. First
  dispatch is a trivial test task. (The ACP procedure is marked experimental
  in `~/Admin-Manual/DOCS/TOOLING_CONVENTIONS/acp-sdlc-semantic-review.md`;
  this is its first product use.)
- CodeGraph first, at no model cost: symbol inventory, call order and
  dependency order from the graph; model effort only for describing what each
  behaviour is for. Final check maps every old simulation function to a spec
  line or "deliberately dropped".
- Resumable: architecture doc, task list with states, and review notes live
  in the repo and KStore.
- Architecture Law (`/home/robin/Admin-Manual/CLAUDE.md`, repo `README.md`):
  ARCHITECTURE.md is a complete snapshot before coding; CHECKLIST.md tasks
  derive only from it, self-contained, naming the exact symbols they wire.

## Verified engine facts (web research, 2026-11-09)

- Pin UE 5.8.4. Linux install is Epic's prebuilt zip.
- MetaHuman Creator runs in the Linux editor since 5.7. Fallback: author on
  Windows and copy `Content/MetaHumans/`.
- A MetaHuman's head shape cannot be edited while the game runs, so the face
  photo is a texture fit onto a roster head.
- NOT confirmed, proven in Step 1: ray tracing stability on Linux with driver
  595.99; running a face-landmark model in a packaged Linux build; exact disk
  footprint; how far headless authoring goes.

## Target architecture

New repository layout:
- `DOCS/PRD.md`, `DOCS/ARCHITECTURE.md`, `CHECKLIST.md`, `CLAUDE.md`.
- `SquashShowcase/` - the Unreal project (`Content/` under Git LFS).
- `oracle/` - Node scripts that import the old simulation from the old repo
  at a pinned commit and write recordings to `oracle/golden/`.
- `.claude/settings.json` - only the plugins and MCP servers this project
  uses (codegraph, kstore).

| Module | Role |
|---|---|
| `SquashSim` | Pure C++ simulation, no engine object types. Fixed 1/120 s step, 4 substeps, doubles, old coordinate frame, bit-exact seeded RNG. Includes the chance enumerator, footstep planner, shot solvers, interference table, AI. |
| `SquashGame` | Fixed-step driver, input sampling, match flow, replay. Publishes a read-only snapshot each tick. |
| `SquashPresentation` | Ball, MetaHuman character and animation, court builder (two courts), wall-visibility controller, floor overlays (lanes, paths, markers), camera director, audio. |
| `SquashFace` | Landmark detection, photo-to-head texture fit, skin-tone match, template picker. |
| `SquashUI` | Menu, roster and photo flow, HUD. |
| `SquashEditorTools` | Level/material generation scripts, parity checker, contrast and frame-time checks. |

Hard boundary: presentation only reads the snapshot (ball state and events;
per player position, facing, planned foot placements and times, stroke phase,
contact point and time; lanes and previews; match phase and score). Only
tick-stamped input events flow back. Animation realises the simulation's
decisions and never changes them: authored key poses plus full-body IK onto
the planned foot placements, racket face on the contact point at contact time.

Glass: one translucent layer per wall; a controller decides per frame, from
the camera position, whether each wall is "looked through" (clear,
reflections capped) or "backdrop" (dense inside-face frit, stable matte).

## Parity method (KEEP items only)

- Recordings from the old simulation in Node, one JSON Lines file per
  scenario with commit, parameters, seed and status tag.
- Ball trajectories per KEEP shot, nick scenarios, rule outcomes.
- Tolerances: ball within 1 mm over 3 s with identical event order; seeded
  rallies compared statistically.
- IMPROVE items are recorded as "before" only and never gate.

## Build sequence

| Step | Deliverable | Exit criterion |
|---|---|---|
| 0. New repo and PRD | Create the new repository with a trimmed project-scoped `.claude/settings.json` and its own CodeGraph index; write `DOCS/PRD.md` from the decisions in this plan (purpose, controls, shot ideals, positioning and footwork, interference, courts and visibility, players and face photo, scope); save to KStore | Operator approves the PRD; per-message overhead lower in the new repo |
| 1. Feasibility spikes | UE 5.8.4 installed; empty C++ project builds and packages from the command line; glass box with ray tracing rendered offscreen; one MetaHuman assembled on Linux; landmark model run in a packaged build; a photo fitted onto that head; trivial GLM dispatch proving the route and model | Each passes or its fallback is chosen; disk use measured |
| 2. Spec | `DOCS/ARCHITECTURE.md` derived from the approved PRD (all code-only numbers for KEEP items read from the old repo; entity table with engaged-by column) and `CHECKLIST.md` (self-contained tasks naming their symbols, each with an owned-files manifest) | Operator approves both |
| 3. Oracle and sim port | Recordings; `SquashSim` | All KEEP scenarios within tolerance |
| 4. Vertical slice | Show court, one MetaHuman with planned footwork, ball, new controls, floor overlays, back-wall and top-down cameras | Contrast checks pass; operator plays a rally and judges controls and footwork |
| 5. Full match | AI, serve, scoring, interference, replays, HUD, club court | A full solo match completes |
| 6. Face flow and roster | Menu flow, photo fit, tone match, hair set | Photo to preview works in a packaged build |
| 7. Polish and package | Arena, crowd, audio, performance | Frame-time target met; packaged Linux build runs |

## Verification

- Build: `Engine/Build/BatchFiles/Linux/Build.sh SquashShowcaseEditor Linux
  Development -project=...`; package with `RunUAT.sh BuildCookRun ...
  -platform=Linux -build -cook -stage -pak -archive`.
- Parity: checker replays recorded inputs through `SquashSim` and diffs
  against `oracle/golden/`.
- Controls and footwork: scripted input sequences checked against the spec
  (each grid cell from each court region gives the stated shot; side-on at
  contact; recovery begins within the swing; Space behaviours), then operator
  play-test at Step 4.
- Visibility: automated run with `-RenderOffscreen` per camera mode with the
  ball on a grid; ball-to-surround luminance contrast at least 3:1 at every
  point in back-wall and top-down views on both courts (proposed line).
- Performance target (not yet measured): 60 fps at 1440p in play.

## Steps only the operator can do

- Before Step 1: free disk to about 180 GB (119 GB free now); add 32 GB+ swap
  (MetaHuman assembly can exhaust 31 GB RAM); Epic account; download UE 5.8.4.
- Step 1 and Step 6: create the roster in MetaHuman Creator, with guidance.
- Step 2: the three remaining rulings; approve the spec.
- Steps 4 and 7: judge controls, footwork, look and feel.

## Risks and expected shortfalls

| Risk | Mitigation |
|---|---|
| Ray tracing unstable on Linux with this driver | Step 1 gate; newer engine branch; driver 580; software lighting as last resort |
| MetaHuman assembly fails or runs out of memory | Swap; author on Windows |
| Disk runs out | Free space first; delete the zip after extracting; small roster |
| Landmark model fails in a packaged build | Link ONNX Runtime directly |
| GLM struggles with Unreal C++ | Small tasks with full interfaces, compile check after each; Claude takes engine-wiring tasks |
| New controls or human-speed movement feel wrong | Defaults are settings; operator play-test at Step 4 before the full match is built |

Likely to fall short of "indistinguishable from broadcast": weight shift in
the footwork without motion capture (pattern will be right, fluidity less
so); cloth; hair cannot come from the photo; likeness is a face texture on a
template head, so profile and head shape will not match; crowd quality from
free assets.
