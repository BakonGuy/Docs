# Preparing Vehicle Meshes

What the body and wheel meshes need before they go into an AVS vehicle. Most of it is set in the modeling program or at import, and is harder to fix once the vehicle is built.



## Vehicle Body Mesh

The body is the mesh on `VehicleMesh`, the vehicle's root component.

- Face the front of the vehicle along +X, with +Z up. AVS uses the vehicle mesh's X axis as forward, and each wheel drives along its own X axis. `VehicleMesh` is the root, so its rotation cannot be corrected in the Blueprint. Fix the facing in the mesh asset, in the modeling program or with the import rotation.
- Model at real size. Unreal units are centimeters, so a car 4.5 m long is 450 units. AVS distances such as **Wheel Radius** and **Spring Length** are in centimeters too.
- The pivot can be anywhere. The center of mass comes from the collision, and each wheel sits where its component is placed.
- Give the body simple collision: a few boxes or convex hulls in the static mesh asset. Avoid complex collision. Its triangles behave as planes. The collision shape also sets the vehicle's center of mass and how it rotates. See [How the Collision Shape Affects Handling](https://overtorque-creations.com/Dev/Docs/#AVS/Configuration/Physics.md#how-the-collision-shape-affects-handling).

Parts that move on their own, such as doors or a hood on a physics constraint, need to be separate meshes. Their constraints go in the vehicle Blueprint. See [Important Information](https://overtorque-creations.com/Dev/Docs/#AVS/Getting_Started/Important_Information.md#keep-physics-constraints-out-of-physics-assets).



## Wheel Meshes

- Put the pivot at the center of the wheel, on the axle. AVS places the wheel mesh at the wheel component's position and spins it around the mesh's own Y axis. A pivot away from the axle makes the wheel wobble as it turns.
- Point the axle along Y. The wheel stands upright, with its round faces toward +Y and -Y.
- Keep the mesh to the tire and rim. **Auto Wheel Radius** measures the radius as half the mesh's height. A part that sticks out past the tire makes the measured radius too large. To use a mesh like that, turn off **Auto Wheel Radius** and enter the tire's radius in **Wheel Radius**.
- One wheel mesh can be used on both sides. A wheel component rotated to face the other way needs **Invert Torque**. See [Making Wheels Drive, Steer and Brake](https://overtorque-creations.com/Dev/Docs/#AVS/Components/Wheels.md#making-wheels-drive-steer-and-brake).

The wheel mesh's collision only matters for physics mode wheels, and for raycast wheels once they are detached. Physics mode wheels need a simple sphere collision to stay stable at speed. See [Physics Wheels Need Sphere Collision](https://overtorque-creations.com/Dev/Docs/#AVS/Getting_Started/Important_Information.md#physics-wheels-need-sphere-collision).



## Skeletal Mesh Vehicles

A vehicle whose wheels are bones in one skeletal mesh is set up differently. See [Skeletal Mesh Wheels](https://overtorque-creations.com/Dev/Docs/#AVS/Skeletal_Mesh/Skeletal_Wheels.md) and [Skeletal Mesh AnimBP](https://overtorque-creations.com/Dev/Docs/#AVS/Skeletal_Mesh/Skeletal_Animation.md).



## Next Steps

- [Quick Start](https://overtorque-creations.com/Dev/Docs/#AVS/Getting_Started/Quick_Start.md): build a drivable car from these meshes.
- [Wheels](https://overtorque-creations.com/Dev/Docs/#AVS/Components/Wheels.md): placing wheel components and choosing a wheel mode.
- [Physics](https://overtorque-creations.com/Dev/Docs/#AVS/Configuration/Physics.md): mass, center of mass and the collision shape.
