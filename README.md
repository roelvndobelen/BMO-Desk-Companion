# BMO — My Raspberry Pi Desk Companion

A personal Adventure Time-inspired build combining an animated character, voice assistant, touchscreen controls, and useful home integrations.

BMO runs on a Raspberry Pi 5 inside a customized 3D-printed enclosure. The goal is to give him a little personality while making him genuinely useful on my desk.

This page documents the features, hardware, installed software, and operation of my build. It does not contain application code or installation commands. The interface and voice commands are currently configured in Dutch; this overview is written in English.

Documentation updated: **21 September 2026**. Software inventory was inspected on the Raspberry Pi on that date.

## Inspiration and credits

A huge shout-out to **brenpoly**: their BMO build inspired me to start this project and develop my own version with additional features and personal touches.

- [Original BMO build video](https://www.youtube.com/watch?v=l5ggH-YhuAw)
- [brenpoly's Be More Agent project](https://github.com/brenpoly/be-more-agent)
- [Original BMO enclosure model on Printables](https://www.printables.com/model/1582055-bmo-from-adventure-time-local-ai-agent-project)

The original project's advertised capabilities and parts list are not automatically the same as this build. In particular, my current version combines local functions with cloud AI rather than running entirely offline.

## What BMO can do

### Voice conversations

- Responds to the “Hey BMO” wake phrase.
- Answers questions aloud with a friendly, playful personality.
- Keeps a short recent conversation history to understand follow-up questions; this is not permanent, unlimited memory.
- Listens for an answer after asking a follow-up question, without requiring the wake phrase again.
- Can use web search when a question calls for current information.
- Offers a touchscreen button to start an interaction.
- Uses cloud speech recognition with a local transcription fallback.

### Expressive face and personality

- Displays a familiar happy face when idle.
- Changes expression while listening, thinking, searching, and speaking.
- Animates the mouth using the strength of the actual speech audio.
- Includes curious, confused, excited, proud, winking, shy, musical, bored, surprised, wide-eyed wonder, and sleeping expressions.
- Keeps the expressions visually consistent through shared proportions.
- Uses selected expressions for relevant events, such as compliments or a completed print.
- Keeps the worried and sad-looking error expressions disabled; those situations use the normal face instead.
- Adds blinking, occasional winks, small smiles, and brief touch reactions.
- Provides an expression selection page for trying the extra faces manually.

Expressions are animated reactions, not evidence of actual feelings or reliable emotion recognition. Not every expression is automatically triggered in every situation.

### The red balloon

Occasionally, a little red balloon floats across the screen for about twelve seconds. BMO switches to his glossy, wide-eyed expression and follows its position with his eyes. His normal face returns afterwards.

The effect is scheduled roughly every 90–180 seconds of eligible idle time. Talking, listening, an open menu, and other foreground activity take priority, so it is not a strict wall-clock schedule. It is silent and does not require an AI request.

### Touchscreen hamburger menu

The touchscreen provides access to conversation controls, volume, timers, music, lights, display settings, system status, printer information, expression previews, and Pi-hole statistics.

### Volume and display settings

- Adjusts speaker volume through touch controls or spoken percentage commands.
- Uses an 80% startup volume setting.
- Provides brightness presets and a night setting.
- Saves the selected normal brightness locally.
- Supports waking a dimmed or blanked display by touch.

### Presence-based sleep and wake

- Enters a quieter, reduced-activity mode after five minutes without detecting someone, when no active interaction prevents sleep.
- Dims the screen to 20% rather than turning it off.
- Wakes when someone is detected again and restores the normal display brightness.
- Makes these automatic transitions without sound effects.
- Keeps the camera running so automatic wake-up remains possible.

The present implementation uses camera-based face detection as a proxy for presence. It does not identify people and can miss someone who is turned away or poorly lit. Sleep is an application mode, not Raspberry Pi suspend or shutdown; background services such as DNS remain running.

### Camera-assisted descriptions

- Captures an image on request so BMO can describe what is in front of him.
- Uses the Raspberry Pi AI Camera for image capture and presence sensing.
- Shows the exact captured photo on the touchscreen during analysis and the spoken description, rather than showing a different live frame.
- Returns to the face when the spoken answer finishes.
- Includes a Close button to dismiss the photo early without stopping the description.
- Uses a temporary, private runtime file rather than a photo archive. Normal completion removes it; the display also has a two-minute expiry safeguard while running.

Visual descriptions use cloud AI. Owning an AI Camera does not mean this build currently performs all vision processing on its onboard accelerator.

### Timers

- Creates and cancels timers through voice commands.
- Offers touchscreen presets of 1, 5, 10, 15, and 30 minutes.
- Shows a countdown on the display, with a larger timer view available.
- Provides a timer-finished alert.

### Music and smart-home controls

- Controls configured Spotify playback: resume, pause, next track, and previous track.
- Shows available now-playing information, including track and artist.
- Can obtain compatible Sonos now-playing information through Home Assistant.
- Controls configured living-room lights through Home Assistant.

These features depend on the connected accounts, compatible devices, and integration availability. BMO is not a standalone music-streaming service or a universal controller for every smart-home product.

### More careful Home Assistant commands

The home-control layer now distinguishes questions from actions instead of acting on isolated words such as “on” or “off.” It is a bounded Dutch-language command parser, not an unrestricted autonomous AI agent.

- Reads device status for questions such as “Are the lights on?” without switching anything.
- Keeps the most recent home-device context for 90 seconds, allowing a follow-up such as “Turn that off again.”
- Asks which device is meant when a request is ambiguous.
- Asks for confirmation before turning “It is dark here” into a lighting action.
- Understands volume percentages written as digits or Dutch number words.
- Checks target availability before sending supported commands.
- Avoids redundant power commands when the device already reports the desired state.
- Gives a short “Oké” acknowledgement after an accepted command, rather than announcing technical service calls.
- Requests separate instructions for conflicting or mixed actions instead of guessing.
- Does not silently turn requests with negation, exceptions, or scheduling conditions into immediate commands.

The configured targets are two living-room lamps connected to switchable plugs, a Samsung television, and a Sonos Beam. The lamp plugs support on/off, not brightness or color control. Other rooms and devices are not automatically discovered or authorized. Service acceptance is not a guarantee that a physical device changed state.

## How BMO works

### From a spoken question to an answer

1. **Listen locally:** the microphone provides audio to the local wake-word detector. BMO waits for “Hey BMO,” unless a follow-up listening window is already open.
2. **Record the request:** after activation, BMO records the spoken request and detects when speech ends.
3. **Recognize speech:** the normal route uses cloud transcription. A local Whisper model is available as a transcription fallback.
4. **Choose the handler:** supported volume, camera, timer, home-control, and music requests are handled by their dedicated modules. Other questions go to the conversational AI.
5. **Use context:** the conversational assistant keeps up to four recent conversation turns. Home-control references use their separate, shorter 90-second context.
6. **Speak:** the text answer is sent to the configured speech service. Audio is played incrementally as it arrives rather than waiting for the entire recording.
7. **Animate and listen again:** mouth movement follows audio amplitude. If the answer contains a follow-up question, BMO opens another listening window; otherwise, he returns to waiting for the wake phrase.

The current speech voice is **Marin**. If cloud speech fails, the application attempts an **eSpeak NG** fallback. Streaming removes the former full-recording download wait, but transcription, answer generation, network latency, and time to first speech audio still cause delays. This is amplitude-based mouth animation, not phoneme-perfect lip synchronization. [Speech streaming reference](https://developers.openai.com/api/docs/guides/text-to-speech)

### Camera requests

For “Wat zie je?” (“What do you see?”), the existing camera service captures a new JPEG. The same image is sent to the vision service and displayed locally. BMO describes the returned answer aloud, then clears the temporary image. The camera is not continuously uploading a live stream to the vision service.

### Separate background services

| Component | Responsibility |
| --- | --- |
| Voice service | Wake phrase, recording, transcription, command routing, conversation, and speech output. |
| Face/display service | Animated face, touchscreen menu, photo overlay, and information displays. |
| Camera service | Image capture and local face-based presence detection. |
| Camera watchdog | Periodic camera-health checks and recovery when needed. |
| Home Assistant | State and service access for configured household devices and weather information. |
| Pi-hole | DNS filtering and filtering statistics, independently of the face animation. |

The voice and display processes exchange small local status messages. The camera supplies requested images over a local socket. These interfaces let the face update while other work is happening. Sleep dims the display and pauses selected application activity; it does not shut down Linux, the camera, or DNS.

### What runs locally and what uses the network?

| Function | Where it runs |
| --- | --- |
| Wake-word detection, face animation, menus, timers, presence detection | On the Raspberry Pi. |
| Fallback speech transcription | On the Raspberry Pi using whisper.cpp. |
| Normal conversation, cloud transcription, visual descriptions, main speech voice | Through configured cloud AI services. |
| Printer monitoring | Across the local network to the printer's Moonraker endpoint. |
| Smart-home commands | Through Home Assistant; individual device integrations may themselves depend on the cloud. |
| Music controls | Through Spotify and the configured playback integrations. |
| Weather display | Reads Home Assistant's configured weather entity; its upstream source may need internet access. |
| DNS filtering | On BMO, with permitted queries forwarded to upstream DNS resolvers. |

### Weather and clock

- Displays the current time.
- Shows available weather information, including temperature and a short description.
- Keeps this information alongside the face rather than replacing it with a full dashboard.

### 3D-printer monitoring

- Reads status from the configured Creality printer through its Moonraker interface.
- Shows estimated remaining time above the face only while a print is running.
- Keeps the main face display free of a progress bar and percentage counter.
- Offers a detailed menu page with print state, filename, progress, layer information when available, elapsed time, estimated remaining time, and nozzle/bed temperatures.
- Reports connection problems instead of presenting stale progress as current.

This integration is read-only: it does not start, cancel, or otherwise control prints. Remaining time is an estimate based on the printer's available data, not a guaranteed finish time. [Moonraker printer-status reference](https://moonraker.readthedocs.io/en/latest/external_api/printer/)

### Pi-hole statistics and network filtering

- Runs Pi-hole on the same Raspberry Pi.
- Filters blocklisted domains for devices whose DNS requests pass through it.
- Shows blocking status, DNS-query totals, blocked-query totals, blocked percentage, active clients, and blocklist size in the touchscreen menu.

This does not block every advertisement. Devices using another DNS route may bypass it, and DNS filtering should not be treated as a reliable way to remove YouTube or Twitch video ads. BMO must stay powered on while the network relies on it for DNS. [Pi-hole documentation](https://docs.pi-hole.net/)

### System status and reliability

- Displays Raspberry Pi temperature, memory usage, and storage usage.
- Shows the local Wi-Fi address and the status of the voice, display, and camera services.
- Keeps connection indicators in the status page rather than cluttering the face.
- Uses background services for startup and a camera watchdog for recovery.

## Hardware used

This is the documented parts list for my build, not a copy of the original creator's shopping list. Unconfirmed details are explicitly marked rather than guessed.

| Part | Quantity | Role and reference |
| --- | --- | --- |
| Raspberry Pi 5 | 1 | Main computer. [Official product page](https://www.raspberrypi.com/products/raspberry-pi-5/) |
| Freenove FNK0078 touchscreen | 1 | Face display and touch controls; the application runs at 800 × 480. [Official model documentation](https://github.com/Freenove/Freenove_Touchscreen_Monitor_for_Raspberry_Pi) |
| Raspberry Pi AI Camera | 1 | Image capture and presence sensing. [Official documentation](https://www.raspberrypi.com/documentation/accessories/ai-camera.html) |
| Pimoroni NVMe Base Duo | 1 | Mounted PCIe/NVMe expansion board. [Product](https://shop.pimoroni.com/products/nvme-base-duo-for-raspberry-pi-5) · [Guide](https://learn.pimoroni.com/article/getting-started-with-nvme-base-duo) |
| USB-A extension cable, 30 cm | 2 | Brings USB connections to the front of the enclosure. [Exact linked cable](https://www.kiwi-electronics.com/nl/usb-extensie-kabel-30cm-703) |
| USB microphone | 1 | Linux reports “USB PnP Sound Device”; the retail model is not yet documented. |
| USB speaker/audio output | 1 setup | Linux reports “UACDemoV1.0”; the retail model is not yet documented. |
| Lexar NM620 NVMe SSD | 1 × 512 GB | Detected by Linux as approximately 476.9 GiB. This inventory does not establish which application data is stored on it. |
| Memory card | Approximately 64 GB | Detected as approximately 58.9 GiB; manufacturer and exact model are not yet documented. |
| Power supply and power cable | 1 setup | Powers the assembled build; exact supply model and rating still to be documented. |
| Display, camera, and PCIe connection cables | As fitted | Connects the installed modules; exact lengths and variants still to be documented. |
| 3D-printed enclosure and fittings | 1 set | Body, faceplate, rear access parts, arms, legs, button pieces, and custom mounting/cover parts. |
| PLA / silk PLA filament | As needed | Materials used during the enclosure build; exact brands and per-part materials still to be documented. |
| Fasteners, spacers, and adhesive | As needed | Assembly and mounting; exact sizes and quantities still to be documented. |

The physical buttons are part of the enclosure; this overview does not claim that they are electrically connected or mapped to actions. No battery, UPS, or additional microcontroller is claimed as installed.

### Enclosure modifications

**[3D-print files and part guide](3d-prints/README.md)** — includes the latest three custom parts and a separate folder containing the original downloaded model set. Read the scale and attribution notes before slicing or redistributing.

- Adjusted screen opening and placement.
- Camera opening, mounts, and revised mounting-post heights.
- USB openings and rear connector clearance adapted to the actual extension cables.
- Revised rear access-cover cable opening.
- A separate glue-on cover to hide the interior behind the horizontal front slot.

Several revisions and fit tests were needed. Prototype variants are not all installed simultaneously, and the source model's license must be checked before sharing modified files.

### External equipment

- A Creality 3D printer for fabrication and print-status monitoring; exact model still to be documented.
- A NETGEAR Nighthawk XR500 router used for this build's network and DNS configuration; this specific router is not a requirement.
- Configured smart lights and a Sonos/Spotify playback setup for the corresponding integrations; exact models still to be documented.

## Software and reference projects

These references identify the platforms and tools used by the documented features; they are not an installation guide or a complete dependency lockfile.

### Installed software inventory

The following programs and libraries were found on BMO. Version numbers are a snapshot of this build, not required versions for another installation. This is the relevant application inventory, not every package in the operating system.

| Installed component | Observed version / state | Purpose |
| --- | --- | --- |
| Debian Linux with Raspberry Pi packages | Debian 13, Trixie | Operating system. |
| Python | 3.13.5 | Main application runtime. |
| Tkinter | System package 3.13.5-1 | Touchscreen and animated interface. |
| OpenAI Python SDK | 3.7.0 | Client for configured cloud AI requests. |
| openWakeWord | 0.6.0 | Local wake-phrase detection. |
| ONNX Runtime | 1.29.0 | Local model execution. |
| NumPy | 2.5.2 | Numerical/audio processing. |
| Pillow | 12.3.0 | Image handling and touchscreen photo rendering. |
| Requests | 2.34.2 | HTTP communication with integrations. |
| Spotipy | 2.26.0 | Spotify integration library. |
| Picamera2 | System package 0.3.37-1 | Camera capture. |
| OpenCV | System package 4.10.0+dfsg-5 | Local face detection. |
| ALSA utilities | 1.2.14-1+rpt1 | Audio recording, playback, and mixer controls through arecord, aplay, and amixer. |
| eSpeak NG | 1.52.0+dfsg-5 | Local fallback speech synthesis. |
| FFmpeg | Package 8:7.1.5-0+deb13u1+rpt2 | Installed media-processing utility; not the primary speech player. |
| Docker CE | Package 5:29.7.2-1~debian.13~trixie | Supporting service containers. |
| whisper.cpp | Local build; exact revision not recorded here | Offline transcription fallback. |
| systemd | System service manager | Starts and supervises the BMO services. |

### Downloaded local models and supporting services

- **Wake-word model:** a local ONNX model is present for the wake-phrase detector.
- **Whisper small model:** the local ggml-small model file and whisper-cli executable are present.
- **OpenCV face detector:** used as a presence signal, not for recognizing a person's identity.
- **Home Assistant Container:** previously installed and its API used successfully during integration checks. Its current container image version was not re-audited for this document.
- **Pi-hole container:** previously installed and DNS blocking tested. Its current container image version was not re-audited for this document.
- **HACS:** previously installed in Home Assistant for custom integrations; it is not BMO's conversation engine.
- **Matter server:** previously present alongside Home Assistant; not used by the direct voice-command parser itself.

Ollama was not found on the command path during this inspection, and a Piper folder was not found in the active application directory. Neither is claimed as the active conversation or speech engine of this build. The original inspiration project's local-model setup should not be confused with my cloud-assisted version.

### Software used on the computer

- **Creality Print:** used on the Windows computer to arrange, slice, and send the 3D-printed parts. It does not run BMO's assistant.
- **SSH:** used to administer the Raspberry Pi remotely.
- **Web browser:** used to access Home Assistant, Pi-hole, and router settings.

### References

| Project | Role |
| --- | --- |
| [Raspberry Pi OS](https://www.raspberrypi.com/software/operating-systems/) | Raspberry Pi operating environment. |
| [Python](https://www.python.org/) and [Tkinter](https://docs.python.org/3/library/tkinter.html) | Assistant logic, face animation, and touchscreen UI. |
| [openWakeWord](https://github.com/dscripka/openWakeWord) | Local wake-phrase detection. |
| [whisper.cpp](https://github.com/ggml-org/whisper.cpp) | Local speech-transcription fallback. |
| [OpenAI Python SDK](https://github.com/openai/openai-python) | Cloud conversation, speech, and vision requests in this version. |
| [Picamera2](https://github.com/raspberrypi/picamera2) | Camera capture. |
| [OpenCV](https://opencv.org/) | Local face detection for presence sensing. |
| [Home Assistant](https://www.home-assistant.io/) | Configured smart-home and media integrations. |
| [Spotify Web API](https://developer.spotify.com/documentation/web-api) | Music playback integration. |
| [Moonraker](https://moonraker.readthedocs.io/) | Read-only printer status. |
| [Pi-hole](https://pi-hole.net/) | DNS-based filtering and statistics. |
| [Docker](https://docs.docker.com/) | Hosting supporting services. |
| [Creality Print](https://github.com/CrealityOfficial/CrealityPrint) | Slicing and preparing enclosure prints. |
| [eSpeak NG](https://github.com/espeak-ng/espeak-ng) | Local fallback speech synthesis. |
| [ALSA](https://www.alsa-project.org/) | Linux audio tools. |
| [Pillow](https://python-pillow.github.io/) | Image processing. |
| [Spotipy](https://spotipy.readthedocs.io/) | Spotify client library. |
| [ONNX Runtime](https://onnxruntime.ai/) | Local model runtime. |
| [HACS](https://www.hacs.xyz/) | Home Assistant custom-integration management. |

## Example requests

The build currently expects Dutch commands. English meanings are included for readers, not as a claim that English command parsing is configured.

| Say to BMO | Meaning / result |
| --- | --- |
| “Hey BMO, wat zie je?” | Capture a photo, display it, and describe it aloud. |
| “Staan de lampen aan?” | Read the configured lamps' states without switching them. |
| “Zet de ronde lamp aan.” | Request the named lamp to turn on. |
| “Zet die weer uit.” | Use recent home-device context to request it off. |
| “Zet de lamp aan.” | Ask which lamp is intended. |
| “Het is donker hier.” | Ask for confirmation before turning on the configured lamps. |
| “Zet de Sonos op dertig procent.” | Set the configured Sonos volume. |
| “Zet je geluid naar 80 procent.” | Adjust BMO's own speaker volume. |
| “Zet een timer voor vijf minuten.” | Start a timer. |

## Operating notes

- Keep BMO powered while the network uses it as its DNS server; display sleep is fine, unplugging it is not equivalent.
- Reserve local IP addresses for BMO and the printer, or maintain their configured addresses when DHCP assignments change.
- The printer's status connection and Home Assistant authorization must remain available for those features to work.
- Cloud AI requires configured credentials and may incur usage charges. Voice output is AI-generated.
- A local transcription fallback does not make the full conversational pipeline offline.
- Keep private tokens, environment files, captured images, and personal network configuration out of public repositories.
- Retain backups before updating the application. This document alone is not a runnable release or a complete reproduction guide.

## Privacy and practical limitations

The wake phrase and presence detection are processed locally, but this is **not a fully offline assistant**. Cloud functions can send recorded speech, recognized conversation text, or an explicitly requested camera image to the configured AI provider. Internet access and paid API usage may be required.

Conversation memory is short-term. Presence detection can fail. Printer times are estimates. Connected services can become unavailable. Reduced-activity mode has not been benchmarked for a specific power saving.

Passwords, API keys, private configuration, and personal network addresses are intentionally excluded from this overview.

## Project status

This is an evolving personal build, not a finished commercial product. The functions above reflect the project work documented so far, not a fresh live test of every integration on the publication date.

The remaining documentation work is to complete the exact audio retail models, storage layout, power supply, printer model, cooling, and assembly-hardware details. Readers should not treat this as a fully specified purchasing checklist until those details are filled in.

BMO and Adventure Time belong to their respective rights holders. This is an unofficial fan project, with no claimed affiliation or endorsement. Credit and license requirements for third-party software and models remain with their original creators.
