# Wheels

Wheels are separate `AVS_Wheel` components. Each one owns its own configuration, so a vehicle is however many wheels you add, wherever you put them — four, six, three, or one.

This page covers configuring them, tuning them, and changing them at runtime. For your first vehicle, follow step 3 of the [Quick Start guide](https://overtorque-creations.com/Dev/Docs/#AVS/Getting_Started/Quick_Start.md) first.



## Basic Understanding

A vehicle's wheel set is built from the `AVS_Wheel` components parented under the vehicle mesh. There is no wheel array to fill in and no axle setup — adding a component in the right place is what adds a wheel.

Because each wheel carries its own configuration, there is no concept of a "front axle" or a "rear axle" in AVS. A wheel steers because its own **Is Steerable Wheel** flag is on, not because of where it sits.

Each wheel runs in one of two modes, raycast or physics, and can switch between them at runtime.



## Wheel Modes: Raycast and Physics

AVS has two wheel modes, and has been designed so you can seamlessly switch between them at runtime depending on your needs.

<!-- side-by-side:50 -->
**Raycast**

This is the more traditional approach, the wheel is a trace against the world. Nothing physical is there.

Fast, stable, and forgiving. It ignores the wheel mesh's collision entirely, so a bad collision shape does not matter unless you intend to detach the wheel.

This is typically the default mode games use. Use it unless you have a specific reason not to.
<!-- split -->
**Physics**

The wheel is a real physics body held on by constraints. It is considerably more expensive, and it inherits collision from your wheel mesh. Friction in this mode is handled entirely by the physics engine, therefore you need to make sure your physics materials are setup properly.

Use it when you need real wheel collision or the [Connect to Bone](https://overtorque-creations.com/Dev/Docs/#AVS/Skeletal_Mesh/Skeletal_Wheels.md) skeletal workflow.

This mode typically requires a simple sphere collision on your wheel mesh to remain stable at speed. Complex collisions can be acceptable for low speed vehicles. Chaos has no true cylinder collision. See [Important Information](https://overtorque-creations.com/Dev/Docs/#AVS/Getting_Started/Important_Information.md) for more information.
<!-- /side-by-side -->



## Adding and Configuring a Wheel

<!-- side-by-side:57 -->
**1. Add the component and position it.** Add an `AVS_Wheel` under the vehicle mesh and move it to the wheel's center. Name it after its position, such as `WheelFrontDriver`.

The component's location is the **center of wheel travel**, not where the wheel rests at ride height.

**2. Set the wheel mesh.** With **Auto Wheel Radius** on, the radius comes from the mesh bounds and you can leave **Wheel Radius** alone. With no mesh assigned, AVS falls back to a sphere at the configured radius, which is fine for prototyping.

**3. Set the wheel mode**, per the decision above.

**4. Assign roles** — driving, steering, braking, handbrake. Covered below.

**5. Tune suspension** last, once the vehicle actually drives.
<!-- split -->
![Vehicle Wheel - Config panel showing Wheel Mode Raycast, wheel mass and tire friction, plus the Drive/Steer, Brakes and Suspension sections](../Assets/Images/tutorials-Creating-Vehicles-10.png "Every wheel carries its own full configuration")
<!-- /side-by-side -->



## Wheel Roles: Driving, Steering, Braking and Handbrake

A wheel does nothing on its own. Four independent flags decide what it participates in, and a normal car uses different combinations front and rear.

<!-- side-by-side:57 -->
**Is Driving Wheel** takes torque from the drivetrain. **Invert Torque** reverses it, which you need for wheels rotated 180 degrees — mirrored wheels on the passenger side are the usual case.

**Is Steerable Wheel** turns with steering input, up to **Max Steering Angle** (30 degrees by default). **Invert Steering** is what rear steering wheels need.

**Is Braking Wheel** is on by default and responds to the brake. **Brake Torque** per wheel is how you set brake bias — more at the front than the rear is the realistic arrangement.

**Is Handbrake Wheel** is off by default. Enable it on the rear wheels only. Enabling it on all four makes the handbrake stop the vehicle instead of breaking rear traction.
<!-- split -->
![Wheel Dynamics section showing the Powertrain, Steering and Brakes groups with their flags](../Assets/Images/_placeholder.png "Four independent roles — a wheel can have any combination")
<!-- /side-by-side -->



## Wheel Role Combinations

A conventional rear wheel drive car ends up as:

| | Front wheels | Rear wheels |
|---|---|---|
| Is Driving Wheel | off | **on** |
| Is Steerable Wheel | **on** | off |
| Is Braking Wheel | **on** | **on** |
| Is Handbrake Wheel | off | **on** |
| Brake Torque | higher | lower |

Front wheel drive swaps the driving wheels. All wheel drive enables driving on all four. Four wheel steering enables steering on the rear too, with **Invert Steering** on and a smaller max angle so it does not feel twitchy.



## Tire Friction

**Tire Friction** is a coefficient pair: **X is longitudinal** (accelerating and braking), **Y is lateral** (cornering). Both default to `1.4`.

Tune the two values independently:

| Want | Do |
|---|---|
| More responsive acceleration | Raise X |
| Sharper turn-in | Raise Y |
| A car that drifts | Lower Y, leave X alone |
| A car that spins its wheels | Lower X, or raise gear torque |

> This applies to **raycast wheels**. Physics wheels take friction from the Physics Material instead, so set it there.



## Arcade Wheel Friction

**Arcade Wheel Friction** (raycast only, on by default) applies friction at the wheel center instead of the contact point. This resists tipping.

Turn it off for physically correct behavior, which requires more care with the center of mass.



## Wheel Spin Enabled

**Wheel Spin Enabled** lets the wheel spin up when the drivetrain asks for more than the surface can deliver.

In 1.5 that spin feeds back into vehicle behavior, so burnouts actually work.

Disable it per wheel if you want a wheel that stays planted no matter what.



## Tuning Suspension

Do this last, and do it in game rather than in the details panel.

<!-- side-by-side:57 -->
**1. Get a baseline in the editor.** Turn on **Editor Preview** on a wheel to draw its travel in the viewport. What you are checking is that **Spring Length** fits inside the wheel well — a spring longer than the bodywork will push the wheel through it.

**2. Drive it with the config assist HUD.** The HUD's spring sliders apply to every wheel live while you drive. This is far faster than editing components and pressing play repeatedly. Press **Shift + F1** to get the cursor.

**3. Copy the values back.** HUD values override every wheel and are not saved. Once it feels right, put the numbers into the components, then set per wheel differences — softer rear, stiffer front — from that baseline.
<!-- split -->
![Wheel with Editor Preview enabled, drawing suspension travel in the viewport](../Assets/Images/_placeholder.png "Editor Preview shows whether spring length actually fits the wheel well")
<!-- /side-by-side -->



## Suspension Settings Reference

| Setting | Default | Unit | Raise it to |
|---|---|---|---|
| **Spring Length** | `25` | cm | Allow more travel |
| **Spring Strength** | `25` | N/mm | Hold more weight, less body roll |
| **Spring Damping** | `1.0` | kNs/m | Settle faster, bounce less |



## Physics Mode Suspension Settings

Physics wheel mode adds three more settings:

- **Has Spring** — whether the wheel is sprung at all.
- **Spring Hard Lock** — stop dead at the travel limit instead of damping past it.
- **Physics Downforce** — a constant force down the wheel's local -Z, `50` N by default.



## Attach/Detach vs Add/Remove Wheels

These are two different systems. **Detach** takes a wheel off the vehicle. **Remove** deletes the wheel component.

<!-- side-by-side:50 -->
**Attach/Detach** — the wheel comes off

`Detach()` breaks the wheel's attachment and lets the mesh simulate as a loose physics body. The component still exists and the vehicle still knows about it.

This is what you want for damage. A derby game blowing wheels off wrecked cars, a crash knocking a wheel loose, a jack lifting a wheel for repair — all Detach.

It is fully reversible. `Attach(ResetPosition, Force)` puts it back; pass `Force` to rebuild the constraints from scratch.
<!-- split -->
**Add/Remove** — the wheel set changes

`AddWheel()` **installs** a new wheel component 
`RemoveWheel()` **destroys** one, along with its mesh. 
Both then recompute the vehicle's wheel set.

This is for modular vehicles: Such as a truck gaining a third axle, or a configurator building vehicles from parts.

`RemoveWheel` is destructive. You cannot undo it, you would have to add a new wheel component and install it.
<!-- /side-by-side -->

If you only want the wheel to come off, use Detach. Use Add/Remove only when the vehicle has a different number of wheels than it did before.



## Detaching and Attaching Wheels at Runtime

For the whole-vehicle case there are two convenience calls on the vehicle:

- `DetachAllWheels()` — every wheel comes off at once. This is your explosion.
- `ResetAllWheels()` — snaps every wheel back to position and re-attaches any that were detached. This is your respawn.

```cpp
// Destroy: every wheel comes off and simulates as a loose body.
Vehicle->DetachAllWheels();

// Respawn: snap every wheel back into place and re-attach any that were detached.
Vehicle->ResetAllWheels();
```

A single wheel, for damage on one corner:

```cpp
if( UAVS_Wheel* Wheel = Vehicle->GetWheels()[WheelIndex] )
{
	Wheel->Detach();

	// Later, put it back. Force rebuilds the constraints from scratch.
	Wheel->Attach(/*ResetPosition*/ true, /*Force*/ true);
}
```

Detaching also clears that wheel's effects and refreshes contact modification, so you do not need to clean up after it.



## Collide When Detached

**Collide When Detached** decides whether a loose wheel collides with its own vehicle body. Leave it off unless you want wheels bouncing off the car they came from.

> Raycast wheels normally ignore the wheel mesh collision, but a detached wheel is a real physics body. If you plan to detach raycast wheels, give the mesh proper collision or it will fall through the world.



## Adding and Removing Wheels at Runtime

`AddWheel` and `RemoveWheel` are **protected**, so call them from inside your vehicle Blueprint or a C++ subclass, not from an outside actor.

Order matters when adding:

1. Create the `AVS_Wheel` component and attach it **under the vehicle mesh**. The wheel set is built from the vehicle mesh's children, so a wheel parented anywhere else will never be found.
2. Set its configuration — mode, roles, radius, suspension.
3. Call `AddWheel`. It runs the normal construct and initialize path, creates the wheel's mesh, refreshes the wheel set, and reapplies the vehicle's current physics state.

`RemoveWheel` handles the teardown itself: it detaches the wheel, destroys the wheel mesh, cleans up, drops it from the arrays, and destroys the component. Do not destroy the component yourself first.

```cpp
void AMyVehicle::AddSpareAxle()
{
	// 1. Create the component and attach it under the vehicle mesh.
	//    Wheels parented anywhere else are never found.
	UAVS_Wheel* NewWheel = NewObject<UAVS_Wheel>(this);
	NewWheel->SetupAttachment(VehicleMesh);
	NewWheel->RegisterComponent();
	NewWheel->SetRelativeLocation(FVector(-180.0f, 90.0f, 0.0f));

	// 2. Configure it before installing.
	NewWheel->WheelStaticMesh = SpareWheelMesh;
	NewWheel->WheelConfig.WheelMode = EWheelMode::Raycast;
	NewWheel->WheelConfig.IsDrivingWheel = true;
	NewWheel->WheelConfig.IsBrakingWheel = true;

	// 3. Install it. This constructs, initializes, rebuilds the wheel set,
	//    and reapplies the vehicle's current physics state.
	AddWheel(NewWheel);
}

void AMyVehicle::DropSpareAxle(UAVS_Wheel* Wheel)
{
	// Detaches, destroys the wheel mesh, cleans up and destroys the component.
	// Do not destroy the component yourself.
	RemoveWheel(Wheel);
}
```

> Changing the wheel set while driving is allowed, the change is immediate. Removing a driving wheel redistributes torque instantly, and removing a loaded wheel drops that corner of the vehicle.



## Reading Wheel State and Output Data

Live simulation output sits on the wheel's **Wheel Data**. Which value to use depends on what you are building:

<!-- side-by-side:57 -->
**Suspension animation and weight transfer.** **Suspension Force** (Newtons) is how loaded a corner is. **Current Spring Length** gives compression for driving a visual spring.

**Traction control, slip indicators, effects.** **Slip** is total slip; above `1.0` the wheel is demanding more grip than the surface can provide. **Slip2D** splits it into X (longitudinal: wheelspin and lockup) and Y (lateral: sliding).

**Landings and impacts.** **Contact Normal Speed** is speed into the contact normal, in cm/s.

**Surface reactions.** `GetContactSurfaceType` returns the physical surface under the wheel. `GetHasContact` returns whether the wheel is touching anything.
<!-- split -->
![Blueprint reading Suspension Force and Slip from a wheel's Wheel Data output](../Assets/Images/_placeholder.png "Wheel Data carries the live output from the physics simulation")
<!-- /side-by-side -->

Alongside those: `GetRotationSpeed`, `GetWheelTorque`, `GetWheelVelocity`, `GetIsLocked`, `GetIsBrakeLocked`, and `GetSuspensionSettings`.

If you are driving effects from any of this, use the [Wheel Effects](https://overtorque-creations.com/Dev/Docs/#AVS/Wheel_Effects/Overview.md) system rather than polling on Tick — it already handles per-wheel lifetime, surface changes, and pooling.



## Wheel Trace Settings

Contact detection uses a trace in both wheel modes, and suspension rides on the result.

**Trace Type** is `Line` by default, which is fastest and correct for most ground. Switch to `Sphere` or `Sweep` when wheels catch on grating, narrow beams, or sharp edges. The wider shape stops a single ray passing through a gap.

**Trace Channel** picks what the trace tests against. If your wheels are dropping through something they should touch, this is the first thing to check.

`AddWheelTraceIgnoreActor` adds an actor to the wheel's ignore list at runtime. Use it when a vehicle carries or tows something the wheels should not trace against.
