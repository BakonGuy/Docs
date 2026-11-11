# Physics

How the vehicle body responds to its wheels: mass, center of mass, drag and flipping, and how to move a vehicle without breaking its physics. The example is the compact car from the [Quick Start](https://overtorque-creations.com/Dev/Docs/#AVS/Getting_Started/Quick_Start.md).

Wheel and suspension settings are on the [Wheels](https://overtorque-creations.com/Dev/Docs/#AVS/Components/Wheels.md) page. Every physics setting is listed in [Vehicle Settings](https://overtorque-creations.com/Dev/Docs/#AVS/Reference/Vehicle_Settings.md).

Apply the [recommended project settings](https://overtorque-creations.com/Dev/Docs/#AVS/Getting_Started/Project_Settings.md) before tuning anything here.



## How Vehicle Physics Works

The vehicle mesh (`VehicleMesh`) is an ordinary Unreal physics body. Every physics tick, each wheel pushes on it with its suspension and grip forces.

How the body responds to those forces depends on its mass, its center of mass and its drag, the same as any other physics object.



## Setting Vehicle Mass

Mass is set on `VehicleMesh`, in Unreal's own physics settings: turn on **Mass (kg)** override and enter a value. AVS does not change it.

For the example compact car, use `1100` kg. A midsize sedan is `1500` kg or more. Under `1000` kg is very light for a car.

Set mass before tuning tire friction. Each wheel's grip is calculated from the load it carries, so changing mass changes how the same friction values feel.



## Setting the Center of Mass

<!-- side-by-side:57 -->
1. Add a **Center of Mass** component directly under `VehicleMesh`.
2. Move it slightly above wheel height, between the axles.
3. Turn on **Visualize Center of Mass** under **Advanced Vehicle System → Debug**, and check where the physics body's center of mass actually is.
<!-- split -->
![Vehicle in the viewport with the center of mass visualizer enabled, showing the marker below the chassis](../Assets/Images/_placeholder.png "Visualize Center of Mass draws the center of mass the body is using")
<!-- /side-by-side -->

The higher the center of mass is above the wheels, the more the body rolls in corners. At wheel height it does not roll. Below the wheels, it leans into the turn.

The [Center of Mass](https://overtorque-creations.com/Dev/Docs/#AVS/Components/Center_Of_Mass.md) page covers the component and the center of mass functions.



## How the Collision Shape Affects Handling

AVS does not set the vehicle body's inertia. Unreal calculates it from the shapes in `VehicleMesh`'s collision and the vehicle's mass. Inertia decides how hard the body is to rotate, so it sets how quickly the vehicle rolls, pitches and turns.

Two vehicles with the same mass and the same AVS settings, but different body collision, rotate differently. Without a Center of Mass component, their centers of mass differ too, because Unreal places the center of mass from the collision as well.

Welded objects count. A mesh with collision attached directly to `VehicleMesh` adds its shape and mass to the body. See [Attaching Objects](https://overtorque-creations.com/Dev/Docs/#AVS/Components/Attaching_Objects.md).

To change the inertia without changing the collision, use **Inertia Tensor Scale** in `VehicleMesh`'s advanced **Physics** settings. A higher value on an axis makes the body harder to rotate around it.



## Using a Separate Collision Mesh

Vehicles that should handle the same can share one collision mesh, with a different visible body on each.

1. Set `VehicleMesh` to a mesh that only provides the collision. Use the same one on every vehicle that shares the handling.
2. Turn on **Hidden in Game** on `VehicleMesh`, so it does not draw.
3. Add a Static Mesh component under `VehicleMesh` with the visible body. Set its **Collision Presets** to `NoCollision`, so none of its collision welds into the vehicle's body.

The shared collision then sets the inertia, and the center of mass when there is no Center of Mass component, the same way on every vehicle.



## Flipped Vehicles

**Upside Down Angle Threshold** (`90` degrees) sets when the vehicle counts as upside down. `0` is upright and `180` is fully inverted. With the default, a vehicle counts as flipped once it tips past its side.

`GetIsUpsideDown` returns the current state. **OnVehicleFlipped** fires whenever it changes, with **bIsNowUpsideDown** true when the vehicle has just flipped over.

AVS has no built-in function for righting a vehicle. What happens after a flip is up to your own code, for example a recovery prompt started from **OnVehicleFlipped**.



## Moving a Vehicle

`TeleportVehicle(Location, Rotation, KeepRelativeVelocity)` moves the vehicle, its wheels and any hitched trailers together.

Physics mode wheels are separate physics bodies. Setting the actor transform directly moves the vehicle body and leaves them behind.

**KeepRelativeVelocity** true keeps the vehicle's velocity, turned to match the new rotation. False arrives stopped.

`SetVehiclePositionWithVirtualPivot(LocalPivot, WorldTransform, Teleport)` places the vehicle so that a point on it lands on a world transform. Use it when a spawn marker stands for something other than the vehicle's origin, such as a trailer coupler.

`SetPhysics(Simulate)` turns simulation on or off for the vehicle body and all its wheels. **Start With Physics** (on) decides whether it simulates when spawned.



## Drag

AVS sets the vehicle mesh's linear damping itself on every 25 TPS tick. The value it uses while driving, **Base Linear Drag**, is taken from the mesh's **Linear Damping** when the vehicle is built. To change the vehicle's drag, change **Linear Damping** on `VehicleMesh`.

When the vehicle is almost stopped with no throttle, AVS raises the damping to as much as `15` to help it settle.

**Dynamic Air Drag** adds extra damping while the vehicle slides sideways. It is off by default, and is not shown in the details panel. Turn it on from Blueprint or C++.



## Wheels Push Physics

With **Wheels Push Physics** on (the default), each wheel's force is also applied, in reverse, to the simulating object it stands on. A vehicle on a loose platform pushes the platform down.

With raycast wheels, this can rock a vehicle back and forth on another physics object. Turning it off stops the wheels pushing on the object.



## Vehicle Max Angular Velocity

**Vehicle Max Angular Velocity** (`700` deg/s) limits how fast the vehicle body can spin, for example after a crash. It only applies to the vehicle mesh.

Physics mode wheels are limited by the project's own **Max Angular Velocity** setting instead. See [Project Settings](https://overtorque-creations.com/Dev/Docs/#AVS/Getting_Started/Project_Settings.md).



## When a Vehicle Counts as Stopped

A vehicle counts as at rest after its speed stays below **Rest Velocity Threshold** (`25` cm/s) for 3 seconds. Passive mode and the network rest state both use this.

A vehicle creeping slower than the threshold also counts as stopped. Lower the threshold if slow vehicles are treated as stopped while they still move.



## Reading Physics State

- `GetAirSpeed` returns the vehicle's speed, in your **Speed Units**.
- `GetAcceleration` returns the change in speed per second.
- `GetSlip` returns sideways speed, from `0` below 1000 cm/s to `1` at 2500 cm/s. It is not tire slip. Each wheel has its own slip in its **Wheel Data**.
- `AnyWheelContact` returns whether any wheel is touching a surface.



## Fixing Vehicle Physics Problems

### Vehicle Shakes While Parked

See [A Vehicle That Shakes, Jitters or Bounces](https://overtorque-creations.com/Dev/Docs/#AVS/Advanced/Debugging.md).

### Vehicle Rocks Back and Forth on Another Physics Object

Turn off **Wheels Push Physics**.

### Physics Wheels Are Left Behind When the Vehicle Is Moved

Move the vehicle with `TeleportVehicle`.



## Next Steps

- [Center of Mass](https://overtorque-creations.com/Dev/Docs/#AVS/Components/Center_Of_Mass.md): placing and changing the center of mass.
- [Wheels](https://overtorque-creations.com/Dev/Docs/#AVS/Components/Wheels.md): suspension and grip.
- [Debugging](https://overtorque-creations.com/Dev/Docs/#AVS/Advanced/Debugging.md): tools for a vehicle that does not behave.
