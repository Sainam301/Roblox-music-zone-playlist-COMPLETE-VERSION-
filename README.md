# 🛠️ Installation & Placement Guide
To make the system work correctly, place the components in these exact locations in Roblox Studio:
⚬ Script Location: Place your LocalScript directly inside StarterPlayer \rightarrow StarterPlayerScripts.
⚬ Zone Part Naming: Create a Part in Workspace where players stand to hear music. Rename this Part to Song part #1 (or match the exact name set in 		your script). Ensure it is Anchored = true and CanCollide = false.
⚬ Music Audio Files: Drag your Roblox Sound instances directly inside the LocalScript.
⚬ Announcements (Optional): If using announcements, place your announcement Sound object (e.g., MallAnnouncement) directly inside the LocalScript alongside your songs.
# ⚙️ Configuration & Variables
⚬ START_SONG_NAME: Set this string to the exact Name property of the Sound instance inside your script that should play first (e.g., "#2" or "clip").
⚬ AUDIO_VERSION: Selects the active acoustic profile. Options are "Mall", "Concert", or "Hall".
⚬ ANNOUNCEMENT_TRIGGER: Defines when announcements trigger—set to "Every Song" or list target track names in a table like {"Love is"}.
⚬ ANNOUNCEMENTS_ENABLED: A boolean flag (true/false) at the top of the script that lets you toggle announcement triggers on or off completely.
⚬ _G.AudioEffectsEnabled: A global boolean flag (true/false) allowing external scripts to dynamically toggle live spatial audio effects.
⚬ zonePart: Automatically detects your zone boundary part in Workspace using WaitForChild("Song part #1").
# 🎛️ Audio Effects Architecture
Both scripts programmatically instantiate and configure real-time audio effects at runtime:
⚬ EqualizerSoundEffect (clubEQ): Shapes frequency response on active music tracks: ⚬ LowGain: Controls sub-bass and low-end impact (e.g., boost for 	"Concert", cut for "Mall"). ⚬ MidGain: Controls vocal and instrumental warmth across all acoustic environments. ⚬ HighGain: Manages high-frequency 	clarity and sparkle.
⚬ ReverbSoundEffect (clubReverb): Adds spatial width and acoustic dimension: ⚬ DecayTime: Length of the spatial echo tail (e.g., 5.5s for cavernous 	"Hall" acoustics). ⚬ Density & Diffusion: Controls spatial reflection density and environment openness. ⚬ WetLevel: Sets the balance of processed 	environment audio over the raw track.
# 🔍 Output & Console Debugging
	Monitor script status via the Roblox Studio Output Window (View ➔ Output):
⚬	print(): Confirms active zone tracking (✅ Bound to zone part...), song initialization (▶️ Attempting to play song...), and live playback (🔊 		Music playing now!).
⚬	warn(): Alerts you if the designated zone part is missing (⚠️ Zone part missing...) or if no Sound objects are parented inside the script (⚠️ 		No 	sound instances found...).
