# Custom Wheel Effects

How to write your own wheel effect. This page builds one in C++: an effect that plays a sound and smoke when a wheel slides sideways. The same steps work in Blueprint.

The built-in effects are written the same way, with the same functions. Every effect function is listed in [Wheel Effect Settings](https://overtorque-creations.com/Dev/Docs/#AVS/Reference/Wheel_Effect_Settings.md).



## How an Effect Runs

An effect is a class based on `AVS_WheelEffect`. The one you add to the vehicle is a template. Each wheel makes its own copy of it, then calls four functions on that copy:

1. **Requirements Met** decides whether the effect should exist on this wheel at all. Return false and the copy is thrown away.
2. **Init Effect** runs once, when the copy is created.
3. **Tick Effect** runs every tick while the effect exists. This is where it decides how strongly to play.
4. **Clear Effect** runs when the effect is removed.

Each function receives the wheel it belongs to.



## Step 1: Create the Effect Class

In Blueprint, create a new Blueprint class with `AVS_WheelEffect` as its parent.

In C++, inherit from `UAVS_WheelEffect` with the same class specifiers the built-in effects use. **DisplayName** is the name shown in the effect type list.

```cpp
UCLASS(BlueprintType, Blueprintable, EditInlineNew, DefaultToInstanced, meta=(DisplayName="Lateral Slip"))
class ULateralSlipEffect : public UAVS_WheelEffect
{
	GENERATED_BODY()
```



## Step 2: Add Settings for the Thresholds

The built-in effects all expose a start value and a full value. Following the same pattern keeps your effect familiar to anyone who has set up the built-in ones.

```cpp
public:
	// Slip where the effect starts, and where it reaches full intensity.
	UPROPERTY(EditAnywhere, BlueprintReadWrite, Category="Conditions", meta=(ClampMin="0.0"))
	float MinSlip = 0.5f;

	UPROPERTY(EditAnywhere, BlueprintReadWrite, Category="Conditions", meta=(ClampMin="0.0"))
	float FullSlip = 1.5f;
```



## Step 3: Add an Output

`FAVS_WheelEffectOutput` is the sound and particle block the built-in effects use. Adding one gives your effect the same placement, audio and particle settings, and lets you drive it with `StartOutput`, `UpdateOutput` and `StopOutput`.

```cpp
	UPROPERTY(EditAnywhere, BlueprintReadWrite, Category="", meta=(ShowOnlyInnerProperties))
	FAVS_WheelEffectOutput Output;
```

Write your own output only if the standard block cannot do what you need. Brake Squeal is the built-in example of an effect with its own audio output.



## Step 4: Decide When the Effect Exists

Override **Requirements Met** to skip wheels the effect does not apply to. This effect has nothing to play without a sound or particles.

```cpp
bool ULateralSlipEffect::RequirementsMet_Implementation(UAVS_Wheel* Wheel)
{
	return IsValid(Output.Sound) || IsValid(Output.NiagaraSystem);
}
```

For effects in **Global Wheel Effects**, AVS checks this again whenever the wheel lands, leaves the ground or changes surface. An effect that stops passing is removed, and one that starts passing is created.



## Step 5: Drive the Output Every Tick

**Tick Effect** reads the wheel, works out an intensity from `0` to `1`, and starts or updates the output.

```cpp
void ULateralSlipEffect::TickEffect_Implementation(UAVS_Wheel* Wheel, float DeltaTime)
{
	if( !IsValid(Wheel) || !Wheel->GetHasContact() )
	{
		StopOutput();
		return;
	}

	// Slip2D is only calculated for raycast wheels. A physics wheel reports zero here.
	const float LateralSlip = FMath::Abs(Wheel->WheelData.Slip2D.Y);
	if( LateralSlip < MinSlip )
	{
		StopOutput();
		return;
	}

	const float Intensity = FMath::Clamp(FMath::GetRangePct(MinSlip, FullSlip, LateralSlip), 0.0f, 1.0f);

	if( IsOutputActive() ) UpdateOutput(Output, Intensity);
	else StartOutput(Output, Intensity, true); // true = looping, false = one-shot
}
```

The values an effect most often reads:

- **Slip** and **Slip2D** on the wheel's **Wheel Data**, for skids and wheelspin. Raycast wheels only.
- **Suspension Force**, for how much load the wheel is carrying.
- **Contact Normal Speed**, or `GetContactImpactSpeed()`, for impacts.
- `GetContactStartedThisFrame()`, which is true on the frame the wheel lands.
- `GetContactSurfaceType()`, for the surface under the wheel.



## Step 6: Clean Up in Clear Effect

When an effect is removed, AVS calls **Clear Effect** and then stops the standard output for you. An effect that only uses `FAVS_WheelEffectOutput` does not need to override **Clear Effect** at all.

Override it when your effect spawned something itself. Deactivate particle systems instead of destroying them, so particles already in the air finish and fade. `StopOutput` does the same for the standard output.



## Step 7: Add It to a Vehicle

Compile, then add the effect to one of the vehicle's effect slots the same way as a built-in effect. Your class appears in the effect type list under its **DisplayName**.



## Keeping State in an Effect

Each wheel has its own copy of the effect, so a copy can safely hold state such as a smoothed value or a timer.

A copy starts with the template's values. Set state up in **Init Effect**, which runs once for every new copy. In C++, marking state properties `DuplicateTransient` also stops the template's values being copied.

Copies do not last forever:

- **Surface effects** are removed when the wheel leaves the ground, and created fresh when it lands or changes surface.
- **Global effects** last as long as **Requirements Met** keeps passing.

State that has to survive driving from one surface onto another needs a global effect whose **Requirements Met** does not depend on contact or surface.



## The Complete Effect

```cpp
UCLASS(BlueprintType, Blueprintable, EditInlineNew, DefaultToInstanced, meta=(DisplayName="Lateral Slip"))
class ULateralSlipEffect : public UAVS_WheelEffect
{
	GENERATED_BODY()

public:
	UPROPERTY(EditAnywhere, BlueprintReadWrite, Category="Conditions", meta=(ClampMin="0.0"))
	float MinSlip = 0.5f;

	UPROPERTY(EditAnywhere, BlueprintReadWrite, Category="Conditions", meta=(ClampMin="0.0"))
	float FullSlip = 1.5f;

	UPROPERTY(EditAnywhere, BlueprintReadWrite, Category="", meta=(ShowOnlyInnerProperties))
	FAVS_WheelEffectOutput Output;

	virtual bool RequirementsMet_Implementation(UAVS_Wheel* Wheel) override;
	virtual void TickEffect_Implementation(UAVS_Wheel* Wheel, float DeltaTime) override;
};
```

```cpp
bool ULateralSlipEffect::RequirementsMet_Implementation(UAVS_Wheel* Wheel)
{
	return IsValid(Output.Sound) || IsValid(Output.NiagaraSystem);
}

void ULateralSlipEffect::TickEffect_Implementation(UAVS_Wheel* Wheel, float DeltaTime)
{
	if( !IsValid(Wheel) || !Wheel->GetHasContact() )
	{
		StopOutput();
		return;
	}

	// Slip2D is only calculated for raycast wheels. A physics wheel reports zero here.
	const float LateralSlip = FMath::Abs(Wheel->WheelData.Slip2D.Y);
	if( LateralSlip < MinSlip )
	{
		StopOutput();
		return;
	}

	const float Intensity = FMath::Clamp(FMath::GetRangePct(MinSlip, FullSlip, LateralSlip), 0.0f, 1.0f);

	if( IsOutputActive() ) UpdateOutput(Output, Intensity);
	else StartOutput(Output, Intensity, true);
}
```

In Blueprint, override the same events with the same logic. `StartOutput`, `UpdateOutput` and `StopOutput` are available there too.



## Next Steps

- [Wheel Effects](https://overtorque-creations.com/Dev/Docs/#AVS/Wheel_Effects/Overview.md): setting effects up on a vehicle.
- [Wheel Effect Settings](https://overtorque-creations.com/Dev/Docs/#AVS/Reference/Wheel_Effect_Settings.md): every effect setting and function.
