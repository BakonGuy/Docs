# Recording a Vehicle with Take Recorder

Recording a vehicle while you drive it, then playing it back in Sequencer.



## Add Your Own Wheel Meshes

A wheel component with no children creates its own wheel mesh when the game starts. Take Recorder does not record those meshes correctly. Add the wheel meshes in the vehicle Blueprint instead:

1. Under each wheel component, add a Static Mesh component with the wheel mesh, or a Sphere Collision.
2. Make it the wheel component's first child. AVS uses the first child as the wheel mesh, and does not create one of its own.

**Wheel Static Mesh** is not used once a wheel has a child. See [Adding a Wheel](https://overtorque-creations.com/Dev/Docs/#AVS/Components/Wheels.md#adding-a-wheel).



## Recording

Record the vehicle with Take Recorder as normal while you drive it.



## Playing Back

Turn on **Cinematic Playback** under **Advanced Vehicle System → General** on the recorded vehicle. It only takes effect when no player controls the vehicle, so it changes nothing while you record. During playback it releases the wheel locks that would hold the wheels still. See [Cinematic Playback](https://overtorque-creations.com/Dev/Docs/#AVS/Configuration/General.md#cinematic-playback).

A vehicle moved by Sequencer is not possessed, so it can go passive. See [Tick and Performance](https://overtorque-creations.com/Dev/Docs/#AVS/Advanced/Tick_And_Performance.md).



## Next Steps

- [General Vehicle Settings](https://overtorque-creations.com/Dev/Docs/#AVS/Configuration/General.md): what **Cinematic Playback** changes.
- [Wheels](https://overtorque-creations.com/Dev/Docs/#AVS/Components/Wheels.md): wheel meshes and wheel modes.
