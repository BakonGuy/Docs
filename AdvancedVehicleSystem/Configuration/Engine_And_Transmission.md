# Engine and Transmission

AVS does not simulate a full engine and drivetrain. Torque comes from the gear you are in, based on your current speed, and RPM is a cosmetic value derived from engine load and throttle.

This keeps setup to a table of numbers, and is enough to drive audio, HUDs, and shift behavior convincingly.



## Configuring the Gears Array

The gear table decides how a vehicle accelerates. Build it from a target top speed rather than adjusting numbers until it stops feeling wrong.

<!-- side-by-side:57 -->
**1. Decide the vehicle's top speed.** That is the **End Speed** of your last gear, in whatever unit **Speed Units** is set to.

**2. Work backwards.** Each gear's End Speed is roughly where the next one takes over. A four gear car topping out at 75 might run 30 / 50 / 65 / 75.

**3. Set Start Speed** to where the gear becomes useful — usually the previous gear's End Speed, or slightly below it so they overlap.

**4. Set shift points.** **Up Shift** is the speed the transmission changes up at, **Down Shift** the speed it changes back down. Leave a gap between them or the transmission will hunt back and forth at the boundary.

**5. Set torque.** **Max Torque** applies at Start Speed, **Min Torque** at End Speed, interpolated between. Beyond End Speed it falls off exponentially.
<!-- split -->
![Gears array with four elements expanded, showing end speed, start speed, shift points, RPM and torque values](../Assets/Images/tutorials-Creating-Vehicles-11.png "Gear 0 is always reverse; everything after it is forward")
<!-- /side-by-side -->

Gear 0 is always reverse. Everything after it is a forward gear.

Torque is in **hectonewton meters (hNm)**, equal to 100 Nm — a value of `50` applies 5000 Nm at the wheel.

**High RPM** and **Low RPM** are cosmetic. They produce an RPM value for audio and UI and are not used in any calculation, so set them to whatever sounds right.

A working four gear starting point:

| | Gear 0 (reverse) | Gear 1 | Gear 2 | Gear 3 |
|---|---|---|---|---|
| End Speed | 20.0 | 30.0 | 65.0 | 75.0 |
| Start Speed | 0.0 | 5.0 | 25.0 | 50.0 |
| Up Shift | 100.0 | 20.0 | 50.0 | 80.0 |
| Down Shift | 0.0 | 0.0 | 15.0 | 45.0 |
| High RPM | 5500.0 | 5500.0 | 5500.0 | 5500.0 |
| Low RPM | 950.0 | 950.0 | 950.0 | 950.0 |
| Max Torque | 30.0 | 30.0 | 30.0 | 30.0 |
| Min Torque | 5.0 | 5.0 | 5.0 | 5.0 |

> Changing **Speed Units** after building a table does not convert the numbers. Your gear speeds will silently mean something else. Pick the unit first.



## Shifter Position vs Current Gear

Two separate things, and confusing them is the most common source of "my vehicle won't move".

<!-- side-by-side:50 -->
**The shifter** is the PRND position — Park, Reverse, Neutral, Drive. It is what the player controls.

`SetShifterInput` sets it directly. `MoveShifterInput(bMoveUp)` steps through positions, which is what you bind to shift up and shift down keys.

A vehicle spawns in **Park**. It will not move until something shifts it into Drive.
<!-- split -->
**The gear** is the entry in the Gears array currently supplying torque. With Automatic Transmission on, AVS picks it.

`GetSelectedGear` returns the gear chosen by input. `GetCurrentGear` returns the gear actually in use.

They differ during the shift delay. For a tachometer, read `GetCurrentGear` — it is what the engine is actually doing.
<!-- /side-by-side -->



## Transmission Modes: Automatic and Manual

<!-- side-by-side:50 -->
**Automatic with manual shifter** (the default)

**Automatic Transmission** on, **Automatic Shifter Positon** off. AVS picks gears; the player still moves the shifter between Drive and Reverse.

Suits driving sims and anything where reversing should be deliberate.
<!-- split -->
**Fully automatic**

Both on. The vehicle moves itself between Drive and Reverse based on input, so the player never shifts.

Suits arcade and casual driving. Requires the combined input function, covered below.
<!-- /side-by-side -->

For a manual transmission, turn **Automatic Transmission** off and drive gear selection yourself with `SetManualGear`.

**Gear Switch Time** (`0.5` s) is how long a change takes. Enable **Zero Throttle While Shifting** under Engine to cut throttle during the change.

### Automatic Shifter Positon Setup

It needs a specific input arrangement, and it will not work without it.

1. In project input settings, remove the separate brake axis. Add your brake keys to the throttle axis with a scale of `-1.0`, so one axis carries both.
2. In the event graph, replace `SetThrottleInput` with **`SetThrottleAndBrakeInput`**, fed from that axis.

Negative values then act as brake while moving forward, and as throttle once the vehicle is in reverse. Full walkthrough with screenshots in the [Quick Start guide](https://overtorque-creations.com/Dev/Docs/#AVS/Getting_Started/Quick_Start.md).



## Starting and Stopping the Engine

A vehicle spawns with the engine off and in Park. Handle both, or nothing happens when the player presses a key.

<!-- side-by-side:57 -->
The simplest approach is **Start With Engine Running** on, and shifting into Drive on BeginPlay. That is what the Quick Start does, and it is right for a prototype.

For anything with an ignition, leave it off and call `StartEngine` — a convenience for `SetEngineRunning(true)`. **Ignition Time** (`0.5` s) is the delay before the engine is actually running, which is the window your starter sound plays into.

`SetEngineRunning` is replicated, so call it from the owning client or the server. `SetLocalEngineRunning` exists for local-only cosmetic cases.

**Idle RPM** (`1000`) and **Idle Max RPM** (`7500`) set the RPM range in neutral, from no throttle to full. These feed engine audio when the vehicle is not moving.
<!-- split -->
![Advanced Vehicle System - Engine settings showing Start With Engine Running, Ignition Time and the idle RPM range](../Assets/Images/_placeholder.png "A vehicle spawns with the engine off and in Park")
<!-- /side-by-side -->

Building a gear table in code, for a vehicle that changes transmission at runtime:

```cpp
TArray<FVehicleGear> NewGears;

FVehicleGear Reverse;
Reverse.StartSpeed = 0.0f;
Reverse.EndSpeed   = 20.0f;
Reverse.UpShift    = 100.0f;
Reverse.MaxTorque  = 30.0f;  // [hNm] 30 = 3000 Nm at the wheel
Reverse.MinTorque  = 5.0f;
NewGears.Add(Reverse);

FVehicleGear First;
First.StartSpeed = 5.0f;
First.EndSpeed   = 30.0f;
First.UpShift    = 20.0f;
First.DownShift  = 0.0f;
First.MaxTorque  = 30.0f;
First.MinTorque  = 5.0f;
NewGears.Add(First);

// Replaces the whole table. Gear 0 is always reverse.
Vehicle->SetGearArray(NewGears);
```



## GearChanged and ShifterChanged Events

**GearChanged** fires with the old and new gear index. **ShifterChanged** fires when the PRND position moves. Both are Blueprint events, and both are overridable in C++.

These are where shift audio, dash indicators, and transmission animation belong — rather than polling the current gear on Tick and comparing it to last frame.



## Changing the Gears Array at Runtime

- `SetGearArray` replaces the whole table.
- `SetGearItem` replaces a single gear by index.

Useful for a vehicle that gets an upgrade, or a configurator swapping transmissions.

Both take effect immediately, so avoid replacing the table mid-shift.
