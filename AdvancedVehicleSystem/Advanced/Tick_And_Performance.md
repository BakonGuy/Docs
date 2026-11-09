# Tick and Performance

How AVS saves work on vehicles that are not moving, which tick event to put your own logic on, and how to profile vehicles.

The tick settings are listed in [Vehicle Settings](https://overtorque-creations.com/Dev/Docs/#AVS/Reference/Vehicle_Settings.md).



## How Passive Mode Works

A parked vehicle does not need most of the work a moving one does. When a vehicle comes to rest, AVS puts it in passive mode and skips that work.

A vehicle goes passive when all of these are true:

- **Allow Passive Mode** is on.
- The vehicle has finished initializing.
- It has stayed below **Rest Velocity Threshold** (`25` cm/s) for 3 seconds.
- It is not a hitched trailer.
- It is not possessed by a player or an AI controller.
- Its engine RPM has settled: at idle if the engine is running, or at zero if it is off.

While passive, AVS's 25 TPS tick stops, and the wheel components and engine audio stop updating. With **Passive Tick Gatekeeping** on (the default), the vehicle's standard Tick event stops as well.

Blueprint logic on Tick or on **AVS_25TPS** therefore stops running once the vehicle settles.



## Choosing a Tick Event

AVS adds three tick events to the standard Tick:

| Event | Runs |
|---|---|
| **AVS_AlwaysTick** | Every frame, passive or not, before the others. |
| **AVS_PassiveTick** | Every frame while passive, with **Passive Tick Gatekeeping** on. |
| **AVS_25TPS** | 25 times a second while not passive. At low frame rates it runs at most twice per frame, so it can fall below 25. |
| **Tick** | Every frame while not passive. With **Passive Tick Gatekeeping** off, also while passive. |

- Logic that must run on a parked vehicle, such as an interaction prompt, goes on **AVS_AlwaysTick**.
- Logic that only matters while parked goes on **AVS_PassiveTick**.
- Logic that does not need to update every frame, such as HUD values and audio parameters, goes on **AVS_25TPS**. Above 25 frames per second, it runs less often than Tick.
- Logic that must update every frame on a moving vehicle goes on Tick.

**Passive State Changed** fires whenever the vehicle enters or leaves passive mode, with the new state.



## Keeping a Vehicle Awake

A possessed vehicle never goes passive. A vehicle moved without being possessed can: by a sequencer, a traffic system, or your own movement code. Movement that does not show up as physics velocity is not seen by the rest check, and the vehicle goes passive while it is being moved.

Override **Determine Passive State** and return false while your system is in control:

<!-- side-by-side:57 -->
```cpp
bool AConvoyVehicle::DeterminePassiveState_Implementation()
{
	// Moved by a convoy system that never possesses it.
	if( bConvoyMovementActive ) return false;

	return Super::DeterminePassiveState_Implementation();
}
```
<!-- split -->
![Determine Passive State overridden in a vehicle Blueprint, returning false while an external movement system is active](../Assets/Images/_placeholder.png "Determine Passive State overridden in Blueprint")
<!-- /side-by-side -->

In Blueprint, override the **Determine Passive State** function the same way, and call the parent for the normal checks.

Turning **Allow Passive Mode** off also keeps the vehicle awake, all the time.

In C++, `IsPassiveMode()` returns the state on any AVS component, and a custom `AVS_Component` can override `OnPassiveStateChanged` to stop its own tick.



## Profiling with stat AVS

<!-- side-by-side:57 -->
Type `stat AVS` in the console, in the editor or in game.

It shows timings for AVS's work: the whole actor tick, **AVS_AlwaysTick**, the 25 TPS tick and its parts (engine, forces, brakes, wheels, cosmetics), wheel component ticks, wheel effects, the physics thread, and contact modification.

It also counts vehicles and wheels:

- **Active Vehicles**: vehicles that are not passive. More than the number actually moving means something is keeping vehicles awake.
- **Simulated Wheels**: every wheel being simulated.
- **Wheels In Contact**: wheels touching the ground.
<!-- split -->
![The stat AVS overlay showing the tick pipeline breakdown and vehicle counters in PIE](../Assets/Images/_placeholder.png "The stat AVS overlay")
<!-- /side-by-side -->

The counters are compiled out of builds without stats, such as shipping builds.



## Performance with Many Vehicles

AVS has not been the bottleneck in released games using it. When a game with many vehicles slows down, profile it before changing anything.

At scale, the costs are particles, lights, LOD, materials and network relevancy. In the AVS demo project, exhaust particles cost more than anything else, and several vehicles without them have little performance impact.

Using complex collision on level meshes makes little to no difference to performance.



## Fixing Tick Problems

### Blueprint Tick Stops When the Vehicle Parks

The vehicle went passive. Move the logic to **AVS_AlwaysTick** or **AVS_PassiveTick**.

### Vehicle Stops Responding While Moved by Something Other Than a Player

The vehicle went passive while unpossessed. Override **Determine Passive State** as in Keeping a Vehicle Awake.



## Next Steps

- [Debugging](https://overtorque-creations.com/Dev/Docs/#AVS/Advanced/Debugging.md): tools for a vehicle that does not behave.
- [Vehicle Settings](https://overtorque-creations.com/Dev/Docs/#AVS/Reference/Vehicle_Settings.md): every tick setting and event.
