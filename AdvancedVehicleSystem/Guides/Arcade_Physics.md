# Tuning AVS for Arcade-Style Vehicles

AVS is built for realistic vehicle physics. These changes make a vehicle quicker to respond and more forgiving, for arcade-style handling. Each one can be used on its own.



## More Grip

Raise the wheels' grip, so the vehicle accelerates harder and turns more sharply.

- Raycast wheels: raise **Tire Friction** on each wheel. X is grip for accelerating and braking, Y for cornering.
- Physics wheels: raise **Friction** on the wheels' physical material.

Raycast tire friction well above the default `1.4` can make a vehicle shake while parked. Raise it in small steps and check the vehicle still sits still.



## Faster Steering

Under **Advanced Vehicle System → Steering** on the vehicle:

- Raise the **Steering Falloff Curve**'s values at higher speeds, to allow more steering at speed.
- Raise **Steering Speed**, so the wheels turn in faster.
- Raise **Steering Recenter Speed**, so the wheels straighten faster when the input is released.

See [Steering](https://overtorque-creations.com/Dev/Docs/#AVS/Configuration/Steering.md).



## Less Body Roll

Move the [Center of Mass](https://overtorque-creations.com/Dev/Docs/#AVS/Components/Center_Of_Mass.md) component down, toward wheel height. The lower it is, the less the body rolls in corners. At wheel height the body does not roll.

Below wheel height, the body leans into turns, the opposite way to a real car.



## More Engine Power

Raise **Max Torque** on each gear, for harder acceleration. See [Engine and Transmission](https://overtorque-creations.com/Dev/Docs/#AVS/Configuration/Engine_And_Transmission.md).

With physics wheels, too much torque makes the vehicle misbehave. Raise it in steps until it feels right.



## Lighter Vehicle

Lower **Mass (kg)** on `VehicleMesh`. A lighter vehicle accelerates and turns more quickly.

Too light, and the wheels lose traction and the vehicle slides. Under about `1000` kg, a car can also shake.



## Drifting

For raycast wheels, lower the Y value of **Tire Friction** on the rear wheels. The rear slides more in turns.

For handbrake turns, lower the rear wheels' sideways grip only while the handbrake is held:

1. Turn on **Is Handbrake Wheel** on the rear wheels only. Handbrake wheels lock completely.
2. When the handbrake changes, set the Y value of **Tire Friction** in each rear wheel's **Wheel Config**: lower while `GetHandbrakeApplied` is true, and back to its original value when it is false.

The handbrake state is replicated, so in multiplayer every machine can set the same friction from `GetHandbrakeApplied`. See [Changing Tire Grip at Runtime](https://overtorque-creations.com/Dev/Docs/#AVS/Components/Wheels_At_Runtime.md#changing-tire-grip-at-runtime).

Physics mode wheels take their grip from the wheel's physical material. Swap it with `SetPhysMaterialOverride` the same way.



## Scaled Down Vehicles

A scaled down vehicle, such as an RC car or a toy, is simulated at its real size, so it behaves differently from the full size version. Tune it at the size it is played at.



## Next Steps

- [Wheels](https://overtorque-creations.com/Dev/Docs/#AVS/Components/Wheels.md): tire friction and suspension.
- [Steering](https://overtorque-creations.com/Dev/Docs/#AVS/Configuration/Steering.md): the falloff curve and smoothing.
- [Physics](https://overtorque-creations.com/Dev/Docs/#AVS/Configuration/Physics.md): mass and center of mass.
