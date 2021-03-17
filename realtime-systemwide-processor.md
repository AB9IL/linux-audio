# Configuring Pulseaudio for System-Wide Audio EQ, Compression, and Limiting

Computers running Ubuntu, Linux Mint, Debian, or other operating distros with PulseAudio must configure PulseAudio to load the LADSPA or LV2 plugins, route audio data through them, and provide processed audio as the default sink.  This is done with two files: **/etc/pulse/default.pa** and **/etc/asound.conf**.

First, paste the following code into **/etc/asound.conf**

pcm.pulse {
    type pulse
}
ctl.pulse {
    type pulse
}
pcm.!default {
    type pulse
}
ctl.!default {
    type pulse
}

Next, paste the following code to the top of /etc/pulse/default.pa to enable the alsa-sink module:

<code>
load-module module-alsa-sink device=hw:0,0 sink_name=processed_output
</code>

Finally, paste the following code to the bottom of **/etc/pulse/default.pa**.  The
system will send all sounds to the default sink, **ladspa_output.tap_eq**.
Note that the limiter sends its output to the ALSA sink we called *processed_output* above:

<code>
#EQ and Dynamics
load-module module-ladspa-sink sink_name=ladspa_output.fastLookaheadLimiter label=fastLookaheadLimiter plugin=fast_lookahead_limiter_1913 master=processed_output control=6,0,0.3

load-module module-ladspa-sink sink_name=ladspa_output.tap_dynamics_m label=tap_dynamics_m plugin=tap_dynamics_m master=ladspa_output.fastLookaheadLimiter control=4,700,15,15,13

load-module module-ladspa-sink sink_name=ladspa_output.tap_eq label=tap_equalizer plugin=tap_eq master=ladspa_output.tap_dynamics_m control=-6,-6,-3,0,0,0,0,0,100,200,400,1000,3000,6000,12000,15000 

set-default-sink ladspa_output.tap_eq

</code>

The ideal settings for each stage of processing depend on the nature of the audio content, and should be determined through testing.  See the examples given below, for ALSA, that are suitable for normal voice, broadcast, and hardcore hf-radio usage.  Note that PulseAudio takes the control parameters as a string of comma separated numbers.
