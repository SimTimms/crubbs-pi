# Game Mechanics Reference

A reference document for designing gameplay elements, plots, missions, and story content for the React Three Fiber space simulation.

---

## World Overview

A 3D space environment rendered in real time. The player pilots a spaceship through a dark expanse containing a space station, a fuel station, two planets, scattered radio beacons, space debris, and an asteroid belt. The world is large — objects are thousands of units apart — and navigation requires managing limited resources.

---

## The Player Ship

### Movement & Controls

| Key | Action |
|-----|--------|
| W | Thrust forward |
| S | Thrust reverse |
| A | Yaw left |
| D | Yaw right |
| Q | Strafe left |
| E | Strafe right |
| SPACE | Undock (when docked) |
| M | Toggle minimap |

- **No drag in space** — velocity persists until countered by thrust
- **Y-axis locked** — ship travels in a flat plane (no vertical movement)
- **Thrust multiplier** — UI slider lets player push 0.5×–3× thrust (dangerous above 2×)
- Camera follows ship; mouse look + scroll zoom (FOV 10°–50°)

### Resources

All resources range 0–100. Hitting zero on hull or O2 has severe consequences.

| Resource | Drain | Refill | Effect at Zero |
|----------|-------|--------|----------------|
| Power | 1/sec per active thrust key | Passive (not shown) | Unknown — needs design |
| Fuel | 1/sec while thrusting | 10/sec while docked | Cannot thrust |
| O2 | 1/sec always | 10/sec while docked | Death (implied) |
| Hull Integrity | On collision (scaled by impact speed) | None currently | Ship destroyed |

### Thrust Multiplier (Slider)
- **0.5×–1.9×**: Normal, safe operation
- **2×–3×**: "DANGER" mode — faster travel but higher G-forces and collision damage risk

### G-Force
- Sustained acceleration builds a blackout overlay proportional to G-force
- High thrust multiplier + aggressive maneuvering → dangerous G accumulation

---

## Physics & Collision

- **Physics is Newtonian** — no drag, momentum carries the ship indefinitely
- **Collision damage** scales with impact speed × 1.2 multiplier
  - Slow docking at <4 m/s: safe
  - High-speed collision: potentially fatal
- **Bounce/restitution**: 0.4 — collisions deflect rather than stop the ship
- **Collidable objects**: Space station, fuel station, docking bays, all 54 debris pieces, ~1560 asteroids

---

## Docking

### Docking Ports
Both the **Space Station** (at [0, 0, -500]) and the **Fuel Station** (at [0, 0, 160]) have docking bays.

### Docking Process
1. Approach docking port at **< 4 m/s** relative velocity
2. Align with docking bay collision zone (2-unit radius port, 9 units forward from ship nose)
3. Automatic dock on contact at safe speed
4. Press **SPACE** to undock (ejects at 4 m/s)

### While Docked
- Fuel refills at 10/sec
- O2 refills at 10/sec
- Access to **Missions** dialog
- Access to **Cargo** management
- Hull cannot be repaired (potential gameplay hook)

---

## Missions (Current)

Missions are assigned at docking stations and load cargo into the ship's hold.

| Mission | Destination | Cargo Required | Quantity |
|---------|-------------|----------------|----------|
| Mars Run | Red Planet [3500, -5000, -12000] | Food | 20 units |
| Neptune Run | Neptune [0, -7000, 0] | Data Cores | 15 units |

Missions are currently structured as delivery runs. No completion/reward mechanics are implemented yet — this is a core design opportunity.

---

## Cargo System

- Cargo items stored as `{ name, quantity }` entries in an inventory array
- Items are visible in the HUD (top left), clickable to eject
- **Ejection mechanic**: Two-step confirmation → select quantity → items launched as physics cubes
  - Inherit ship velocity + -3 m/s backward impulse + random spread
  - Cubes persist 60 seconds before despawning
  - Max 200 simultaneous ejected cubes

---

## Laser / Targeting System

- **Mouse hold**: Fire cyan laser beam from ship nose (max range 1000 units)
- **Power cost**: 2/sec while firing
- **Spotlight**: 100-unit range illumination while laser is active (toggle button)
- **Raycast targeting**: Detects Space Station meshes and Radio Beacons
- **Impact visualization**: Pulsing orange sphere + distance label at hit point
- **Target lock**: Clicking objects (station, beacons, docking bays) sets a target in the HUD
  - HUD shows target name + relative velocity
  - "Docking velocity" hint appears when <4 m/s near a docking bay

---

## Radio Beacons

10 beacons scattered in the region between the origin and the space station.

| Beacon | Position | Has Audio |
|--------|----------|-----------|
| 0 | [80, 0, -1760] | Yes (`beacon-001.mp3`) |
| 1 | [-220, 0, -850] | Yes (`radio.mp3`) |
| 2–9 | Scattered in -1760 to -310 Z range | No |

### Interaction
- **Laser hit**: Beacon pulses faster (breathing rate 6×), shows "RadioBeacon" label
- **Click**: Triggers audio playback if beacon has an audio file
- Audio managed by App.tsx; play/pause button appears in UI after interaction

