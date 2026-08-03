# How to use

Configuration Options & Defaults
⚬	START_SONG_NAME: Set this string to the exact Name property of the Sound instance inside your script that you want to play first (e.g., "Upbeat Pop Track").
⚬	AUDIO_VERSION: Controls which acoustic preset gets loaded on startup. Options are "Custom", "Mall", "Concert", or "Hall".
⚬	ANNOUNCEMENT_INTERVAL: (Announcements script only) Integer defining the delay in seconds between scheduled announcement plays (default is 180).
⚬	_G.AudioEffectsEnabled: Global boolean flag (true/false) that allows external scripts to toggle live audio processing on or off in real time.
⚬	zonePart: Automatically searches workspace for a part named "song", "Song part", or "Song". If found, bounds music playback to that 3D box; if missing, defaults to playing world-wide.
Audio Effects Architecture
Both scripts generate native Roblox audio instances programmatically at runtime and attach them directly to the active playback source:
⚬	EqualizerSoundEffect (clubEQ): Modifies tonal frequencies directly on the playing track:
⚬	LowGain: Bass adjustment in decibels.
⚬	MidGain: Mid-range vocal/instrument presence in decibels.
⚬	HighGain: Treble and high-frequency sparkle in decibels.
⚬	ReverbSoundEffect (clubReverb): Generates 3D spatial depth:
⚬	DecayTime: Length of the echo tail in seconds.
⚬	Density & Diffusion: Echo reflection tightness and space fullness.
⚬	WetLevel: Balance of processed spatial audio mixed over the dry track.
Output & Console Debugging
Roblox Studio routes all logging to the Output window (View tab -> Output):
⚬	print(): Displays track switches (▶️ Attempting to play song...), zone detections (✅ Bound to zone part...), and successful playback triggers (🔊 Music playing now!).
⚬	warn(): Triggers yellow warning messages if the designated zone part cannot be located (⚠️ Zone part missing...) or if no Sound objects are parented inside the LocalScript.

# Free published script down here


# 1. Standard Zone Playlist (No Announcements)

--==============================================================================
-- Script Name: MusicZonePlaylist.lua
-- Description: Zone-based audio player with spatial reverb and equalizer presets.
--==============================================================================

local Players = game:GetService("Players")
local RunService = game:GetService("RunService")

local localPlayer = Players.LocalPlayer

-- Configuration
local START_SONG_NAME = "Upbeat Pop Track" -- Replace with your desired starting track description/name
local AUDIO_VERSION = "Custom" -- Options: "Custom", "Mall", "Concert", "Hall"
_G.AudioEffectsEnabled = true

-- Zone part detection (searches common name variations)
local zonePart = workspace:FindFirstChild("song") 
	or workspace:FindFirstChild("Song part") 
	or workspace:FindFirstChild("Song")

if not zonePart then
	warn("⚠️ [AudioSystem] Zone part missing in Workspace! Sound will default to playing everywhere.")
else
	print("✅ [AudioSystem] Bound to zone part: " .. zonePart.Name)
end

-- Playback & Effects Setup
local soundPlayback = Instance.new("Sound")
soundPlayback.Name = "LocalPlaylistPlayback"
soundPlayback.Parent = script
soundPlayback.Volume = 0.5

local clubReverb = Instance.new("ReverbSoundEffect")
clubReverb.Name = "ClubReverb"
clubReverb.Parent = soundPlayback

local clubEQ = Instance.new("EqualizerSoundEffect")
clubEQ.Name = "ClubEQ"
clubEQ.Parent = soundPlayback

-- Apply Acoustic Presets
if AUDIO_VERSION == "Custom" then
	-- Spatial "Sound Dimension" acoustic profile
	clubReverb.DecayTime = 2.5       
	clubReverb.Density = 0.8         
	clubReverb.Diffusion = 1.0       
	clubReverb.WetLevel = -6         
	clubEQ.LowGain = 12             
	clubEQ.MidGain = 8               
	clubEQ.HighGain = 12             
elseif AUDIO_VERSION == "Mall" then
	clubReverb.DecayTime = 3.8       
	clubReverb.Density = 0.7         
	clubReverb.Diffusion = 1.0       
	clubReverb.WetLevel = -4         
	clubEQ.HighGain = 2              
	clubEQ.MidGain = 4               
	clubEQ.LowGain = -10 
elseif AUDIO_VERSION == "Concert" then
	clubReverb.DecayTime = 3.0       
	clubReverb.Density = 1.0         
	clubReverb.Diffusion = 0.8       
	clubReverb.WetLevel = -3         
	clubEQ.HighGain = -3  
	clubEQ.MidGain = 2    
	clubEQ.LowGain = 8    
