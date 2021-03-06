## Switching to [Gassitpi](https://github.com/shivasiddharth/GassistPi) based radio:
**Benefits:**
* Get a google home like thing
* Can play songs by request from YouTube
* Voice control everything
* Standard Raspbian OS (no fighting volumio when updating, etc.)
* Simple Python based control

**Downsides**
* ~No web UI (yet?)~ Using this https://github.com/open-dynaMIX/simple-mpv-webui
  * May not work with multiple mpv instances running unless I create a wrapper for multiple running instances? definitely not a version one feature...
* Pretty slow calling up songs and radio stations
  * Will get around the radio station bit by having multiple mpvs start up as services and then cycle between muting them.

## MPV commands:
### For streaming radio stations
* Start with: `mpv --no-video --playlist=/path/to/playlist/stereoio.pls --playlist-start=pos --input-ipc-server=/tmp/mpvsocket --mute=yes`
    - Probably want this to kick on at startup.
    - `stereoio.pls` is just a plaintext file with radio urls on each line.
    - `pos` is a 0-based index value of which line in the playlist you want to play. Should be read from a file representing the current physical position of the tuner knob.
    - `mute=yes` starts the radio muted, this is so the rpi can powercycle without the radio playing through the speakers
    - `--input-ipc-server=/tmp/mpvsocket` allows you to pass valid json to the running mpv instance. The following commands are ones we will want:
        1. `echo '{"command": ["playlist-pos", 1] }' | socat - /tmp/mpvsocket [change to playlist position 1]`
        2. `{ "command": ["cycle", "mute"] }`
        3. `{ "command": ["cycle", "pause"] }`
* Can have multiple instances of MPV running with different /tmp/sockets. Probably don't want too many since the Rpi is small and might get over worked, but it could be worth trying to have the 4 radio stations below all running as separate processes. We could have them running as services as well so that they kickoff at startup... hmmmmmm.... They would all have a different `/tmp/socket` file where commands are sent (by a python script? by a node script? also running as a service). When the tuner enters/leaves the stations range it sends the command to cycle that stations mute status.
  * We would probably need to keep track of the currently playing station somewhere so that asking google to tune to a different sation or asking for a stop sends the message to the right socket.
  * Probably means we should use the more explicit `{ "command": ["set_property", "mute", true|false] }` set of commands
  * This may also break the simple-mpv-webui...
  * Since this set of 4 radios will all be services controlled by systemd, we can have mode switching (changing between radio, youtube, etc) idle them all and then start them up again when we switch back. Will just want to add some kind of startup message so that when the radio doesn't start right away people don't think it is broken.

## Web radio urls:
* Rock the Craddle Radio: https://rockthecradle.stream.publicradio.org/rockthecradle.mp3
* the current: https://current.stream.publicradio.org/kcmp.mp3
* MPR news: https://nis.stream.publicradio.org/nis.mp3
* classical MPR: https://cms.stream.publicradio.org/cms.mp3?cb=8600250482
* Jazz 88: http://quarrel.str3am.com:7110/live_mp3

## YT playlists:
* Vern's Dance Party: https://www.youtube.com/watch?list=PLWTMAv08KxuPRxkX2qAJBcVwmjn8RodGP
* Vern's Mix tape: https://www.youtube.com/watch?list=PLWTMAv08KxuPfzsHDTKvjmAi02Elhz6ZQ

## HifiBerry Amp2
* GPIO you can't use: https://www.hifiberry.com/docs/hardware/gpio-usage-of-hifiberry-boards/

## `systemctl` tidbits:
* ~while the `start` and `stop` commands require root, `is_active` and `is_inactive` do not~
  * Manage by updating the visudo file to create a group for managing the radio systemd files
    You still need to pass `sudo` with the command, but it will not prompt for a password.
* the systemd files will need a user name in them, I think...
* Consider using a [systemd template file](https://fedoramagazine.org/systemd-template-unit-files/)

## Controlling via web?:
* have node interact with the python script: https://medium.com/@HolmesLaurence/integrating-node-and-python-6b8454bfc272

## Other audio sources?:
* Try adding pianobar, the cli based Pandora client [maybe this](https://creekhousemedia.com/2017/12/08/make-a-raspberry-pi-pandora-player/)

## ToDo:
- [X] Order DAC (HiFiBerry AMP2)
- [ ] Wire up volume knob [try this?](https://gist.github.com/thijstriemstra/6396142f426aeffb0c1c6507fb2acd7b)
- [X] Add platform for varicap tuner base
- [ ] find way to wire varicap to pot
- [ ] Wire up tuner knob (same way as volume knob?)
- [ ] Switch to turn off microphone
- [ ] Switch for changing music source? (radio, YouTube, Pandora, Spotify?)
- [ ] Mute button
- [ ] Stop button
- [ ] Wire up as much as I can for temporary use even if not using yet.
- [ ] Wire lights to on/off switch
- [ ] Since stereoio can be controlled by voice, have a physical switch to read physical settings and change if different from settings generated by voice control.
- [X] systemctl service files for mpv radio stations.
- [ ] Scripted construction of service files.
- [ ] Fork GassistPi to add support for my multi-station idea.
- [ ] gpiozero based physical controls ([handy example](https://github.com/themagpimag/essentials-gpiozero/blob/master/09-Potentiometer/ch9listing4.py))
- [ ] Can I script the edits to visudo?


For rotary encoders:
https://gitlab.com/tibernut/radioPi
