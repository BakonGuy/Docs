# Wheel Settings

Every setting on the `AVS_Wheel` component, grouped the way the details panel groups them. For how to set a wheel up, see [Wheels](https://overtorque-creations.com/Dev/Docs/#AVS/Components/Wheels.md).



## General

| Setting | Default | Description |
|---|---|---|
| **Wheel Static Mesh** | none | The mesh AVS creates as the wheel. With none set, AVS uses a sphere. |
| **Wheel Mode** | `Raycast` | `Raycast` or `Physics`. Can be changed at runtime with `SetWheelMode`. |
| **Auto Wheel Radius** | on | Measures **Wheel Radius** as half the wheel mesh's height. |
| **Wheel Radius** | `30` cm | Used when **Auto Wheel Radius** is off. Sets where the trace finds the ground, in both wheel modes. |

A wheel uses its first child component as its wheel mesh. AVS only creates one from **Wheel Static Mesh** when the wheel has no children.



## Trace Settings

| Setting | Default | Description |
|---|---|---|
| **Trace Channel** | `TraceTypeQuery1` | The channel the suspension trace tests against, in both wheel modes. In a default project this is the Visibility channel. |
| **Trace Type** | `Line` | `Line` traces a single line. `Sphere` traces a sphere of the wheel's radius. |
| **Trace Ignore Actors** | empty | Actors the trace ignores. `AddWheelTraceIgnoreActor` adds to it at runtime. |

> `Sweep` appears in **Trace Type** but is not implemented. A wheel set to it never finds the ground.



## Skeletal Mesh

| Setting | Default | Description |
|---|---|---|
| **Connect to Bone** | off | Constrains a skeletal mesh bone to the wheel, so the bone follows it. Physics mode only. |
| **Bone Name** | none | The bone to constrain, and to snap the wheel to when it is a child of a skeletal mesh. |

See [Skeletal Wheels](https://overtorque-creations.com/Dev/Docs/#AVS/Skeletal_Mesh/Skeletal_Wheels.md).



## Physics

| Setting | Default | Description |
|---|---|---|
| **Wheel Mass** | `15` kg | Rim and tire mass used by the wheel simulation. Does not change the wheel mesh's physics body. |
| **Wheel Spin Enabled** | on | Lets the wheel spin when it gets more torque than it has grip for. Off, the wheel stays planted regardless of torque or friction. |
| **Physics Material** | `Tire` | Applied to the wheel mesh. Provides the grip for physics mode wheels. |
| **Override Weight** | off | Sets the wheel mesh's physics body to **Override Weight Kg**. Off, AVS sets the body to `100` kg. |
| **Override Weight Kg** | `0` kg | The body mass used when **Override Weight** is on. |
| **Collide When Detached** | off | Whether a detached wheel collides with its own vehicle. |



## Tires

| Setting | Default | Description |
|---|---|---|
| **Tire Friction** | `1.4, 1.4` | Grip coefficients. X is along the wheel, for accelerating and braking. Y is across it, for cornering. Raycast mode only. |
| **Arcade Wheel Friction** | on | Applies the friction force at the wheel's center instead of the contact point. Raycast mode only. |



## Powertrain

| Setting | Default | Description |
|---|---|---|
| **Is Driving Wheel** | off | Receives torque from the drivetrain. Each driving wheel receives the gear table's full torque. |
| **Invert Torque** | off | Reverses the torque direction, for wheels facing the opposite way. Also mirrors wheel effect offsets. |



## Steering

| Setting | Default | Description |
|---|---|---|
| **Is Steerable Wheel** | off | Turns with the vehicle's steering input. |
| **Max Steering Angle** | `30` degrees | The furthest the wheel turns. |
| **Invert Steering** | off | Reverses the steering direction, for wheels that turn opposite to the front. |



## Brakes

| Setting | Default | Description |
|---|---|---|
| **Is Braking Wheel** | on | Slows with the brake input. |
| **Is Handbrake Wheel** | off | Locks completely while the handbrake is applied. |
| **Brake Torque** | `2500` Nm | Braking force for this wheel. Setting it per wheel sets the brake balance. |
| **Rolling Resistance** | `0.01` | Constant drag, from `0` to `1`. Applied as a minimum brake input on every wheel. |

Wheels also lock without these settings: driving wheels lock while the shifter is in Park, and any wheel with a braking, handbrake or driving role locks below the vehicle's **Brake Lock Speed** with the brake held at `0.95` or more.



## Suspension Dynamics

| Setting | Default | Description |
|---|---|---|
| **Editor Preview** | off | Draws the wheel at both ends of its travel in the viewport. |
| **Spring Length** | `25` cm | Total suspension travel, centered on the wheel component. |
| **Spring Strength** | `25` N/mm | How hard the spring pushes back. |
| **Spring Damping** | `1.0` kNs/m | How quickly suspension movement settles. |

Past full compression, AVS adds an upward force, easing in up to the vehicle's weight, to push the wheel back out of the ground. This only applies with the `Line` trace type.



## Physics Mode Suspension

These only apply to physics mode wheels.

| Setting | Default | Description |
|---|---|---|
| **Has Spring** | on | Whether the wheel's constraint has a linear spring. |
| **Spring Hard Lock** | off | On, the wheel stops at the end of its travel. Off, it is damped past it. |
| **Physics Downforce** | `50` N | A constant force down the wheel's local -Z. Can be changed at runtime with `SetSpringDownforce`. |



## Wheel Effects

| Setting | Default | Description |
|---|---|---|
| **Override Global Effects** | off | Replaces the vehicle's global wheel effects for this wheel. An empty override silences them. |
| **Global Effects Override** | empty | The replacement global effects. |
| **Effects For Default Surface Override** | empty | Replaces the vehicle's default surface effects. An empty one falls back to the vehicle's. |
| **Surface Effect Overrides** | empty | Replaces the vehicle's effects for specific surfaces. An empty entry silences that surface. |

See [Wheel Effects](https://overtorque-creations.com/Dev/Docs/#AVS/Wheel_Effects/Overview.md).



## Wheel Data Output

Live values the simulation writes to the wheel's **Wheel Data** each physics tick.

| Value | Unit | Description |
|---|---|---|
| **Suspension Force** | N | How much load the wheel is carrying. |
| **Current Spring Length** | cm | How far the spring is currently extended. |
| **Slip** | | Total tire slip. Above `1.0`, the wheel is asking for more grip than it has. |
| **Slip2D** | | Slip split into X, along the wheel, and Y, across it. |
| **Contact Normal Speed** | cm/s | Speed into the contact surface. |
| **Angular Velocity** | rad/s | How fast the wheel is spinning. |
| **Last Trace** | | The suspension trace's last result. Also available from `GetLastTouch`. |

> **Slip** and **Slip2D** are only calculated for raycast wheels. A physics wheel reads zero, or the last raycast value if it was switched from raycast mode.



## Wheel Functions

On each `AVS_Wheel`:

| Function | Description |
|---|---|
| `Detach()` | Takes the wheel off. It becomes a loose physics body. |
| `Attach(ResetPosition, Force)` | Puts a detached wheel back. **Force** rebuilds its constraints. |
| `SetWheelMode(NewMode)` | Switches between `Raycast` and `Physics`. |
| `ChangeStaticMesh(NewMesh)` | Swaps the wheel mesh and reattaches the wheel. |
| `SetWheelTorque(OverrideVehicle, TargetSpeed, Torque, Reverse)` | Drives the wheel directly. **TargetSpeed** is in cm/s, **Torque** in hNm. |
| `LockWheel(Lock)` | Locks or releases the wheel. |
| `SetWheelPosition(Location, Rotation)` | Moves the wheel relative to its parent. |
| `ResetPosition()` | Returns the wheel to its configured position. |
| `ResetSteering()` | Centers the wheel's steering and clears its steering input. |
| `ResetWheelCollisions()` | Rebuilds the wheel's collision setup. |
| `SetSpringDownforce(Downforce)` | Sets **Physics Downforce**. |
| `ClearAllEffects()` | Stops the wheel's effects. They are rebuilt on the next tick. |
| `AddWheelTraceIgnoreActor(Actor)` | Adds an actor to **Trace Ignore Actors**. |
| `GetHasContact()` | Whether the trace is hitting something. |
| `GetContactSurfaceType()` | The physical surface under the wheel. |
| `GetIsAttached()` | Whether the wheel is attached. |
| `GetIsLocked()` / `GetIsBrakeLocked()` | Whether the wheel is locked, and whether that lock comes from the brakes. |
| `GetRotationSpeed()` | The wheel's surface speed, in the vehicle's **Speed Units**. |
| `GetWheelAngVelInRadians()` | The wheel's angular velocity, in rad/s. |
| `GetWheelVelocity(Local)` | The wheel's velocity, in world space or relative to the wheel. |
| `GetWheelTorque()` | The wheel's current drive torque, in hNm. |
| `GetSuspensionSettings()` | Outputs **Spring Length**, **Spring Strength**, **Spring Damping** and **Physics Downforce**. |
| `GetLastTouch()` | The last trace result. |
| `GetWheelMesh()` | The wheel mesh component. |

On the vehicle:

| Function | Description |
|---|---|
| `GetWheels()` | Every wheel on the vehicle. |
| `DetachAllWheels()` | Detaches every wheel. |
| `ResetAllWheels()` | Returns every wheel to position and reattaches detached ones. |
| `AddWheel(NewWheel)` | Protected. Installs a wheel component already attached under the vehicle mesh. |
| `RemoveWheel(Wheel)` | Protected. Detaches the wheel, destroys its mesh and destroys the component. |



## Wheel Reprojection

Set on the vehicle, under **Advanced Vehicle System → Experimental**. Reprojection hides the real wheel mesh and shows a **projected mesh** in its place, aligned to the wheel every frame, so wheels do not look tilted by the physics engine.

| Setting | Default | Description |
|---|---|---|
| **Wheel Reprojection** | off | Turns reprojection on. |
| **Reprojection Smooth Alpha** | `0.75` | How quickly the projected mesh follows, from `0` to `1`. |
| **Wheel Reprojection Camber** | off | Adds cosmetic camber driven by suspension compression. |
| **Camber Compressed** | `30, 15` | Wheel height in cm, and camber angle in degrees, at the compressed end. |
| **Camber Decompressed** | `-30, -15` | The same, at the extended end. |

Camber is visual only and does not change how the wheel simulates.
