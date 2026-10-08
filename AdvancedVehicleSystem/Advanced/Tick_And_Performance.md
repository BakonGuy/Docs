# Tick and Performance

AVS keeps large numbers of vehicles cheap by skipping work on vehicles that are not moving.
The mechanism is passive mode, and it is the reason a Blueprint Tick can stop running.



## Basic Understanding

A vehicle that has come to rest does not need most of what a moving one does, so AVS stops doing it. That state is called **passive mode**, and it is what makes a street full of parked vehicles practical.

Passive mode also gates the standard Tick event, and AVS's own 25 TPS tick with it. Blueprint logic placed on either stops running once the vehicle settles.

AVS provides three additional tick events with different guarantees, and picking the right one is the whole answer to that problem.



## Passive Mode

When a vehicle comes to rest it enters **passive mode**, a low resource state. A parked car costs a fraction of a moving one.

`IsPassiveMode` reads the current state on a vehicle or any AVS component.



## Passive Tick Gatekeeping

**Passive Tick Gatekeeping** (on by default) means that while passive, only **AVS_AlwaysTick** and **AVS_PassiveTick** run. The standard Tick and **AVS_25TPS** both stop.

This is intentional and is where the performance saving comes from. Use one of the other tick events rather than disabling gatekeeping.



## Tick Event: AVS_AlwaysTick

Runs every tick regardless of passive mode or gatekeeping, before everything else.

Use it for anything that must work on a parked vehicle — interaction prompts, proximity checks, a door that opens while the car sits there.



## Tick Event: AVS_PassiveTick

Runs only while passive **and** Passive Tick Gatekeeping is on, after AlwaysTick. With gatekeeping off, it never runs.

Use it for logic that only matters while resting, so you are not paying for it while driving.



## Tick Event: AVS_25TPS

Runs at a fixed 25 ticks per second while the vehicle is not passive. At very low frame rates it catches up at most two steps per frame, so it can fall below 25.

Use it for anything that does not need per-frame precision — HUD values, audio parameters, light updates. Cheaper and more predictable than the standard tick.

AVS uses it internally for its own engine, forces, brake, wheel, and cosmetic work.



## Tick Event: Tick

The standard Unreal tick, gated by passive mode.

Use it only for logic that needs per-frame updates on a moving vehicle.



## Choosing a Tick Event

<!-- side-by-side:57 -->
Two rules cover most cases:

**If it must run while parked, use AVS_AlwaysTick.**

**If it does not need to be per-frame, use AVS_25TPS.**

Most Blueprint logic placed on the standard Tick belongs in one of the two.
<!-- split -->
![Blueprint event graph showing the AVS_AlwaysTick, AVS_PassiveTick and AVS_25TPS events](../Assets/Images/_placeholder.png "Four ticks, each with a different guarantee")
<!-- /side-by-side -->



## When a Vehicle Should Not Go Passive

Passive mode assumes the vehicle is at rest because nobody is driving it.

A vehicle possessed by a player or an AI controller never goes passive, so normal gameplay is already covered.

The risk is anything that moves a vehicle **without possessing it**: a sequencer, a convoy or traffic system driving vehicles directly, or your own movement code. Those can move a vehicle in ways the rest check does not register as motion, and the vehicle goes passive underneath them.



## Allow Passive Mode

Setting **Allow Passive Mode** to false keeps a vehicle awake permanently.

This works, but gives up the optimization entirely. Prefer the override below unless the vehicle should genuinely never rest.



## Determine Passive State Override

<!-- side-by-side:57 -->
Override **Determine Passive State** and return false while your system is in control. The vehicle then rests only when actually idle.

In C++, `DeterminePassiveState` is a `BlueprintNativeEvent`, so override the `_Implementation` and fall through to the stock decision.
<!-- split -->
![Determine Passive State overridden in a vehicle Blueprint, returning false while an external movement system is active](../Assets/Images/_placeholder.png "Override Determine Passive State rather than disabling passive mode outright")
<!-- /side-by-side -->

```cpp
bool AConvoyVehicle::DeterminePassiveState_Implementation()
{
	// This vehicle is moved by a convoy system that never possesses it,
	// so the stock checks cannot tell it is in use.
	if( bConvoyMovementActive ) return false;

	return Super::DeterminePassiveState_Implementation();
}
```



## Stock Passive Conditions

The stock implementation returns passive only when **all** of these are true:

- **Allow Passive Mode** is enabled
- the vehicle has finished initializing
- the vehicle is locally at rest — below **Rest Velocity Threshold** for 3 seconds
- it is not currently syncing as a trailer
- it is not possessed by a player or an AI controller
- engine RPM has settled — at idle if the engine is running, or at zero if it is off



## Passive State Changed Event

**Passive State Changed** fires whenever the state flips.

This is where you re-enable anything that depends on ticking.



## Passive State on Components

Passive state is pushed down to every AVS component, so components do not need their own wiring. Engine audio, for example, stops ticking its RPM and throttle updates while passive.

If you write your own `AVS_Component`, override **On Passive State Changed** to react. Gating your own tick is the usual response.



## Profiling with stat AVS

<!-- side-by-side:57 -->
`stat AVS` opens the plugin's own stat group in PIE or in game.

The timing breakdown covers the whole pipeline: total actor tick, always tick, the 25 TPS batch split into engine, forces, brakes, wheels and cosmetics, then wheel component ticks, wheel effects, physics thread work, and contact modification.

All counters compile out entirely in builds without the stats system, so there is no shipping cost.
<!-- split -->
![The stat AVS overlay showing the tick pipeline breakdown and vehicle counters in PIE](../Assets/Images/_placeholder.png "Check Active Vehicles first — it explains most surprises")
<!-- /side-by-side -->



## stat AVS Counters

Check the three counters before the timings.

| Counter | Means |
|---|---|
| **Active Vehicles** | How many are not passive. Higher than the number actually moving means something is keeping vehicles awake. |
| **Simulated Wheels** | Total wheels being simulated. |
| **Wheels In Contact** | How many are touching ground. |



## Performance with Many Vehicles

AVS has shipped in a number of released games without being the bottleneck. When a game with many vehicles slows down, profile it first.

At scale, the costs that matter are particles, lights, LOD, materials and network relevancy. In the AVS demo project, exhaust particles cost more than anything else; six or seven vehicles without them barely register.

Wheel collision complexity makes little to no difference to performance. Complex collision is avoided for a different reason — it treats each triangle as a plane, so objects clip through it.
