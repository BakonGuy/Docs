# Vehicle Settings

Every setting and function on `AVS_Vehicle`, grouped the way the details panel groups them. Select `ClassName(self)` in the components list, or press **Class Defaults**, to find the settings under **Advanced Vehicle System**.

Speeds are in the vehicle's **Speed Units** unless a unit is given.



## General

| Setting | Default | Description |
|---|---|---|
| **Speed Units** | `MPH` | The unit for every speed AVS takes or returns, including the gear table. Changing it does not convert existing values. |
| **Brake Lock Speed** | `10` | Below this speed, with the brake at `0.95` or more, wheels with a braking, handbrake or driving role lock to hold the vehicle still. `0` turns this off. |
| **Cinematic Playback** | off | While on and the vehicle is not player controlled, AVS releases every wheel lock so recorded wheel motion plays back. Turn it on after recording. |

**Speed Units** options: `(mph) Miles Per Hour`, `(km/h) Kilometers Per Hour`, `(ft/s) Feet Per Second`, `(m/s) Meters Per Second`, `(kn) Knots` and `Furlong Per Fortnight`.



## Tick

| Setting | Default | Description |
|---|---|---|
| **Allow Passive Mode** | on | Lets the vehicle enter passive mode when it comes to rest. |
| **Passive Tick Gatekeeping** | on | While passive, the standard Tick stops and **AVS_PassiveTick** runs. |

