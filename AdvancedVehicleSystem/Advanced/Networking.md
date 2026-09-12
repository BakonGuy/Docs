# Networking

AVS vehicles replicate out of the box and the defaults work for most projects. You need this page when tuning for a specific connection profile, or when something is desyncing.



## Basic Understanding

Physics vehicles are difficult to replicate because the server and clients never simulate identically. Instead of keeping every simulation in step, AVS replicates vehicle state and has clients follow it.

Two things follow from that, and they cover most of this page:

**The vehicle's network type** decides how a client follows the server — buffered and smooth, or predicted and immediate.

**Input functions only work from the owning client or the server.** Calling them anywhere else does nothing and reports no error.



## Network Type: History Interpolation

This is the default.

Clients buffer incoming states and play them back slightly behind the server. Smooth, stable, and forgiving of jitter.

The cost is latency, since remote vehicles are always a little behind reality.



## Network Type: Client Prediction

Clients simulate ahead and correct against server state.

More immediate, at the cost of correction artifacts when a prediction turns out wrong.

Use it when remote vehicle responsiveness matters more than smoothness.



## Calling Input Functions on Client and Server

This is the most common cause of multiplayer problems with AVS.

<!-- side-by-side:57 -->
Input functions are replicated, but **where you call them matters**. In single player, call them from anywhere. In multiplayer they only work from the **owning client** — the client possessing the pawn — or from the server.

Calling `SetThrottleInput` on a non-owning client does nothing, and produces no error.

`isOwningClient` is the check to make before calling input functions from anything that might run elsewhere.

`SetLocalEngineRunning` is the deliberate exception, for local-only cosmetic changes.
<!-- split -->
![Blueprint checking isOwningClient before calling a vehicle input function](../Assets/Images/_placeholder.png "Check ownership before calling input functions")
<!-- /side-by-side -->

```cpp
void AMyVehicle::ApplyThrottle(float Value)
{
	// Input functions do nothing on a client that does not own this pawn.
	if( !isOwningClient() ) return;

	SetThrottleInput(Value);
}
```

The same applies to hitching. `Hitch` and `HitchToOverlapped` must come from the owning client or the server.



## Core Network Settings

Settings under **Advanced Vehicle System → Network**. These three are usually left alone.

| Setting | Default | What it does |
|---|---|---|
| **Movement Replication** | `true` | Whether movement replicates at all |
| **Sync Location** | `true` | Replicate position |
| **Sync Rotation** | `true` | Replicate rotation |



## Advanced Network Settings

| Setting | Default | Raise it to | Lower it to |
|---|---|---|---|
| **Net Send Rate** | `0.05` s | Save bandwidth | Update more often |
| **Net Time Behind** | `0.15` s | Absorb jitter on poor connections | Tighten responsiveness |
| **Net Lerp Start** | `0.35` | Tolerate more drift before correcting | Correct sooner |
| **Net Position Tolerance** | `0.1` | Ignore small differences | Correct more precisely |
| **Net Smoothing** | `10.0` | Correct faster, more visibly | Correct gently |



## Net Time Behind

**Net Time Behind** is the main trade on this page.

More absorbs packet loss and jitter, at the cost of remote vehicles lagging further behind reality.

Less is tighter, but makes bad connections obvious.



## Net Send Rate

Do not lower **Net Send Rate** without measuring.

With many vehicles it is the setting most likely to cost you real bandwidth.



## Network Rest State

Vehicles that come to a stop stop sending updates, rather than repeatedly sending an unchanged position.

**Rest Velocity Threshold** under **Advanced Vehicle System → Physics** decides when that kicks in, and it is the same threshold passive mode uses.

> If parked vehicles are drifting out of sync on clients, that threshold is too high.



## Trailer Networking

Trailer sync is automatic once a hitch connection is made. Reworked in 1.5 with an adaptive send rate — trailers update more when they need to and back off while resting.

The current implementation supports a single tow hitch and trailer hitch pair. Hitch chains are not yet supported, so a truck towing a trailer towing another trailer is outside what the networking handles today.



## Disabling Movement Replication

`SetMovementReplicationEnabled` disables replication temporarily. 
It does not change the configured **Movement Replication** setting.

Use it when something else takes over the vehicle's movement — a cinematic, attaching it to another actor, or your own movement code.

Turning it off for the duration stops AVS and your system fighting over the transform, and re-enabling restores normal behavior.
