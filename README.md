-- LocalScript para StarterPlayerScripts (funciona mobile)
local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local Camera = workspace.CurrentCamera
local LocalPlayer = Players.LocalPlayer

local RETICLE_RADIUS = 80
local gui = Instance.new("ScreenGui")
gui.Parent = LocalPlayer.PlayerGui
gui.Name = "AimESP_Mobile_GUI"
gui.ResetOnSpawn = false

-- RGB Aim Circle
local aimCircle = Instance.new("ImageLabel")
aimCircle.AnchorPoint = Vector2.new(0.5, 0.5)
aimCircle.Position = UDim2.new(0.5,0,0.5,0)
aimCircle.Size = UDim2.new(0, RETICLE_RADIUS*2, 0, RETICLE_RADIUS*2)
aimCircle.BackgroundTransparency = 1
aimCircle.Image = "rbxassetid://6023426915"
aimCircle.Parent = gui

-- AIM Button (mobile)
local aimButton = Instance.new("TextButton")
aimButton.Size = UDim2.new(0,100,0,100)
aimButton.Position = UDim2.new(1,-110,1,-110)
aimButton.AnchorPoint = Vector2.new(0,1)
aimButton.BackgroundColor3 = Color3.fromRGB(40,40,40)
aimButton.TextColor3 = Color3.fromRGB(255, 255, 255)
aimButton.TextScaled = true
aimButton.Text = "AIM"
aimButton.AutomaticSize = Enum.AutomaticSize.None
aimButton.BorderSizePixel = 0
aimButton.Parent = gui

------------------ ESP ------------------
local function createESPModel(plr)
    if plr.Character and plr.Character:FindFirstChild("HumanoidRootPart") and not plr.Character:FindFirstChild("ESPBox") then
        local box = Instance.new("BoxHandleAdornment")
        box.Name = "ESPBox"
        box.Adornee = plr.Character.HumanoidRootPart
        box.AlwaysOnTop = true
        box.ZIndex = 5
        box.Size = Vector3.new(4,7,2)
        box.Color3 = Color3.new(1,0,0)
        box.Transparency = 0.7
        box.Parent = plr.Character
    end
end

local function updateESPs()
    for _, plr in ipairs(Players:GetPlayers()) do
        if plr ~= LocalPlayer then createESPModel(plr) end
    end
end

Players.PlayerAdded:Connect(function(plr)
    plr.CharacterAdded:Connect(function() task.wait(1) updateESPs() end)
end)
RunService.RenderStepped:Connect(updateESPs)

------------------ AIMBOT UTILS ------------------
local function worldToScreen(pos)
    local v, onScreen = Camera:WorldToViewportPoint(pos)
    return Vector2.new(v.X, v.Y), onScreen, v.Z
end

local function isVisible(part)
    local origin = Camera.CFrame.Position
    local dir = (part.Position - origin)
    local ray = Ray.new(origin, dir)
    local partHit = workspace:FindPartOnRayWithIgnoreList(ray, {LocalPlayer.Character}, false, true)
    return (not partHit) or (partHit:IsDescendantOf(part.Parent))
end

local function getClosestPlayerInAimCircle()
    local closest, minDist = nil, RETICLE_RADIUS
    for _, plr in ipairs(Players:GetPlayers()) do
        if plr ~= LocalPlayer and plr.Character and plr.Character:FindFirstChild("Humanoid") 
        and plr.Character:FindFirstChild("HumanoidRootPart") and plr.Character.Humanoid.Health > 0 then
            local pos,onScreen,depth = worldToScreen(plr.Character.HumanoidRootPart.Position)
            if onScreen and depth > 0 then
                local center = Vector2.new(Camera.ViewportSize.X/2, Camera.ViewportSize.Y/2)
                local dist = (pos-center).Magnitude
                if dist < RETICLE_RADIUS and dist < minDist and isVisible(plr.Character.HumanoidRootPart) then
                    minDist = dist
                    closest = plr
                end
            end
        end
    end
    return closest
end

------------------ RGB Animation & Aimbot ------------------
local aiming = false

aimButton.MouseButton1Down:Connect(function()
    aiming = true
end)
aimButton.MouseButton1Up:Connect(function()
    aiming = false
end)

RunService.RenderStepped:Connect(function()
    local color = Color3.fromHSV((tick()%5)/5,1,1)
    aimCircle.ImageColor3 = color

    if aiming then
        local closest = getClosestPlayerInAimCircle()
        if closest and closest.Character and closest.Character:FindFirstChild("HumanoidRootPart") then
            Camera.CFrame = CFrame.new(Camera.CFrame.Position, closest.Character.HumanoidRootPart.Position)
        end
    end
end)
