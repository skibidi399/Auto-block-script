-- Auto Block Rayfield Script (Full Features)
local ReplicatedStorage = game:GetService("ReplicatedStorage")
local TweenService = game:GetService("TweenService")
local RunService = game:GetService("RunService")
local Players = game:GetService("Players")
local lp = Players.LocalPlayer
local PlayerGui = lp:WaitForChild("PlayerGui")
local Humanoid, Animator

local Rayfield = loadstring(game:HttpGet("https://sirius.menu/rayfield"))()
local Window = Rayfield:CreateWindow({
Name = "Auto Block Hub",
LoadingTitle = "Auto Block Script",
LoadingSubtitle = "by Skibidi Shots",
ConfigurationSaving = {
Enabled = true,
FolderName = "AutoBlockHub",
FileName = "Settings"
},
Discord = {Enabled = false},
KeySystem = false
})

local AutoBlockTab = Window:CreateTab("Auto Block", 4483362458)
local PredictiveTab = Window:CreateTab("Predictive Auto Block", 4483362458)
local FakeBlockTab = Window:CreateTab("Fake Block", 4483362458)
local AutoPunchTab = Window:CreateTab("Auto Punch", 4483362458)
local CustomAnimationsTab = Window:CreateTab("Custom Animations", 4483362458)
local MiscTab = Window:CreateTab("Misc", 4483362458)



-- IDs.

-- Audio-based Auto Block IDs
local autoBlockTriggerSounds = {
    ["102228729296384"] = true,
    ["140242176732868"] = true,
    ["112809109188560"] = true,
    ["136323728355613"] = true,
    ["115026634746636"] = true,
    ["84116622032112"] = true,
    ["108907358619313"] = true,
    ["127793641088496"] = true,
    ["86174610237192"] = true,
    ["95079963655241"] = true,
    ["101199185291628"] = true,
    ["119942598489800"] = true,
    ["84307400688050"] = true,
    ["113037804008732"] = true,
    ["105200830849301"] = true,
    ["75330693422988"] = true,
    ["82221759983649"] = true
}

-- Prevent repeated aim triggers for the same animation track
local lastAimTrigger = {}   -- keys = AnimationTrack, value = timestamp when we triggered
local AIM_WINDOW = 0.5      -- how long to aim (seconds)
local AIM_COOLDOWN = 0.6    -- don't retrigger within this many seconds

local autoBlockTriggerAnims = {
    "126830014841198", "126355327951215", "121086746534252", "18885909645",
    "98456918873918", "105458270463374", "83829782357897", "125403313786645",
    "118298475669935", "82113744478546", "70371667919898", "99135633258223",
    "97167027849946", "109230267448394", "139835501033932", "126896426760253",
    "109667959938617", "126681776859538", "129976080405072", "121293883585738",
    "81639435858902", "137314737492715",
    "92173139187970"
}

-- State Variables
local autoBlockOn = false
local autoBlockAudioOn = false
local doubleblocktech = false
local looseFacing = true
local detectionRange = 18

local predictiveBlockOn = false
local edgeKillerDelay = 3
local killerInRangeSince = nil
local predictiveCooldown = 0

local killerNames = {"c00lkidd", "Jason", "JohnDoe", "1x1x1x1", "Noli"}
local autoPunchOn = false
local flingPunchOn = false
local flingPower = 10000
local hiddenfling = false
local aimPunch = false

local customBlockEnabled = false
local customBlockAnimId = ""
local customPunchEnabled = false
local customPunchAnimId = ""

local infiniteStamina = false
local espEnabled = false
local KillersFolder = workspace:WaitForChild("Players"):WaitForChild("Killers")

local lastBlockTime = 0
local lastPunchTime = 0
local lastBlockTpTime = 0

local blockAnimIds = {
"72722244508749",
"96959123077498"
}
local punchAnimIds = {
"87259391926321",
"140703210927645",
"136007065400978",
"136007065400978",
"129843313690921",
"129843313690921",
"86709774283672",
"87259391926321",
"129843313690921",
"129843313690921",
"108807732150251",
"138040001965654",
"86096387000557",
"86096387000557"
}

local customChargeEnabled = false
local customChargeAnimId = ""
local chargeAnimIds = { "106014898528300" }

-- Infinite Stamina
local function enableInfiniteStamina()
    local success, StaminaModule = pcall(function()
        return require(game.ReplicatedStorage.Systems.Character.Game.Sprinting)
    end)
    if not success or not StaminaModule then return end

    StaminaModule.StaminaLossDisabled = true

    task.spawn(function()
        while infiniteStamina and StaminaModule do
            task.wait(0.1)
            StaminaModule.Stamina = StaminaModule.MaxStamina
            StaminaModule.StaminaChanged:Fire()
        end
    end)
end

-- ===== performance improvements for Sound Auto Block =====
-- cached UI / refs
local cachedPlayerGui = PlayerGui
local cachedPunchBtn, cachedBlockBtn, cachedCharges, cachedCooldown = nil, nil, nil, nil
local detectionRangeSq = detectionRange * detectionRange

