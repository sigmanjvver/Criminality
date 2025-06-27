-- Services and locals
local Players = game:GetService("Players")
local LocalPlayer = Players.LocalPlayer
local Mouse = LocalPlayer:GetMouse()
local RunService = game:GetService("RunService")

-- Silent Aim internal state (local)
local SilentAim = false
local SilentAimFOV = 100
local SilentAimPart = "Head"

-- Drawing FOV Circle setup
local Drawing = Drawing or error("Drawing API not available")
local FOVCircle = Drawing.new("Circle")
FOVCircle.Transparency = 1
FOVCircle.Thickness = 2
FOVCircle.Color = Color3.fromRGB(255, 0, 0)
FOVCircle.Filled = false
FOVCircle.Visible = false

-- Update FOV Circle position and visibility
RunService.RenderStepped:Connect(function()
    if SilentAim then
        FOVCircle.Position = Vector2.new(Mouse.X, Mouse.Y + 36) -- UI offset
        FOVCircle.Radius = SilentAimFOV
        FOVCircle.Visible = true
    else
        FOVCircle.Visible = false
    end
end)

-- Find closest target within FOV based on selected part
local function getClosestTarget()
    local closestPlayer = nil
    local shortestDistance = math.huge
    for _, plr in pairs(Players:GetPlayers()) do
        if plr ~= LocalPlayer and plr.Character and plr.Character:FindFirstChild(SilentAimPart) and plr.Character:FindFirstChild("Humanoid") and plr.Character.Humanoid.Health > 0 then
            local partPos, onScreen = workspace.CurrentCamera:WorldToViewportPoint(plr.Character[SilentAimPart].Position)
            if onScreen then
                local mousePos = Vector2.new(Mouse.X, Mouse.Y)
                local screenPos = Vector2.new(partPos.X, partPos.Y)
                local dist = (screenPos - mousePos).Magnitude
                if dist < shortestDistance and dist <= SilentAimFOV then
                    shortestDistance = dist
                    closestPlayer = plr
                end
            end
        end
    end
    return closestPlayer
end

-- Metatable hook to override mouse.Hit for silent aim
local mt = getrawmetatable(game)
local oldIndex = mt.__index
setreadonly(mt, false)
mt.__index = newcclosure(function(self, key)
    if self == Mouse and key == "Hit" and SilentAim then
        local targetPlayer = getClosestTarget()
        if targetPlayer and targetPlayer.Character then
            local partName = SilentAimPart
            if partName == "Random" then
                local parts = {"Head", "Torso"}
                partName = parts[math.random(1, #parts)]
            end
            local targetPart = targetPlayer.Character:FindFirstChild(partName)
            if targetPart then
                return targetPart.CFrame
            end
        end
    end
    return oldIndex(self, key)
end)
setreadonly(mt, true)

-- Orion UI
local OrionLib = loadstring(game:HttpGet('https://raw.githubusercontent.com/jensonhirst/Orion/main/source'))()
local Window = OrionLib:MakeWindow({
    Name = "CrimHub 🎯",
    HidePremium = false,
    SaveConfig = true,
    ConfigFolder = "CrimHubConfig"
})

local Combat = Window:MakeTab({Name = "Combat", Icon = "rbxassetid://4483345998"})

Combat:AddToggle({
    Name = "Silent Aim",
    Default = false,
    Callback = function(state)
        SilentAim = state
    end
})

Combat:AddSlider({
    Name = "Silent Aim FOV",
    Min = 50,
    Max = 300,
    Default = 100,
    Increment = 5,
    ValueName = "pixels",
    Callback = function(val)
        SilentAimFOV = val
    end
})

Combat:AddDropdown({
    Name = "Silent Aim Target Part",
    Default = "Head",
    Options = {"Head", "Torso", "Random"},
    Callback = function(selected)
        SilentAimPart = selected
    end
})
