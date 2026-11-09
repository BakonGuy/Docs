# Center of Mass

Where a vehicle's center of mass sits changes how much its body rolls in corners. `AVS_CenterOfMass` is a component you place to set it. This page sets the center of mass for the car from the [Quick Start](https://overtorque-creations.com/Dev/Docs/#AVS/Getting_Started/Quick_Start.md).



## How the Center of Mass Is Chosen

Without a Center of Mass component, the vehicle uses Unreal's own center of mass, calculated from the vehicle mesh's collision.

The component is a marker with no settings. When it is attached directly to `VehicleMesh`, its location becomes the center of mass. One nested under any other component is ignored. If there is more than one, only the first found is used.

Two functions can change it from code:

1. `SetExactCenterOfMass(Location)` sets the center of mass relative to the vehicle mesh's pivot, and overrides the component.
2. `SetCenterOfMassOffset(Offset)` adds an offset on top of whichever of the above applies. With neither the component nor `SetExactCenterOfMass`, the offset is measured from the vehicle mesh's pivot.



## Adding a Center of Mass Component

<!-- side-by-side:57 -->
1. Add an `AVS_CenterOfMass` component directly under `VehicleMesh`.
2. Move it to the middle of the car, between the axles, slightly above wheel height.
3. Turn on **Visualize Center of Mass** under **Advanced Vehicle System → Debug**. It draws the center of mass the physics body is actually using.
4. Drive the car, then adjust the height in small steps.
<!-- split -->
![Center of mass component positioned inside a vehicle chassis with the visualizer enabled](../Assets/Images/_placeholder.png "Position the marker, then check it with the visualizer")
<!-- /side-by-side -->



## Center of Mass Height

The higher the center of mass is above the wheels, the more the body rolls in corners. At wheel height it does not roll at all. Below the wheels, the body leans into the turn, which a real vehicle does not do.

Small changes in height make a large difference.



## Changing the Center of Mass at Runtime

Each call to `SetCenterOfMassOffset` replaces the previous offset, so calling it repeatedly with the same value does not move the center of mass further. It can shift the center of mass as cargo is loaded.

`GetExactCenterOfMass` returns the center of mass the physics body is using, in vehicle mesh space, and whether that value is valid. It is not valid until the physics body exists, so it cannot be read during construction.



## Fixing Center of Mass Problems

### Center of Mass Component Has No Effect

Check that it is attached directly to `VehicleMesh`. Before AVS 1.5.2, the component and the visualizer did not work correctly.

### Car Leans Into Corners

The center of mass is below the wheels. Raise it above wheel height.



## Next Steps

- [Physics](https://overtorque-creations.com/Dev/Docs/#AVS/Configuration/Physics.md): mass, drag and moving a vehicle.
- [Component Settings](https://overtorque-creations.com/Dev/Docs/#AVS/Reference/Component_Settings.md): every component setting.