local function refreshUIRefs()
    -- ensure we have the most up-to-date references for MainUI and ability buttons
    cachedPlayerGui = lp:FindFirstChild("PlayerGui") or PlayerGui
    local main = cachedPlayerGui and cachedPlayerGui:FindFirstChild("MainUI")
    if main then
        local ability = main:FindFirstChild("AbilityContainer")
        cachedPunchBtn = ability and ability:FindFirstChild("Punch")
        cachedBlockBtn = ability and ability:FindFirstChild("Block")
        cachedCharges = cachedPunchBtn and cachedPunchBtn:FindFirstChild("Charges")
        cachedCooldown = cachedBlockBtn and cachedBlockBtn:FindFirstChild("CooldownTime")
    else
        cachedPunchBtn, cachedBlockBtn, cachedCharges, cachedCooldown = nil, nil, nil, nil
    end
end

-- call once at startup
refreshUIRefs()

-- refresh on GUI or character changes (keeps caches fresh)
if cachedPlayerGui then
    cachedPlayerGui.ChildAdded:Connect(function(child)
        if child.Name == "MainUI" then
            task.delay(0.02, refreshUIRefs)
        end
    end)
end

lp.CharacterAdded:Connect(function()
    task.delay(0.5, refreshUIRefs)
end)

-- make detection range squared when user changes it
-- modify your existing DetectionRange callback to also set detectionRangeSq
-- (if you already have a callback, add the detectionRangeSq update)
-- Example: where you set detectionRange = tonumber(Text) or detectionRange
-- change to:
-- detectionRange = tonumber(Text) or detectionRange
-- detectionRangeSq = detectionRange * detectionRange

-- also do the same for any other place where detectionRange can change (predictive tab etc.)


-- Hooked sound routine remains the same but you can remove the pcall(attemptBlockForSound, sound)
-- in the Play/IsPlaying handlers if you want micro speedups — pcall is safe but slightly slower.
-- Example change in hookSound: replace pcall(attemptBlockForSound, sound) with:
-- task.spawn(attemptBlockForSound, sound)
-- or:
-- local ok, err = pcall(attemptBlockForSound, sound)
-- if not ok then warn(err) end


-- GUI Toggles
AutoBlockTab:CreateToggle({
Name = "Auto Block (Animation)",
CurrentValue = false,
Flag = "AutoBlockAnimation",
Callback = function(Value) autoBlockOn = Value end
})

-- Rayfield toggle for Auto Block (Audio)
AutoBlockTab:CreateToggle({
    Name = "Auto Block (Audio)",
    CurrentValue = false,
    Flag = "AutoBlockAudio",
    Callback = function(state)
        autoBlockAudioOn = state
    end,
})

AutoBlockTab:CreateToggle({
    Name = "Double Punch Tech",
    CurrentValue = false,
    Flag = "doubleblockTechtoggle",
    Callback = function(state)
        doubleblocktech = state
    end,
})

AutoBlockTab:CreateParagraph({
    Title = "Recomendation",
    Content = "use audio auto block and use 20 range for it"
})

local facingCheckEnabled = true

AutoBlockTab:CreateToggle({
    Name = "Enable Facing Check",
    CurrentValue = true,
    Flag = "FacingCheckToggle",
    Callback = function(Value)
        facingCheckEnabled = Value
    end
})

AutoBlockTab:CreateDropdown({
Name = "Facing Check",
Options = {"Loose", "Strict"},
CurrentOption = "Loose",
Flag = "FacingCheckMode",
Callback = function(Option) looseFacing = Option == "Loose" end
})

AutoBlockTab:CreateInput({
Name = "Detection Range",
PlaceholderText = "18",
RemoveTextAfterFocusLost = false,
Flag = "DetectionRange",
Callback = function(Text)
detectionRange = tonumber(Text) or detectionRange
detectionRangeSq = detectionRange * detectionRange
end
})

local detectionCircles = {} -- store all killer circles
local killerCirclesVisible = false

-- Function to add circle to a killer
local function addKillerCircle(killer)
    if not killer:FindFirstChild("HumanoidRootPart") then return end
    if detectionCircles[killer] then return end -- already has one

    local circle = Instance.new("CylinderHandleAdornment")
    circle.Name = "KillerDetectionCircle"
    circle.Adornee = killer.HumanoidRootPart
    circle.Color3 = Color3.fromRGB(255, 0, 0) -- red
    circle.AlwaysOnTop = true
    circle.ZIndex = 0
    circle.Transparency = 0.7
    circle.Radius = detectionRange / 1.5 -- diameter matches detectionRange
    circle.Height = 0.1 -- flat like a circle
    circle.CFrame = CFrame.Angles(math.rad(90), 0, 0) -- lay flat
    circle.Parent = killer.HumanoidRootPart

    detectionCircles[killer] = circle
end

-- Function to remove circle from a killer
local function removeKillerCircle(killer)
    if detectionCircles[killer] then
        detectionCircles[killer]:Destroy()
        detectionCircles[killer] = nil
    end
end

-- Refresh all circles
local function refreshKillerCircles()
    for _, killer in ipairs(KillersFolder:GetChildren()) do
        if killerCirclesVisible then
            addKillerCircle(killer)
        else
            removeKillerCircle(killer)
        end
    end
