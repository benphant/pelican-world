# Audio

All music and sound in the documented game is synthesized with the Web Audio API: oscillators, noise, and filters. There are no audio files.

## Signal chain

```
voice (N detuned oscs) → lowpass → envelope Gain ─┬→ music Gain(.42) ─┐
                                                  └→ reverb send → Convolver(2.2 s procedural IR) → Gain(.32) ─┤
noise (looping white-noise buffer) → biquad → Gain ─→ sfx Gain(.75) ──┤
                                                                      master → DynamicsCompressor(-16 dB, 4:1) → out
```

- The **reverb impulse** is generated in code: `(Math.random()*2-1) * Math.pow(1 - i/len, 3)` (exponentially decaying white noise), one buffer per channel.
- **Wind:** looping noise through a bandpass, gain `min(.12, speed² × .00025)`, center `300 + speed × 40`. Faster riding is louder and thinner.

## Look-ahead scheduler (beats that do not hitch)

```js
this.timer = setInterval(() => this.schedule(), 25); // poll every 25 ms
schedule() {
  if (!this.ctx || !this.region || !this.musicOn || this.paused) return;
  if (this.nextT < c.currentTime) this.nextT = c.currentTime + .05;
  // After a hitch or a background tab, catch up — do not fire a burst of late notes
  const spb = 60 / R.bpm / 2;                         // eighth-note length
  while (this.nextT < c.currentTime + .15) {          // schedule 150 ms ahead on the audio clock
    try { this.playStep(this.step, this.nextT, spb, R); }
    catch (e) { /* one bad note must not kill the scheduler */ }
    const swing = R.style === 'jazz' ? (this.step % 2 ? -.33 : .33) * spb * .5 : 0;
    this.nextT += spb + swing; this.step++;
  }
}
```

## Per-stage music is data

Documented examples (scales and instruments follow the stage, not a requirement for every new game):

```js
music: { style: 'waltz', bpm: 168, root: 60, scale: [0,2,4,5,7,9,11],
         prog: [[0,4,7],[5,9,0],[7,11,2] /* … */], lead: 'accordion' }
music: { style: 'jazz',  bpm: 132, root: 58, scale: [0,2,3,5,7,9,10],
         prog: [[2,5,9,12],[7,11,14,17],[0,4,7,11] /* … */], lead: 'piano' }
music: { style: 'pentatonic', bpm: 96, root: 62, scale: [0,2,4,7,9], lead: 'guzheng' }
music: { style: 'japan', bpm: 92, root: 64, scale: [0,1,5,7,8], lead: 'koto' }
music: { style: 'hawaii', bpm: 104, root: 65, scale: [0,2,4,7,9], lead: 'uke' }
music: { style: 'arabic', bpm: 118, root: 57, scale: [0,1,4,5,7,8,10], lead: 'oud' }
```

Accompaniment branches on `style`:

- Waltz: 6 eighths per bar, oom-pah-pah.
- Jazz: walking bass and ride, snare on 2 and 4.
- Arabic: doum/tek hand drum.
- Japanese / Chinese-inspired beds: long pads plus a drum.

**Melody:** a random walk on the scale (step −2 to +2 degrees, clamped to 2–13). Beat one of each bar **snaps to the nearest chord tone**, so the line does not "go out." Note density is a style parameter (waltz 0.7, Japan 0.4).

**Timbres** (documented recipes):

- Accordion = two saws detuned ±8 cents, plus 5.5 Hz vibrato.
- Guzheng = triangle pluck, 1/3 chance of a 1.059× (semitone) upward slide.
- Piano = triangle pluck plus a sine one octave up.
- Oud = saw pluck with a filter envelope.

`env()` is an ADSR. Plucks use `exponentialRamp` down to 0.0001 (an exponential ramp cannot target 0).

## Sound effects (all a few lines of synthesis)

| Cue | Recipe |
|---|---|
| Collect | Square arpeggio; root rises with the combo |
| Gold collect | 7-note rising arpeggio plus reverb |
| Jump | Sine 300 Hz sliding to 2.4×, plus a band-pass noise |
| Land | Low-passed noise |
| Spring ramp | Sine sliding to 3.5×, 14 Hz vibrato |
| Crash | Low-passed noise, a falling square, plus a squawk (saw 520 Hz falling, Q 6, 30 Hz vibrato) |
| Bell | Sine 2100 Hz × [1, 2.76, 5.4] (inharmonic partials = metal), twice |
| Brake | 2400 Hz saw, high Q, vibrato |
| Fireworks | Low-passed noise burst plus high-passed crackle |

## Required lesson: music dies after "play again"

**Symptom:** after a pause or a results screen, "retry" or "back to title" leaves music silent forever. SFX still work.

**Cause:** `togglePause(true)` set `audio.paused = true`, but `toTitle()` wrote `G.state` directly and **never called `togglePause(false)`**. The flag stayed stuck. The scheduler returns immediately on `if (this.paused) return;`.

**Fix (three belts):**

```js
// 1) Single source of truth: derive the flag from game state every frame
audio.paused = G.state === 'paused';
// 2) Also clear it on every entry
function toTitle() { G.state = 'title'; /* … */ audio.paused = false; /* … */ }
setRegion(r) {
  this.region = r; this.step = 0; this.paused = false;
  if (this.ctx) {
    this.nextT = /* now */;
    if (this.ctx.state !== 'running') this.ctx.resume();
  }
}
// 3) AudioContext may be suspended (background, iOS). Resume on any click and on foreground
addEventListener('pointerdown', () => audio.init(), { capture: true });
document.addEventListener('visibilitychange', () => {
  if (!document.hidden && audio.ctx && audio.ctx.state !== 'running') audio.ctx.resume();
});
```

**How to test:** add a counter `nMusic` on `voice()` that only counts notes sent to the music bus. After each action, wait 1.5 s and assert the counter is still rising. Cover:

- Retry, and two retries in a row
- Back to title → start
- Fail → retry
- Clear → play again
- Mute → unmute
- Stage change
- Every stage in the stage select
- Results → play again

Headless Chrome needs `--autoplay-policy=no-user-gesture-required` to emit audio.

**General rule:** a **boolean plus several exits will miss one**. If a value can be derived from state, do not store a second copy.

Documented mix defaults: compressor −16 dB at 4:1; music 0.42; SFX 0.75; reverb 0.32.
