# Retrocausal Level Generation (Inverse Math)

[Architecture Idea](architecture.md)

This document describes the mathematical architecture for level preprocessing. The goal is to generate viable paths in a "bullet hell" environment, mathematically ensuring that the player can dodge obstacles, while simultaneously forcing moments of high visual tension ("near-misses").

## 1. Terminology and Data Structures

- **STC (Spatiotemporal Capsule):** Represents the "perfect case scenario" of the player's path. Between two instants of time (Nodes t1 and t2), the area the player occupies is a geometric Capsule (two semicircles connected by a rectangle).
    - **Radius Calculation:** Glitchee's collision mask radius + Variable Error Margin.
    - **Radius Dynamics:** In certain segments, the error margin decreases, narrowing the STC. This increases the density of projectiles around the capsule, forcing "near-misses" and increasing the procedural difficulty.
- **Node:** The center of an STC's semicircle, associated with an exact millisecond (time t).
- **OPM (Offensive Potential Mask):** The maximum area (polygon) a trap can hit, joining all its attack possibilities, limited by the scenario's geometry (walls) and the laws of physics (friction).
- **PD (Probability Distribution):** Mapping of all possible initial values for a trap's variables (e.g., initial angle). Starts with 100% possibilities. Prohibited values are marked with 0.

## 2. Trap Classification

- **Immutable:** Agents that are not influenced by the environment or other traps. They only influence others. Example: Fixed or rotating cannons.
- **Mutable:** Agents whose position or state is altered by the game's physics and the actions of Immutable traps. Example: Disks pushed by shots.

## 3. The Core of the Algorithm: Causal Regression

Instead of positioning traps and testing if the path is possible (brute force), the algorithm starts from the ready path (STC) and calculates which trap configurations would cause the player's death. These configurations are then prohibited.

**General Step-by-Step:**

1. Check if a trap's OPM overlaps the STC of segment t1-t2.
2. If there is spatial overlap, the Genuine Intersection is calculated.
3. The ends of this intersection are used to perform the regressive raycast to the trap (considering projectile thickness).
4. The initial property (firing angle, spawn time) that would cause that exact impact is calculated.
5. The surgical prohibition is applied to the trap's PD.

## 4. STC Resolution (The 3-Zone Logic)

To avoid broad temporal prohibitions (Over-constraining) and resolve the danger of fast projectiles crossing segments (Tunneling), the STC is not treated as a single temporal block, but geometrically sliced:

1. **Node N1 Semicircle:** If the shot hits only the back semicircle of the capsule, it is assumed that the collision would occur exactly at time t1. The limiting angle of the trap is associated and regressed from this specific time.
2. **Node N2 Semicircle:** Same logic, but at the front end of the capsule, associating the impact with future time t2.
3. **The Rectangle (Intermediate Segment):** If the shot hits the rectangular area connecting N1 and N2, the exact point of impact on the edge of the rectangle is calculated. Using linear interpolation between t1 and t2 (based on the distance from that point to N1), the exact millisecond tx when Glitchee will pass through there is found. The regression occurs from tx.

This approach ensures that the system prohibits only the exact trap configuration that would cause a collision at that specific micro-instant, preserving the PD as much as possible.

## 5. Genuine Intersection and Clipped OPM

If the scenario has obstacles (walls) that cut the trap's OPM, the impact analysis does not start from the raw edges of the STC:

1. The polygonal intersection between the OPM (already clipped by walls) and the STC is calculated.
2. Instead of using the nominal ends of the semicircles/rectangle, the algorithm uses the vertices formed by the crossing edges between the OPM and the STC.
3. The regression (raycast with shot thickness) starts from these precise geometric vertices to find the prohibited angles on the trap.

## 6. Kinematics and Complex Interactions (Friction and Multiple Impacts)

So that Mutable traps do not enter infinite loops and so that the retrocausal calculation does not suffer from combinatorial explosion, the system implements constant friction, elastic collisions, and state-locking.

### 6.1 Friction Limiting the OPM