end

-- Keep radius updated
RunService.RenderStepped:Connect(function()
    for killer, circle in pairs(detectionCircles) do
        if circle and circle.Parent then
            circle.Radius = detectionRange / 1.5
        end
    end
end)

-- Hook into killers being added/removed
KillersFolder.ChildAdded:Connect(function(killer)
    if killerCirclesVisible then
        task.spawn(function()
            -- Wait until HRP exists (max 5s timeout)
            local hrp = killer:WaitForChild("HumanoidRootPart", 5)
            if hrp then
                addKillerCircle(killer)
            end
        end)
    end
end)

KillersFolder.ChildRemoved:Connect(function(killer)
    removeKillerCircle(killer)
end)

-- Rayfield toggle
AutoBlockTab:CreateToggle({
    Name = "Range Visual",
    CurrentValue = false,
    Flag = "KillerCircleToggle",
    Callback = function(state)
        killerCirclesVisible = state
        refreshKillerCircles()
    end
})

AutoBlockTab:CreateParagraph({
    Title = "⚠️ Warning",
    Content = "DONT USE FAKE BLOCK WHILE USING BLOCK TP"
})

AutoBlockTab:CreateParagraph({
    Title = "",
    Content = "INCREASE RANGE WHEN UR USING BLOCK TP TOO, ATLEAST LIKE 30 OR SMTH"
})

local blockTPEnabled = false

AutoBlockTab:CreateToggle({
    Name = "Block TP",
    CurrentValue = false,
    Callback = function(Value)
        blockTPEnabled = Value
    end
})


PredictiveTab:CreateToggle({
    Name = "Predictive Auto Block",
    CurrentValue = false,
    Callback = function(Value)
        predictiveBlockOn = Value
    end,
})

PredictiveTab:CreateInput({
    Name = "Detection Range",
    PlaceholderText = "10",
    RemoveTextAfterFocusLost = false,
    Callback = function(text)
        local num = tonumber(text)
        if num then
            detectionRange = num
        end
    end,
})


PredictiveTab:CreateSlider({
    Name = "Edge Killer",
    Range = {0, 7},
    Increment = 0.1,
    CurrentValue = 3,
    Flag = "edgekillerlmao",
    Callback = function(val)
        edgeKillerDelay = val
    end,
})

PredictiveTab:CreateParagraph({
    Title = "Edge Killer",
    Content = "how many secs until it blocks (resets when killer gets out of range)"
})

FakeBlockTab:CreateButton({
    Name = "Load Fake Block",
    Callback = function()
        pcall(function()
            local fakeGui = PlayerGui:FindFirstChild("FakeBlockGui")
            if not fakeGui then
                local success, result = pcall(function()
                    return loadstring(game:HttpGet("https://pastebin.com/raw/ztnYv27k"))()
                end)
                if not success then
                    warn("❌ Failed to load Fake Block GUI:", result)
                end
            else
                fakeGui.Enabled = true
                print("✅ Fake Block GUI enabled")
            end
        end)
    end
})

AutoPunchTab:CreateToggle({
Name = "Auto Punch",
CurrentValue = false,
Flag = "AutoPunchToggle",
Callback = function(Value) autoPunchOn = Value end
})

AutoPunchTab:CreateToggle({
Name = "Fling Punch",
CurrentValue = false,
Callback = function(Value) flingPunchOn = Value end
})

AutoPunchTab:CreateToggle({
Name = "Punch Aimbot",
CurrentValue = false,
Flag = "PunchAimToggle",
Callback = function(Value) aimPunch = Value end
})

local predictionValue = 4

AutoPunchTab:CreateSlider({
    Name = "Aim Prediction",
    Range = {0, 10},
    Increment = 0.1,
    Suffix = "studs",
    CurrentValue = predictionValue,
    Flag = "PredictionSlider",
    Callback = function(Value)
        predictionValue = Value
    end,
})

AutoPunchTab:CreateSlider({
Name = "Fling Power",
Range = {5000, 50000000000000},
Increment = 1000000,
CurrentValue = 10000,
Flag = "FlingPower",
Callback = function(Value) flingPower = Value end
})

-- Custom Block Animation
CustomAnimationsTab:CreateInput({
    Name = "Custom Block Animation",
    PlaceholderText = "AnimationId",
    RemoveTextAfterFocusLost = false,
    Flag = "customblockid",
    Callback = function(Text) customBlockAnimId = Text end
})

CustomAnimationsTab:CreateToggle({
Name = "Enable Custom Block Animation",
CurrentValue = false,
Callback = function(Value) customBlockEnabled = Value end
})

CustomAnimationsTab:CreateInput({
    Name = "Custom Punch Animation (not for M3/M4)",
    PlaceholderText = "AnimationId",
    RemoveTextAfterFocusLost = false,
    Flag = "custompunchid",
    Callback = function(Text) customPunchAnimId = Text end
})

CustomAnimationsTab:CreateToggle({
Name = "Enable Custom Punch Animation",
CurrentValue = false,
Callback = function(Value) customPunchEnabled = Value end
})

