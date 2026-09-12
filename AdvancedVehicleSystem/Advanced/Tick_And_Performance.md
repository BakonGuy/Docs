# Tick and Performance

AVS keeps large numbers of vehicles cheap by skipping work on vehicles that are not moving.
The mechanism is passive mode, and it is the reason a Blueprint Tick can stop running.



## Basic Understanding

A vehicle that has come to rest does not need most of what a moving one does, so AVS stops doing it. That state is called **passive mode**, and it is what makes a street full of parked vehicles practical.

Passive mode also gates the standard Tick event, which is the part that surprises people. Blueprint logic placed on Tick stops running once the vehicle settles.

AVS provides three additional tick events with different guarantees, and picking the right one is the whole answer to that problem.



## Passive Mode

When a vehicle comes to rest it enters **passive mode**, a low resource state. A parked car costs a fraction of a moving one.

`IsPassiveMode` reads the current state on a vehicle or any AVS component.



## Passive Tick Gatekeeping

**Passive Tick Gatekeeping** (on by default) means passive mode also gates the standard Tick event.

This is intentional and is where the performance saving comes from. Use one of the other tick events rather than disabling gatekeeping.



## Tick Event: AVS_AlwaysTick

Runs every tick regardless of passive mode or gatekeeping, before everything else.

Use it for anything that must work on a parked vehicle — interaction prompts, proximity checks, a door that opens while the car sits there.



## Tick Event: AVS_PassiveTick

Runs only while passive, after AlwaysTick.

Use it for logic that only matters while resting, so you are not paying for it while driving.



## Tick Event: AVS_25TPS

Runs at a fixed 25 ticks per second.

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

This works, but gives up the optimisation entirely. Prefer the override below unless the vehicle should genuinely never rest.



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
- the vehicle is locally at rest, per **Rest Velocity Threshold**
- it is not currently syncing as a trailer
- it is not possessed by a player or an AI controller
- engine RPM is at or below idle (or the engine is off)



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



## Common Performance Problems

Most AVS performance problems are one of these three:

1. **Vehicles not going passive.** Check Active Vehicles. Usually something is nudging them, or Allow Passive Mode got turned off and never back on.
2. **Blueprint logic on standard Tick** that should be on AVS_25TPS.
3. **Physics wheel mode used where raycast would do.** Physics wheels are considerably more expensive. Use them when you need real wheel collision, not by default.
