-- LINE HUB | FIX GUI NOT SHOWING
-- Toggle GUI: B

-- SERVICES
local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local UIS = game:GetService("UserInputService")
local LP = Players.LocalPlayer
local Camera = workspace.CurrentCamera

-- ================= REMOVE GUI ANTIGA =================
pcall(function()
    if LP.PlayerGui:FindFirstChild("LineHub") then
        LP.PlayerGui.LineHub:Destroy()
    end
end)

-- ================= CHARACTER =================
local function GetChar()
    local c = LP.Character or LP.CharacterAdded:Wait()
    return c, c:WaitForChild("Humanoid"), c:WaitForChild("HumanoidRootPart")
end

local Character, Humanoid, HRP = GetChar()
LP.CharacterAdded:Connect(function()
    Character, Humanoid, HRP = GetChar()
end)

-- ================= GUI =================
local Gui = Instance.new("ScreenGui")
Gui.Name = "LineHub"
Gui.ResetOnSpawn = false
Gui.Parent = LP:WaitForChild("PlayerGui") -- AQUI ESTÁ A CORREÇÃO

local Main = Instance.new("Frame", Gui)
Main.Size = UDim2.new(0, 380, 0, 520)
Main.Position = UDim2.new(0.05,0,0.18,0)
Main.BackgroundColor3 = Color3.fromRGB(18,18,18)
Main.BorderSizePixel = 0
Main.Active = true
Main.Draggable = true
Main.Visible = true

Instance.new("UICorner", Main).CornerRadius = UDim.new(0,14)

local Title = Instance.new("TextLabel", Main)
Title.Size = UDim2.new(1,0,0,45)
Title.BackgroundTransparency = 1
Title.Text = "⚡ Line Hub"
Title.Font = Enum.Font.GothamBold
Title.TextSize = 24
Title.TextColor3 = Color3.new(1,1,1)

-- ================= BUTTON SYSTEM =================
local function ToggleButton(text, y, callback)
    local b = Instance.new("TextButton", Main)
    b.Size = UDim2.new(0.88,0,0,36)
    b.Position = UDim2.new(0.06,0,0,y)
    b.Text = text.." [OFF]"
    b.Font = Enum.Font.GothamMedium
    b.TextSize = 14
    b.TextColor3 = Color3.new(1,1,1)
    b.BackgroundColor3 = Color3.fromRGB(35,35,35)
    b.BorderSizePixel = 0
    Instance.new("UICorner", b)

    local state = false
    b.MouseButton1Click:Connect(function()
        state = not state
        b.Text = text.." ["..(state and "ON" or "OFF").."]"
        b.BackgroundColor3 = state and Color3.fromRGB(60,90,60) or Color3.fromRGB(35,35,35)
        callback(state)
    end)
end

-- ================= SPEED CUSTOM =================
local SpeedOn = false
local SpeedValue = 16

local SpeedBox = Instance.new("TextBox", Main)
SpeedBox.Size = UDim2.new(0.88,0,0,30)
SpeedBox.Position = UDim2.new(0.06,0,0,95)
SpeedBox.PlaceholderText = "Speed (ex: 30)"
SpeedBox.Text = ""
SpeedBox.Font = Enum.Font.Gotham
SpeedBox.TextSize = 14
SpeedBox.TextColor3 = Color3.new(1,1,1)
SpeedBox.BackgroundColor3 = Color3.fromRGB(30,30,30)
Instance.new("UICorner", SpeedBox)

SpeedBox.FocusLost:Connect(function()
    local v = tonumber(SpeedBox.Text)
    if v then SpeedValue = math.clamp(v, 16, 150) end
end)

RunService.Heartbeat:Connect(function()
    if SpeedOn then
        Humanoid.WalkSpeed = SpeedValue
    end
end)

-- ================= FLY AVANÇADO =================
local Fly = false
local FlySpeed = 75
local BV, BG
local keys = {}

UIS.InputBegan:Connect(function(i,g)
    if g then return end
    keys[i.KeyCode] = true
end)

UIS.InputEnded:Connect(function(i)
    keys[i.KeyCode] = false
end)

RunService.RenderStepped:Connect(function()
    if Fly and HRP then
        local dir = Vector3.zero
        if keys[Enum.KeyCode.W] then dir += Camera.CFrame.LookVector end
        if keys[Enum.KeyCode.S] then dir -= Camera.CFrame.LookVector end
        if keys[Enum.KeyCode.A] then dir -= Camera.CFrame.RightVector end
        if keys[Enum.KeyCode.D] then dir += Camera.CFrame.RightVector end
        if keys[Enum.KeyCode.Space] then dir += Vector3.new(0,1,0) end
        if keys[Enum.KeyCode.LeftControl] then dir -= Vector3.new(0,1,0) end

        BV.Velocity = dir * FlySpeed
        BG.CFrame = Camera.CFrame
    end
end)

local function ToggleFly(state)
    Fly = state
    if Fly then
        BV = Instance.new("BodyVelocity", HRP)
        BV.MaxForce = Vector3.new(9e9,9e9,9e9)
        BG = Instance.new("BodyGyro", HRP)
        BG.MaxTorque = Vector3.new(9e9,9e9,9e9)
    else
        if BV then BV:Destroy() end
        if BG then BG:Destroy() end
    end
end

-- ================= ANTI KNOCKBACK REAL =================
local AntiKB = false
local lastVel = Vector3.zero

RunService.RenderStepped:Connect(function()
    if not AntiKB or not HRP then return end
    local cur = HRP.AssemblyLinearVelocity
    if (cur - lastVel).Magnitude > 35 then
        HRP.AssemblyLinearVelocity = lastVel
    end
    lastVel = HRP.AssemblyLinearVelocity
end)

-- ================= BUTTONS =================
ToggleButton("🏃 Speed Custom", 140, function(s)
    SpeedOn = s
    if not s then Humanoid.WalkSpeed = 16 end
end)

ToggleButton("✈️ Fly Avançado", 185, ToggleFly)

ToggleButton("🛡️ Anti-Knockback (RoVibes)", 230, function(s)
    AntiKB = s
end)

-- ================= TOGGLE GUI =================
UIS.InputBegan:Connect(function(i,g)
    if not g and i.KeyCode == Enum.KeyCode.B then
        Main.Visible = not Main.Visible
    end
end)
