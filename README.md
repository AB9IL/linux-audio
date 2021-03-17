# linux-audio
Scripts, configs, and snipptes useful for audio on GNU/Linux

### Suggested LADSPA / LV2 Plugins for audio processing on your Linux machine:
My suggestion is to stock up on audio plugins; there is a great variety and you may want one of these someday for a specific task:

swh-plugins
tap-plugins
zam-plugins
lsp-plugins
lsp-plugins-lv2

### Modifying Your Debian / Ubuntu (and other) Audio
  
Plan to change defaults / presets by lediting **/etc/asound.conf** and **/etc/pulse/default.pa**  If you are on ALSA only (no PulseAudio), edit global settings in **/etc/asound.conf** or your own user settings in **~/.asoundrc**.

Normally, use **Alsamixer** or **Pavucontrol** to set volume levels.  Save with:
```bash
sudo alsactl store
```

If you edit default levels by hand, look in **/var/lib/alsa**.  Edit and save as root.

### Guides and Examples for LADSPA / LV2 audio processing in Linux:
[Realtime System Wide Linux Audio Processing](/realtime-systemwide-processor.md)
[Audio Processing WITH Pulseaudio and Dmix](/linux-audio-processor-with-dmix.md)
[Audio Processing Without PulseAudio and Dmix](/linux-audio-processor-nopulse-nodmix.md)
