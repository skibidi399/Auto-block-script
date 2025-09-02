-- Separate Fake Block GUI (Top Left, Draggable, Modes Restored, Independent)

local Players = game:GetService("Players")
local localPlayer = Players.LocalPlayer

if game.CoreGui:FindFirstChild("FakeBlockGui") then
    game.CoreGui:FindFirstChild("FakeBlockGui"):Destroy()
end

local gui = Instance.new("ScreenGui")
gui.Name = "FakeBlockGui"
gui.ResetOnSpawn = false
gui.Parent = game.CoreGui

local frame = Instance.new("Frame")
frame.Position = UDim2.new(0, 10, 0, 10)
frame.Size = UDim2.new(0, 170, 0, 140)
frame.BackgroundColor3 = Color3.fromRGB(40, 40, 40)
frame.Active = true
frame.Draggable = true
frame.Parent = gui

local showToggle = Instance.new("TextButton")
showToggle.Position = UDim2.new(0, 5, 0, 115)
showToggle.Size = UDim2.new(0, 160, 0, 20)
showToggle.Text = "Hide GUI"
showToggle.Parent = frame

local normalToggle = Instance.new("TextButton")
normalToggle.Position = UDim2.new(0, 5, 0, 5)
normalToggle.Size = UDim2.new(0, 160, 0, 30)
normalToggle.Text = "Normal Mode"
normalToggle.Parent = frame

local m3toggle = Instance.new("TextButton")
m3toggle.Position = UDim2.new(0, 5, 0, 40)
m3toggle.Size = UDim2.new(0, 160, 0, 30)
m3toggle.Text = "M3&4 Mode"
m3toggle.Parent = frame

local fakeBlockBtn = Instance.new("TextButton")
fakeBlockBtn.Position = UDim2.new(0, 5, 0, 75)
fakeBlockBtn.Size = UDim2.new(0, 160, 0, 30)
fakeBlockBtn.Text = "Fake Block"
fakeBlockBtn.Parent = frame


-- Logic
local mode = "Normal"
normalToggle.MouseButton1Click:Connect(function()
    mode = "Normal"
    normalToggle.BackgroundColor3 = Color3.fromRGB(0, 150, 0)
    m3toggle.BackgroundColor3 = Color3.fromRGB(40, 40, 40)
end)

m3toggle.MouseButton1Click:Connect(function()
    mode = "M3&4"
    m3toggle.BackgroundColor3 = Color3.fromRGB(0, 150, 0)
    normalToggle.BackgroundColor3 = Color3.fromRGB(40, 40, 40)
end)

showToggle.MouseButton1Click:Connect(function()
    frame.Visible = not frame.Visible
end)


local function playFakeBlockAnimation(animId)
    local char = localPlayer.Character
    if char then
        local humanoid = char:FindFirstChildOfClass("Humanoid")
        if humanoid then
            local anim = Instance.new("Animation")
            anim.AnimationId = "rbxassetid://" .. animId
            local track = humanoid:LoadAnimation(anim)
            track:Play()
        end
    end
end


fakeBlockBtn.MouseButton1Click:Connect(function()
    if mode == "Normal" then
        playFakeBlockAnimation("72722244508749") -- Correct Normal Animation
    elseif mode == "M3&4" then
        playFakeBlockAnimation("96959123077498") -- Correct M3&4 Animation
    end
end)
