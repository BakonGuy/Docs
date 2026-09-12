# Debugging

Tools for diagnosing a vehicle that is not behaving correctly, in the order to try them.



## Basic Understanding

There are three tools, and they answer different questions.

**The config assist HUD** shows live vehicle state while you drive, and lets you change spring values without leaving the game.

**The debug visualizers** draw things you cannot otherwise see — the real center of mass, suspension traces.

**`stat AVS`** reports what the plugin is costing, and how many vehicles are actually awake.



## Config Assist HUD

The plugin includes a HUD for building and tuning vehicles. It shows live vehicle state and lets you change spring values while driving, which is much faster than editing components and pressing play again.

<!-- side-by-side:57 -->
Setup is covered in step 5 of the [Quick Start guide](https://overtorque-creations.com/Dev/Docs/#AVS/Getting_Started/Quick_Start.md). Once it is running, press **Shift + F1** to get your cursor back so you can use the sliders.

Read the state panel before changing anything. Air speed, shifter position, current gear, torque, engine status and wheel count answer most questions immediately. A vehicle in Park with the engine off is a different problem from one in Drive producing no torque.

<!-- split -->
![In-game vehicle with the setup HUD showing air speed, shifter position, current gear, torque and live spring sliders](../Assets/Images/tutorials-Creating-Vehicles-16.png "The config assist HUD running in game")
<!-- /side-by-side -->



## Config Assist HUD Spring Sliders

Spring values from the HUD apply to **every wheel**, overriding your per-wheel configuration, and are not saved.

That is what makes it fast for finding a baseline, and why you copy the numbers back into the components when you are done.



## Debug Visualizers

Under **Advanced Vehicle System → Debug**:

| Setting | Shows |
|---|---|
| **Visualize Center of Mass** | The actual physics center of mass, after every offset |
| **Suspension Debug** | Suspension traces and state |
| **Display Debug Menu** | The in-game debug menu, also toggleable with `ShowDebugMenu` |

Enable **Visualize Center of Mass** first when a vehicle handles strangely with no obvious cause. A center of mass in an unexpected position explains a large share of handling problems, and it cannot be confirmed by looking at the vehicle.



## Wheel Editor Preview

**Editor Preview**, under the wheel's Suspension Dynamics, draws wheel travel in the viewport.

This makes it obvious when spring length does not fit the wheel well.



## Profiling with stat AVS

`stat AVS` opens the plugin's stat group — the full tick pipeline plus **Active Vehicles**, **Simulated Wheels**, and **Wheels In Contact**.

Check the counters before the timings. See [Tick and Performance](https://overtorque-creations.com/Dev/Docs/#AVS/Advanced/Tick_And_Performance.md) for how to read it.



## Useful Console Commands

```text
stat AVS          Plugin tick pipeline and vehicle counters
stat Physics      Engine-side physics cost
stat FPS          Frame rate, for comparing against the physics rate
```



## A Vehicle That Shakes, Jitters or Bounces

Four settings account for most cases. Work through them in this order.

**1. Vehicle mass is too low.** Under `1000` kg is very light for a car. A light body reacts violently to suspension forces meant for a heavier one. See [Vehicle Mass](https://overtorque-creations.com/Dev/Docs/#AVS/Configuration/Physics.md).

**2. Tire friction is too high.** Friction well above the default `1.4` on the Y axis is a frequent cause. Put it back to `1.4` and confirm the shaking stops before tuning it again.

**3. Project physics settings have not been applied.** Substepping in particular. See [Recommended Project Settings](https://overtorque-creations.com/Dev/Docs/#AVS/Getting_Started/Project_Settings.md).

**4. Spring strength does not match the mass.** A vehicle riding on fully compressed springs has no travel left to absorb anything. Check it with **Editor Preview** and aim for roughly half compression at rest.

If all four check out and the vehicle still shakes only while standing still, it is worth confirming the behavior persists in a fresh project. Physics sleep behavior has changed between engine releases and has caused this symptom before.



## Physics Wheels Reacting Harshly to Bumps and Curbs

A wheel in **physics mode** is a rigid body. It reacts as though perfectly solid — rolling a marble at a curb rather than a tire deforming over it, since there is no tire deformation model.

A sharp edge therefore transfers its full impulse into the vehicle.

Raycast wheels do not have this behavior, and have been the default since 1.4. Switching the wheel mode is the first thing to try.

Suspension values will not tune it away, because the harshness is in the collision rather than in the spring.



## Common Problems and Fixes

| Symptom | Usual cause |
|---|---|
| Vehicle will not move | Engine off, or still in Park. A vehicle spawns with both. Then check a wheel has **Is Driving Wheel** on. |
| Stops responding after sitting still | It went passive. See [Tick and Performance](https://overtorque-creations.com/Dev/Docs/#AVS/Advanced/Tick_And_Performance.md). |
| Wheels bounce or misbehave at speed | Physics wheel mode with non-sphere collision, or engine contact offsets left at default. |
| Wheels will not reach high speed | Engine **Max Angular Velocity** too low. See [Project Settings](https://overtorque-creations.com/Dev/Docs/#AVS/Getting_Started/Project_Settings.md). |
| Skeletal mesh parts jitter violently | Physics constraints defined inside the mesh's Physics Asset. Move them to the vehicle Blueprint. |
| Inputs do nothing in multiplayer | Called from a non-owning client. See [Networking](https://overtorque-creations.com/Dev/Docs/#AVS/Advanced/Networking.md). |
| Vehicle see-saws on another physics object | Turn off **Wheels Push Physics**. |
| Effects cut off abruptly | An effect destroying its particles instead of deactivating them. See [Custom Effects](https://overtorque-creations.com/Dev/Docs/#AVS/Wheel_Effects/Custom_Effects.md). |
| Vehicle shoved around by a character | A character collision is welded to the vehicle. See [Attaching Objects](https://overtorque-creations.com/Dev/Docs/#AVS/Components/Attaching_Objects.md). |
| Actors vanish at distance while driving | Network relevancy using a stale view location. See [Networking](https://overtorque-creations.com/Dev/Docs/#AVS/Advanced/Networking.md). |
| Skeletal wheels sit or scale wrong | A bone with non-1.0 scale. Apply the scale in your modeling software. |



## Getting Support

If none of that covers it, the `#avs_support` channels on [Discord](https://discord.gg/8YvWARS) are the fastest way to reach me, or send an email.

Include your Unreal version, your AVS version, and whether you are using raycast or physics wheels. Those three answer most of the first round of questions.



## Finding What Is Pushing a Vehicle

When a vehicle is being shoved by something and it is not obvious what, let the engine tell you.

Select `VehicleMesh` on the vehicle, add its **On Component Hit** event, and print the **Other Component** to the screen.

Reproduce the problem and the print spams whatever component is doing the pushing, usually a character mesh/capsule or a weapon mesh that still has collision enabled.

