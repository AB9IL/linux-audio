# Configuring Pulseaudio: System-Wide Audio EQ, Compression, and Limiting

Computers running Ubuntu, Linux Mint, Debian, or other operating distros with PulseAudio must configure PulseAudio to load the LADSPA or LV2 plugins, route audio data through them, and provide processed audio as the default sink.  This is done with two files: **/etc/pulse/default.pa** and **/etc/asound.conf**.

First, paste the following code into **/etc/asound.conf**

```bash
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
```

Next, paste the following code to the top of /etc/pulse/default.pa to enable the alsa-sink module:

```bash
load-module module-alsa-sink device=hw:0,0 sink_name=processed_output
```

Finally, paste the following code to the bottom of **/etc/pulse/default.pa**.  The
system will send all sounds to the default sink, **ladspa_output.tap_eq**.
Note that the limiter sends its output to the ALSA sink we called *processed_output* above:

```
#EQ and Dynamics
load-module module-ladspa-sink sink_name=ladspa_output.fastLookaheadLimiter label=fastLookaheadLimiter plugin=fast_lookahead_limiter_1913 master=processed_output control=6,0,0.3

load-module module-ladspa-sink sink_name=ladspa_output.tap_dynamics_m label=tap_dynamics_m plugin=tap_dynamics_m master=ladspa_output.fastLookaheadLimiter control=4,700,15,15,13

load-module module-ladspa-sink sink_name=ladspa_output.tap_eq label=tap_equalizer plugin=tap_eq master=ladspa_output.tap_dynamics_m control=-6,-6,-3,0,0,0,0,0,100,200,400,1000,3000,6000,12000,15000 

set-default-sink ladspa_output.tap_eq
```

The ideal settings for each stage of processing depend on the nature of the audio content, and should be determined through testing.  Here are some examples given suitable for normal voice, broadcast, and hardcore hf-radio usage.  Note that PulseAudio takes the control parameters as a string of comma separated numbers. 

After making any changes to /etc/asound.conf, restart ALSA with the following command:

```
sudo alsa reload
```

Processor settings for ALSA (not Pulse):

```bash
#Broadcast Quality
EQ	 controls	[3 3 0 -8 0 0 0 0 100 300 500 1000 3000 6000 12000 15000]
Compressor		[4 700 8 20 13]
Limiter			[6 0 1.0]

#Normal Voice Quality
EQ controls 		[-20 0 -6 0 0 0 -10 -30 100 300 500 1000 3000 6000 12000 15000]
Compressor		[4 300 15 15 13]
Limiter			[10 0 0.3]
		
#Hardcore Ham DX / Contesting
EQ controls 		[-50 -50-50 -5 -5 -50 -50 -50 100 300 500 1000 3000 6000 12000 15000]
Compressor		[4 300 20 20 13]
Limiter			[15 0 0.3]
```

Processor plugin settings for PulseAudio.  These are not complete Pulseaudio source and sink configs.  Look into the **default.pa** or command line syntax to fully set up these processor plugins:

```bash
#Broadcast Quality
plugin=fast_lookahead_limiter_1913 master=alsa_output.pci-0000_05_01.0.analog-surround-71 control=6,0,1.0
plugin=tap_dynamics_m master=ladspa_output.fastLookaheadLimiter control=4,700,8,20,13
plugin=tap_eq master=ladspa_output.tap_dynamics_m control=3,3,0,-8,0,0,0,0,100,300,500,1000,3000,6000,12000,15000 

#Normal Voice Quality
plugin=fast_lookahead_limiter_1913 master=alsa_output.pci-0000_05_01.0.analog-surround-71 control=10,0,0.3
plugin=tap_dynamics_m master=ladspa_output.fastLookaheadLimiter control=4,300,15,15,13
plugin=tap_eq master=ladspa_output.tap_dynamics_m control=-50,-50,-50,-5,-5,-50,-50,-50,100,300,500,1000,3000,6000,12000,15000 

#Hardcore Ham DX / Contesting
plugin=fast_lookahead_limiter_1913 master=alsa_output.pci-0000_05_01.0.analog-surround-71 control=15,0,0.3
plugin=tap_dynamics_m master=ladspa_output.fastLookaheadLimiter control=4,300,20,20,13
plugin=tap_eq master=ladspa_output.tap_dynamics_m control=-6,-6,-3,0,0,0,0,0,100,200,400,1000,3000,6000,12000,15000 
