# Animation

The documented rule: **do not keyframe the main motion**. Derive it from physical quantities — wheel rate from speed, crank rate from the wheel, body bob from crank phase, lean from steer times speed. Smooth every value with exponential damping or a spring so nothing snaps.

## Frame-rate-independent damping

```js
export const damp = (a, b, lambda, dt) => lerp(a, b, 1 - Math.exp(-lambda * dt));
```

- Larger `lambda` follows more tightly. Documented values: steer 9, lean 8, camera position 7, camera target 12, FOV 3, finish deceleration 1.5.
- **Never write `x += (target - x) * 0.1`.** That changes feel completely between 30 FPS and 144 FPS.
- Clamp `dt` to at most `1/20` second so tabbing away and back does not teleport objects.

## Locomotion (documented bicycle runner)

```js
let target = 15 * diff;
if (up) target = 24 * diff;
if (down) target = 3;
if (P.crashT > 0) target = 4;          // diff = 1 + stageIndex * 0.05
const acc = target > P.speed ? (up ? 9 : 5) : (down ? 16 : 4);
// Accelerate slowly, brake quickly: the asymmetry is what feels like a vehicle
P.speed += clamp(target - P.speed, -acc * dt, acc * dt);
P.steer = damp(P.steer, steerIn, 9, dt); // keyboard -1/0/1 becomes a smooth analog
const lat = (5.5 + P.speed * .22) * (P.air ? .7 : 1);
P.lean = damp(P.lean, P.steer * clamp(P.speed / 14, .2, 1.1) * .32, 8, dt);
```

- Jump: `vy = 7.2`, gravity `20` (stronger than real gravity, so jumps are snappy).
- Ramps: height `h * k²` (`k` is progress on the ramp). Launch when `k > .93` with `vy = 9.5 + speed * .12`.
- Landing: landing SFX, `shake = .15`, the pelican pouch is squashed by the impulse, and a clean big landing also sheds particles. The combination is what reads as weight.
- Crash: multiply speed by 0.15, `shake = .6`, 2.2 s of invulnerability flashing at 12 Hz; hit obstacles rock with `sin(knock*30)*knock*.3`.
- Heading: `rotation.y = path.yaw(s) - steer * .18` so the nose leads the turn.

## Pedal cycle

```js
const rWheel = .36;
st.wheel -= o.speed / rWheel * dt;                 // ω = v / r (no slip)
const pedaling = o.airborne ? 0.15 : 1;            // almost no pedaling in the air
st.crank -= o.speed / rWheel * dt * .42 * pedaling; // 0.42 = gear ratio
B.crank.rotation.x = st.crank;
B.pedals.forEach(p => p.rotation.x = -st.crank);   // pedals counter-rotate; feet stay level
B.updateChain(-st.chainOff);                       // 64 instanced chain links slide on a path
B.steer.rotation.y = -o.steer * .45;
lean.rotation.z = -o.lean;                         // whole group (including rider) leans
```

## Body follows crank phase

```js
const ph = st.crank;
PL.torso.position.y = 1.1 + Math.abs(Math.sin(ph)) * .015 * Math.min(1, o.speed / 6);
// |sin| bobs twice per revolution
PL.torso.rotation.z = Math.sin(ph) * .04 * Math.min(1, o.speed / 6) + o.lean * .25;
// sway + lean the body more than the bike in a corner
PL.torso.rotation.x = -.05 + Math.min(o.speed, 25) * .004;          // tuck as speed rises
PL.head.rotation.x = Math.sin(ph * 2) * .05;                        // head nods at 2× body
PL.head.rotation.y = o.steer * .35 + (o.idle ? Math.sin(st.t * .7) * .4 : 0);
```

Keep amplitudes small (1.5 cm, 0.04 rad) and scale by `min(1, speed/6)` so a parked character does not twitch. Large amplitudes look like a seizure.

## Two-bone IK so legs stay on the pedals

Legs are not clips. Each frame, solve from hip to the actual pedal position. Speed, jumps, and lean then cannot detach the feet.

