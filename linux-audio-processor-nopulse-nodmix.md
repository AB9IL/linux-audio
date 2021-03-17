# Configuring ALSA (no Pulseaudio, no dmix)

System-wide audio processing without PulseAudio requires configuring ALSA.  ALSA will take raw audio, pass the data through a series of LADSPA plugins in real time, and export processed audio as the default sound.  All of this is set up, system wide, in the file **/etc/asound.conf**.

First, it is necessary for the audio data to be in float format.  That is accomplished in the first section of asound.conf, for the virtual device "ladcomp."   Next, the "tap_equalizer" plugin is set to recieve audio data from the first audio interface, "plughw:0,0" and process it according to the controls.  There is flexibility here - the gain levels and bands can be changed to suit the EQ task at hand.

Next, the programmable "tap_dynamics_m" plugin is used for audio amplitude compression.  There are several different compression, gating, and expansion presets for this plugin; here number 13 was chosen since it combines compression and mild gating to reduce background noise.  Some makeup gain is used after the audio is compressed, to bring it back to a nominal level.  Preset 7 also works well.

Finally, the "Fast Lookahead Limiter" is applied to hard limit the audio for additional lowering of the peak-to-average ratio and prevent voice peaks from going above a set limit.  This is important when feeding audio to radio transmitters in order to prevent damage, distortion, or triggering internal protective circuits.  Few things sound worse than a single sideband transmitter driven so hard that the ALC kicks in, or an FM transmitter modulated strongly enough to reach the deviation limiter.  The "Fast Lookahead Limiter" does its job with finesse.

Here is the actual speech processor data.  Paste it into the file **/etc/asound.conf**.  If there presently is no **/etc/asound.conf**, then create it.  For individual users on a system who want a customized audio configuration, this data may be written to the file **~/.asoundrc** (a hidden file in the user's home directory).

<code>
# ALSA / LADSPA Speech Processor
#Place this in /etc/asound.conf or ~/.asoundrc

#Convert audio to float data for the equalizer plugin.
pcm.ladcomp {
    type plug
    slave.pcm "plughw:0,0";
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
                        controls [-20 0 -6 0 0 0 -10 -30 100 300 500 1000 3000 6000 12000 15000]
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
                        controls [3 300 15 15 13]
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
                        controls [10 0 0.3]
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
type plug;
slave.pcm "hw:0,0";
}

ctl.!default {
type hw           
card 0
}
</code>
