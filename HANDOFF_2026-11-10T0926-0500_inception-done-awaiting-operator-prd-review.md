# Handoff: squash (Unreal Engine 5 rebuild), inception done, waiting on the operator

Written 2026-11-10 09:26 (UTC-5) by Claude (orchestrator seat). The operator
paused this project to budget model quota. This file is for whichever agent
or person picks it up next, on any machine. It assumes you have seen none of
the earlier conversation.

## 1. What this project is

A native, GPU-heavy squash game in Unreal Engine 5 with MetaHuman players. It
is a proof of concept whose whole point is realism: it should look like a real
squash match between real people, with the user's own face on their player.
Ease of building and breadth of device support are explicitly not goals.

It replaces an existing three.js browser game with cartoon players. None of
that game's code is carried over. Its specifications and tuned numbers are.

**Purpose of the game (the operator's words, paraphrased):** to habituate
optimal timing, shot choice and technique passively. The on-screen player is
the teacher. The user only chooses the shot and its timing; the game moves the
player with ideal footwork and positioning. Every design decision is judged
against that purpose.

## 2. Where everything is

| Thing | Location |
|---|---|
| This repository | `https://github.com/rebots-online/squash`, branch `master` |
| Product requirements, the authority | `DOCS/PRD.md` |
| Approved plan with build steps 0 to 7 | `DOCS/APPROVED_PLAN_2026-11-09.md` |
| Project rules for agents | `CLAUDE.md` |
| Donor game (reference only) | Tag `donor-threejs-v1` in this repository (commit `180ec040da06c7c444f2ace3e10730b2bcdbe173`); the same game is the template repository `rebots-online/squash-arcade` |

If the PRD and the plan disagree, the PRD wins. The one known difference is
the target platform (see section 4).

On the original machine the repository is at
`/home/robin/CascadeProjects/squash-showcase` and a donor checkout at
`/home/robin/CascadeProjects/squash` (do not pull there; it would lose the
donor files). On another machine, clone this repository once and get the
donor beside it with `git worktree add ../squash-donor donor-threejs-v1`.

## 3. Current state

- Step 0 of the plan is complete: repository created, PRD written, plan
  approved, project notes written.
- There is **no Unreal project, no architecture document, no task list and no
  product code**. The engine is not installed.
- Nothing is in progress. The working tree was clean at handoff.

The order of work is fixed and must not be skipped:

1. Operator approves the PRD.
2. Step 1 feasibility spikes.
3. `DOCS/ARCHITECTURE.md`, a complete snapshot including an entity table.
4. `CHECKLIST.md`, self-contained tasks derived only from the architecture.
5. Product code.

Do not write product code before the operator has approved items 1, 3 and 4.

## 4. Decisions already made (do not reopen without the operator)

Full detail is in the PRD. In brief:

- **Engine:** Unreal Engine 5.8.4 (pinned), C++ modules, MetaHuman. Epic's
  licence is accepted.
- **Platform:** Windows desktop with an RTX 3090-class GPU is the first
  shipped target. Linux is the development environment on the same dual-boot
  machine. A Windows build can only be made on Windows, so Windows sessions
  are needed at four checkpoints: Step 1 feasibility, Step 4 first playable
  slice, Step 6 roster and face photo, Step 7 final packaging. No web version.
- **Assets:** free only. Proprietary licences are fine. Licences that
  restrict use (for example non-commercial only) are not.
- **Players:** developers author a small MetaHuman roster once and ship it.
  End users pick a body and supply a photo. The photo is fitted as a texture
  onto a roster head, because a MetaHuman's head shape cannot be changed
  while the game runs.
- **Courts:** a glass show court in an arena, and a club hardwood court with
  a glass back wall. A wall between the camera and the play must never hide
  the players or ball, and see-through must never hurt playability.
- **Controls:** the user never steers.
  - Left hand holds one key of `Q W E / A S D / Z X C`. Rows are height on
    the front wall (high, mid, low). Columns are the literal direction the
    ball leaves the racket (left, straight, right). From the right side of
    the court, left is cross-court, centre is straight, right is a boast;
    mirrored on the left side.
  - Right hand presses `J`, `K` or `L` to hit soft, medium or firm. The press
    starts the step-and-swing, and its timing sets the quality.
  - `Space` held reins the player in: wait for a later ball, stay out of the
    opponent's lane, or stop (which is the appeal for a let). The player is
    deliberately eager by default so that reaching for Space is instinctive.
- **Shot ideals:** every shot aims at the ideal for its type (drop to the
  nick zone, drive dying in the back corner, lob falling steeply near the
  back wall, and so on). The game never aims for less.
- **Footwork:** never "arrive, plant, swing, then move". The player arrives
  one step short, side-on; the last step is the swing; the same step finishes
  as the recovery toward the T. Foot placements are planned by the simulation
  and the body is solved onto them.
- **Interference:** an explicit let and stroke table checked against the
  current World Squash rules, automatic clearing that is visibly intentional,
  and floor overlays showing the opponent's lane and the recovery path.
- **Kept from the donor game** (must match recordings of its simulation):
  ball physics, nick model, wall and tin collisions, scoring and serve rules,
  strike reach gate, input buffer, the basic drives and boast.
- **Improved, not copied:** lob, drop from deep, kill, back-wall boast, all
  positioning and spacing, footwork, recovery, movement speeds, interference.

## 5. Waiting on the operator

Nothing else can proceed until these are done:

1. Review `DOCS/PRD.md` and fill the two parts marked `[OPERATOR]`: the
   section 0 inception gate and the section 12 deployment vector.
2. Say whether a Forgejo remote is wanted as well as GitHub.
3. Rule on the items marked `[PROPOSED]` in the PRD. They are suggestions
   from the agent, not decisions. The main ones:
   - Enable the ball-hits-player check (the donor code never fires it).
   - Make hit-stop a visual effect only (in the donor it pauses the
     simulation).
   - Holding a shot key is preparation only; power comes from the fire key.
   - A mistimed hit is a looser version of the same shot, never a different
     shot.
   - Under pressure, footwork is a compressed version of the same pattern.
   - A boast is only chosen when the player is clearly in the outer third.
   - A short explained freeze after each let or stroke call.
   - Ball-to-surround contrast of at least 3:1; 60 fps at 1440p.
4. Download Unreal Engine 5.8.4 with an Epic account (the Linux prebuilt zip
   for development; the Windows install for the Windows checkpoints).

## 6. What the next agent does once unblocked

Step 1 feasibility spikes, each of which either passes or has its fallback
chosen:

- An empty C++ Unreal project builds and packages from the command line.
- A glass box with hardware ray tracing renders offscreen.
- One MetaHuman is assembled.
- A face-landmark model runs inside a packaged build.
- A photo is fitted onto that MetaHuman's head.
- A trivial coding task is dispatched to the coder model and comes back with
  the model confirmed (see section 7).
- Disk use is measured.

Then write `DOCS/ARCHITECTURE.md` and `CHECKLIST.md` for operator approval.

Facts already checked by web research (2026-11-09):

- Unreal 5.8.4 is stable; Linux ships as a prebuilt zip.
- MetaHuman Creator runs in the Linux editor since 5.7. Assembly may exhaust
  31 GB of RAM. Fallback: author on Windows and copy `Content/MetaHumans/`.
- Unreal has no webcam capture on Linux.

Not yet confirmed, to be proven in Step 1: ray tracing stability on Linux
with NVIDIA driver 595.99; running the landmark model in a packaged build;
the true disk footprint; how much can be authored without the editor GUI.

## 7. How the work is meant to be done

- **Roles.** Claude orchestrates and reviews; its quota is scarce. Coding is
  done by GLM-5.3 with a 1M-token context, dispatched through `zclaude --acp`
  with `acpx`. Confirm the model from the returned metadata on every run.
  Each task carries a list of the files it owns. The coder never accepts its
  own work. Codex is a fallback orchestrator only.
- **Donor code.** Never modify it and never copy its source. Use it for
  tuned numbers and as a reference: its simulation (`src/squash/*.js`) runs
  headless in Node, and `node --test tests/*.test.mjs` runs its tests. Use a
  code graph tool to inventory it before spending model effort.
- **Known donor bugs** (do not reproduce): the ball-hits-player and
  interference checks are called with the wrong arguments and never fire;
  hit-stop pauses simulation time; the lob lands far too short.
- **Parity.** Record the donor simulation's output for the "kept" items as
  JSON Lines files, and require the new simulation to match (ball within
  1 mm over 3 s, same event order). "Improved" items are recorded only as a
  before reference and never gate.
- **Conventions on the original machine.** The global rules live in
  `/home/robin/Admin-Manual`. Continuity notes live in a store called KStore;
  this file exists so that the project is also resumable without it. Scratch
  files go in `.tmp/`, never `/tmp`. Commit and push after every task.
- **Working with this operator.** Ask questions in plain text, not through
  multiple-choice dialogs. Report honestly when something was not done.

## 8. Machine notes (original machine only)

These apply only to the dual-boot desktop this was started on:

- Linux system drive: 196 GB free after clearing build output on 2026-11-10.
  That is enough for the engine. No repartitioning is needed.
- 31 GB RAM and 30 GB swap. No extra swap is needed.
- Windows is already installed. Its system drive is 256 GB, so the engine
  must go on a second drive there. Free space on a Windows-usable drive has
  not been measured; check it at the first Windows session.

## 9. History of this handoff

| Commit | What |
|---|---|
| `180ec04` | The three.js game (tag `donor-threejs-v1`) |
| `83ee3e2` | Three.js game retired from the working tree |
| `d4a2744` | Inception PRD and project notes |
| `71afec6` | PRD: Windows is the first target, Linux the development environment |
| `5c82bfe` | Approved plan copied into the repository; this handoff |
| next commit | Project moved into `rebots-online/squash` |