```js
const T = /* pedal in the leg's local space */;
const H = /* hip */;
const L1 = .42, L2 = .44;                            // thigh, shin
const d = Math.min(T.distanceTo(H), L1 + L2 - .001); // avoid acos of NaN at full extension
const v = T.clone().sub(H).normalize();
const cosA = clamp((L1*L1 + d*d - L2*L2) / (2*L1*d), -1, 1);
const A = Math.acos(cosA);
let p = V(0, v.z, -v.y); if (p.z > 0) p.negate(); p.normalize(); // knee plane: always forward
const K = H.clone().addScaledVector(v, L1 * cosA).addScaledVector(p, L1 * Math.sin(A));
seg(thigh, H, K); seg(shin, K, T);
// seg(): place a Y-axis cylinder between two points (midpoint, length, quaternion)
```

Wings (or arms) use the same idea: the "hand" must hold the bar. The grip point rotates with the steer axis; an orthonormal basis from shoulder → grip (`makeBasis(X, Y, Z)`) becomes the wing orientation, so the wings follow the bars in a turn.

## Secondary motion

| Part | Method | Parameters (documented) |
|---|---|---|
| Pouch | Spring-damper: `v += ((target - y) * 160 - v * 6) * dt` | stiffness 160, damping 6; landing impulse `v -= 3.5`; target drops slightly under accel and by 0.15 in the air; Y scale clamped 0.55–1.5 |
| Wings | `slerp` between folded (on the bars) and open; `wingBlend` chases at rate 8 | Open when airborne, celebrating, or crashing; flap 11 / 14 / 22; wingspan scales up when open |
| Blink | Random every 2–5 s, lasts 0.14 s | Squint 0.6 on crash |
| Scarf | A chain of nodes with per-node phase lag (`- i * .9`, `- i * 1.3`); amplitude grows with index and speed | Max flutter around speed 15 |
| Crash | Head shake at 25 Hz (`sin(t*25)*.2`) | Plus a squawk |
| Celebrate | Body bounce at 10 Hz, head pitch −0.35, wings open and flap | Triggered after the finish |

Principle: primary motion (pedaling) sets the rhythm; secondary motion lags and overshoots. Weight is the lag and the overshoot.

## Life in the world

- Obstacles each have `move(o, t)`: a camel bobs `sin(t*1.5)`, a crab sidesteps, a taxi travels the road at 5 m/s, struck props rock.
- Paris pigeons: take off when the player is within 14 m, `y += dt*6` and scatter sideways, wings at `sin(t*40)`, hide after 4 s.
- New York traffic lights cycle every 11 s (green 5, yellow 1.5, red); all heads share the lamp materials, so a color change is a material tweak.
- Steam, flags, and particles (snow, petals, leaves, sand) wrap inside a 70×34×110 box around the camera, so a fixed particle count covers the whole stage.

## Camera: spring follow plus a sense of speed

```js
// Chase: back off with speed, rise on jumps, follow only 1/4 of lateral motion (less nausea)
outPos.copy(base).addScaledVector(fwd, -5.6 - P.speed * .05)
  .add(V(0, 2.5 + P.y * .4, 0)).addScaledVector(right, -P.u * .25);
outLook.copy(base).addScaledVector(fwd, 6).add(V(0, 1.2 + P.y * .5, 0)); // look 6 m ahead, not at the bike
camPos.lerp(tp, 1 - Math.exp(-dt * 7));
camLook.lerp(lp, 1 - Math.exp(-dt * 12)); // position lags the look: a slight drift in turns
camera.fov = damp(camera.fov, 58 + P.speed * .55 + (K.up() ? 4 : 0), 3, dt);
if (shake > 0) {
  camera.position.x += (Math.random() - .5) * shake;
  shake = Math.max(0, shake - dt * 1.5);
}
```

- Two more cameras, toggled with C in the example: close (3 m back, 1.9 high) and side (4.5 m to the right).
- **Title:** slow orbit, `a = t * .18`, radius 6, height `2.2 + sin(t*.3)*.4`.
- **Finish:** swing to the front quarter, rise and pull back with `finishT`, follow with `1 - exp(-dt*2)` for a slow end beat.
- On a cut, call a `snapCamera()` that teleports immediately and set `cutNext = true` so post skips motion blur for that frame and auto-exposure converges instantly (`uRate = 1`).
