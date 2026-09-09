# Engine Audio

## Setting up Engine Sounds for your Vehicle

<!-- side-by-side:32 -->
AVS includes the `Vehicle_EngineAudio` component as an optional feature for quick vehicle setups. Add the component to your vehicle and position it where the audio source should be.
<!-- split -->
![Engine audio component positioned over the hood of a muscle car, with its Config section showing Engine Start Sound, Engine Sound, High RPM and Engine Attenuation](../assets/images/components-EngineAudio-01.png "The component's transform sets where the engine is heard from")
<!-- /side-by-side -->

## Configuration

| Setting | What it does |
|---|---|
| **Engine Start Sound** | A simple audio file. It is blended into the engine sound loop. |
| **Engine Sound** | A looping Sound Cue taking two parameters, `RPM` and `Throttle`. Neither is required if your engine sound doesn't need to be driven dynamically. |
| **High RPM** | The maximum RPM the Engine Sound Cue expects. Used to clamp the input parameters and prevent audio artifacts. |
| **Engine Attenuation** | A [standard Unreal audio attenuation asset](https://dev.epicgames.com/documentation/en-us/unreal-engine/distance-based-sound-attenuation-in-unreal-engine). |

## Setting up the Sound Cue

Below is an example of a finished sound cue. Building cues is outside the scope of this documentation since it isn't part of AVS itself, but the demo project includes many different engine setups you can learn from.

![Sound Cue graph with Idle, Low RPM and High RPM sections, each feeding a Continuous Modulator into a final Crossfade by Param node](../assets/images/components-EngineAudio-02.png "Looping wave players crossfaded by RPM, then modulated by throttle")
