# Advanced Vehicle System Documentation

This is the documentation for AVS — setup guides, component references, and the settings that matter.

> **This documentation covers AVS 1.5 and newer.**
>
> If you are on **1.4 or earlier**, use the [Legacy documentation](https://overtorque-creations.com/Dev/Docs/#AVS_Legacy/README.md) instead. You can also switch between them with the asset selector in the sidebar.

If you're looking for what AVS is and what it does, that's on the [asset page](https://overtorque-creations.com/Dev/AdvancedVehicleSystem/). These pages assume you already have the plugin and want to build something with it.

## Where to start

If this is your first vehicle, work through the [Quick Start guide](https://overtorque-creations.com/Dev/Docs/#AVS/Tutorials/Quick_Start.md). It takes you from an empty project to something you can drive, and most of the other pages assume you've been through it.

Before you get too far, read [Important Information](https://overtorque-creations.com/Dev/Docs/#AVS/General/Important_Information.md). It covers a handful of behaviors that are easy to trip over and miserable to debug afterwards — wheel collision shapes, parent function calls, and a couple of skeletal mesh quirks.

Then take a look at the [recommended project settings](https://overtorque-creations.com/Dev/Docs/#AVS/General/Project_Settings.md). None of it is required, but physics wheel mode is noticeably more stable with those changes made.

## How these pages are organized

**General** covers engine and project level concerns — the recommended project settings, the gotchas in Important Information, and how to tune AVS toward arcade handling.

**Components** is the reference for the optional components you add to a vehicle: wheels, engine audio, lights, exhaust, and the hitch and trailer system.

**Tutorials** holds the start-to-finish walkthroughs — the Quick Start guide, plus a legacy version for Unreal older than 5.2.

**Skeletal Mesh** has two pages, and picking the right one matters:

- [Skeletal Wheels](https://overtorque-creations.com/Dev/Docs/#AVS/Tutorials/Skeletal_Mesh/Skeletal_Wheels.md) if your wheels live inside the skeletal mesh and you're using the "Connect to Bone" feature with physics wheels.
- [Skeletal Animation](https://overtorque-creations.com/Dev/Docs/#AVS/Tutorials/Skeletal_Mesh/Skeletal_Animation.md) if you have separated wheel meshes, which is what raycast wheels require.

## Elsewhere

- [Vehicle Quick Reference](https://docs.google.com/spreadsheets/d/1pkKlGzj8sBwvM5CaRuJEFr3M6ok60kjFWUP30DUFpR8) — every vehicle setting in one sheet
- [Discord](https://discord.gg/8YvWARS) — questions, help, and the fastest way to reach me
- [Trello](https://trello.com/b/4ny2jJrL/vehicle-system-plugin) — upcoming features and what I'm working on
- [FAB listing](https://www.fab.com/listings/e1457ad1-297b-4a70-aecb-5c6716d9494f)

## Something missing?

These docs are still a work in progress and a few pages are thinner than I'd like. If something is unclear, out of date, or just isn't here, let me know on the Discord server or by email. That's usually what decides which page I write next.
