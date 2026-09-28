/*
====================================================================
AUDIO PROGRAMMING - WORKSHOP 2

====================================================================
*/

// -----------------------------------------------------------------
// 1. NOISE GENERATORS
// -----------------------------------------------------------------

// White noise (equal energy per frequency) - Watch playback levels!
(
{
	WhiteNoise.ar(0.1 !2)
}.play
)

// Pink noise (equal energy per octave - natural sound to human hearing)
(
{
	PinkNoise.ar(0.5 !2)
}.play
)


// -----------------------------------------------------------------
// 2. FILTERS (SUBTRACTIVE SYNTHESIS)
// -----------------------------------------------------------------

// Simple Low-Pass Filter (LPF)
(
{
	var noise = PinkNoise.ar(0.5) !2;
	var cutoff = 1000;

	LPF.ar(noise, cutoff)
}.play
)

(
Ndef(\pink, {|cutoff = 1000|
	var noise = PinkNoise.ar(0.5) !2;
	LPF.ar(noise, cutoff)
}).play
)

Ndef(\pink).set(\cutoff, 1000);
Ndef(\pink).gui

// LPF with interactive cutoff frequency via MouseX
(
{
	var cutoff = MouseX.kr(20, 10000, 1);
	var noise = PinkNoise.ar !2;

	LPF.ar(noise, cutoff)
}.play
)

/*
TASK: Try swapping LPF in the example above with other filter UGens:
- HPF  (High Pass Filter)
- BPF  (Band Pass Filter)
- BRF  (Band Reject Filter)
- RLPF (Resonant Low Pass Filter)
- RHPF (Resonant High Pass Filter)
*/

// Resonant Low-Pass Filter (RLPF)
// MouseX controls cutoff frequency, MouseY controls resonance (rq)
(
{
	var freq = 80;
	var detune = 0.99;
	var sound = Pulse.ar([freq, freq * detune], 0.5, 0.5);
	var cutoff = MouseX.kr(50, 8000, 1);
	var res = MouseY.kr(1, 0.1); // Lower value = higher resonance

	RLPF.ar(sound, cutoff, res)
}.scope
)


// -----------------------------------------------------------------
// 3. DYNAMIC TIME GENERATORS (Line & XLine)
// -----------------------------------------------------------------

// Line.kr(start, end, dur) creates a line from a start value to end value
(
{
	var freq = 80;
	var detune = 0.99;
	var width = Line.kr(0.1, 0.9, 10); // Sweeps pulse width over 10 seconds
	var sound = Pulse.ar([freq, freq * detune], width, 0.5);
	var cutoff = MouseX.kr(50, 8000, 1);

	LPF.ar(sound, cutoff)
}.scope
)


// -----------------------------------------------------------------
// 4. ENVELOPES & TRIGGERS
// -----------------------------------------------------------------

// Visualizing Envelopes:
// Percussive (Attack-Release / AR) envelope
Env.perc(attackTime: 0.01, releaseTime: 1).plot;
Env.perc(0.5, 1, 1, \sine).plot;
Env.perc(0.5, 1, 1, \welch).plot;

// Linear (Attack-Sustain-Release / ASR) envelope
Env.linen(attackTime: 0.5, sustainTime: 1, releaseTime: 2).plot;

// Envelopes triggered periodically with Impulse.kr
(
{
	var freq = 100;
	var vol = 0.5;
	var detune = 0.99;
	var trigger = Impulse.kr(2); // 2 triggers per second
	var env = EnvGen.kr(Env.perc(0.01, 0.4), trigger);

	Saw.ar([freq, freq * detune], vol * env)
}.play
)

(
Ndef(\bounce, { |freq = 100, vol = 0.5, detune = 0.99, speed = 2 |

	var trigger = Impulse.kr(speed);
	var env = EnvGen.kr(Env.perc(0.01, 0.4), trigger);

	Saw.ar([freq, freq * detune], vol * env)
}).play
)

Ndef(\bounce).gui

(
// Define custom specs
Spec.add(\freq, [40, 1000, \exponential]);
Spec.add(\vol, [0, 1, \linear]);
Spec.add(\detune, [0.5, 1.5, \linear]);
Spec.add(\speed, [1, 10, \linear]);
)

Ndef(\bounce).clear


// -----------------------------------------------------------------
// 5. LANGUAGE-SIDE RANDOMNESS vs SERVER-SIDE TRIGGERS
// -----------------------------------------------------------------

// Language-side evaluation (runs once when executed):
100.rand;                    // Random integer between 0 and 100
{ 100.rand }.dup(10);        // Array of 10 random numbers
rrand(20, 40);               // Random number between 20 and 40
rrand(0.0, 1.0).round(0.1);  // Fractional number rounded to 1 decimal place

// Server-side random triggers (TRand / TExpRand update on every trigger):
(
{
	var trigger = Impulse.kr(2);
	var release = TRand.kr(0.2, 1, trigger);
	var freq = TExpRand.kr(100, 600, trigger).round(100);
	var vol = TRand.kr(0.2, 0.6, trigger);
	var detune = 0.99;
	var env = EnvGen.kr(Env.perc(0.01, release), trigger);

	Saw.ar([freq, freq * detune], vol * env)
}.play
)


// -----------------------------------------------------------------
// 6. APPLIED EXAMPLES
// -----------------------------------------------------------------

// Example A: Wind Emulator
(
{
	var noise = PinkNoise.ar(1);
	var modulator = LFNoise1.kr(1, 50, 150);

	RLPF.ar(noise, modulator, 0.3) !2
}.play
)

// Example B: Simple 808-Style Kick Drum
(
{
	var freq = MouseX.kr(40, 100, 1);
	var sound = SinOsc.ar(freq) !2;
	var release = MouseY.kr(0.1, 2);
	var pulse = Impulse.kr(2);
	var env = EnvGen.kr(Env.perc(0.00001, release), pulse);

	sound * env

}.play
)

// Example C: Acceleration & Filter Sweep Synthesis
(
{
	var pulse = Impulse.kr(Line.kr(1, 10, 20));
	var env = EnvGen.kr(Env.perc(0.01, 0.5), pulse, 2000, 200);

	RLPF.ar(PinkNoise.ar([0.5, 0.5]), env, Line.kr(0.2, 0.02, 20))
}.play
)