See [Tick and Performance](https://overtorque-creations.com/Dev/Docs/#AVS/Advanced/Tick_And_Performance.md).



## Steering

| Setting | Default | Description |
|---|---|---|
| **Steering Input Smoothing** | `Ease` | `Instant` jumps to the input. `Constant` moves toward it at a fixed rate. `Ease` slows as it gets close. |
| **Steering Falloff Curve** | see below | Maximum steering input (Y) at each speed (X). Sliding sideways adds to it, up to `1.0` extra at 2500 cm/s of sideways speed. |
| **Steering Speed** | `2.5` | How fast steering moves toward the input. With `Constant`, `1.0` goes from center to full in 1 second. |
| **Steering Recenter Speed** | `2.5` | Used in place of **Steering Speed** whenever the input is closer to center than the current steering. |

The default **Steering Falloff Curve** allows `1.0` at `0`, `0.8` at `20`, `0.4` at `60` and `0.3` at `120`.



## Engine

| Setting | Default | Description |
|---|---|---|
| **Start With Engine Running** | off | Starts the engine when the vehicle spawns. Still waits for **Ignition Time**. |
| **Zero Throttle While Shifting** | off | Sets the throttle to `0` while no gear is engaged. |
| **Ignition Time** | `0.5` s | Delay between starting the engine and the engine running. |
| **Idle RPM** | `1000` | RPM with no gear engaged and no throttle. |
| **Idle Max RPM** | `7500` | RPM with no gear engaged and full throttle. |



## Transmission

| Setting | Default | Description |
|---|---|---|
| **Automatic Transmission** | on | AVS picks the gear from the shift points. Off, gears are selected with `SetManualGear`. |
| **Automatic Shifter Positon** | off | AVS moves the shifter between Park, Reverse, Neutral and Drive from the inputs. Needs `SetThrottleAndBrakeInput`. |
| **Gear Switch Time** | `0.5` s | How long a gear change takes. No gear supplies torque during it. |
| **Gears** | 2 gears | The gear table. Gear `0` is reverse; every gear after it is a forward gear. |

Each entry in **Gears**:

| Setting | Description |
|---|---|
| **End Speed** | The gear's intended top speed. Torque is at **Min Torque** here. |
| **Start Speed** | The gear's intended lowest speed. Torque is at **Max Torque** here. |
| **Up Shift** | Automatic transmission only. Changes up above this speed. |
| **Down Shift** | Automatic transmission only. Changes down below this speed. |
| **High RPM** | RPM at **End Speed**. Cosmetic. |
| **Low RPM** | RPM at **Start Speed**. Cosmetic. |
| **Max Torque** | Torque at **Start Speed**, in hNm (`1` = 100 Nm), at each driving wheel. |
| **Min Torque** | Torque at **End Speed**, in hNm, at each driving wheel. |

A new vehicle starts with two gears, each with **End Speed** `100`, **Start Speed** `0`, **Up Shift** `100`, **Down Shift** `0`, **High RPM** `5500`, **Low RPM** `950`, **Max Torque** `30` and **Min Torque** `5`. A gear added in the editor starts with every value at `0`.

Below **Start Speed**, torque ramps up toward **Max Torque**. Past **End Speed**, it starts again from **Max Torque** and falls to `0` over a further quarter of the gear's speed range.



## Physics

| Setting | Default | Description |
|---|---|---|
| **Start With Physics** | on | The vehicle simulates physics when it spawns. |
| **Disable Skeletal Collisions** | on | Turns off collision on the vehicle's skeletal mesh components during construction. Components tagged `KeepCollision` are skipped. |
| **Physics Material** | `VehicleChassis` | Applied to the vehicle mesh. Clear it to keep the mesh's own. |
| **Vehicle Max Angular Velocity** | `700` deg/s | How fast the vehicle body can spin. `0` removes the limit. |
| **Wheels Push Physics** | on | Each wheel's force is also applied, in reverse, to the simulating object under it. |

Under **Physics → Advanced**:

| Setting | Default | Description |
|---|---|---|
| **Rest Velocity Threshold** | `25` cm/s | Below this for 3 seconds, the vehicle counts as at rest, for passive mode and the network rest state. |
| **Upside Down Angle Threshold** | `90` degrees | Tilt past which the vehicle counts as upside down. `0` is upright, `180` fully inverted. |

Not shown in the details panel, settable from Blueprint or C++:

| Value | Default | Description |
|---|---|---|
| **Base Linear Drag** | the mesh's **Linear Damping** | The linear damping AVS applies while moving. |
| **Dynamic Air Drag** | off | Adds `0.02` to `0.25` damping while sliding, above `7`. |

AVS sets the vehicle mesh's linear damping itself on every 25 TPS tick. Below `0.5` with no throttle it raises it as high as `15`, to help the vehicle settle.



## Wheel Effects

**Global Wheel Effects**, **Effects For Default Surface** and **Surface Effects**. See [Wheel Effect Settings](https://overtorque-creations.com/Dev/Docs/#AVS/Reference/Wheel_Effect_Settings.md).



## Network

| Setting | Default | Description |
|---|---|---|
| **Network Type** | `History Interpolation` | `History Interpolation` plays received states back slightly behind. `Client Prediction` is unfinished. |
| **Movement Replication** | on | Replicates location, rotation and velocity. Inputs replicate either way. |
| **Sync Location** | on | Applies replicated location. |
| **Sync Rotation** | on | Applies replicated rotation. |

Under **Network → Advanced**:

| Setting | Default | Description |
|---|---|---|
| **Net Send Rate** | `0.05` s | Time between the states the owner sends. |
| **Net Time Behind** | `0.15` s | How far behind the owner a client plays received states. |
| **Net Lerp Start** | `0.35` s | How long before each state a client starts blending toward it. |
| **Net Position Tolerance** | `0.1` cm | Per axis. A state closer than this is skipped. |
| **Net Use Smoothing** | on | Smooths corrections while the vehicle is simulating. |
| **Net Smoothing** | `10` | Speed of smoothed corrections. Higher is faster. |

See [Networking](https://overtorque-creations.com/Dev/Docs/#AVS/Advanced/Networking.md).



## Debug

| Setting | Default | Description |
|---|---|---|
| **Display Debug Menu** | off | Shows the config assist HUD while a local player possesses the vehicle. |
| **Visualize Center of Mass** | off | Draws the center of mass the physics body is using. |
| **Suspension Debug** | off | Draws each wheel's trace, force and spring length. Development builds only. |



## Experimental

Wheel reprojection hides the real wheel mesh and shows a projected mesh in its place, aligned to the wheel every frame, so wheels do not look tilted by the physics engine.

| Setting | Default | Description |
|---|---|---|
| **Wheel Reprojection** | off | Turns reprojection on. |
| **Wheel Reprojection Camber** | off | Adds cosmetic camber driven by suspension compression. |
| **Camber Compressed** | `30, 15` | Wheel height in cm, and camber in degrees, at the compressed end. |
| **Camber Decompressed** | `-30, -15` | The same, at the extended end. |
| **Reprojection Smooth Alpha** | `0.75` | How closely the projected mesh follows each frame, from `0` to `1`. |

Camber is visual only. It does not change how the wheel simulates.



## Input Functions

| Function | Description |
|---|---|
| `SetThrottleInput(Value)` | `0` to `1`. Owning client or server only. |
| `SetBrakeInput(Value)` | `0` to `1`. Owning client or server only. |
| `SetThrottleAndBrakeInput(Value)` | `-1` to `1`. Negative is brake, or throttle while in Reverse with **Automatic Shifter Positon** on. |
| `SetSteeringInput(Value)` | `-1` to `1`. Owning client or server only. |
| `SetHandbrakeInput(Value)` | Applies or releases the handbrake. |
| `SetShifterInput(Position)` | Sets the shifter: `Park`, `Reverse`, `Neutral` or `Drive`. |
| `MoveShifterInput(MoveUp)` | Moves the shifter one position, toward Park when `MoveUp` is true. Returns the new position. |
| `SetManualGear(GearNum)` | Selects a forward gear with **Automatic Transmission** off. |
| `StartEngine()` / `SetEngineRunning(Running)` | Starts or stops the engine, through the server. |
| `SetLocalEngineRunning(Running)` | Sets the engine state on this machine only. |
| `SetInputHost(Vehicle)` | Copies throttle, brake, steering and handbrake from another vehicle. Built for trailers. |
| `GetThrottleInput`, `GetBrakeInput`, `GetSteeringInput`, `GetHandbrakeInput`, `GetShifterInput` | The current inputs. |
| `GetCurrentSteeringInput` | The raw input on the controlling client, the replicated input elsewhere. |

Throttle, brake and steering inputs are rounded to steps of `0.1`.



## Vehicle State Functions

| Function | Description |
|---|---|
| `GetAirSpeed` | The vehicle's speed. |
| `GetAcceleration` | Change in speed per second. |
| `GetThrottle`, `GetBrake` | The smoothed throttle and brake, as applied. |
| `GetBrakeApplied`, `GetHandbrakeApplied` | Whether the brake or handbrake is on. |
| `GetCurrentSteering` | The steering the wheels use, after smoothing and the falloff curve. |
| `GetEngineRunning`, `GetEngineRPM` | Engine state and RPM. |
| `GetTorque` | The torque currently given to each driving wheel, in hNm. |
| `GetTargetSpeed` | The speed the throttle is asking for in the current gear. |
| `GetSelectedGear` | The gear the transmission wants. |
| `GetCurrentGear` | The gear actually engaged. `-1` while between gears or out of gear. |
| `GetSlip` | Sideways speed, from `0` below 1000 cm/s to `1` at 2500 cm/s. |
| `GetIsUpsideDown` | Whether the tilt is past **Upside Down Angle Threshold**. |
| `AnyWheelContact` | Whether any wheel is touching a surface. |
| `GetInitializationState` | Whether AVS has finished setting the vehicle up. |
| `IsCinematic` | **Cinematic Playback** is on and no player controls the vehicle. |
| `GetSpeedUnitData` | The speed unit, its abbreviation, and conversion factors to and from cm/s. |
| `GetMaxSteeringInputAtSpeed(Speed)` | The falloff curve's value at a speed. |
| `GetPhysicsTickDelta` | The last Chaos substep time. |
| `GetVehicleMesh`, `GetWheels` | The vehicle mesh and every wheel. |



## Other Functions

| Function | Description |
|---|---|
| `SetGearArray(Gears)`, `SetGearItem(Index, Gear)` | Replace the gear table, or one gear, and refresh the engaged gear. |
| `SetPhysics(Simulate)` | Turns simulation on or off for the vehicle and its wheels. |
| `TeleportVehicle(Location, Rotation, KeepRelativeVelocity)` | Moves the vehicle, its physics wheels and hitched trailers together. |
| `SetVehiclePositionWithVirtualPivot(LocalPivot, WorldTransform, Teleport)` | Places the vehicle so a local pivot lands on a world transform. |
| `SetExactCenterOfMass(Location)` | Sets the center of mass relative to the vehicle mesh pivot. |
| `SetCenterOfMassOffset(Offset)` | Sets an offset added to the center of mass. Each call replaces the last. |
| `GetExactCenterOfMass` | The center of mass in vehicle mesh space, and whether it is valid yet. |
| `DetachAllWheels`, `ResetAllWheels` | Detach every wheel, or reattach and reposition them. |
| `ClearAllWheelEffects` | Stops every wheel's effects. They are rebuilt on the next tick. |
| `AddWheel(Wheel)`, `RemoveWheel(Wheel)` | Protected. Install or destroy a wheel at runtime. |
| `RefreshContactModification(Instant)` | Rebuilds which wheel meshes are kept from colliding with the vehicle. |
| `SetLightsActive(Group, Active)`, `GetLightsActive(Group)`, `ToggleLightsActive(Group)` | Light groups. |
| `UpdateLights` | Protected. Applies light group changes. |
| `UnhitchAll` | Releases every hitch on the vehicle. |
| `SetMovementReplicationEnabled(Enabled)` | Pauses or resumes movement replication without changing **Movement Replication**. |
| `isOwningClient` | True on the machine that controls this vehicle. |
| `ShowDebugMenu(Display)` | Shows or hides the config assist HUD. |
| `UpdateSuspensionPreview` | Redraws **Editor Preview** on every wheel. |



## Events

| Event | Fires |
|---|---|
| **GearChanged** | When a gear change completes, with the old and new gear index. |
| **ShifterChanged** | When the shifter is set. |
| **OnVehicleFlipped** | When the vehicle becomes upside down or rights itself. |
| **UpdateLightDecorations** | For each light controller, after its intensity is set. |
| **Determine Passive State** | Each frame, to decide whether the vehicle may be passive. Overridable. |
| **Passive State Changed** | When the vehicle enters or leaves passive mode. |
| **Owner Changed** | On every machine, when the vehicle is possessed or unpossessed. |
| **AVS_25TPS** | 25 times a second while not passive. |
| **AVS_AlwaysTick** | Every frame, before everything else. |
| **AVS_PassiveTick** | Every frame while passive, with **Passive Tick Gatekeeping** on. |
