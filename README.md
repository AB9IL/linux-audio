# linux-audio
Scripts, configs, and snipptes useful for audio on GNU/Linux

### Suggested LADSPA / LV2 Plugins for audio processing on your Linux machine:
My suggestion is to stock up on audio plugins; there is a great variety and you may want one of these someday for a specific task:

swh-plugins
tap-plugins
zam-plugins
lsp-plugins
lsp-plugins-lv2

For audio compression and gating, I find the Tap-Dynamics module to be especially useful and moddable.  For example, you can adjust its default behavior by making changes to the source code.  For example, in the file "tap_dynamics_presets.h" preset number 7 can provide good amplitude compression and gating if written as shown:

	{ /* Compressor/Gate, threshold at -20 dB */
		5, 
		{
			{-80.0f, -110.0f},
			{-62.0f, -90.0f},
			{-20.0f, -20.0f},
			{0.0f, -10.0f},
			{20.0f, -8.0f},
		},
	},
  
  ### Modifying Your Debian / Ubuntu (and other) Audio
  
  Plan to change defaults / presets by editing **/etc/asound.conf** and **/etc/pulse/default.pa**
  
  If editing levels by hand, look in **/var/lib/alsa**
  
  