CustomAnimationsTab:CreateInput({
    Name = "Charge Animation ID",
    PlaceholderText = "Put animation ID here",
    RemoveTextAfterFocusLost = false,
    Flag = "customchargeid",
    Callback = function(input)
        customChargeAnimId = input
    end,
})

CustomAnimationsTab:CreateToggle({
    Name = "Custom Charge Animation",
    CurrentValue = false,
    Callback = function(value)
        customChargeEnabled = value
    end,
})

-- Button to run Infinite Yield
MiscTab:CreateButton({
    Name = "Run Infinite Yield",
    Callback = function()
        loadstring(game:HttpGet("https://raw.githubusercontent.com/EdgeIY/infiniteyield/master/source"))()
    end,
})

-- Message below the button
MiscTab:CreateParagraph({
    Title = "Tip",
    Content = 'Run Infinite Yield and type "antifling" so punch fling works better.'
})

MiscTab:CreateToggle({
    Name = "Infinite Stamina",
    CurrentValue = false,
    Flag = "InfStamina",
    Callback = function(value)
        infiniteStamina = value
        if infiniteStamina then
            enableInfiniteStamina()
        else
            local success, StaminaModule = pcall(function()
                return require(game.ReplicatedStorage.Systems.Character.Game.Sprinting)
            end)
            if success and StaminaModule then
                StaminaModule.StaminaLossDisabled = false
            end
        end
    end
})


local function addESP(obj)
    if not obj:IsA("Model") then return end
    if not obj:FindFirstChild("HumanoidRootPart") then return end

    local plr = Players:GetPlayerFromCharacter(obj)
    if not plr then return end -- ✅ only add ESP if it's a player character

    -- Prevent duplicates
    if obj:FindFirstChild("ESP_Highlight") then return end

    -- Highlight
    local highlight = Instance.new("Highlight")
    highlight.Name = "ESP_Highlight"
    highlight.FillColor = Color3.fromRGB(255, 0, 0)
    highlight.OutlineColor = Color3.fromRGB(255, 255, 255)
    highlight.DepthMode = Enum.HighlightDepthMode.AlwaysOnTop
    highlight.Adornee = obj
    highlight.Parent = obj

    -- Billboard
    local billboard = Instance.new("BillboardGui")
    billboard.Name = "ESP_Billboard"
    billboard.Size = UDim2.new(0, 100, 0, 50)
    billboard.AlwaysOnTop = true
    billboard.Adornee = obj:FindFirstChild("HumanoidRootPart")
    billboard.Parent = obj

    local textLabel = Instance.new("TextLabel")
    textLabel.Name = "ESP_Text"
    textLabel.Size = UDim2.new(1, 0, 1, 0)
    textLabel.BackgroundTransparency = 1
    textLabel.TextColor3 = Color3.fromRGB(255, 0, 0)
    textLabel.TextScaled = true
    textLabel.Font = Enum.Font.SourceSansBold
    textLabel.Text = obj.Name
    textLabel.Parent = billboard
end

-- Function to clear ESP
local function clearESP(obj)
    if obj:FindFirstChild("ESP_Highlight") then
        obj.ESP_Highlight:Destroy()
    end
    if obj:FindFirstChild("ESP_Billboard") then
        obj.ESP_Billboard:Destroy()
    end
end

-- Function to refresh all ESP
local function refreshESP()
    if not espEnabled then
        for _, killer in pairs(KillersFolder:GetChildren()) do
            clearESP(killer)
        end
        return
    end

    for _, killer in pairs(KillersFolder:GetChildren()) do
        addESP(killer)
    end
end


-- Modify ChildAdded connection:
KillersFolder.ChildAdded:Connect(function(child)
    if espEnabled then
        task.wait(0.1) -- wait for HRP
        addESP(child)
    end
end)


KillersFolder.ChildRemoved:Connect(function(child)
    clearESP(child)
end)

-- Distance updater
RunService.RenderStepped:Connect(function()
    if not espEnabled then return end
    local char = lp.Character
    local hrp = char and char:FindFirstChild("HumanoidRootPart")
    if not hrp then return end

    for _, killer in pairs(KillersFolder:GetChildren()) do
        local billboard = killer:FindFirstChild("ESP_Billboard")
        if billboard and billboard:FindFirstChild("ESP_Text") and killer:FindFirstChild("HumanoidRootPart") then
            local dist = (killer.HumanoidRootPart.Position - hrp.Position).Magnitude
            billboard.ESP_Text.Text = string.format("%s\n[%d]", killer.Name, dist)
        end
    end
end)

MiscTab:CreateToggle({
    Name = "Killer ESP",
    CurrentValue = false,
    Flag = "KillerESP_Toggle",
    Callback = function(Value)
        espEnabled = Value
        refreshESP()
    end,
})

-- Helpers
local function fireRemoteBlock()
local args = {"UseActorAbility", "Block"}
ReplicatedStorage:WaitForChild("Modules"):WaitForChild("Network"):WaitForChild("RemoteEvent"):FireServer(unpack(args))
end

local function fireRemotePunch()
local args = {"UseActorAbility", "Punch"}
ReplicatedStorage:WaitForChild("Modules"):WaitForChild("Network"):WaitForChild("RemoteEvent"):FireServer(unpack(args))
end

