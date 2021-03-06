# Audio example for UE5

Demonstration of the UE5 audio features, including:

- Exploration of MetaSound, starting with a A440 generator
- Procedurally generated wind MetaSound
- Setting MetaSound float parameters from Blueprints
- Invoking MetaSound triggers from Blueprints
- Changing Submix reverb when entering an area (not using Audio Volumes, which
  will be deprecated according to the UE5 release notes)
- Audio Modulation plugin Control Bus, Control Bus Mix, and Patch Parameters to
  mix sounds


## Panning Method
Per Aaron McLeran's comments in [Unreal Engine's Audio Rendering:
Retrospectives and Case Study Analysis](https://youtu.be/QwMAKXBTAC8?t=1178),
the panning method was changed from Linear (default setting) to Equal Power via
this setting in Config/DefaultEngine.ini:

```ini
[/Script/Engine.AudioSettings]
PanningMethod=EqualPower
MonoChannelUpmixMethod=EqualPower
```

## Cue Points

USoundWave and the Metasound Wave Player node supports Cue points on WAV files.
This is implemented in Runtime/Engine/Private/Audio.cpp in
`FWaveModInfo::ReadWaveInfo`. For this to work, your WAV file needs to contain
*cue* chunks (chunks with 4CC ID `'cue '`.  To author these, you'll need a WAV
exporter/authoring tool that supports them.  At the moment, Audacity (my
primary tool) does not.

See also:

- [Adding Cue Points to Wav files in C](https://bleepsandpops.com/post/37792760450/adding-cue-points-to-wav-files-in-c)


## Credits

The EchoThief_FortWordenTunnel IR (impulse response) asset was created from the
[EchoThief](http://www.echothief.com/) library.

The
[Fully Automatic Pistol (UZI) - Pistol Firing Burst (1)](https://freesound.org/people/FilmmakersManual/sounds/522545/)
sound is from
[FilmmakersManual](https://freesound.org/people/FilmmakersManual/sounds/522544/).


## See Also

- [New Audio Features](https://docs.unrealengine.com/5.0/en-US/AudioFeatures/)
- [Audio Modulation Mixing in UE4.26 Preview 1 - Dan Reynolds](https://www.youtube.com/watch?v=BV5l_7-w7cg)



