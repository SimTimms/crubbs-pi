# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project

A 3D space exploration game built with React Three Fiber. The player pilots a spaceship through a scaled solar system with orbital mechanics, NPC ships, docking, combat, scanning, and narrative systems.

The application code lives in `my-r3f-app/`.

## Commands

Run all commands from `my-r3f-app/`:

```bash
npm run dev       # Start Vite dev server with HMR
npm run build     # tsc -b && vite build
npm run lint      # ESLint
npm run preview   # Preview production build
```

There are no tests configured.

## Architecture

**Entry flow:** `main.tsx` → `App.tsx` → `AppShell.tsx` → `SceneLayer.tsx` → `Scene.tsx` (React Three Fiber `<Canvas>`)

**App layers** (`src/components/App/`):
- `SceneLayer` — R3F Canvas with all 3D objects
- `HudLayer` — HUD overlays (nav, radio, inbox, scanning, power)
- `DialogLayer` — Modal dialogs (docking, messages, selection, comms)
- `ControlLayer` — Keyboard and mobile input capture
- `AudioLayer` — Audio element setup

**Global state pattern:**
- Module-level refs in `src/context/*.ts` — high-frequency ship physics state (position, velocity, quaternion, fuel, O2, hull). These avoid React re-renders.
- React context for save data (`SaveStore.ts`) and time-sensitive UI events (`MessageStore.ts`)
- Config constants in `src/config/*.ts` — see Config section below

**Ship physics** (`src/hooks/useShipPhysics.ts` + `src/hooks/shipPhysics/`):
- `useFrame` loop runs physics step each frame
- Submodules handle: inputs, gravity, collisions, docking, resource drain, engine audio, thruster light
- Ship state (THRUST, fuel, O2, hull integrity, etc.) is in `src/context/ShipState.ts`

**Scene composition** (`src/components/Scene.tsx`):
- `OrbitCamera` (`Camera.tsx`) — mouse drag (quaternion yaw/pitch) + scroll (FOV zoom)
- `SolarSystem` — orbital planets using scaled AU distances
- `Sun`, `AsteroidBelt`, `SpaceParticles`, `NebulaClouds`, `SkySphere` — environment
- `Spaceship` — player ship with GLB model + physics integration
- `AIShip`, `GhostFleet` — NPC ships
- `SpaceStation`, `FuelStation`, `RadioBeacon`, `LandingPad` — world objects
- `RailgunWarning`, `LaserRay` — combat
- `AutopilotController` — orbital autopilot maneuvers

**Autopilot** (`src/autopilot/`):
- 14 files covering approach, circularization, orbit insertion, hyperbolic capture, SOI transitions
- All maneuvers work via the `AutopilotCtx` interface in `types.ts`

**Config files** (`src/config/`):
- `solarConfig.ts` — solar system scale, planet sizes
- `worldConfig.ts` — planet/station/beacon definitions, audio paths
- `scanRanges.ts` — HUD sensor ranges (proximity, magnetic, drive signature, radio)
- `damageConfig.ts` — collision multipliers, railgun damage, O2/fuel drain/refill rates
- `neptuneConfig.ts` — Neptune no-fly zone, railgun timing
- `commsConfig.ts` — comms delay simulation (speed of light scaling)
- `ghostFleetConfig.ts` — NPC ship/station names, fleet spawn radius

**Narrative** (`src/narrative/`):
- `inboxMessages.ts` — story messages with player choices
- `npcDialogues.ts` — NPC conversation trees
- `radioChatter.ts` — background radio lines
- `shipRegistry.ts` — NPC ship names and factions
- `contacts.ts` — contact list entries
- `commsDelay.ts` — message delay calculator

**3D models** are `.glb`/`.gltf` files in `my-r3f-app/public/`, loaded via `useGLTF`.

## Key Patterns

- Use `useFrame` for per-frame updates; avoid state changes inside `useFrame`
- Ship physics state lives in module-level refs (`src/context/ShipState.ts`, `ShipPos.ts`, etc.) — read directly, never set via React state
- Camera rotation is quaternion-based in `Camera.tsx` — avoid Euler angles to prevent gimbal lock
- `tsc` strict mode is on (`noUnusedLocals`, `noUnusedParameters`) — unused variables will fail the build
- Config values should live in `src/config/` — never hardcode magic numbers in components
- Debug flags are scattered; consolidate into `src/config/debugConfig.ts` when adding new ones

## Known Issues / Refactoring Notes

See `my-r3f-app/REFACTORING.md` for a full analysis of dead code, hardcoded values to move to config, and recommended folder reorganization.