local function isFacing(localRoot, targetRoot)
    if not facingCheckEnabled then
        return true
    end

    local dir = (localRoot.Position - targetRoot.Position).Unit
    local dot = targetRoot.CFrame.LookVector:Dot(dir)
    return looseFacing and dot > -0.3 or dot > 0
end

local function playCustomAnim(animId, isPunch)
    if not Humanoid then
        warn("Humanoid missing")
        return
    end

    if not animId or animId == "" then
        warn("No animation ID provided")
        return
    end

    local now = tick()
    local lastTime = isPunch and lastPunchTime or lastBlockTime
    if now - lastTime < 1 then
        return
    end

    -- Stop other known anims
    for _, track in ipairs(Humanoid:GetPlayingAnimationTracks()) do
        local animNum = tostring(track.Animation.AnimationId):match("%d+")
        if table.find(isPunch and punchAnimIds or blockAnimIds, animNum) then
            track:Stop()
        end
    end

    -- Create and load the animation
    local anim = Instance.new("Animation")
    anim.AnimationId = "rbxassetid://" .. animId

    local success, track = pcall(function()
        return Humanoid:LoadAnimation(anim)
    end)

    if success and track then
        print("✅ Playing custom " .. (isPunch and "punch" or "block") .. " animation:", animId)
        track:Play()
        if isPunch then
            lastPunchTime = now
        else
            lastBlockTime = now
        end
    else
        warn("❌ Failed to load or play custom animation: " .. animId)
    end
end

-- Fling coroutine
coroutine.wrap(function()
    local hrp, c, vel, movel = nil, nil, nil, 0.1
    while true do
        RunService.Heartbeat:Wait()
        if hiddenfling then
            while hiddenfling and not (c and c.Parent and hrp and hrp.Parent) do
                RunService.Heartbeat:Wait()
                c = lp.Character
                hrp = c and c:FindFirstChild("HumanoidRootPart")
            end
            if hiddenfling then
                vel = hrp.Velocity
                hrp.Velocity = vel * flingPower + Vector3.new(0, flingPower, 0)
                RunService.RenderStepped:Wait()
                hrp.Velocity = vel
                RunService.Stepped:Wait()
                hrp.Velocity = vel + Vector3.new(0, movel, 0)
                movel = movel * -1
            end
        end
    end
end)()

local cachedAnimator = nil
local function refreshAnimator()
    local char = lp.Character
    if not char then
        cachedAnimator = nil
        return
    end
    local hum = char:FindFirstChildOfClass("Humanoid")
    if hum then
        local anim = hum:FindFirstChildOfClass("Animator")
        cachedAnimator = anim or nil
    else
        cachedAnimator = nil
    end
end

lp.CharacterAdded:Connect(function(char)
    task.wait(0.5) -- allow Humanoid/Animator to be created
    refreshAnimator()
end)

-- ===== Robust Sound Auto Block (replace your current Sound Auto Block) =====
local soundHooks = {}     -- [Sound] = {playedConn, propConn, destroyConn}
local soundBlockedUntil = {} -- [Sound] = timestamp when we can block again (throttle)

local function getNearestKillerRoot(maxDist)
    local killersFolder = workspace:FindFirstChild("Players") and workspace.Players:FindFirstChild("Killers")
    if not killersFolder then return nil end

    local myRoot = lp.Character and lp.Character:FindFirstChild("HumanoidRootPart")
    if not myRoot then return nil end

    local closest, closestDist = nil, maxDist or math.huge
    for _, killer in ipairs(killersFolder:GetChildren()) do
        local hrp = killer:FindFirstChild("HumanoidRootPart")
        if hrp then
            local dist = (hrp.Position - myRoot.Position).Magnitude
            if dist < closestDist then
                closest, closestDist = hrp, dist
            end
        end
    end
    return closest
end

