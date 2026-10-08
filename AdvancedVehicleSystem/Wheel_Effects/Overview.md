# Wheel Effects

Wheel effects are the sounds and particles a wheel makes against the ground: tire smoke, skid marks, rolling noise, impacts and brake squeal. This page sets up tire smoke, skid marks and rolling sound on a rear wheel drive car.

Every effect setting is listed in [Wheel Effect Settings](https://overtorque-creations.com/Dev/Docs/#AVS/Reference/Wheel_Effect_Settings.md). Writing your own effect is covered in [Custom Effects](https://overtorque-creations.com/Dev/Docs/#AVS/Wheel_Effects/Custom_Effects.md).

Wheel effects are new in 1.5 and replace the SurfaceEffects system.



## How Wheel Effects Work

An effect watches one wheel. Every tick it decides how strongly it should play, from `0` to `1`, and drives a sound, a particle system, or both at that intensity.

Effects are set up once on the vehicle and copied to every wheel. Each wheel gets its own copy, so four wheels can smoke independently.

AVS ships with four effects:

| Effect | Plays when |
|---|---|
| **Roll** | The wheel is rolling along a surface. |
| **Skid / Slip** | The wheel is sliding or spinning. |
| **Bump / Impact** | The wheel hits something, such as a landing or a curb. |
| **Brake Squeal** | The brakes are working hard. |



## Global or Surface Effects

The vehicle has three places to put effects.

<!-- side-by-side:50 -->
**Global Wheel Effects** play on every surface. Rolling noise and brake squeal go here, since they sound the same on any ground.
<!-- split -->
**Effects For Default Surface** play on any surface without its own entry. **Surface Effects** hold effects for specific surfaces, such as dirt or gravel.
<!-- /side-by-side -->

Smoke and skid marks change with the surface, so for the example car they go in **Effects For Default Surface**.



## Adding Tire Smoke

<!-- side-by-side:57 -->
1. Select the vehicle and open **Advanced Vehicle System → Wheel Effects**.
2. Add an entry to **Effects For Default Surface** and set its type to **Skid / Slip**.
3. Set **Niagara System** to your smoke.
4. Turn on **Attach To Wheel**, so the smoke follows the wheel.
5. Play, and slide the car.
<!-- split -->
![Wheel Effects section on a vehicle with an effects array expanded, showing an effect type selected](../Assets/Images/_placeholder.png "Effects are set up once on the vehicle")
<!-- /side-by-side -->

By default the effect starts at `500` cm/s of sideways speed and reaches full strength at `1500` cm/s. To make smoke appear sooner, lower **Min Skid Speed**.

If your Niagara system has a parameter for how dense the smoke is, put its name in **Niagara Intensity Param**. AVS sets it from `0` to `1` as the slide gets stronger. **Parameter Intensity Range** rescales that if your system expects other numbers, such as `0` to `100`.



## Adding Skid Marks

Skid marks are a second **Skid / Slip** effect in the same slot, with a different setup:

1. Add another **Skid / Slip** entry to **Effects For Default Surface**.
2. Set **Niagara System** to your skid mark system.
3. Leave **Attach To Wheel** off, so the marks stay on the road as the car drives away.
4. Leave **Use Contact Point** on, so they appear where the tire touches the ground.

Separate effects can have separate thresholds. The marks can start at a lower speed than the smoke.



## Choosing a Slip Source

**Slip Source** decides what a **Skid / Slip** effect responds to.

The default, `Skid Speed (Classic)`, responds to the wheel sliding sideways, or to any movement while the wheel is locked. It works in both wheel modes.

The other three respond to tire slip, which only raycast wheels calculate:

- `X Slip` responds to wheelspin and lockup. Use it for burnout smoke.
- `Y Slip` responds to sliding sideways. Use it for cornering smoke.
- `Combined Slip` responds to both.

These use **Min Slip** and **Full Slip** for their thresholds. On physics wheels they never play.



## Different Effects on Different Surfaces

To give the car dust on dirt:

1. Add an entry to **Surface Effects** and set its key to your dirt physical surface.
2. Add a **Skid / Slip** effect to that entry with your dust particles.

On dirt, the wheel now uses the dirt entry. On any surface without an entry, it uses **Effects For Default Surface**.

> Do not use `Default` as a key in **Surface Effects**. AVS never reads it. Set up the default surface in **Effects For Default Surface**.



## Adding Rolling Sound

1. Add a **Roll** effect to **Global Wheel Effects**.
2. Set **Sound** to a looping tire sound.
3. Set **Random Start Time Max** to the length of the loop.

Without step 3, all four wheels start the loop at the same point and play in sync.

Rolling sound starts at `100` cm/s and reaches full volume at `2000` cm/s. To vary pitch or volume with speed, put a parameter name in **Audio Intensity Param**.



## Changing Effects for One Wheel

Every wheel uses the vehicle's effects unless it overrides them. Each wheel has three overrides, matching the vehicle's three slots: **Override Global Effects**, **Surface Effect Overrides** and **Effects For Default Surface Override**.

An override replaces the vehicle's effects for that slot. It does not add to them.

To silence a wheel's global effects, turn on **Override Global Effects** and leave the list empty. To silence one surface on one wheel, add that surface to **Surface Effect Overrides** with an empty list.

> An empty **Effects For Default Surface Override** does not silence the wheel. It falls back to the vehicle's default surface effects. There is currently no way for one wheel to opt out of those.



## When Effects Restart

Each wheel's surface effects stop when the wheel leaves the ground, and start fresh when it lands or moves onto a different surface.

Global effects keep running, unless an effect requires contact. The built-in Roll, Skid / Slip and Bump / Impact effects all require contact, so they also stop in the air and start fresh on landing.

`ClearAllWheelEffects()` on the vehicle restarts every wheel's effects. Call it after changing effects at runtime.



## Fixing Wheel Effect Problems

### Skid Effect Never Plays on Physics Wheels

The effect is using `X Slip`, `Y Slip` or `Combined Slip`. Those need raycast wheels. Use `Skid Speed (Classic)`.

### All Four Wheels Sound Like One

Set **Random Start Time Max** to the length of the looping sound.

### Skid Marks Follow the Car

Turn off **Attach To Wheel** on the skid mark effect.



## Next Steps

- [Custom Effects](https://overtorque-creations.com/Dev/Docs/#AVS/Wheel_Effects/Custom_Effects.md): writing your own effect in Blueprint or C++.
- [Wheel Effect Settings](https://overtorque-creations.com/Dev/Docs/#AVS/Reference/Wheel_Effect_Settings.md): every effect setting, with defaults.
