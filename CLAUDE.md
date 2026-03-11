# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project

A 3D interactive visualization built with React Three Fiber. The app renders a 3D scene with GLB models, a dynamic 24-hour day/night cycle, and interactive binocular-style camera controls.

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

**Entry flow:** `main.tsx` → `App.tsx` (wraps with `TimeProvider`) → `Scene.tsx` (React Three Fiber `<Canvas>`)

**Global time state** (`src/context/TimeProvider.tsx`):
- Provides `time` (0–24h), `t` (0–1 normalized), and `speed` (cycle duration in seconds, default 10s)
- All time-aware components consume this context via `useTime()`
- One full day/night cycle completes in `speed` seconds of real time

**Scene composition** (`src/components/Scene.tsx`):
- `BinocularCameraControls` — mouse drag (yaw/pitch via quaternions) + scroll (FOV 10–50°)
- `SunCycle` — orbiting directional light + ambient light with color interpolation across 5 keyframes (sunrise, midday, sunset, night, loop)
- `GLBModel` — loads static GLB files; traverses scene to find meshes named "Bedroom" and animates their emissive intensity based on time
- `AnimatedModel` — loads GLB files with animation clips; plays a named clip at configurable speed

**3D models** are stored as `.glb` files in `my-r3f-app/public/` and loaded at runtime via `useGLTF`.

**Unused components:** `Camera.tsx`, `DeskMan.tsx`, `DragRotate.tsx` — these are superseded by `BinocularCamera.tsx` and `AnimatedModel.tsx`.

## Key Patterns

- Use `useFrame` for per-frame updates (animation loop integration)
- Use `useRef` for camera state and mesh references that should not trigger re-renders
- Camera rotation is quaternion-based — avoid Euler angles to prevent gimbal lock
- `tsc` strict mode is on (`noUnusedLocals`, `noUnusedParameters`) — unused variables will fail the build