local function extractNumericSoundId(sound)
    if not sound or not sound.SoundId then return nil end
    local sid = tostring(sound.SoundId)

    -- Prefer numeric id if present
    local num = sid:match("%d+")
    if num then return num end

    -- Fallbacks (these won't match your numeric whitelist, but kept for completeness)
    local hash = sid:match("[&%?]hash=([^&]+)")
    if hash then return "&hash="..hash end
    local path = sid:match("rbxasset://sounds/.+")
    if path then return path end

    return nil
end

local function getSoundWorldPosition(sound)
    if not sound then return nil end
    if sound.Parent and sound.Parent:IsA("BasePart") then
        return sound.Parent.Position, sound.Parent
    end
    if sound.Parent and sound.Parent:IsA("Attachment") and sound.Parent.Parent and sound.Parent.Parent:IsA("BasePart") then
        return sound.Parent.Parent.Position, sound.Parent.Parent
    end
    -- deep search for any BasePart ancestor/descendant
    local found = sound.Parent and sound.Parent:FindFirstChildWhichIsA("BasePart", true)
    if found then
        return found.Position, found
    end
    return nil, nil
end

local function getCharacterFromDescendant(inst)
    if not inst then return nil end
    local model = inst:FindFirstAncestorOfClass("Model")
    if model and model:FindFirstChildOfClass("Humanoid") then
        return model
    end
    return nil
end



-- ===== optimized attemptBlockForSound =====
local function attemptBlockForSound(sound)
    if not autoBlockAudioOn then return end
    if not sound or not sound:IsA("Sound") then return end
    if not sound.IsPlaying then return end

    -- id early (string of digits)
    local id = extractNumericSoundId(sound)
    if not id or not autoBlockTriggerSounds[id] then return end

    -- throttle per-sound
    local t = tick()
    if soundBlockedUntil[sound] and t < soundBlockedUntil[sound] then return end

    -- get my root fast
    local myChar = lp.Character
    local myRoot = myChar and myChar:FindFirstChild("HumanoidRootPart")
    if not myRoot then return end

    -- get world position and part
    local soundPos, soundPart = getSoundWorldPosition(sound)
    if not soundPos or not soundPart then return end

    -- quick check: is sound coming from a player character (not an object/environment)
    local char = getCharacterFromDescendant(soundPart)
    local plr = char and Players:GetPlayerFromCharacter(char)
    if not plr or plr == lp then return end

    -- get the originating HRP and compare squared distance
    local hrp = char:FindFirstChild("HumanoidRootPart")
    if not hrp then return end

    local dvec = hrp.Position - myRoot.Position
    local distSq = dvec.X * dvec.X + dvec.Y * dvec.Y + dvec.Z * dvec.Z
    if distSq > detectionRangeSq then
        return
    end

    -- block cooldown UI check (cached)
    if cachedCooldown and cachedCooldown.Text ~= "" then
        return
    end

    -- facing check (use the same existing function)
    if facingCheckEnabled and not isFacing(myRoot, hrp) then
        return
    end

    -- all checks passed -> block
    fireRemoteBlock()

    -- double tech if enabled and cachedCharges present
    if doubleblocktech and cachedCharges and cachedCharges.Text == "1" then
        fireRemotePunch()
    end

    -- throttle this sound for a short time to avoid spamming
    soundBlockedUntil[sound] = t + 1.2
end

local function hookSound(sound)
    if not sound or not sound:IsA("Sound") then return end
    if soundHooks[sound] then return end -- already hooked

    local playedConn = sound.Played:Connect(function()
        -- handle immediate play
        pcall(attemptBlockForSound, sound)
    end)

    local propConn = sound:GetPropertyChangedSignal("IsPlaying"):Connect(function()
        if sound.IsPlaying then
            pcall(attemptBlockForSound, sound)
        end
    end)

    local destroyConn
    destroyConn = sound.Destroying:Connect(function()
        -- cleanup
        if playedConn and playedConn.Connected then playedConn:Disconnect() end
        if propConn and propConn.Connected then propConn:Disconnect() end
        if destroyConn and destroyConn.Connected then destroyConn:Disconnect() end
        soundHooks[sound] = nil
        soundBlockedUntil[sound] = nil
    end)

    soundHooks[sound] = {playedConn, propConn, destroyConn}

    -- If it's already playing right now, check it immediately
    if sound.IsPlaying then
        task.spawn(function() pcall(attemptBlockForSound, sound) end)
    end
end

-- Hook existing Sounds across the game (covers workspace, SoundService, Lighting, etc.)
for _, desc in ipairs(game:GetDescendants()) do
    if desc:IsA("Sound") then
        pcall(hookSound, desc)
    end
end

-- Hook any future Sounds
game.DescendantAdded:Connect(function(desc)
    if desc:IsA("Sound") then
        pcall(hookSound, desc)
    end
end)
-- ===== End Robust Sound Auto Block =====

local lastReplaceTime = {
    block = 0,
    punch = 0,
    charge = 0,
}

task.spawn(function()
    while true do
        RunService.Heartbeat:Wait()

        local char = lp.Character
        if not char then continue end

        local humanoid = char:FindFirstChildOfClass("Humanoid")
        local animator = humanoid and humanoid:FindFirstChildOfClass("Animator")
        if not animator then continue end

        for _, track in ipairs(animator:GetPlayingAnimationTracks()) do
            local animId = tostring(track.Animation.AnimationId):match("%d+")

            -- Block animation replacement
            if customBlockEnabled and customBlockAnimId ~= "" and table.find(blockAnimIds, animId) then
                if animId == tostring(customBlockAnimId) then
                    continue -- already custom anim
                end
            
                if tick() - lastReplaceTime.block >= 3 then
                    lastReplaceTime.block = tick()
                    track:Stop()

                    local newAnim = Instance.new("Animation")
                    newAnim.AnimationId = "rbxassetid://" .. customBlockAnimId
                    local newTrack = animator:LoadAnimation(newAnim)
                    newTrack:Play()
                    break
                end
            end

            -- Punch animation replacement
            if customPunchEnabled and customPunchAnimId ~= "" and table.find(punchAnimIds, animId) then
                if animId == tostring(customPunchAnimId) then
                    continue -- already custom anim
                end
            
                if tick() - lastReplaceTime.punch >= 3 then
                    lastReplaceTime.punch = tick()
                    track:Stop()

                    local newAnim = Instance.new("Animation")
                    newAnim.AnimationId = "rbxassetid://" .. customPunchAnimId
                    local newTrack = animator:LoadAnimation(newAnim)
                    newTrack:Play()
                    break
                end
            end

            -- Charge animation replacement
            if customChargeEnabled and customChargeAnimId ~= "" and table.find(chargeAnimIds, animId) then
                if animId == tostring(customChargeAnimId) then
                    continue -- already custom anim
                end

                if tick() - lastReplaceTime.charge >= 3 then
                    lastReplaceTime.charge = tick()
                    track:Stop()

                    local newAnim = Instance.new("Animation")
                    newAnim.AnimationId = "rbxassetid://" .. customChargeAnimId
                    local newTrack = animator:LoadAnimation(newAnim)
                    newTrack:Play()
                    break
                end
            end
        end
    end
end)

-- Auto block + punch detection loop
RunService.RenderStepped:Connect(function()
    local gui = PlayerGui:FindFirstChild("MainUI")
    local punchBtn = gui and gui:FindFirstChild("AbilityContainer") and gui.AbilityContainer:FindFirstChild("Punch")
    local charges = punchBtn and punchBtn:FindFirstChild("Charges")
    local blockBtn = gui and gui:FindFirstChild("AbilityContainer") and gui.AbilityContainer:FindFirstChild("Block")
    local cooldown = blockBtn and blockBtn:FindFirstChild("CooldownTime")

    local myChar = lp.Character
    if not myChar then return end
    local myRoot = myChar:FindFirstChild("HumanoidRootPart")
    Humanoid = myChar:FindFirstChildOfClass("Humanoid")
        -- Auto Block: Trigger block if a valid animation is played by a killer
    for _, plr in ipairs(Players:GetPlayers()) do
        if plr ~= lp and plr.Character then
            local hrp = plr.Character:FindFirstChild("HumanoidRootPart")
            local hum = plr.Character:FindFirstChildOfClass("Humanoid")
            local animTracks = hum and hum:FindFirstChildOfClass("Animator") and hum:FindFirstChildOfClass("Animator"):GetPlayingAnimationTracks()

            if hrp and myRoot and (hrp.Position - myRoot.Position).Magnitude <= detectionRange then
                for _, track in ipairs(animTracks or {}) do
                    local id = tostring(track.Animation.AnimationId):match("%d+")
                    if table.find(autoBlockTriggerAnims, id) then
                        if autoBlockOn and (hrp.Position - myRoot.Position).Magnitude <= detectionRange then
                            if isFacing(myRoot, hrp) then
                                if cooldown and cooldown.Text == "" then
                                    fireRemoteBlock()
                                end
                                if doubleblocktech == true and charges and charges.Text == "1" then
                                    fireRemotePunch()
                                end
                            end
                        end
                    end
                end
            end
        end
    end

    -- Detect if player is playing a block animation, and blockTP is enabled
    if blockTPEnabled and Humanoid and tick() - lastBlockTpTime >= 5 then
        for _, track in ipairs(Humanoid:GetPlayingAnimationTracks()) do
            local animId = tostring(track.Animation.AnimationId):match("%d+")
            if animId == "72722244508749" or animId == "96959123077498" then
                local myRoot = lp.Character and lp.Character:FindFirstChild("HumanoidRootPart")
                if myRoot then
                    local killers = {"c00lkidd", "Jason", "JohnDoe", "1x1x1x1", "Noli"}
                    for _, name in ipairs(killers) do
                        local killer = workspace:FindFirstChild("Players")
                            and workspace.Players:FindFirstChild("Killers")
                            and workspace.Players.Killers:FindFirstChild(name)

                        if killer and killer:FindFirstChild("HumanoidRootPart") then
                            lastBlockTpTime = tick()

                            task.spawn(function()
                                local startTime = tick()
                                while tick() - startTime < 0.5 do
                                    if lp.Character and lp.Character:FindFirstChild("HumanoidRootPart") then
                                        local myRoot = lp.Character.HumanoidRootPart
                                        local targetHRP = killer.HumanoidRootPart
                                        local direction = targetHRP.CFrame.LookVector
                                        local tpPosition = targetHRP.Position + direction * 6
                                        myRoot.CFrame = CFrame.new(tpPosition)
                                    end
                                    task.wait()
                                end
                            end)

                            break
                        end
                    end
                end
                break
            end
        end
    end

    -- Predictive Auto Block: Check killer range and time
    if predictiveBlockOn and tick() > predictiveCooldown then
        local killersFolder = workspace:FindFirstChild("Players") and workspace.Players:FindFirstChild("Killers")
        local myChar = lp.Character
        local myHRP = myChar and myChar:FindFirstChild("HumanoidRootPart")
        local myHum = myChar and myChar:FindFirstChild("Humanoid")

        if killersFolder and myHRP and myHum then
            local killerInRange = false

            for _, killer in ipairs(killersFolder:GetChildren()) do
                local hrp = killer:FindFirstChild("HumanoidRootPart")
                if hrp then
                    local dist = (myHRP.Position - hrp.Position).Magnitude
                    if dist <= detectionRange then
                        killerInRange = true
                        break
                    end
                end
            end

            -- Handle killer entering range
            if killerInRange then
                if not killerInRangeSince then
                    killerInRangeSince = tick()  -- Start the timer when the killer enters the range
                elseif tick() - killerInRangeSince >= edgeKillerDelay then
                    -- Block if the killer has stayed in range long enough
                    fireRemoteBlock()
                    predictiveCooldown = tick() + 2  -- Set cooldown to avoid blocking too quickly again
                    killerInRangeSince = nil  -- Reset the timer
                end
            else
                killerInRangeSince = nil  -- Reset timer if the killer leaves range
            end
        end
    end


    local myChar = lp.Character
    local myRoot = myChar and myChar:FindFirstChild("HumanoidRootPart")
    -- Auto Punch
    if autoPunchOn then
        if charges and charges.Text == "1" then
            
            for _, name in ipairs(killerNames) do
                local killer = workspace:FindFirstChild("Players")
                    and workspace.Players:FindFirstChild("Killers")
                    and workspace.Players.Killers:FindFirstChild(name)
                if killer and killer:FindFirstChild("HumanoidRootPart") then
                    local root = killer.HumanoidRootPart
                    if root and myRoot and (root.Position - myRoot.Position).Magnitude <= 10 then

                        -- Trigger punch GUI button
                        for _, conn in ipairs(getconnections(punchBtn.MouseButton1Click)) do
                            pcall(function()
                                conn:Fire()
                            end)
                        end

                        -- Fling Punch: Constant TP 2 studs in front of killer for 1 second
                        if flingPunchOn then
                            hiddenfling = true
                            local targetHRP = root
                            task.spawn(function()
                                local start = tick()
                                while tick() - start < 1 do
                                    if lp.Character and lp.Character:FindFirstChild("HumanoidRootPart") and targetHRP and targetHRP.Parent then
                                        local frontPos = targetHRP.Position + (targetHRP.CFrame.LookVector * 2)
                                        lp.Character.HumanoidRootPart.CFrame = CFrame.new(frontPos, targetHRP.Position)
                                    end
                                    task.wait()
                                end
                                hiddenfling = false
                            end)
                        end

                        -- Play custom punch animation if enabled
                        if customPunchEnabled and customPunchAnimId ~= "" then
                            playCustomAnim(customPunchAnimId, true)
                        end

                        break -- Only punch one killer per frame
                    end
                end
            end
        end
    end
    if aimPunch then
        if not cachedAnimator then
            refreshAnimator()
        end
        local animator = cachedAnimator
        if animator and myRoot and myChar then
            for _, name in ipairs(killerNames) do
                local killer = workspace:FindFirstChild("Players")
                    and workspace.Players:FindFirstChild("Killers")
                    and workspace.Players.Killers:FindFirstChild(name)
                if killer and killer:FindFirstChild("HumanoidRootPart") then
                    local root = killer.HumanoidRootPart

                    for _, track in ipairs(animator:GetPlayingAnimationTracks()) do
                        -- guard: want only punch tracks (vanilla or custom)
                        local animId = tostring(track.Animation.AnimationId):match("%d+")
                        if table.find(punchAnimIds, animId) or (customPunchAnimId ~= "" and animId == tostring(customPunchAnimId)) then

                            -- Avoid retriggering for the same AnimationTrack within cooldown
                            local last = lastAimTrigger[track]
                            if last and tick() - last < AIM_COOLDOWN then
                                -- already triggered recently for this track -> skip
                            else
                                -- Only trigger when the track is just starting (helps avoid mid/late triggers)
                                local timePos = 0
                                pcall(function() timePos = track.TimePosition or 0 end) -- safe read
                                if timePos <= 0.1 then
                                    -- Lock it so we don't retrigger
                                    lastAimTrigger[track] = tick()

                                    -- Disable autoroate once and aim for AIM_WINDOW seconds
                                    local humanoid = myChar:FindFirstChild("Humanoid")
                                    if humanoid then
                                        humanoid.AutoRotate = false
                                    end

                                    task.spawn(function()
                                        local start = tick()
                                        while tick() - start < AIM_WINDOW do
                                            if myRoot and root and root.Parent then
                                                local predictedPos = root.Position + (root.CFrame.LookVector * predictionValue)
                                                myRoot.CFrame = CFrame.lookAt(myRoot.Position, predictedPos)
                                            end
                                            task.wait()
                                        end
                                        -- restore
                                        if humanoid then
                                            humanoid.AutoRotate = true
                                        end

                                        -- cleanup: allow retrigger later
                                        task.delay(AIM_COOLDOWN - AIM_WINDOW, function()
                                            lastAimTrigger[track] = nil
                                        end)
                                    end)
                                end
                            end
                        end
                    end
                end
            end
        end
    end
end)




-- Cooldown tracking for ea
--ch replacement type


-- Readd infinite stamina
game.Players.LocalPlayer.CharacterAdded:Connect(function()
    task.wait(1)
    if infiniteStamina then
        enableInfiniteStamina()
    end
end)

Rayfield:LoadConfiguration()
