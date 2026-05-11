# Traps

This document details trap ideas designed for the game's deterministic engine. Each trap is designed to work with the **Iterative Forward Rejection (IFR)** system, ensuring they can be simulated forward and "nudged" until a safe path for the player is guaranteed.

*Engineering Note:* Traps with infinite movement and no friction are prohibited, as they make the forward simulation loop unpredictable. Every moving trap must eventually reach a *State-Lock* (rest) or exit the simulation bounds.

## Progression and Difficulty Cycles (Sawtooth Curve)

The game uses a procedural progression architecture in a **Sawtooth Curve** format.

- **Initial Cycle:** The first cycle introduces basic traps (e.g., Static Shredder and Delay Sniper) to acclimate the player to the drawing and temporal execution mechanics.
- **Internal Scaling:** Within the same cycle, difficulty increases progressively (narrowing the STC's error margin and increasing trap density).
- **Cycle Break:** Upon advancing to the next cycle, difficulty undergoes a "soft reset," and **a new trap type is introduced** into the procedural generation pool.

# V1.0: LAUNCH FEATURES

Below are the 13 traps of the first version, structured to support procedural generation.

## Category 1: Direct Immutables (5 Traps)

*Agents that are not influenced by the environment or other traps. They are validated by nudging their initial firing angles or temporal offsets.*

1. **Static Shredder (Shotgun Node)**: Fires a cone of 5 simultaneous projectiles. IFR nudges the base orientation of the cone.
2. **Delay Sniper (Wind-up Sniper)**: Fixes a harmless aiming laser. After 2 seconds, an instant shot fires. IFR nudges the firing timestamp (phase shift).
3. **Continuous Sweeper Laser (Sweeper)**: A continuous beam that sweeps like a lighthouse. IFR nudges the starting rotation or the sweep speed.
4. **Intermittent Grid (Blinking Laser Grid)**: Laser walls that blink in rhythmic intervals. IFR nudges the temporal offset (duty cycle).
5. **Defective Homing Cannon (Drunk Homing)**: Projectile with a predictable, locked turn rate. IFR nudges the initial launch angle.

## Category 2: Kinematic Cascade Mutables (5 Traps)

*Entities submitted to the Single-Trigger State-Lock law. They interact with the simulation via isolated vector transfer.*

1. **Kinetic Puck**: A static disk that slides when hit by a projectile. IFR validates that its sliding path doesn't hit the player.
2. **Accelerator Lens**: A translucent block that doubles the speed of any projectile passing through it.
3. **Ricochet Block**: Reflects projectiles like a moving mirror when it is in motion.
4. **Corrupted Trail**: Leaves a deadly "wall" of data on the floor while it is sliding.
5. **Crystal Ball**: Explodes into 8 static shards only when it comes to a complete stop.

## Category 3: System Anomalies (3 Traps)

*Aimed at manipulating player perception and generating choreographed near-misses.*

1. **"Fake-Out" Wall**: Appears solid during planning but becomes intangible during execution.
2. **Corrupted Signal**: An Immutable projectile that freezes in mid-air for 500ms before resuming at double speed.
3. **Memory Sweeper**: A massive barrier that requires the player to use a "Dash" or "Ghost" skill to survive.
