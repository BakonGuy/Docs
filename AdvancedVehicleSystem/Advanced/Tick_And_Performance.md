# Tick and Performance

AVS keeps large numbers of vehicles cheap by skipping work on vehicles that are not moving.
The mechanism is passive mode, and it is the reason a Blueprint Tick can stop running.



## Passive Mode

When a vehicle comes to rest it enters **passive mode**, a low resource state. A parked car costs a fraction of a moving one, which is what makes a street full of parked vehicles practical.

**Passive Tick Gatekeeping** (on by default) means passive mode also gates the standard Tick event. Blueprint logic on Tick stops running once the vehicle settles.

This is intentional and is where the performance saving comes from. Use one of the other tick events rather than disabling gatekeeping.



## Tick Events: AlwaysTick, PassiveTick, 25TPS and Tick

<!-- side-by-side:57 -->
**AVS_AlwaysTick** runs every tick regardless of passive mode or gatekeeping, before everything else.

Use it for anything that must work on a parked vehicle — interaction prompts, proximity checks, a door that opens while the car sits there.

**AVS_PassiveTick** runs only while passive, after AlwaysTick.

Use it for logic that only matters while resting, so you are not paying for it while driving.

**AVS_25TPS** runs at a fixed 25 ticks per second.

Use it for anything that does not need per-frame precision — HUD values, audio parameters, light updates. Cheaper and more predictable than the standard tick. AVS uses it internally for its own engine, forces, brake, wheel, and cosmetic work.

**Tick** is the standard Unreal tick, gated by passive mode. Use it only for logic that needs per-frame updates on a moving vehicle.
<!-- split -->
![Blueprint event graph showing the AVS_AlwaysTick, AVS_PassiveTick and AVS_25TPS events](../Assets/Images/_placeholder.png "Four ticks, each with a different guarantee")
<!-- /side-by-side -->

Two rules cover most cases: **if it must run while parked, use AlwaysTick. If it does not need to be per-frame, use AVS_25TPS.** Most Blueprint logic placed on Tick belongs in one of the two.



## Disabling Passive Mode and Determine Passive State

Passive mode assumes the vehicle is at rest because nobody is driving it. That assumption breaks when something other than player input is moving the vehicle.

<!-- side-by-side:57 -->
A vehicle possessed by a player or an AI controller never goes passive, so normal gameplay is already covered. The risk is anything that moves a vehicle **without possessing it**: a sequencer, a convoy or traffic system driving vehicles directly, or your own movement code. Those can move a vehicle in ways the rest check does not register as motion, and the vehicle goes passive underneath them.

**Simple fix:** set **Allow Passive Mode** to false on those vehicles. This works, but gives up the optimisation entirely.

**Better fix:** override **Determine Passive State** and return false while your system is in control. The vehicle then rests only when actually idle.

**Passive State Changed** fires whenever the state flips, which is where you re-enable anything that depends on ticking.
<!-- split -->
![Determine Passive State overridden in a vehicle Blueprint, returning false while an external movement system is active](../Assets/Images/_placeholder.png "Override Determine Passive State rather than disabling passive mode outright")
<!-- /side-by-side -->

The stock implementation returns passive only when **all** of these are true:

- **Allow Passive Mode** is enabled
- the vehicle has finished initializing
- the vehicle is locally at rest, per **Rest Velocity Threshold**
- it is not currently syncing as a trailer
- it is not possessed by a player or an AI controller
- engine RPM is at or below idle (or the engine is off)

In C++, `DeterminePassiveState` is a `BlueprintNativeEvent`, so override the `_Implementation` and fall through to the stock decision:

```cpp
bool AConvoyVehicle::DeterminePassiveState_Implementation()
{
	// This vehicle is moved by a convoy system that never possesses it,
	// so the stock checks cannot tell it is in use.
	if( bConvoyMovementActive ) return false;

	return Super::DeterminePassiveState_Implementation();
}
```



## Passive State on Components

Passive state is pushed down to every AVS component, so components do not need their own wiring. Engine audio, for example, stops ticking its RPM and throttle updates while passive.

If you write your own `AVS_Component`, override **On Passive State Changed** to react — gating your own tick is the usual response. `IsPassiveMode` reads the current state.



## Profiling with stat AVS

`stat AVS` opens the plugin's own stat group in PIE or in game.

<!-- side-by-side:57 -->
Check the three counters before the timings:

- **Active Vehicles** — how many are not passive. If this is higher than the number actually moving, something is keeping vehicles awake.
- **Simulated Wheels** — total wheels being simulated.
- **Wheels In Contact** — how many are touching ground.

The timing breakdown covers the whole pipeline: total actor tick, always tick, the 25 TPS batch split into engine, forces, brakes, wheels and cosmetics, then wheel component ticks, wheel effects, physics thread work, and contact modification.

All counters compile out entirely in builds without the stats system, so there is no shipping cost.
<!-- split -->
![The stat AVS overlay showing the tick pipeline breakdown and vehicle counters in PIE](../Assets/Images/_placeholder.png "Check Active Vehicles first — it explains most surprises")
<!-- /side-by-side -->



## Common Performance Problems

Most AVS performance problems are one of these three:

1. **Vehicles not going passive.** Check Active Vehicles. Usually something is nudging them, or Allow Passive Mode got turned off and never back on.
2. **Blueprint logic on standard Tick** that should be on AVS_25TPS.
3. **Physics wheel mode used where raycast would do.** Physics wheels are considerably more expensive. Use them when you need real wheel collision, not by default.
