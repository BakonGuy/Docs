# Wheel Effects

Skid marks, tire smoke, rolling audio, surface reactions, brake squeal — anything a wheel should produce as it interacts with the ground.

New in 1.5, replacing the old SurfaceEffects system.



## Basic Understanding

An effect is a self-contained object. It reads the wheel every tick, decides whether it should be running, calculates a 0-1 intensity, and hands that intensity to an output — a sound, a Niagara system, or both.

Effects live in arrays, and there are two kinds of array:

**Global Wheel Effects** always apply, whatever the wheel is driving on. Tire roll audio and brake squeal belong here.

**Surface Effects** apply only on a specific physical surface. This is how dirt produces different smoke from tarmac.

Effects are normally configured on the **vehicle** and inherited by every wheel. Wheels can then override what they need, so common changes only have to be made once.



## Adding a Wheel Effect

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



## Effect Type: Roll

The wheel is turning on a surface. Contact is implied.

Use it for tire roll noise and light dust from ordinary driving.

Intensity ramps from **Min Speed** (`100` cm/s) to **Full Speed** (`2000` cm/s), and holds at full above that.



## Effect Type: Skid / Slip

The wheel is losing grip. This covers skid marks, smoke and squeal.

Its **Slip Source** setting decides what it reacts to, covered in the next two sections.

> A common setup is **two** skid effects: one on `X Slip` for burnout smoke, one on `Y Slip` for cornering squeal. They are separate objects, so they can have completely different sounds, particles, and thresholds.



## Effect Type: Bump / Impact

A one-shot on impacts into the contact normal — landings, curbs, potholes.

Intensity ramps from **Min Strength** (`200` cm/s) to **Full Strength** (`1000` cm/s), with a **Cooldown** (`0.2` s) between firings.

> Keep the cooldown. Without it, rough ground triggers an impact sound every frame the wheel makes contact.



## Effect Type: Brake Squeal

Surface independent, audio only. Worn brake noise, with a wear value you can change at runtime.

It is also the shipped example of a custom effect that implements its own output instead of the standard one.



## Skid Effect Slip Sources

**Slip Source** changes what the Skid / Slip effect responds to.

| Source | Responds to | Reach for it when |
|---|---|---|
| `Skid Speed (Classic)` | Lateral speed, or full planar speed while locked | You want the original AVS behavior, or are updating an older project |
| `X Slip` | Longitudinal slip | Burnout smoke and lockup marks |
| `Y Slip` | Lateral slip | Cornering squeal and drift smoke |
| `Combined Slip` | Total slip | One effect covering both, when you do not need to tell them apart |

The classic source uses **Min Skid Speed** (`500` cm/s) and **Full Skid Speed** (`1500` cm/s).

The calculated sources use **Min Slip** (`0.5`) and **Full Slip** (`1.5`) instead. The settings you are not using hide themselves.



## Skid Effect Response Settings

**Response Speed** (`5.0`) is how fast intensity chases changing slip. Lower smooths out brief spikes; higher snaps.

**Min Wheel Speed** (`100` cm/s) is a floor below which nothing plays. This is what stops a stationary wheel from squealing.



## Surface Effects

<!-- side-by-side:57 -->
**Surface Effects** is a map from physical surface to a set of effects. Add an entry, pick the surface, and configure effects for it.

A wheel on a surface with an entry uses that entry.

A wheel on a surface without one falls back to **Effects For Default Surface**.
<!-- split -->
![Surface Effects map with entries for several physical surfaces, each holding its own effects array](../Assets/Images/_placeholder.png "One entry per surface; anything unlisted falls back to the default slot")
<!-- /side-by-side -->



## The Effects For Default Surface Slot

`Default` is a real value in Unreal's surface enum, and once you use it as a key you cannot add another entry with an unassigned key.

That is why **Effects For Default Surface** is its own slot. Configure the default there, and treat `Default` as "unassigned" when adding map entries.



## Per-Wheel Effect Overrides

Wheels inherit from the vehicle and override only what they need.

| Override | Replaces |
|---|---|
| **Override Global Effects** | The vehicle's global effects for this wheel |
| **Effects For Default Surface Override** | The default surface entry |
| **Surface Effect Overrides** | The vehicle's entry for one specific surface |

Overrides **replace, they do not merge**. Overriding the Dirt surface on a wheel means the vehicle's Dirt effects no longer apply to it at all — not that yours are added on top.

Useful cases: a spare wheel that should not make noise, a steel wheel with a different surface sound, or heavier smoke on the driven axle only.



## Silencing a Single Wheel

Enable **Override Global Effects** on that wheel and leave the array empty.

An empty override disables the effects rather than inheriting them.



## Effect Output: Attach To Wheel and Use Contact Point

<!-- side-by-side:57 -->
**Attach To Wheel** decides whether the effect follows the wheel or stays where it spawned.

Skid marks want it **off** — the marks should stay on the ground as the car drives away. Smoke and dust usually want it **on**, so the effect travels with the wheel.

**Use Contact Point** (on by default) spawns at the surface contact rather than the wheel component's location. Leave it on for anything that should appear where rubber meets road.
<!-- split -->
![Placement settings on an effect output showing Attach To Wheel and Use Contact Point](../Assets/Images/_placeholder.png "Attach To Wheel off for marks, on for smoke")
<!-- /side-by-side -->

Every built-in effect except Brake Squeal shares this output block.



## Effect Output: Location and Rotation Offset

**Location Offset** and **Rotation Offset** fine-tune placement, in AVS wheel space.

Wheels using **Invert Torque** are corrected automatically, so a mirrored wheel does not need mirrored offsets.



## Parameter Intensity Range

**Parameter Intensity Range** maps the effect's normalized 0-1 intensity into whatever range your assets expect.

If your Niagara system wants 0-100, set it here rather than rebuilding the system.



## Audio Output Settings

Set a **Sound** and an **Attenuation**, then use **Audio Intensity Param** to feed the mapped intensity into a Sound Cue or MetaSound — driving volume, pitch, or a crossfade.

**Audio Fade Out Duration** (`0.1` s) fades continuous audio to silence before the component is released, so effects do not cut off abruptly.

> Set **Random Start Time Max** to the length of the looping audio. Without it, all four wheels start the same loop at the same position and sound like a single loud wheel.



## Particle Output Settings

Niagara and Cascade are both supported, each with an optional float parameter receiving the mapped intensity.

Use Niagara for new work.



## Effect Templates and Runtime Instances

Effects you configure are **templates**. Each wheel duplicates them into its own runtime instance, so wheels never share state.

**Surface effects are recreated** when contact or surface changes, and any state inside them resets.

**Global effects persist** across those changes and keep their state.



## Clearing Effects at Runtime

`ClearAllWheelEffects()` on the vehicle refreshes everything.

Call it after changing effect configuration at runtime.



## Custom Effects

Effects are extendable in Blueprint and C++, using the same functions the built-in ones use. See [Custom Effects](https://overtorque-creations.com/Dev/Docs/#AVS/Wheel_Effects/Custom_Effects.md).
