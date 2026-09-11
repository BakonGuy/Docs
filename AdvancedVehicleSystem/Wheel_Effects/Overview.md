# Wheel Effects

Skid marks, tire smoke, rolling audio, surface reactions, brake squeal — anything a wheel should produce as it interacts with the ground.

New in 1.5, replacing the old SurfaceEffects system. Effects are now an array of self-contained objects. Each one decides when it should run and how intense it is, then hands that intensity to an output.



## Adding a Wheel Effect

Effects are normally configured on the **vehicle** and inherited by every wheel. Wheels can then override what they need, so common changes only have to be made once.

<!-- side-by-side:57 -->
**1. Open Advanced Vehicle System → Wheel Effects** on the vehicle.

**2. Pick the right slot.** For an effect that should always run regardless of surface — tire roll audio, brake squeal — use **Global Wheel Effects**. For an effect that depends on what you are driving on, use **Effects For Default Surface**.

**3. Add an entry and choose an effect type.** Roll, Skid / Slip, Bump / Impact, Brake Squeal, or one of your own.

**4. Set the conditions** that decide when it fires — speed and slip thresholds, covered below.

**5. Set the output** — a sound, a Niagara system, or both.

**6. Drive it.** Effects only exist at runtime.
<!-- split -->
![Wheel Effects section on a vehicle with an effects array expanded, showing an effect type selected](../Assets/Images/_placeholder.png "Configure once on the vehicle; every wheel inherits it")
<!-- /side-by-side -->



## Effect Types: Roll, Skid, Bump and Brake Squeal

<!-- side-by-side:50 -->
**Roll** — the wheel is turning on a surface. Contact is implied.

Use it for tire roll noise and light dust from ordinary driving. Intensity ramps from **Min Speed** (`100` cm/s) to **Full Speed** (`2000` cm/s) and holds at full above that.

**Bump / Impact** — a one-shot on impacts into the contact normal.

Landings, curbs, potholes. Ramps from **Min Strength** (`200` cm/s) to **Full Strength** (`1000` cm/s), with a **Cooldown** (`0.2` s) between firings.

Keep the cooldown. Without it, rough ground triggers an impact sound every frame the wheel makes contact.
<!-- split -->
**Skid / Slip** — the wheel is losing grip.

Skid marks, smoke, squeal. Its **Slip Source** setting decides what it reacts to, covered below.

**Brake Squeal** — surface independent, audio only.

Worn brake noise, with a wear value you can change at runtime. It is also the shipped example of a custom effect that implements its own output instead of the standard one.
<!-- /side-by-side -->

### Skid Effect Slip Sources

**Slip Source** changes what the Skid / Slip effect responds to.

| Source | Responds to | Reach for it when |
|---|---|---|
| `Skid Speed (Classic)` | Lateral speed, or full planar speed while locked | You want the original AVS behavior, or are updating an older project |
| `X Slip` | Longitudinal slip | Burnout smoke and lockup marks |
| `Y Slip` | Lateral slip | Cornering squeal and drift smoke |
| `Combined Slip` | Total slip | One effect covering both, when you do not need to tell them apart |

The classic source uses **Min Skid Speed** (`500` cm/s) and **Full Skid Speed** (`1500` cm/s). The calculated sources use **Min Slip** (`0.5`) and **Full Slip** (`1.5`) instead, and the settings you are not using hide themselves.

Two more settings to check:

- **Response Speed** (`5.0`) — how fast intensity chases changing slip. Lower smooths out brief spikes; higher snaps.
- **Min Wheel Speed** (`100` cm/s) — a floor below which nothing plays. This is what stops a stationary wheel from squealing.

> A common setup is **two** skid effects: one on `X Slip` for burnout smoke, one on `Y Slip` for cornering squeal. They are separate objects, so they can have completely different sounds, particles, and thresholds.



## Surface Effects

Surface Effects is how dirt produces different effects from tarmac.

<!-- side-by-side:57 -->
**Surface Effects** is a map from physical surface to a set of effects. Add an entry, pick the surface, and configure effects for it.

A wheel on a surface with an entry uses that entry. A wheel on a surface without one falls back to **Effects For Default Surface**.

**The `Default` quirk:** `Default` is a real value in Unreal's surface enum, and once you use it as a key you cannot add another entry with an unassigned key. That is why **Effects For Default Surface** is its own slot — configure the default there, and treat `Default` as "unassigned" when adding map entries.
<!-- split -->
![Surface Effects map with entries for several physical surfaces, each holding its own effects array](../Assets/Images/_placeholder.png "One entry per surface; anything unlisted falls back to the default slot")
<!-- /side-by-side -->



## Per-Wheel Effect Overrides

Wheels inherit from the vehicle and override only what they need.

| Override | Replaces |
|---|---|
| **Override Global Effects** | The vehicle's global effects for this wheel |
| **Effects For Default Surface Override** | The default surface entry |
| **Surface Effect Overrides** | The vehicle's entry for one specific surface |

Overrides **replace, they do not merge**. Overriding the Dirt surface on a wheel means the vehicle's Dirt effects no longer apply to it at all — not that yours are added on top.

This is also how you silence a wheel: enable **Override Global Effects** and leave the array empty. An empty override disables the effects rather than inheriting them.

Useful cases: a spare wheel that should not make noise, a steel wheel with a different surface sound, or heavier smoke on the driven axle only.



## Effect Output and Placement

Every built-in effect except Brake Squeal shares the same output block.

<!-- side-by-side:57 -->
**Attach To Wheel** decides whether the effect follows the wheel or stays where it spawned.

Skid marks want it **off** — the marks should stay on the ground as the car drives away. Smoke and dust usually want it **on**, so the effect travels with the wheel.

**Use Contact Point** (on by default) spawns at the surface contact rather than the wheel component's location. Leave it on for anything that should appear where rubber meets road.

**Location Offset** and **Rotation Offset** fine-tune from there, in AVS wheel space. Wheels using Invert Torque are corrected automatically, so a mirrored wheel does not need mirrored offsets.
<!-- split -->
![Placement settings on an effect output showing Attach To Wheel and Use Contact Point](../Assets/Images/_placeholder.png "Attach To Wheel off for marks, on for smoke")
<!-- /side-by-side -->

**Parameter Intensity Range** maps the effect's normalized 0-1 intensity into whatever range your assets expect. If your Niagara system wants 0-100, set it here rather than rebuilding the system.

### Audio Output Settings

Set a **Sound** and an **Attenuation**, then use **Audio Intensity Param** to feed the mapped intensity into a Sound Cue or MetaSound — driving volume, pitch, or a crossfade.

Set **Random Start Time Max** to the length of the looping audio. Without it, all four wheels start the same loop at the same position and sound like a single loud wheel.

**Audio Fade Out Duration** (`0.1` s) fades continuous audio to silence before the component is released, so effects do not cut off abruptly.

### Particle Output Settings

Niagara and Cascade are both supported, each with an optional float parameter receiving the mapped intensity. Use Niagara for new work.



## Effect Runtime Behavior

This explains behavior that can otherwise look like a bug:

- Effects you configure are **templates**. Each wheel duplicates them into its own runtime instance, so wheels never share state.
- **Surface effects are recreated** when contact or surface changes. Any state inside them resets.
- **Global effects persist** across those changes and keep their state.
- `ClearAllWheelEffects()` on the vehicle refreshes everything, which is what you call after changing effect configuration at runtime.



## Custom Effects

Effects are extendable in Blueprint and C++, using the same functions the built-in ones use. See [Custom Effects](https://overtorque-creations.com/Dev/Docs/#AVS/Wheel_Effects/Custom_Effects.md).