elseif AUDIO_VERSION == "Hall" then
	clubReverb.DecayTime = 5.5    
	clubReverb.Density = 0.9         
	clubReverb.Diffusion = 1.0       
	clubReverb.WetLevel = -2         
	clubEQ.HighGain = 1              
	clubEQ.MidGain = 1               
	clubEQ.LowGain = 0    
else
	clubReverb.Enabled = false
	clubEQ.Enabled = false
end

-- Index Child Sound Objects
local songs = {}
local startingSongIndex = 1

for _, child in ipairs(script:GetChildren()) do
	if child:IsA("Sound") and child.Name ~= "LocalPlaylistPlayback" then
		table.insert(songs, child)
	end
end

for index, song in ipairs(songs) do
	if song.Name == START_SONG_NAME then
		startingSongIndex = index
		break
	end
end

-- Zone Spatial Check
local function isPlayerInsideZone(player)
	if not zonePart then return true end
	
	local character = player.Character
	if not character then return false end
	local rootPart = character:FindFirstChild("HumanoidRootPart")
	if not rootPart then return false end

	local playerPos = zonePart.CFrame:PointToObjectSpace(rootPart.Position)
	return math.abs(playerPos.X) <= zonePart.Size.X / 2
		and math.abs(playerPos.Y) <= zonePart.Size.Y / 2
		and math.abs(playerPos.Z) <= zonePart.Size.Z / 2
end

-- Playlist Loop
local currentTrackIndex = startingSongIndex

task.spawn(function()
	if #songs == 0 then 
		warn("⚠️ [AudioSystem] No sound instances found inside script!")
		return 
	end
	
	while true do
		local song = songs[currentTrackIndex]
		soundPlayback.SoundId = song.SoundId
		soundPlayback.TimePosition = 0
		
		if isPlayerInsideZone(localPlayer) then
			soundPlayback:Play()
		end
		
		soundPlayback.Ended:Wait()
		task.wait(1)
		
		currentTrackIndex = currentTrackIndex + 1
		if currentTrackIndex > #songs then
			currentTrackIndex = 1
		end
	end
end)

-- Real-Time State Controller
RunService.RenderStepped:Connect(function()
	clubReverb.Enabled = _G.AudioEffectsEnabled
	clubEQ.Enabled = _G.AudioEffectsEnabled

	if isPlayerInsideZone(localPlayer) then
		if not soundPlayback.IsPlaying and soundPlayback.TimePosition < soundPlayback.TimeLength then
			soundPlayback:Play()
		end
		soundPlayback.Volume = 0.5 
	else
		soundPlayback.Volume = 0 
	end
end)

# 2. Zone Playlist With Announcement System

--==============================================================================
-- Script Name: MusicZonePlaylistWithAnnouncements.lua
-- Description: Zone-based audio player with spatial acoustic tuning and 
--              intermittent announcement ducking/overlay capability.
--==============================================================================

local Players = game:GetService("Players")
local RunService = game:GetService("RunService")

local localPlayer = Players.LocalPlayer

-- Configuration
local START_SONG_NAME = "Upbeat Pop Track" -- Replace with your desired starting track description/name
local AUDIO_VERSION = "Custom"
local ANNOUNCEMENT_INTERVAL = 180 -- Time in seconds between announcements
_G.AudioEffectsEnabled = true

-- Zone part detection
local zonePart = workspace:FindFirstChild("song") 
	or workspace:FindFirstChild("Song part") 
	or workspace:FindFirstChild("Song")

-- Music Playback & Audio Effects
local musicPlayback = Instance.new("Sound")
musicPlayback.Name = "LocalPlaylistPlayback"
musicPlayback.Parent = script
musicPlayback.Volume = 0.5

local clubReverb = Instance.new("ReverbSoundEffect")
clubReverb.Name = "ClubReverb"
clubReverb.Parent = musicPlayback

local clubEQ = Instance.new("EqualizerSoundEffect")
clubEQ.Name = "ClubEQ"
clubEQ.Parent = musicPlayback

-- Announcement Playback Object (Bypasses Reverb/EQ for Clarity)
local announcementPlayback = Instance.new("Sound")
announcementPlayback.Name = "LocalAnnouncementPlayback"
announcementPlayback.Parent = script
announcementPlayback.Volume = 0.8

