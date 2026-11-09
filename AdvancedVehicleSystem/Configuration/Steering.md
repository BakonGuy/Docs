# Steering

How steering input turns the wheels, and how to tune it. The example is the rear wheel drive car from the [Quick Start](https://overtorque-creations.com/Dev/Docs/#AVS/Getting_Started/Quick_Start.md), steering with its front wheels.

Every steering setting is listed in [Vehicle Settings](https://overtorque-creations.com/Dev/Docs/#AVS/Reference/Vehicle_Settings.md) and [Wheel Settings](https://overtorque-creations.com/Dev/Docs/#AVS/Reference/Wheel_Settings.md).



## How Steering Works

Steering input goes through three steps before it reaches a wheel:

1. The **Steering Falloff Curve** limits how much of the input is allowed at the current speed.
2. The vehicle's steering moves toward that limited input, at the rate set by the smoothing settings.
3. Each steerable wheel turns by that steering, multiplied by its own **Max Steering Angle**.

The first two steps are set on the vehicle. The third is set on each wheel.



## Setting Up Steerable Wheels

<!-- side-by-side:57 -->
1. Select a front wheel and turn on **Is Steerable Wheel**.
2. Leave **Max Steering Angle** at `30` degrees. This is how far the wheel turns at full steering.
3. Do the same on the other front wheel.
<!-- split -->
![Wheel Dynamics section showing the Powertrain, Steering and Brakes groups with their flags](../Assets/Images/_placeholder.png "Is Steerable Wheel and Max Steering Angle on a front wheel")
<!-- /side-by-side -->

The car now steers. The rest of this page tunes how it steers.



## Steering Speed and Steering Recenter Speed

**Steering Speed** (`2.5`) is how fast the steering moves toward the input. Higher is faster.

**Steering Recenter Speed** (`2.5`) is used in its place whenever the input is closer to center than the current steering, such as when the stick is released or eased off. Set it higher than **Steering Speed** and the wheels straighten faster than they turn in.

Tune both at low speed, where the falloff curve allows full steering.



## Steering Input Smoothing

**Steering Input Smoothing** decides how the steering moves toward the input:

- `Ease` (the default) moves quickly at first and slows as it gets close.
- `Constant` moves at a fixed rate. A **Steering Speed** of `1.0` goes from center to full in 1 second.
- `Instant` matches the input immediately. The speed settings are greyed out, since nothing uses them. Use it when your own code already smooths the input.



## Steering Falloff Curve

The **Steering Falloff Curve** maps speed (X), in your **Speed Units**, to the largest steering input allowed (Y).

<!-- side-by-side:57 -->
The default curve allows full steering when stopped, and less as speed rises:

| Speed | Steering allowed |
|---|---|
| `0` | `1.0` |
| `20` | `0.8` |
| `60` | `0.4` |
| `120` | `0.3` |

At 60, the example car's front wheels turn at most `0.4 × 30` = 12 degrees.
<!-- split -->
![Steering falloff curve editor with speed on the X axis and maximum steering input on the Y axis](../Assets/Images/_placeholder.png "Full steering when slow, less as speed rises")
<!-- /side-by-side -->

Lower values at high speed make the car turn less at speed. Higher values allow more steering at speed. A flat curve at `1.0` allows full steering at every speed.

While the car slides sideways, AVS raises the limit. Nothing is added below 1000 cm/s of sideways speed, rising to an extra `1.0` at 2500 cm/s, so a sliding car can reach full steering.

`GetMaxSteeringInputAtSpeed` returns the curve's value at a speed, without the allowance for sliding.



## Four Wheel Steering

To steer with the rear wheels as well:

1. Turn on **Is Steerable Wheel** on the rear wheels.
2. Turn on **Invert Steering** on the rear wheels, so they turn the opposite way to the front. This tightens the turning circle.
3. Set the rear wheels' **Max Steering Angle** to how far they should turn.

Without **Invert Steering**, the rear wheels turn the same way as the front, and the vehicle moves diagonally (crab steering).



## Reading Steering Values

- `GetSteeringInput` returns the replicated input, rounded to steps of `0.1`.
- `GetCurrentSteeringInput` returns the raw input on the controlling client, and the replicated input on other machines.
- `GetCurrentSteering` returns the steering the wheels use, after the falloff curve and smoothing.

A steering wheel mesh or a driver's hands should follow `GetCurrentSteering`. The input values run ahead of it while the smoothing catches up.



## Steering One Wheel from Code

`SetSteeringInput` on a physics mode wheel turns that wheel directly. On a steerable wheel, the vehicle's steering replaces the value whenever the steering changes, so use it on wheels with **Is Steerable Wheel** off.

Raycast wheels only steer from the vehicle's steering. On a raycast wheel, `SetSteeringInput` turns the wheel mesh without changing the direction the wheel pushes.



## Next Steps

- [Physics](https://overtorque-creations.com/Dev/Docs/#AVS/Configuration/Physics.md): mass, body limits and moving a vehicle.
- [Arcade Physics](https://overtorque-creations.com/Dev/Docs/#AVS/Guides/Arcade_Physics.md): steering and grip for arcade handling.
- [Vehicle Settings](https://overtorque-creations.com/Dev/Docs/#AVS/Reference/Vehicle_Settings.md): every steering setting.
