# Engine Audio

`AVS_EngineAudio` drives a looping engine sound from the vehicle's RPM and throttle, and plays a start sound on ignition.



## Setting up Engine Sounds for your Vehicle

<!-- side-by-side:32 -->
The Vehicle System includes the `AVS_EngineAudio` component as an optional feature for quick vehicle setups. To use it add the component to your vehicle and position it where the audio source should be.
<!-- split -->
![Engine audio component positioned over the hood of a muscle car, with its Config section showing Engine Start Sound, Engine Sound, High RPM and Engine Attenuation](../Assets/Images/components-EngineAudio-01.png "The component's transform sets where the engine is heard from")
<!-- /side-by-side -->



## Engine Audio Settings

**Engine Start Sound** is a simple audio file, it will be blended into the Engine Sound Loop.

**Engine Sound** is a looping Sound Cue that takes in 2 parameters, _RPM_ and _Throttle_. These parameters are not required if your engine sound does not need to be driven dynamically.

**High RPM** is the maximum RPM that the Engine Sound Cue expects, its used to clamp the input parameters preventing audio artifacts.

**Engine Attenuation** is a [standard unreal audio attenuation file](https://docs.unrealengine.com/en-US/Engine/Audio/DistanceModelAttenuation/index.html)



## Setting up the Sound Cue

Here is an example of a setup sound cue. How to setup the cues is outside the scope of this documentation, as it is not part of the Vehicle System itself. The demo project does include many different engine setups you can use to learn from.

MetaSounds work here too, and are the better choice for new work. The `RPM` and `Throttle` parameters are set the same way.

![Sound Cue graph with Idle, Low RPM and High RPM sections, each feeding a Continuous Modulator into a final Crossfade by Param node](../Assets/Images/components-EngineAudio-02.png "Looping wave players crossfaded by RPM, then modulated by throttle")



## Runtime Control

The component follows the vehicle's engine state on its own, so you normally do not touch it. When you need to:

- `SetEngineRunning(bool)` — starts or stops the engine audio, including the start sound.
- `StopSound()` — stops playback outright.



## Engine Audio and Passive Mode

Engine audio stops ticking its RPM and throttle updates while the vehicle is passive, which is handled automatically.

See [Tick and Performance](https://overtorque-creations.com/Dev/Docs/#AVS/Advanced/Tick_And_Performance.md).
