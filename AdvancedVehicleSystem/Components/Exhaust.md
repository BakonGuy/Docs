# Exhaust

`AVS_ExhaustController` drives exhaust smoke particles from the vehicle's RPM and throttle. This page adds exhaust smoke to the car from the [Quick Start](https://overtorque-creations.com/Dev/Docs/#AVS/Getting_Started/Quick_Start.md).

The component is optional. It is listed in [Component Settings](https://overtorque-creations.com/Dev/Docs/#AVS/Reference/Component_Settings.md).



## How the Exhaust Controller Works

The controller has no settings. It finds every Cascade particle system component attached beneath it and drives two vector parameters on each:

- **SmokeVelocity**: X goes from `40` to `150` as RPM rises from `0` to `10000`.
- **SmokeSize**: X goes from `30` to `80` as throttle goes from `0` to `1`.

The particles are on while the engine is running and the vehicle is slower than `40` in its **Speed Units**. Otherwise they are off.

Niagara systems are not driven by the controller.



## Adding Exhaust Smoke

<!-- side-by-side:40 -->
1. Add an `AVS_ExhaustController` component to the car, at the tailpipe.
2. Add a **Cascade Particle System** component under it, and set its template to `P_VehicleExhaust`, the exhaust smoke included with the plugin.
3. For two tailpipes, add a second particle system component under the same controller.
<!-- split -->
![Exhaust controller component on the rear of a muscle car with two particle systems attached, positioned at the tailpipe](../Assets/Images/Components/Exhaust/components-ExhaustController-01.png "Particle systems attached under the controller, at the tailpipes")
<!-- /side-by-side -->

To make your own exhaust smoke, start from a copy of `P_VehicleExhaust`, or read **SmokeVelocity** and **SmokeSize** as vector parameters in your own Cascade system.



## Next Steps

- [Engine Audio](https://overtorque-creations.com/Dev/Docs/#AVS/Components/Engine_Audio.md): engine sound driven by RPM and throttle.
- [Tick and Performance](https://overtorque-creations.com/Dev/Docs/#AVS/Advanced/Tick_And_Performance.md): exhaust particles are the largest cost in the demo project.
