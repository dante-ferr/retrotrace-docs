# Iterative Forward Rejection (The Choreographer)

[Architecture Idea](architecture.md)

This document describes an alternative mathematical architecture for level generation. Instead of inverting the physics (retrocausal), this approach uses fast forward-simulation and "smart rejection" to ensure the player's path is safe while allowing for complex, chaotic physical interactions.

## 1. Core Concept: The Path is King

In this model, the **STC (Spatiotemporal Capsule)** path is pre-defined or generated first. This path represents the "perfect" journey the player will take through the level.

The goal of the generator is then to "choreograph" traps around this path so they provide high visual tension (near-misses) but never actually intersect the STC.

## 2. The Generation Loop (Forward Simulation)

Unlike the Retrocausal approach, which calculates prohibitions backwards, the Iterative Forward Rejection loop works as follows:

1. **Path Definition:** Create the STC path (a series of spatiotemporal capsules).
2. **Trap Placement:** Randomly (or via heuristics) place a trap with initial parameters (angle, velocity, spawn time).
3. **Forward Simulation:** Run the game's actual physics engine forward in small "temporal chunks" (e.g., 500ms intervals).
4. **Collision Check:** At each step of the simulation, check if any part of the trap (or its projectiles) intersects the STC for that specific time.
5. **Smart Rejection & Perturbation:**
    * **If it hits the STC:** Instead of failing, slightly "nudge" the trap's parameters (e.g., rotate it by 2 degrees, or delay firing by 10ms) and re-simulate.
    * **If it misses the STC:** Keep the trap and move to the next one.
6. **Interaction Handling:** Because we are simulating forward, if a bullet hits a movable block, the block's movement is naturally calculated. We only need to check if the *resulting* movement of the block hits the STC.

## 3. Advantages over Retrocausal

- **Handles Chaos:** Multiple interacting objects (bullet -> block -> block -> player) are trivial to simulate forward. Retrocausal math becomes exponentially complex or impossible with each interaction.
- **Simpler Math:** We use the exact same physics code for generation as we do for the actual game. No need to write complex "Inverse Physics" for every new trap type.
- **Visual Tension:** By "nudging" a trap's parameters until it *just barely* misses the STC, we can mathematically guarantee "near-miss" moments that look intentional and exciting.
- **Performance:** Forward simulation of a few objects is extremely fast. We can run hundreds of "nudge-and-test" iterations in milliseconds.

## 4. Multi-Body Interactions and Cascades

When a trap is added that interacts with another (e.g., a cannon firing at a movable shield), the system simulates the entire "causal chain" forward.

1. **Cannon fires.**
2. **Bullet hits Shield.**
3. **Shield moves.**
4. **Collision check:** Does the Bullet hit the STC? Does the Shield hit the STC?
5. **Rejection:** If *any* part of the chain hits the STC, the initial Cannon parameters are nudged and the entire chain is re-simulated.

This ensures that even the most complex "Rube Goldberg" style trap interactions remain safe for the player's pre-defined path.

## 5. Summary of the Workflow

1. Generate a "Cool Path" (STC).
2. Place a Trap.
3. Simulate Forward.
4. Did it hit the Path?
    * **Yes:** Nudge parameters and GOTO 3.
    * **No:** Lock parameters and place the next Trap.
5. Repeat until the level density is sufficient.

## 6. The "Blame Game" (Resolving Chaotic Collisions)

When a complex interaction occurs (e.g., a Cannon shoots a Disk, the Disk hits the STC), the algorithm must decide which entity to adjust. This is resolved efficiently using the **"Last-In" Rule**.

Instead of reverse-tracing physics to assign blame, the algorithm places and fully validates traps one at a time. The level is strictly maintained in a "mathematically safe state." 
- If a Cannon is placed and safely validated, and then a Disk is placed which gets hit by the Cannon and diverted into the player, the **Disk** is blamed and nudged.
- The fault always lies with the entity that broke the previously validated safe state.

To prevent infinite loops where a new trap cannot fit safely, a **Backtracking Safety Valve** is used:
1. **Nudge Limit:** If an entity is nudged X times and still fails, it is discarded.
2. **Type Swap:** A different trap type is attempted in that slot.
3. **True Backtracking:** If no trap can fit, the system deletes the *previously* placed trap and perturbs the one before it, un-sticking the algorithm.

## 7. Performance and Hardware Considerations

**Rust's Computational Power**
Rust is exceptionally well-suited for Iterative Forward Rejection. Because the forward simulation relies on standard 2D vector kinematics (which are mathematically lightweight) without the overhead of deep reverse-state tracking or physics engine rendering, Rust can execute tens of thousands of "place-simulate-check" loops per second.

**Preprocessing vs. Runtime Generation (Smartphones)**
The processing is **entirely doable in real-time on modern smartphones**. 
- The 2D math required for simulating a handful of bouncing entities over a few seconds is minuscule compared to rendering complex graphics.
- Levels can be procedurally generated *on the device* during a short loading screen (e.g., < 0.1s) when the player starts a run, eliminating the need to pre-generate and download massive level databases.
- Because Rust is compiled to highly optimized native machine code (via GDExtension), the generation loop will be imperceptible to the user.
