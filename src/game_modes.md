# Game Modes

This document describes the gameplay structure and the player's life cycle in the Glitchee universe. The system was designed to be modular, allowing content updates via backend without the need for new heavy binary deployments.

## 1. Story Mode: The Awakening

The "Extended Tutorial". Here the player meets Glitchee, understands his existential boredom, and learns to deal with the traps of each hardware architecture.

[Lore](lore.md)

- **Structure:** Divided into several "Machines", located in different worlds.
- **Sawtooth Progression:**
    - Each machine has N levels.
    - With each new machine, a new trap is introduced.
    - Difficulty rises gradually within the machine and undergoes a "soft reset" when jumping to the next (returning to a level slightly higher than the start of the previous machine).
- **Goal:** Infect the core of the **Alien Hivemind** and unlock the "Universal Administrator" status.
- **Reward:** Unlock *Infinite Simulation* mode and exclusive classic hardware skins.

## 2. Zero-Day Exploit (Global Competition)

The social and competitive mode, focused on daily retention and community events.

- **Concept:** Every week, the backend (automated pipeline in Rust) generates a set of 10 fixed levels for all players worldwide.
- **Availability:** Unlocked early (after World 3 - Home Router). Serves to keep engagement if the player gets stuck on a difficult story level.
- **Ranking (Hz/Efficiency):** Players are ranked by:
    1. **Code Efficiency:** Total length of the drawn line (the shorter, the better).
    2. **Near-Misses:** Number of times Glitchee passed within less than 1 collision unit without dying.
    3. **Execution Time:** Real time spent completing the challenge.
- **Dynamics:** Levels are downloaded via JSON transparently. It's the "Main Stage" for players to show who the best route engineer is.

## 3. Infinite Simulation (VM Sandbox)

The true *Endgame*. Here, the procedural engine in Rust is pushed to the limit, generating challenges that the original machines could never support.

- **Lore:** Glitchee uses the processing power of the Hivemind to run Virtual Machines (VMs). He recreates old environments (like the "Potato Box"), but with "Compilation Flags" that alter the laws of physics.
- **100% Procedural Generation:** Unlike Story Mode, walls, trajectories, and traps are generated in real-time by the causal regression algorithm.
- **Mutation Modifiers (Mutators):** Every 50 levels, a new "Error Flag" is activated in the VM:
    - `-packet-loss`: Projectiles blink (become intermittently invisible).
    - `-overclock-cpu`: Global speed increased by 25%.
    - `-bit-flip`: Touch commands inverted.
- **Legendary Traps (Post-Level 1000):** Introduction of "impossible" mechanics that Glitchee invented in the Hivemind, such as bullet teleportation portals and space-time distortions on the player's spline.

## Player Journey Summary

| Phase | Active Mode | Player Focus | Goal |
| --- | --- | --- | --- |
| **Early Game** | Story (Worlds 1-3) | Learn basic mechanics. | Survive the Potato Box. |
| **Mid Game** | Story + Zero-Day | Collect skins and compete. | Master mutable traps. |
| **Late Game** | Story (Worlds 7-10) | Solve complex puzzles. | Subjugate the Hivemind. |
| **End Game** | Infinite Simulation | Break level records. | Reach Level 1000+ with Mutators. |

## Technical Notes for Development (Rust/Godot)

1. **Seeds:** All procedural levels must be based on `Seeds`. This allows two players to play the same infinite level just by sharing the seed code.
2. **Validation:** The engine in Rust must validate if the generated level has at least one 5-star success path before presenting it to the player.
3. **Persistence:** Infinite mode progress is saved locally, but "Maximum Level Reached" records are synchronized with the Zero-Day server.
