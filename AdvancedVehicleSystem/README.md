# Advanced Vehicle System Documentation

This is the documentation for AVS: setup guides, component pages, and a reference for every setting.

> **This documentation covers AVS 1.5 and newer.**
>
> If you are on **1.4 or earlier**, use the [Legacy documentation](https://overtorque-creations.com/Dev/Docs/#AVS_Legacy/README.md) instead. You can also switch between them with the asset selector in the sidebar.

If you're looking for what AVS is and what it does, that's on the [asset page](https://overtorque-creations.com/Dev/AdvancedVehicleSystem/). These pages assume you already have the plugin and want to build something with it.



## Recommended Reading Order

If this is your first vehicle, work through the [Quick Start guide](https://overtorque-creations.com/Dev/Docs/#AVS/Getting_Started/Quick_Start.md). It takes you from an empty project to something you can drive, and most of the other pages assume you've been through it.

Before you get too far, read [Important Information](https://overtorque-creations.com/Dev/Docs/#AVS/Getting_Started/Important_Information.md). It covers wheel collision shapes, skeletal mesh collisions and constraints, and what passive mode does to your Tick event.

Then take a look at the [recommended project settings](https://overtorque-creations.com/Dev/Docs/#AVS/Getting_Started/Project_Settings.md). None of it is required, but physics wheel mode is noticeably more stable with those changes made.



## Coming from 1.4

AVS 1.5 rewrote the entire plugin in C++. Most of it carries over through Unreal's redirect system. The differences that affect existing projects:

- **Blueprint events no longer need a parent call.** Older tutorials will tell you to add them; you don't need to.
- **SurfaceEffects has been replaced** by the modular [Wheel Effects](https://overtorque-creations.com/Dev/Docs/#AVS/Wheel_Effects/Overview.md) system.
- **Wheelspin now affects the vehicle** rather than being a visual effect.
- **Components are named `AVS_*`**, such as `AVS_Wheel` and `AVS_Hitch`.

Back up your project before updating and give the upgrade a real test before committing to it.



## Migrating a Vehicle Between Projects

> **Enable the AVS plugin in the destination project before migrating anything into it.**

If the plugin is not enabled first, Unreal shows a series of popups about missing classes, and accepting them **corrupts the assets being migrated**. There is no repair for it afterwards. Migrate again into a correctly prepared project.

With the plugin enabled first, a migration from the demo project should produce exactly one error, in `HUD_Demo`, about a "Project Version" node that cannot be migrated. That one is expected and harmless.



## Getting a Version FAB Does Not Have Yet

New AVS releases reach FAB for the newest engine version first, to limit the impact on established projects while the release settles.

The GitHub repository usually carries builds for other engine versions before FAB does. Access is arranged through the Discord server.



## Documentation Structure

Most pages teach one feature by setting it up on an example vehicle, then cover tuning and known problems. They only explain the settings the example needs.

The Reference section lists every setting and function, with defaults and units, in the order the details panel shows them. Each teaching page links to its reference page.

- Getting Started: the Quick Start, and the two pages to read before going far into a project.
- Configuration: settings for the vehicle as a whole: general settings, engine and transmission, steering, and physics.
- Components: everything you add to a vehicle: wheels, the camera pivot, center of mass, the hitch and trailers, lights, engine audio and exhaust. Also attaching your own objects to a vehicle.
- Wheel Effects: tire smoke, skid marks and other wheel sounds and particles, and writing your own effects.
- Skeletal Mesh: using a skeletal mesh for the vehicle's visuals.
- Guides: arcade handling, and dividing gear torque across driving wheels.
- Advanced: networking, tick and performance, debugging, and C++ overrides for the drivetrain and physics forces.
- Reference: [Vehicle Settings](https://overtorque-creations.com/Dev/Docs/#AVS/Reference/Vehicle_Settings.md), [Wheel Settings](https://overtorque-creations.com/Dev/Docs/#AVS/Reference/Wheel_Settings.md), [Wheel Effect Settings](https://overtorque-creations.com/Dev/Docs/#AVS/Reference/Wheel_Effect_Settings.md), [Camera Pivot Settings](https://overtorque-creations.com/Dev/Docs/#AVS/Reference/Camera_Pivot_Settings.md), [Component Settings](https://overtorque-creations.com/Dev/Docs/#AVS/Reference/Component_Settings.md) and [Common Functions](https://overtorque-creations.com/Dev/Docs/#AVS/Reference/Common_Functions.md).

The Skeletal Mesh section has two pages:

- [Skeletal Wheels](https://overtorque-creations.com/Dev/Docs/#AVS/Skeletal_Mesh/Skeletal_Wheels.md) if your wheels live inside the skeletal mesh and you're using the "Connect to Bone" feature with physics wheels.
- [Skeletal Animation](https://overtorque-creations.com/Dev/Docs/#AVS/Skeletal_Mesh/Skeletal_Animation.md) if you have separated wheel meshes, which is what raycast wheels require.



## Links and Resources

- [Vehicle Quick Reference](https://docs.google.com/spreadsheets/d/1pkKlGzj8sBwvM5CaRuJEFr3M6ok60kjFWUP30DUFpR8): every vehicle setting in one sheet
- [Discord](https://discord.gg/8YvWARS): questions, help, and the fastest way to reach me
- [Trello](https://trello.com/b/4ny2jJrL/vehicle-system-plugin): upcoming features and what I'm working on
- [FAB listing](https://www.fab.com/listings/e1457ad1-297b-4a70-aecb-5c6716d9494f)



## Feedback and Support

These docs are still a work in progress and a few pages are thinner than I'd like. If something is unclear, out of date, or just isn't here, let me know on the Discord server or by email. That's usually what decides which page I write next.
