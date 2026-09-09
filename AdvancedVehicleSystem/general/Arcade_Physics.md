# Tuning AVS for Arcade-Style Vehicles

AVS is built for realistic vehicle physics, but with the right tuning it can be pushed toward an arcade feel. These are the adjustments that get you snappier, more forgiving handling.

## Increase Tire Friction

Use a higher friction value on your wheels. More grip gives you:

- more responsive acceleration
- sharper, more immediate turning

Where you set it depends on your wheel mode:

- **Physics Wheels** — set it in your Physics Material.
- **Raycast Wheels** — set the `TireFriction` variable (X = longitudinal, Y = lateral).

## Adjust Steering Responsiveness

Arcade handling benefits from faster, more exaggerated steering.

- **Steering Curve** — a mapping from air speed to max steering input. Raise the curve values to allow stronger steering input at higher speeds.
- **Steering Speed** — how quickly the vehicle responds to steering changes. Raise it for snappier response.
- **Steering Recenter Speed** — how fast steering returns to center with no input. Raising it makes the steering feel tighter and more controlled.

## Lower the Center of Mass

Use the included **Center of Mass** component to shift the vehicle's center of gravity downward.

- A center of mass slightly above wheel height gives a stable but responsive feel.
- Placing it below the wheels reduces flipping, but the vehicle may lean the wrong way when turning.

## Boost Engine Power

Raise the **Max Torque** values for each gear for more aggressive acceleration. Arcade vehicles usually benefit from immediate throttle response.

> With physics wheels it is possible to set Max Torque too high. There's no single correct number — play with it until it feels right.

## Reduce Vehicle Mass

Lower the **Mass** value on the `VehicleMesh` component to make the vehicle feel lighter and more nimble. This amplifies acceleration and makes turning more responsive.

> Going too low affects traction and makes the vehicle slide.

## Add Drifty Handling

When using raycast wheels, reduce lateral friction in the wheel config to allow more sliding through turns.
