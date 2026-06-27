# BUILDOC — Shootr2
> Carry this document into every new chat session. Read it fully before writing any code. Update it at the end of every session using the instructions at the bottom.

---

## 1. Project Identity

**Working title:** Shootr2 (name subject to change)
**Engine:** Unity 6000 (Unity 6)
**Render pipeline:** URP
**Input system:** Unity Input System Package (new) — legacy input disabled and confirmed
**Platforms:** Windows, macOS, Linux
**Repo:** https://github.com/thepr0phet/shootr2

**One-paragraph vision:**
Shootr2 is a lightweight local split-screen FPS for small friend groups (2–4 players), inspired by Halo customs and old CoD split-screen. The tone is simple, fun, and moderately fast-paced — not goofy, not serious. No gore. The default perspective is first-person; third-person may be added later as an option but is not a priority. The aesthetic is intentionally low-poly and minimalist; only features and visuals that have been explicitly decided on should be added. The game is primarily a "fun thing me and my brothers can play together" with no firm release target. Simplicity, performance, and maintainability are the top priorities.

---

## 2. Core Design Decisions
> These are locked decisions. Do not re-litigate or suggest alternatives unless the user explicitly opens them up.

| Decision | Choice | Reason |
|---|---|---|
| Default perspective | First-person (third-person possible later, not planned) | Player preference |
| Movement | Acceleration-based with separate forward/strafe speeds, air control multiplier, deadzone jitter fix | Ported feel from fps-unity FPSMotor |
| Look | Smoothed X/Y with gun lag effect | Ported feel from fps-unity FPSLook |
| Crouch/prone input | Tap B = crouch, Hold B = prone (hold timer in code, one binding) | CoD-style |
| Weapon pickup | None — players always spawn with weapon | Simplicity |
| Weapon damage model | Hitscan, raycast-based with spread | Lightweight, fits small maps |
| Networking | Local split-screen first, NGO layer added later as separate concern | Don't degrade local experience |
| Multiplayer scope | 2–4 players, same machine split-screen primary; multi-machine possible later | |
| Art style | Low-poly, minimalist, programmer art acceptable long-term | Lightweight + intentional |
| Game mode architecture | Modular — modes built from stat presets, gun sets, map, ruleset | Easy to add new modes |
| Code approach | Start fresh, use fps-unity as reference only | fps-unity is too netcode-entangled to copy cleanly |

---

## 3. Planned Game Modes
> Not all built — this is the intended list for reference.

- **Deathmatch** — standard free-for-all, primary mode to build first
- **Instagib** — one-hit kill variant, high priority
- **Team Juggernaut** — CoD-style, one player is the juggernaut, team tries to kill and take the role
- **Search & Destroy** — or similar objective mode, lower priority

---

## 4. Architecture

### Folder Structure
```
Assets/
  _Game/
    Characters/       # Player prefab, controller, state machine, input reader
    Input/            # Input Actions asset + generated C# wrapper
    Weapons/          # Weapon base class, individual weapon scripts
    Modes/            # Game mode logic, ruleset ScriptableObjects
    Maps/             # Map-specific scripts, spawn point managers
    UI/               # HUD, kill feed, scoreboard, menus
    Core/             # Game manager, match manager, player registry
  _Art/               # Materials, meshes, textures (minimal)
  _Settings/          # URP renderer asset, quality settings
```

### Files
> None yet — to be populated as files are created. Format:
> `Assets/_Game/Folder/FileName.cs` — what it owns — key public members

---

## 5. Current State

**Working:**
- Unity 6 project created and pushed to shootr2 repo
- .gitignore in place (fixed from `gitignore` → `.gitignore`, PackageCache excluded)
- URP and Input System packages confirmed installed
- Legacy input confirmed disabled

**Next session starts here:**
1. Create folder structure (`_Game`, subfolders) in Unity
2. Create `GameInputActions.inputactions` asset in `Assets/_Game/Input/`
3. Write `PlayerStateMachine.cs`
4. Write `PlayerController.cs` (movement from FPSMotor reference, look from FPSLook reference, new Input System)
5. Write `GameBootstrapper.cs` (single player test harness)
6. Build player prefab in scene
7. Build placeholder map (plane + cover boxes + spawn point)
8. Hit Play and verify basic movement
9. Resolve Xbox controller binding via Input Debugger