-- Apply Presets
if AUDIO_VERSION == "Custom" then
	clubReverb.DecayTime = 2.5       
	clubReverb.Density = 0.8         
	clubReverb.Diffusion = 1.0       
	clubReverb.WetLevel = -6         
	clubEQ.LowGain = 12             
	clubEQ.MidGain = 8               
	clubEQ.HighGain = 12             
elseif AUDIO_VERSION == "Mall" then
	clubReverb.DecayTime = 3.8       
	clubReverb.Density = 0.7         
	clubReverb.Diffusion = 1.0       
	clubReverb.WetLevel = -4         
	clubEQ.HighGain = 2              
	clubEQ.MidGain = 4               
	clubEQ.LowGain = -10 
elseif AUDIO_VERSION == "Concert" then
	clubReverb.DecayTime = 3.0       
	clubReverb.Density = 1.0         
	clubReverb.Diffusion = 0.8       
	clubReverb.WetLevel = -3         
	clubEQ.HighGain = -3  
	clubEQ.MidGain = 2    
	clubEQ.LowGain = 8    
elseif AUDIO_VERSION == "Hall" then
	clubReverb.DecayTime = 5.5    
	clubReverb.Density = 0.9         
	clubReverb.Diffusion = 1.0       
	clubReverb.WetLevel = -2         
	clubEQ.HighGain = 1              
	clubEQ.MidGain = 1               
	clubEQ.LowGain = 0    
else
	clubReverb.Enabled = false
	clubEQ.Enabled = false
end

-- Index Songs and Announcements
local songs = {}
local announcements = {}
local startingSongIndex = 1

for _, child in ipairs(script:GetChildren()) do
	if child:IsA("Sound") and child.Name ~= "LocalPlaylistPlayback" and child.Name ~= "LocalAnnouncementPlayback" then
		table.insert(songs, child)
	end
end

local announcementFolder = script:FindFirstChild("Announcements")
if announcementFolder then
	for _, child in ipairs(announcementFolder:GetChildren()) do
		if child:IsA("Sound") then
			table.insert(announcements, child)
		end
	end
end

for index, song in ipairs(songs) do
	if song.Name == START_SONG_NAME then
		startingSongIndex = index
		break
	end
end

-- Zone Check
local function isPlayerInsideZone(player)
	if not zonePart then return true end
	
	local character = player.Character
	if not character then return false end
	local rootPart = character:FindFirstChild("HumanoidRootPart")
	if not rootPart then return false end

	local playerPos = zonePart.CFrame:PointToObjectSpace(rootPart.Position)
	return math.abs(playerPos.X) <= zonePart.Size.X / 2
		and math.abs(playerPos.Y) <= zonePart.Size.Y / 2
		and math.abs(playerPos.Z) <= zonePart.Size.Z / 2
end

-- Playlist Loop
local currentTrackIndex = startingSongIndex

task.spawn(function()
	if #songs == 0 then return end
	
	while true do
		local song = songs[currentTrackIndex]
		musicPlayback.SoundId = song.SoundId
		musicPlayback.TimePosition = 0
		
		if isPlayerInsideZone(localPlayer) then
			musicPlayback:Play()
		end
		
		musicPlayback.Ended:Wait()
		task.wait(1)
		
		currentTrackIndex = currentTrackIndex + 1
		if currentTrackIndex > #songs then
			currentTrackIndex = 1
		end
	end
end)

-- Announcement Interval Loop
task.spawn(function()
	if #announcements == 0 then return end
	
	while true do
		task.wait(ANNOUNCEMENT_INTERVAL)
		
		if isPlayerInsideZone(localPlayer) then
			local randomAnnouncement = announcements[math.random(1, #announcements)]
			announcementPlayback.SoundId = randomAnnouncement.SoundId
			
			-- Duck music volume during announcement
			musicPlayback.Volume = 0.15
			announcementPlayback:Play()
			
			announcementPlayback.Ended:Wait()
			musicPlayback.Volume = 0.5
		end
	end
end)

-- Real-Time State Controller
RunService.RenderStepped:Connect(function()
	clubReverb.Enabled = _G.AudioEffectsEnabled
	clubEQ.Enabled = _G.AudioEffectsEnabled

	if isPlayerInsideZone(localPlayer) then
		if not musicPlayback.IsPlaying and musicPlayback.TimePosition < musicPlayback.TimeLength then
			musicPlayback:Play()
		end
		if not announcementPlayback.IsPlaying then
			musicPlayback.Volume = 0.5
		end
	else
		musicPlayback.Volume = 0
		announcementPlayback.Volume = 0
	end
end)
