
Test project for electronic drum triggering
===========================================

Commercial module latency
-------------------------

According to https://www.vdrums.com/forum/general/the-lounge/1182869-fastest-lowest-latency-drum-module-available and http://onyx3.com/EDLM, the drum modules have the following measured latencies:

- Roland TD-50\30: **3 ms**    (measured by Chris K)
- Roland TD11\15:  **3 ms**    (measured by Chris K)
- Roland TD15:     **3.15 ms** (measured by 30YearsLater)
- Roland TD12:     **3.60 ms** (measured by 30YearsLater)
- Roland TD-20:    **5.7 ms**  (measured by Chris K)
- Roland TD-4:     **3.8 ms**  (measured by onyx3.com)
- Roland TD-17:    **3.6 ms**  (measured by onyx3.com)
- MIMIC:           **4 ms**    (measured by Chris K)

I measured my Roland TDW-20 module by using a stereo splitter cable and connected the trigger output
of my PD-120 pad directly to one channel of my sound card input and also to the trigger input of the
TDW-20 module. Then I connected the analog audio output of the TDW-20 to the other stereo channel of
my sound card and recorded the signal with Audacity. I now measured the latency between the main
peak of the pad trigger to the first peak of the synthesized signal from the TDW-20. As seen in the
screen shot, there is a latency of about **7 ms**:
![Roland TDW-20 drum module measured latency](roland_td20_latency.jpg)


Project specifications
----------------------

- Research is done using a regular audio card, capture the drum pad output signal and develop
  the algorithms in Octave.

- One goal would be to use a Raspberry Pi Zero as a trigger module. So, it gets a sampled
  audio signal from the GIOP (some external hardware needed) and processes it using a C++
  software. It outputs a MIDI signal. Since the Raspberry Pi Zero has only a slow processor,
  it will not be possible to include the complete drum module.
  As an alternative an Arduino could be used, similar to the [open e-drums](https://open-e-drums.com) project.

- Positional sensing shall be supported.

- Overall latency should be as small as possible. The goal is to get a latency < 10 ms.


Drum triggering algorithms
--------------------------

Brainstorming:

- Peak detection

  - If you calculate the power of the recorded real-valued audio signal, the resulting power curve has
    significant power drops caused by the nature of a sinusoidal signal. A filtering can smooth the
    curve. As a test I have used an Hilbert transform to convert the real-valued signal in a complex
    signal. As a result, the magnitude of that complex signal is much smoother already without having
    modified the actual spectrum of the signal (real-valued signals have mirror symmetric spectrum).

  - To improve the peak detection, we can make use of the known decay curve of the trigger pad in use.
    So, after successfully detecting a peak, we know that this peak causes a slowly decaying power
    curve which has a known shape and we can subtract that known curve from the signal to improve the
    detection of the next pad hit.

- Positional sensing

  - It has shown that if you hit the pad close to the edge, the resulting sound has less low frequencies
    and sounds more crisp. So, the idea is to low-pass filter the signal and at the detection peak we
	calculate the power ratio of the low-pass filtered signal with the unfiltered signal. This is actual
	possible metric for the positional sensing.

- First results

  - The following plot shows how the current status of the algorithms perform. At the beginning there are
    some single hits. Then there comes a region with a snare drum roll. After that, there are single hits
    which start from the middle, move to the edge and go back to the middle of the pad where the hits are
    equal strong. As shown by the black markers, the positional sensing seems to work pretty good. Also,
    the peak detection and velocity estimation seems to be pretty good as well.
    ![First results plot](first_results.jpg)