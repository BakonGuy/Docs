# Steering

Steering happens in two places: the vehicle decides how much input to allow and how fast to get there, and each wheel decides what to do with it. Tuning usually means adjusting both.



## Basic Understanding

Steering input travels through three stages before it reaches a wheel.

**Smoothing.** The vehicle moves its steering value toward your input at a rate set by the smoothing mode, rather than snapping to it.

**Falloff.** A curve scales the maximum input the player can ask for, based on speed. This is what stops a vehicle from being undriveable on a straight.

**The wheel.** Each steerable wheel turns up to its own **Max Steering Angle**, which is the physical limit everything else scales within.



## Step 1: Set up the Steerable Wheels

<!-- side-by-side:57 -->
Mark your front wheels **Is Steerable Wheel** and set **Max Steering Angle** — `30` degrees is a sensible car.

This is the physical limit. Everything else scales within it.
<!-- split -->
![Advanced Vehicle System - Steering settings showing the smoothing mode, steering speed, recenter speed and falloff curve](../Assets/Images/_placeholder.png "Vehicle-level steering settings")
<!-- /side-by-side -->



## Step 2: Pick a Smoothing Mode

`Ease` is the default and suits most vehicles. The modes are covered below.



## Step 3: Tune Steering Speed and Recenter Speed

Tune these at low speed, where the falloff curve is not interfering.

Get turn-in feeling right standing still before worrying about anything else.



## Step 4: Shape the Falloff Curve

Shape the curve last, driving at speed. This is what stops the car from being twitchy on a straight.



## Steering Input Smoothing Modes

<!-- side-by-side:50 -->
`Ease` — steering eases toward the input and slows as it arrives. The default, and the most natural for a vehicle with a real steering rack.

`Constant` — steering moves toward the input at a fixed rate. More mechanical, predictable, and a reasonable choice for heavy machinery.
<!-- split -->
`Instant` — steering matches input immediately, no smoothing.

Suits arcade handling where responsiveness matters more than realism, or when you are feeding in already-smoothed input from your own code.

With `Instant` selected, the speed settings hide themselves — nothing uses them.
<!-- /side-by-side -->



## Steering Speed and Steering Recenter Speed

**Steering Speed** is how quickly the vehicle responds to a change in input.

**Steering Recenter Speed** is a separate rate, used when steering returns to center from zero input.

Real vehicles self-center faster than a driver turns in. Setting recenter faster than turn-in removes a lot of floatiness without changing grip.



## Steering Falloff Curve

A curve mapping **speed** (X) to **maximum steering input** (Y). Without it, full steering lock stays available at every speed and the vehicle becomes undriveable on a straight.

<!-- side-by-side:57 -->
At low speed you want full lock available, so the curve starts at or near `1.0`. As speed climbs, the available input should shrink — a car at motorway speed should barely turn compared to one in a car park.

The curve does not change the physical **Max Steering Angle**. It scales how much of that angle the player can ask for, so a shallow curve at speed means the wheels simply do not turn as far.

Raising the curve's values across the board allows stronger steering at speed, which is one of the main levers for arcade handling. See [Arcade Physics](https://overtorque-creations.com/Dev/Docs/#AVS/Guides/Arcade_Physics.md).
<!-- split -->
![Steering falloff curve editor with speed on the X axis and maximum steering input on the Y axis](../Assets/Images/_placeholder.png "Full lock when slow, progressively less as speed climbs")
<!-- /side-by-side -->

`GetMaxSteeringInputAtSpeed` evaluates the curve at any speed, which is useful for a HUD showing available grip.

> If a vehicle darts unpredictably at speed, the falloff curve is the first thing to look at — before touching tire friction.



## Wheel Steering Settings

On each `AVS_Wheel`, under **Wheel Dynamics → Steering**:

| Setting | Default | What it does |
|---|---|---|
| **Is Steerable Wheel** | `false` | Wheel rotates in yaw with steering input |
| **Max Steering Angle** | `30.0` deg | Maximum angle this wheel steers to |
| **Invert Steering** | `false` | Reverses steering direction for this wheel |



## Four Wheel Steering

Enable **Is Steerable Wheel** on the rear wheels, turn **Invert Steering** on for them, and give them a **smaller** max angle than the front — something like 30 front and 10 rear.

> Matching the front angle makes the vehicle crab-walk sideways. Keep the rear angle small.



## Manual Wheel Steering Control

A wheel that is not marked steerable can still be driven directly with `SetSteeringInput` on the wheel component.

That is the path for anything unusual — crab steering, independently controlled wheels, or a turret-style vehicle where steering does not come from a single axis.



## Reading Steering Values at Runtime

Three values that are easy to confuse:

| Function | Returns | Use it for |
|---|---|---|
| `GetSteeringInput` | The replicated input value | Network-consistent logic |
| `GetCurrentSteeringInput` | Local input on the controlling client, replicated elsewhere | Responsive UI on the local player |
| `GetCurrentSteering` | The **actual** steering after smoothing and falloff | Steering wheel meshes and animation |

For a steering wheel mesh or a driver's hands, `GetCurrentSteering` is what you want. It is the only one that matches what the wheels are actually doing — the input values will run ahead of it while smoothing catches up.
