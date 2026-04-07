# Traps

This document details trap ideas designed for the game's deterministic engine. Each trap was analyzed from the perspective of causal regression, ensuring they can be calculated backward from the player's Spatiotemporal Capsule (STC) without exceeding processing limits.

*Engineering Note:* Traps with infinite movement and no friction were expressly prohibited in the design, as they make deterministic temporal regression impossible by failing to reach *State-Lock* (rest).

## Progression and Difficulty Cycles (Sawtooth Curve)

The game uses a procedural progression architecture in a **Sawtooth Curve** format.

- **Initial Cycle:** The first cycle will introduce only 2 to 3 basic traps (e.g., Static Shredder and Delay Sniper) to acclimate the player to the drawing and temporal execution mechanics.
- **Internal Scaling:** Within the same cycle, difficulty increases progressively with each generated level (decreasing the STC's error margin, increasing trap density, and reducing the valid PD).
- **Cycle Break:** Upon advancing to the next cycle, difficulty drops abruptly, but the new "easy level" establishes a difficulty floor slightly higher than the start of the previous cycle. At this exact moment, **a new trap is introduced** into the procedural generation pool.

# V1.0: LAUNCH FEATURES

Below are the 13 traps of the first version (5 Immutable, 5 Mutable, 3 Anomalies), structured to support procedural generation in a sawtooth curve.

## Category 1: Direct Immutables (5 Traps)

*Agents that do not suffer external influence. They interact directly with the STC by subtracting angles or times from their Probability Distributions (PD).*

**1. Static Shredder (Shotgun Node)** | *Difficulty: Low*

- **Concept:** A cannon that fires a cone of 5 simultaneous projectiles with fixed angles between them.
- **Retrocausality:** Prohibiting the central projectile in the STC bans the entire base axis in the cannon's PD.

**2. Delay Sniper (Wind-up Sniper)** | *Difficulty: Low-Medium*

- **Concept:** Fixes a harmless aiming laser in the scene. After 2 seconds, an instant shot passes through the line.
- **Retrocausality:** Temporal regression bans the exact millisecond of the "Wind-up" in the PD so that the shot does not coincide with the player's $tx$.

**3. Continuous Sweeper Laser (Sweeper)** | *Difficulty: Medium*

- **Concept:** A continuous beam that sweeps the scene like a lighthouse (uninterrupted raycast).
- **Retrocausality:** Spatial intersection of the STC converts Nodes into prohibited spawn angles in the emitter's PD.

**4. Intermittent Grid (Blinking Laser Grid)** | *Difficulty: Medium-High*

- **Concept:** Laser walls that turn on and off in rhythmic intervals.
- **Retrocausality:** Regression focuses on the temporal offset, banning the exact "Phase Shift" in the PD that would coincide with the player's passage.

**5. Defective Homing Cannon (Drunk Homing)** | *Difficulty: High*

- **Concept:** Projectile that tries to follow the player with a predictable and locked turn rate.
- **Retrocausality:** Reverse kinematics undoes the Bézier curve limiting the possible initial angles in the PD.

## Category 2: Kinematic Cascade Mutables (5 Traps)

*Strictly submitted to the Single-Trigger State-Lock law. They interact with retrocausality via isolated vector transfer.*

**1. Kinetic Puck** | *Difficulty: Low-Medium*

- **Concept:** The fundamental trap. A static disk. When hit by a shot from any Immutable, it absorbs the vector, enters State-Lock, and slides until it stops due to friction.
- **Retrocausality:** Regression starts from the disk's collision with Glitchee, reverses friction to the initial rest point, and finds exactly which Immutable projectile provided the vector, banning it at the source.

**2. Accelerator Lens** | *Difficulty: Medium*

- **Concept:** A translucent block that slides in State-Lock when hit. Any Immutable shot that passes through it during its journey has its speed permanently doubled.
- **Retrocausality:** The Lens's OPM crosses the shot's regressive raycast. The Immutable projectile's travel time is recalculated (halved) strictly during the geometric segment and at the exact millisecond where the two entities crossed.

**3. Ricochet Block** | *Difficulty: Medium-High*

- **Concept:** When hit, it slides in State-Lock. Immutable shots that hit it *during* its journey do not move it but are reflected as if it were a moving mirror.
- **Retrocausality:** The algorithm calculates the moving wall at time $tx$ and reflects the regression lines of other traps that would hit it, creating procedural ricochets without changing the block's trajectory.

**4. Corrupted Trail** | *Difficulty: High*

- **Concept:** Slides in State-Lock after the first impact. While moving, it leaves a static "wall" of deadly data on the floor that lasts for 3 seconds.
- **Retrocausality:** The OPM is the stretched polygon of its temporal path. If Glitchee touches the residual tail, the regression reverses the trail's lifespan time to find the position and time of the Mutable, and then finds the original shot.

**5. Crystal Ball** | *Difficulty: Very High*

- **Concept:** Slides in State-Lock after impact. Only when it *stops completely* (end of friction-induced inertia) does it explode, firing 8 static shots in fixed directions.
- **Retrocausality:** Regression begins at the collision with the "shards". The explosion's origin is mathematically reversed by reverse friction to discover which Immutable shot generated the initial push.

## Category 3: System Anomalies (3 Traps)

*Aimed at manipulating player perception and generating "near-misses" choreographed by the algorithm.*

**1. "Fake-Out" Wall (Decoy Hologram)** | *Difficulty: Low*

- **Concept:** Walls that block raycasts (showing prediction ghosts) during planning but become smoke/intangible during execution.
- **Retrocausality:** Purely psychological. The algorithm pre-calculates safe paths passing *through* them intentionally.

**2. Corrupted Signal (Glitch Emitter)** | *Difficulty: Medium*

- **Concept:** Immutable projectile that suddenly stops in mid-air for 500ms, shakes the screen, and resumes flight at double speed.
- **Retrocausality:** The trajectory is calculated as a piecewise linear function. The algorithm bans shots from the PD that would fall exactly in the 500ms blind window over the STC.

**3. Memory Sweeper (RAM Sweeper)** | *Difficulty: High*

- **Concept:** A massive barrier that sweeps the screen and mandatorily requires the player to use the "Packet Overclock" (Dash/Invulnerability) skill.
- **Retrocausality:** The Sweeper's PD is forced to collide *exactly* during the invulnerability frames the player drew during planning, validating the maneuver perfectly.

# V2.0+: FUTURE FEATURES (BACKLOG)

The following traps will be part of subsequent updates to expand the complexity of procedural routes and reactions.

## Category 1: Direct Immutables (Future)

**6. Sine-Wave Cannon:** Fires projectiles in a wave pattern. Regression applies the arcsine function inverting the propagation speed and amplitude.
**7. Return-to-Sender Projectile:** The shot travels and returns along the exact same path. Regression tests the STC against outgoing and incoming kinematics, summing the times.
**8. Expanding Ring Pulse Cannon:** Fires a ring that slowly grows in radius. The regressive collision thickness increases according to the STC's $tx$.
**9. Phasing Bullet:** Ignores the first wall it hits and bounces/destroys only on the second, forcing the regressive raycast to ignore normals of the first geometry hit.
**10. Time-Dilation Trap (Corruption Mine):** Creates an aura where the player's STC speed is forcibly halved, rewriting the time spacing of Nodes in the preprocessing stage.

## Category 2: Kinematic Cascade Mutables (Future)

**6. Wall-Crawler Cog:** When pushed, sticks to the nearest wall and rolls along the perimeter with friction. The OPM becomes a graph search on the geometry's lines/edges.
**7. Attachable Shield (Mutable Armor):** Mutable that, if hit while sliding, sticks to another Mutable, doubling its mass. The extra mass formula is reversed at the exact $tx$ moment of joining.
**8. Triggered Blackhole (Gravitational Well):** Upon stopping sliding, curves the trajectory of shots around it. Triggers Transactional Cluster Logic to re-evaluate the Genuine Intersections of all neighboring Immutables.
**9. Chained Projectile (Bolas):** Two projectiles connected by a laser. The OPM is the dynamic polygon generated by the wire, requiring linear interpolation against the line connecting them.

## Category 3: System Anomalies (Future)

**4. Ghost Follower (Buffer Tracker):** Attacks the STC Nodes the player *already passed through* (time offset). By banning perfect hits on the tail, it generates shots that intentionally graze the back of the neck.
**5. Reversal Trap (Memory Spike):** If the player passes near, inverts polarity and pulls Mutables back in reverse. Friction kinematics are executed with reverse speed starting from rest.
**6. Shrinking Border (Collapsing Firewall):** Physical walls shrink toward the center based on time variable $t$, altering geometry thickness for recursive Billiard Effect.
**7. Panic Splitter (Fractal Stress Projectile):** Shatters into 3 upon approaching within 1 unit of the STC. The core is forced by the PD to pass at 1.0001 units from Glitchee, activating the visual anomaly without direct collision.
**8. Strobe Mine:** Explodes in pulses that last only 1 temporal frame. Requires absolute rigor in Intermediate Rectangle resolution (3 Zones of the STC) to avoid physical "Tunneling".
**9. Quarantine Cell (Virus Absorber):** A trash bin cube positioned where Mutable Commit failed. It absorbs shots, zeroing the restrictions in that area and helping the algorithm find exits.
**10. Out-of-Bounds Turret (Ghost Sniper):** Invisible cannon outside the screen. Run last to fill idle PDs and artificially increase bullet density.