### Design Notes
Beacons are positioned along the route from origin toward the space station. They currently play mystery audio — a strong foundation for narrative delivery, distress signals, coded messages, or lore drops.

---

## Magnetic Scanner (HUD)

- **Toggle**: "MAGNETIC" button (bottom left)
- Detects **metallic debris only** within 10,000 unit scan radius
- **On-screen**: Orange bracket box around object + distance
- **Off-screen**: Diamond indicator at screen edge + distance
- Distance shown in meters (<1000) or km (≥1000)

### Magnetic Debris Types
Hull Fragments, Steel Struts, Metal Containers, Solar Panels, Engine Casings

### Non-Magnetic Debris Types
Meteorites, Water Tanks, Coolant Pods, O2 Tanks, Fuel Pods

---

## Space Debris Field

54 debris objects scattered in a sphere 1500–7500 units from origin. Generated with a fixed seed (9417), so their positions are deterministic and stable.

| Type | Qty | Magnetic | Notes |
|------|-----|----------|-------|
| Hull Fragment | 8 | Yes | Gray metallic box |
| Steel Strut | 6 | Yes | Dark thin box |
| Metal Container | 5 | Yes | Brown metallic box |
| Solar Panel | 5 | Yes | Very flat dark blue, emissive |
| Engine Casing | 4 | Yes | Dark cylinder |
| Meteorite | 8 | No | Rocky brown icosahedron |
| Water Tank | 5 | No | Transparent blue cylinder |
| Coolant Pod | 4 | No | Cyan emissive sphere |
| O2 Tank | 5 | No | Light blue cylinder |
| Fuel Pod | 4 | No | Orange emissive sphere |

All debris have sphere colliders — they can damage the ship on impact.

---

## Asteroid Belt

1560 asteroids placed along the route between Neptune and the Red Planet. Dense and procedural (seed 7331).

- Scale: 10–65 units each
- Only asteroids near the ship's travel plane (Y ± 300) are registered as active colliders
- Visually impassable in bulk — strategic navigation required

---

## World Map (Key Locations)

```
                    [Origin / Start]
                         |
                    Fuel Station [0, 0, 160]
                         |
                    (Beacon field Z: -300 to -2200)
                         |
                    Space Station [0, 0, -500]
                         |
                         |
                    Neptune [0, -7000, 0]
                              \
                           (Asteroid Belt)
                                 \
                              Red Planet [3500, -5000, -12000]
```

Debris is scattered in a large sphere around the origin (1500–7500 units out).

---

## Day/Night Cycle

- Full 24-hour cycle completes in ~10 real seconds (configurable)
- Affects ambient and directional lighting colors across 5 keyframes
- Bedroom mesh in GLB model brightens at night (emissive)
- Currently cosmetic only — no gameplay effect from time of day (design opportunity)

---

## HUD Summary

| Element | Location | What It Shows |
|---------|----------|---------------|
| Power HUD | Top left | PWR, HUL, FUL, O2, G-force, velocity, cargo |
| Target display | Power HUD | Selected target name + relative velocity |
| Magnetic HUD | Viewport overlay | Metallic debris positions + distances |
| Minimap (M) | Full screen | All world objects, ship position |
| G-force blackout | Full screen | Black overlay when accelerating hard |
| Hull breach overlay | Full screen | "SHIP DESTROYED" on hull = 0 |
| Thrust slider | Bottom center | Multiplier 0.5–3× |
| Spotlight toggle | Bottom left | Enable/disable laser spotlight |
| Magnetic toggle | Bottom left | Enable/disable magnetic scanner |
| Audio button | Bottom left | Play/pause beacon audio |
| Docking dialog | Center | Refuel, O2, Missions, Undock |
| Eject dialog | Center | Confirm cargo ejection + quantity |

---

## Events (Internal System)

These are things the game "knows happened" — useful anchors for mission triggers, story beats, or progression logic:

| Event | When |
|-------|------|
| `ShipDocked` | Entered docking bay |
| `ShipUndocked` | Left docking bay |
| `ShipDestroyed` | Hull integrity hit zero |
| `SpaceStationModelHit` | Laser hit the space station mesh |
| `RadioBeaconHit` | Laser swept across a beacon |
| `RadioBeaconClicked` | Player clicked a beacon (plays audio) |

---

## Design Opportunities & Gaps

These systems exist but have incomplete or absent gameplay loops:

- **Power resource**: Drains with thrust but has no low-power consequences yet
- **Hull repair**: Cannot currently be repaired (docking dialogs have no repair option)
- **Mission completion**: No delivery confirmation or reward when cargo reaches destination
- **O2 depletion**: Drain mechanic exists but death/consequence not fully implemented
- **Beacon audio**: Only 2 of 10 beacons have audio files
- **Day/night gameplay**: Time system is visual-only; no events or mechanics tied to time of day
- **Debris collection**: Magnetic scanner highlights debris but no pickup mechanic exists
- **Fuel pods / O2 tanks in debris**: These debris types suggest potential field-scavenging gameplay
- **Red Planet / Neptune**: Both are reachable destinations with no arrival mechanic yet