**Not started:**
- Weapon system
- Split-screen camera rig (2–4 players, wide panel on top for 3-player)
- UI / HUD
- Game mode system
- Networking layer (NGO — future)

---

## 6. Known Issues & Watch List

**Active issues:**
- Xbox One controller on Bazzite (Linux) via Bluetooth presents as generic Joystick in Unity, not Gamepad. Steam Input disabled — issue persists. Resolution: open Input Debugger (Window → Analysis → Input Debugger) after player is in scene, wiggle sticks, find exact control paths, hardcode bindings.

**Architectural watch items:**
- Networking (NGO) must be a layer on top of local simulation — never let it bleed into core gameplay logic. Local play runs with zero network overhead.
- Game mode system must be modular — modes composed of: player stat preset + weapon set + map + ruleset. Don't hardcode mode behavior into core systems.
- Grapple hook: stub the `Grappling` state in PlayerStateMachine from day one. Do not implement until explicitly requested.
- Crouch/prone share one controller binding (B) — tap = crouch, hold = prone. Hold timer lives in PlayerController, not in the input asset.
- Ceiling SphereCast required before allowing crouch→stand transition.
- Split-screen layout: 3-player uses wide panel on top, two panels on bottom.
- Gun lag effect (from FPSLook) is desired — port it when implementing look.
- Movement feel reference: FPSMotor — acceleration/deceleration lerp, separate forward/strafe max speeds, deadzone jitter fix, air control multiplier.

**Do not do:**
- Do not use legacy Unity Input System anywhere
- Do not copy fps-unity code directly — use as reference only, rewrite cleanly
- Do not add NGO/Netcode to any class until networking phase is explicitly started
- Do not add features, visuals, or systems that haven't been explicitly requested
- Do not add gore, over-the-top effects, or visual complexity
- Do not make players pick up weapons from the map
- Do not assume third-person is wanted — first-person is the default

---

## 7. Reference Material

### fps-unity (old project) — https://github.com/thepr0phet/fps-unity
Good logic to port (rewrite cleanly, do not copy):
- `FPSMotor.cs` — acceleration model, deadzone fix, air control, jump
- `FPSLook.cs` — smoothed look, gun lag effect
- `FPSGun.cs` — raycast loop, spread direction, hit marker coroutine, audio pitch variation

Discard entirely:
- `PlayerInputReader.cs` — uses legacy Input system
- All Netcode: `NetworkBehaviour`, `ServerRpc`, `ClientRpc`, `NetworkTransform`, `IsOwner` checks

### Dependencies & Environment

| Item | Detail |
|---|---|
| Unity version | 6000 (Unity 6) |
| Render pipeline | URP (com.unity.render-pipelines.universal) |
| Input package | com.unity.inputsystem — new system confirmed active |
| Networking (future) | Netcode for GameObjects (NGO) — not yet installed |
| Dev OS | Bazzite (Linux) |
| Controller | Xbox One via Bluetooth — presents as Joystick on Linux, not Gamepad |
| Version control | Git / GitHub — shootr2 repo public |

---

## 8. Session Log
> One line per session. Most recent at top.

- **Session 1** — Architecture planned, design decisions locked, reviewed fps-unity codebase, decided to start fresh using fps-unity as reference only, Unity project created and pushed to repo, .gitignore fixed, BUILDOC finalized.

---

## 9. End-of-Session Update Instructions
> For the LLM at the end of each session:

1. **Section 4 (Files)** — add new files. One line: path, what it owns, key public members. Remove deleted files.
2. **Section 5 (Current State)** — update Working / Next / Not Started accurately. Don't mark working unless tested in editor.
3. **Section 6 (Known Issues)** — add new issues, remove resolved ones, keep watch items current.
4. **Section 8 (Session Log)** — one line summary of this session.
5. Do not change Section 2 unless the user explicitly changed a decision this session.
6. Keep the doc concise — remove stale info, don't just append.
7. Tell the user: *"BUILDOC updated — copy the latest version to your repo before closing."*
