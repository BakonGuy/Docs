# Tuning AVS for Arcade-Style Vehicles

AVS is designed for realistic vehicle physics, but with the right tuning, it can be adapted to feel more arcade-like. Below are some common adjustments to help achieve snappier, more forgiving vehicle behavior.

## Increase Tire Friction

Use a **higher friction** value on your wheels. This gives the vehicle more grip, which:

- Increases acceleration responsiveness
- Makes turning sharper and more immediate

> _Note:_ When using Physics Wheels, this is done in your Physics Material. When using Raycast Wheels, this is done by setting the "TireFriction" variable ( X = Longitudinal, Y = Lateral )

## Adjust Steering Responsiveness

Arcade-style handling benefits from faster, more exaggerated steering.

**Steering Curve**

This is a mapping from Air Speed → Max Steering Input. Increase the curve values to allow stronger steering input at higher speeds.

**Steering Speed**

Controls how quickly the vehicle responds to steering changes. Increase this for snappier response.

**Steering Recenter Speed**

Controls how fast the steering returns to center when no input is given. Raising this can make the steering feel tighter and more controlled.

## Lower the Center of Mass

Use the included **Center of Mass** component to shift the vehicle's center of gravity downward:

- A center of mass just slightly above wheel height provides a stable but responsive feel.
- Placing the center below the wheels can reduce flipping, but may cause the vehicle to lean in the wrong direction when turning.

## Boost Engine Power

- Raise the **Max Torque** values for each gear to deliver more aggressive acceleration. Arcade-style vehicles typically benefit from immediate throttle response.
- _It is possible to raise the MaxTorque too high when using physics wheels. You will have to find the value that is right for you by playing around._

## Reduce Vehicle Mass

- Lower the Mass value on the VehicleMesh component to make the vehicle feel lighter and more nimble. This can amplify acceleration and make turning more responsive.
- _Going too low will effect traction and make your vehicle slide_

## Add Drifty Handling

When using raycast wheels, you can reduce lateral friction in the wheel config to allow more sliding during turns
