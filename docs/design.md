Web Vocal Effects Processor

Requirements & Design Document

1. Purpose & Scope

The goal of this project is to build a browser-based, real-time vocal effects processor that:

Captures audio from the user’s microphone

Applies common vocal effects in real time

Outputs processed audio to the system speakers

Provides responsive, musical, low-latency control

Uses only standard Web APIs (no plugins, no backend)

The application targets musicians, voice performers, and audio experimenters, not casual users.

2. Functional Requirements
2.1 Audio I/O

The system SHALL request microphone access via user gesture.

The system SHALL process audio in real time with minimal latency.

The system SHALL output processed audio to the default audio output device.

The system SHALL stop all audio processing cleanly when disabled.

2.2 Effects Chain

The processor SHALL implement the following effects in this order:

Input Gain

Equalizer (3-band)

Compressor

Delay (parallel)

Reverb (parallel)

Master Output Gain

Each effect SHALL operate independently and non-destructively.

2.3 Equalizer (EQ)

Requirements

Three bands: Low, Mid, High

Each band SHALL support gain adjustment in dB

EQ SHALL be implemented using BiquadFilterNode

EQ SHALL support a bypass switch

Design Notes

Low: low-shelf (~100 Hz)

Mid: peaking (~1.5 kHz)

High: high-shelf (~5 kHz)

Bypass sets all EQ gains to 0 dB (neutral)

2.4 Compressor

Requirements

Adjustable threshold

Adjustable ratio

Adjustable makeup gain

Real-time control without audio dropouts

Compressor SHALL support bypass

Design Notes

Implemented using DynamicsCompressorNode

True bypass achieved by setting ratio = 1

Makeup gain applied post-compression

2.5 Delay

Requirements

Adjustable delay time

Adjustable feedback

Adjustable wet/dry mix

Delay SHALL operate in parallel

Delay SHALL support bypass

Design Notes

DelayNode with feedback loop

Wet/dry mix implemented via GainNodes

Bypass achieved by muting wet signal only

2.6 Reverb

Requirements

Adjustable decay time

Adjustable wet/dry mix

Reverb SHALL operate in parallel

Reverb SHALL support bypass

No external impulse files required

Design Notes

Implemented using ConvolverNode

Impulse response generated procedurally

Bypass achieved by muting wet signal

2.7 Effect Bypass

Requirements

Each effect SHALL have an independent bypass switch

Bypass SHALL NOT disconnect AudioNodes

Bypass SHALL NOT introduce clicks or pops

UI changes SHALL reflect bypass state

Design Principle

Bypass = neutral parameter values, not graph modification

2.8 User Interface

Requirements

Effects SHALL be grouped into visual blocks

Blocks SHALL be arranged in responsive rows

Each block SHALL include:

Effect name

Bypass toggle (where applicable)

Relevant controls

Controls SHALL update audio parameters in real time

3. Non-Functional Requirements
3.1 Performance

Audio processing SHALL remain stable under continuous operation

Parameter changes SHALL use smoothing (setTargetAtTime)

No unnecessary node creation during playback

3.2 Reliability

Application SHALL recover cleanly from start/stop cycles

No uncaught JavaScript errors during normal operation

No reliance on browser-specific hacks

3.3 Compatibility

Target browsers: Chromium-based (Chrome, Edge)

Desktop-first design

Uses Web Audio API only

4. System Architecture
4.1 High-Level Signal Flow
Mic
 → Input Gain
 → EQ (low → mid → high)
 → Compressor
 → Makeup Gain
 → Delay (parallel)
 → Reverb (parallel)
 → Output Gain
 → Destination

4.2 Audio Graph Design Principles

Static graph: Nodes are connected once at startup

Parallel routing for time-based effects

Gain-based bypass instead of disconnection

Single AudioContext per session

5. UI-to-Audio Binding Design
5.1 Slider Handling

Sliders SHALL directly reference DOM elements

AudioNodes SHALL NOT be used to infer DOM state

Slider handlers SHALL:

Update display labels

Apply smoothed parameter changes

Respect bypass state

Explicit Mapping Rule

UI controls → AudioNodes (never the reverse)

5.2 Bypass Logic Pattern
Effect	Bypass Strategy
EQ	Set band gains to 0 dB
Compressor	Set ratio to 1
Delay	Set wet gain to 0
Reverb	Set wet gain to 0
6. Error Prevention & Lessons Learned
6.1 Known Pitfalls Avoided

❌ Reading .id from AudioNodes

❌ Disconnecting nodes during playback

❌ Rebuilding graphs on bypass

❌ Unsmooth parameter jumps

6.2 Defensive Design Rules

Always check bypass state before applying slider changes

Never assume AudioNodes have DOM-like properties

Use explicit references for all controls

7. Extensibility

The design supports future additions such as:

Input/output level meters

Preset save/load (JSON)

Recording processed audio

MIDI controller mapping

AudioWorklet-based custom DSP

Mobile-friendly UI adaptations

No architectural changes are required for these features.

8. Success Criteria

The system is considered complete when:

All effects are audible and controllable in real time

No console errors occur during normal use

Bypass switches behave musically and predictably

Start/stop cycles are stable

A musician can clearly hear and shape their vocal sound
