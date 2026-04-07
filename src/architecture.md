# Architecture Idea

The architectural solution is based on the **Inversion of Control via Contract (Trait)** pattern. The Central Engine does not process trap physics; it only delegates generic geometric and temporal calculations to each entity.

## 1. The Entity Contract (The "Retrocausal" Interface)

Any trap inserted into the game must implement four independent mathematical methods. The Central Engine will call these methods in a loop.

1. `Get_OPM(scenario_geometry)`: The trap receives the wall geometry and returns its maximum spatial range polygon.
2. `Intersect_STC(stc_segment)`: The trap receives a Capsule fragment (N1, N2, or Rectangle) and returns the exact coordinates and time `$tx$` of the crossing, if any.
3. `Regress_Origin(intersection_data)`: The trap receives the impact point/time and uses its internal physics (secret to the Engine) to calculate which initial parameter generated that impact. Returns a set of "Prohibited Parameters".
4. `Commit_Restriction(forbidden_params)`: The trap updates its own Probability Distribution (PD) or, if it is Mutable, passes the restriction to the "parent" Immutable trap through the dependency chain.

## 2. The Central Engine Loop (Agnostic)

The Engine does not have `if (trap == "Laser")`. Its linear loop (O(N)) is as follows:

```rust
for trap in level.traps {
    let mpo_polygon = trap.get_opm(walls);
    for segment in stc.segments {
        if mpo_polygon.crosses(segment) {
            let impact_data = trap.intersect_stc(segment);
            let forbidden_params = trap.regress_origin(impact_data);
            trap.commit_restriction(forbidden_params);
            if trap.pd.is_empty() {
                trigger_transactional_rollback();
            }
        }
    }
}
```

## 3. Application Proof: 3 Distinct Traps

Below, we demonstrate how three traps with completely different logical, kinematic, and temporal behaviors respond to the same four methods of the Contract, without requiring changes to the Central Engine.

### Trap A: Sweeping Laser (Immutable, Continuous Angular Motion)

*The Laser sweeps the scene like a lighthouse.*

1. `Get_OPM`: Returns a circular sector (pizza slice) corresponding to the cannon's vision, cut by nearby walls.
2. `Intersect_STC`: Compares the circular sector with the STC semicircles. Discovers that the player passes through point (X,Y) at time `$tx$`.
3. `Regress_Origin`: Internally applies a simple division: `(Impact_Angle) - (Rotation_Speed * tx)`. Returns the prohibited spawn angle.
4. `Commit_Restriction`: Removes that specific angle from its internal PD matrix.

### Trap B: Corrupted Signal (Anomalous Immutable, Shot with Mid-air Pause)

*A shot that travels, freezes in the air for 500ms, and resumes flight at double speed.*

1. `Get_OPM`: Returns a line segment from the cannon to the opposite wall.
2. `Intersect_STC`: Crosses the line with the STC. Identifies the linear impact distance and the exact time `$tx$`.
3. `Regress_Origin`: The trap's internal math uses a piecewise linear function. If the impact distance is before the pause point, it regresses with `Speed 1`. If it is after the pause point, it calculates regression by subtracting the frozen time (500ms) and using `Speed 2`. Returns the prohibited `firing_time`.
4. `Commit_Restriction`: Removes the exact millisecond of the impact from its list of valid spawn times (PD).

### Trap C: Accelerator Lens (Mutable, Dynamic Time Alteration)

*A block that slides. If a shot passes through it, the shot's speed permanently doubles.*

1. `Get_OPM`: Unlike a projectile, the Lens's OPM is the rectangle it forms sliding from rest to the stopping point (based on dynamic friction).
2. `Intersect_STC`: The Lens does not harm the player. It returns *Null* for collisions with Glitchee's STC. However, it implements a variation of the interface to intercept `Regress_Origin` from other traps.
3. `Regress_Origin`: When the regressive raycast of *another* trap crosses the spatial and temporal OPM of the Lens, it intercepts the request. The Lens recalculates the projectile's time, doubling the virtual distance from that point backward, and returns the modified raycast to the engine to continue regression to the original cannon.
4. `Commit_Restriction`: Has no PD of its own. Its restriction (the initial impact that made it slide) passes the Rollback to the original Immutable shot.