The OPM of a mutable trap is not infinite. It is a volume limited by the braking distance calculated by the maximum possible kinetic energy.

- **OPM Calculation:** Maximum Distance (D) = (Initial Velocity Squared) / (2 * Friction).
- **Decelerated Regression:** If the Mutable hits the STC at a distance D, the original velocity required at the moment of impact is calculated by inverting the kinematics: Initial Velocity = SquareRoot(Final Velocity Squared + 2 * Friction * D).

### 6.2 The Billiard Effect (Mutable colliding with Mutable)

To avoid exponential vector calculations, interactive mutable traps have the same mass and undergo perfectly elastic collisions.

- When Disk 1 hits Disk 2, they swap their velocity vectors. Disk 1 stops instantly and Disk 2 inherits the energy to continue the trajectory. The regression becomes a linear chain of vector transfer.

### 6.3 State-Lock (Kinetic Invulnerability to Multiple Impacts)

To avoid infinite branches in the regression if a moving Disk is hit by another shot, the "State-Lock" rule (snowplow effect) is applied.

- **Stationary State:** When stopped, the Disk has a mass equal to that of the projectile, absorbs the impact, and starts moving.
- **In-Motion State:** When sliding, the Disk assumes infinite mass relative to the projectiles. Any shot that hits it during the journey is destroyed or reflected without changing the Disk's vector. It only returns to the Stationary State after coming to a complete stop due to friction. This ensures gameplay predictability and keeps the algorithm at O(N) complexity.

## 7. Practical Resolution Examples

### Example 1: Immutable Trap (C45 - 45-Degree Rotating Cannon)

The C45 rotates continuously and fires every 45 degrees.

1. We define the extreme vertices of the Genuine Intersection between the STC and the OPM of the C45.
2. We perform the regression of the projectile's travel time from these vertices (applying the 3-Zone rule to find the impact time).
3. This gives us two specific firing angles on the C45 that limit the collision area.
4. We mark all intermediate angles as prohibited at that particular time.
5. We regress the rotation of the cannon from these prohibited angles to the initial time (t = 0) to prohibit the spawn angle (initial rotation) of the C45 in the PD.

### Example 2: Mutable Trap (SD - Spike Disk)

The SD kills the player on touch and can be moved by shots from the C45.

1. First, we calculate the interaction between the STC and the SD. We find the interval of SD trajectories that would make it collide with the STC at the time mapped by the 3 Zones.
2. We enter a recursive analysis: we calculate the interaction of the SD with the Immutable traps around it.
3. We evaluate which shots from the C45 would provide the kinetic energy necessary to move the SD in the newly discovered prohibited trajectories (respecting regression with friction).
4. These specific firing angles are passed to the C45's PD and marked as 0 (prohibited).

## 8. Conflict Resolution and Cascade Prevention (Empty Set)

To avoid the "Cascade Effect" and maintain computational efficiency, the algorithm uses Sequential Collapse and Transactional Logic.

### 8.1 The Independence Rule (Immutable Traps)

Projectiles do not collide with projectiles. Therefore, Immutable traps interact only with the STC.

- **The Process:** The algorithm adds immutable traps one by one. For each trap added, it iterates over the entire path (STC), clips its PD, and "locks" the valid values.
- **Isolation:** If the current trap reaches the Empty Set (0 options), it is simply discarded from that position, without affecting the previously validated traps.

### 8.2 Transactional Logic for Clusters (Mutable Traps)

Mutable traps create vector dependency (causal chains).

- **The Process (Calculation Buffer):** When a Mutable is positioned, the algorithm iterates over the STC and propagates reverse restrictions through the causal chain to the cannons. These restrictions are stored in a temporary state (buffer).
- **Commit or Rollback:** If all traps involved maintain at least one valid interval, the algorithm applies the restrictions (Commit). If any trap in the chain reaches the Empty Set, the calculations are discarded (Rollback), the Mutable is rejected, and the PDs of the already validated traps remain intact.
