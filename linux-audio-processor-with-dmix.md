# Linux Audio Processing (Pulse and Dmix)

Audio can be processed system-wide without Pulseaudio, with a few changes to the default ALSA configuration.  Here we configure the compressor to take data from the "dmix" source, which carries any audio running on the system.  Also, the default plugin is configured to take audio from the last processor stage (ladcomp_limiter).

```bash
# ALSA / LADSPA Speech Processor
#Place this in /etc/asound.conf or ~/.asoundrc

#Convert audio to float data for the equalizer plugin.
# Speech Processor
pcm.ladcomp {
    type plug
# Use this for single channel processing
#    slave.pcm "plughw:0,0";
# Use this for sysytemwide processing
    slave.pcm "plug:dmix";
}

#LADSPA Equalizer plugin.
#Set levels and bands as desired.
#Input is float data from microphone
pcm.ladcomp_eq {
      type ladspa
      slave.pcm "plughw:0,0";
      path "/usr/lib/ladspa";
      plugins [
          {
              label tap_equalizer
              input {
                        #the first 8 numbers set levels (dB),  2nd 8 numbers set the center frequencies (hz).
                        controls [3 0 -6 0 0 0 0 0 100 300 500 1000 3000 6000 12000 15000]
              }
          }
      ]
  }

#LADSPA compressor plugin.
#Set parameters as desired.
#Input is from equalizer plugin.
pcm.ladcomp_compressor {
      type ladspa
      slave.pcm "ladcomp_eq";
      path "/usr/lib/ladspa";
      plugins [
          {
              label tap_dynamics_m
              input {
                        #attack time (ms), release time (ms), offset gain (dB), makeup gain (dB), function
                        controls [3 800 12 20 7]
              }
          }
      ]
  }

#LADSPA look-ahead limiter plugin.
#Set parameters as desired.
#Input is compressor plugin.
pcm.ladcomp_limiter {
      type ladspa
      slave.pcm "ladcomp_compressor";
      path "/usr/lib/ladspa";
      plugins [
          {
              label fastLookaheadLimiter
              input {
                        #InputGain,Limit,Releasetime
                        controls [10 0 0.2]
              }
          }
      ]
  }

#Whatever is fed to the audio
#processor is available here.
#EQ, Compression, and Limiting
pcm.!combo {
type plug;
slave.pcm ladcomp_limiter;
}

#default device
pcm.!default {
type plug
#Single channel processing
#slave.pcm "hw:0,0";
#Systemwide processing
slave.pcm "ladcomp_limiter";
}

ctl.!default {
type hw           
card 0
}
