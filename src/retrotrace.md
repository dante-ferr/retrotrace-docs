# RetroTrace

![glitchee.png](glitchee.png)

### Setup

[Arch Linux dev Setup](setup.md)

### Concept & Genre

- **Genre:** Tactical Bullet Hell / Puzzle (Hybrid-Casual).
- **Core Premise:** The tension of a bullet hell combined with the contemplative logic of a path-drawing puzzle. Time is frozen during planning; execution happens in real-time.
- **Target Platform:** Mobile (Portrait mode, One-thumb control).

### Lore & Theme

[Lore](lore.md)

### Core Mechanics

- **Two-State Loop:**
    1. *Planning State (Time = 0):* Player draws a freeform spline (Bézier curve) to navigate around stationary bullets and traps. Bullets show "ghost" vectors of their future trajectories.
    2. *Execution State (Time = 1):* Player hits "Play". Time unfreezes, bullets move, and the avatar traverses the drawn path at a constant speed.
- **Controls:** Draw with one thumb. Scrub backward on the drawn line to undo/rewind without lifting the finger.
- **Win/Loss:** Reach the exit portal/node to win. Any collision with a bullet/trap hit-box results in instant death and rapid reset.

[Game Modes](game_modes.md)

### Visual Identity

- **Visual Style:** "Programmer Art" elevated by heavy post-processing. Low-poly primitive shapes, Unlit materials, heavy Neon/Bloom, Vignette, and Chromatic Aberration.

### Hacks (Active Skills / Procedural Acrobatics)

- **Packet Overclock (Dash):** Aggressive forward lean, determined face, long solid light trail, screen shake.
- **Sector Defrag (Ghost/Intangible):** Glitchee disintegrates into an ASCII/binary particle swarm to pass through walls of bullets.
- **Buffer Overflow (Repel):** Error icon on screen, emits a procedural shockwave that recalculates/bounces incoming bullets away.

### Camera System

- **Planning Phase:** Strict Top-down (Orthographic) for tactical precision.
- **Execution Phase:** Procedural Cinematic Camera. Tilts down to a chase perspective, uses *LookAt* to anticipate upcoming curves on the spline, and dynamically adjusts FOV based on speed and near-misses.

### Level Generation Architectures (Decision Pending)

- [Retrocausal Level Generation (Inverse Math)](retrocausal_level_generation.md)
- [Iterative Forward Rejection (The Choreographer)](iterative_forward_rejection.md)

[Traps](traps.md)
