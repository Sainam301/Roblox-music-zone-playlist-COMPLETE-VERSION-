# 🛠️ Installation & Placement Guide
To make the system work correctly, place the components in these exact locations in Roblox Studio:
⚬	Script Location: Place MusicZonePlaylist (or else) directly inside StarterPlayer ➔ StarterPlayerScripts.
⚬	Zone Part Naming: Create a Part in Workspace where players stand to hear music. Rename this Part to song, Song, or Song part (case-insensitive).
⚬	Music Audio Files: Drag your Roblox Sound instances directly inside the LocalScript.
⚬	Announcements Folder (Announcements Version Only): Create a Folder named Announcements inside the LocalScript, and place your announcement 			Sound objects inside that folder.
# ⚙️ Configuration & Variables
⚬	START_SONG_NAME: Set this string to the exact Name property of the Sound instance inside your script that should play first (e.g., "Upbeat Pop 		Track").
⚬	AUDIO_VERSION: Selects the active acoustic profile. Options are "Custom", "Mall", "Concert", or "Hall".
⚬	ANNOUNCEMENT_INTERVAL: (Announcements script only) Integer defining the delay in seconds between scheduled announcement triggers (default is 		180).
⚬	_G.AudioEffectsEnabled: A global boolean flag (true/false) allowing external scripts to dynamically toggle live spatial audio effects.
⚬	zonePart: Automatically detects your zone boundary part in Workspace. If no matching part exists, the music defaults to playing map-wide.
# 🎛️ Audio Effects Architecture
Both scripts programmatically instantiate and configure real-time audio effects on runtime:
⚬	EqualizerSoundEffect (clubEQ): Shapes frequency response on active music tracks:
⚬	LowGain: Sub-bass and low-end impact (+12dB in Custom mode).
⚬	MidGain: Vocal and instrumental warmth (+8dB in Custom mode).
⚬	HighGain: High-frequency clarity and sparkle (+12dB in Custom mode).
⚬	ReverbSoundEffect (clubReverb): Adds spatial width and acoustic dimension:
⚬	DecayTime: Length of the spatial echo tail (2.5s in Custom mode).
⚬	Density & Diffusion: Spatial reflection density and openness.
⚬	WetLevel: Balance of processed environment audio over the dry track (-6dB in Custom mode).
# 🔍 Output & Console Debugging
	Monitor script status via the Roblox Studio Output Window (View ➔ Output):
⚬	print(): Confirms active zone tracking (✅ Bound to zone part...), song initialization (▶️ Attempting to play song...), and live playback (🔊 		Music playing now!).
⚬	warn(): Alerts you if the designated zone part is missing (⚠️ Zone part missing...) or if no Sound objects are parented inside the script (⚠️ 		No 	sound instances found...).
## 📢 Announcement System Configuration

The announcement version supports three trigger modes controlled by `ANNOUNCEMENT_MODE`:

* **`"EverySong"`**: Automatically picks and plays a random announcement sound immediately after *every* music track finishes.
* **`"CustomSong"`**: Plays a random announcement *only* after a specific song finishes. Set the target song name in `ANNOUNCEMENT_TARGET_SONG = 		"Your Song Name"`.
* **`"Interval"`**: Plays an announcement on a set timer in seconds (configured via `ANNOUNCEMENT_INTERVAL`).

### 🎵 Announcement Sound Names
* Announcement sound objects can be named **anything you like** inside the `Announcements` folder (e.g., `MallRules`, `WelcomeMessage`, 				`SaleNotice`). 
* The script automatically scans the folder and randomly selects one when triggered.
