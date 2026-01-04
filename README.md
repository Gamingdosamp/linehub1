-- LINE HUB | UNIVERSAL ORGANIZADO
-- Toggle GUI: RightShift

-- SERVICES
local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local UserInputService = game:GetService("UserInputService")
local LocalPlayer = Players.LocalPlayer
local Camera = workspace.CurrentCamera

-- CHARACTER
local function GetChar()
    local c = LocalPlayer.Character or LocalPlayer.CharacterAdded:Wait()
    return c, c:WaitForChild("Humanoid"), c:WaitForChild("HumanoidRootPart")
end

local Character, Humanoid, HRP = GetChar()
LocalPlayer.CharacterAdded:Connect(function()
    Character, Humanoid, HRP = GetChar()
end)

-- ================= GUI =================
local Gui = Instance.new("ScreenGui", game.CoreGui)
Gui.Name = "LineHub"

local Main = Instance.new("Frame", Gui)
Main.Size = UDim2.new(0, 360, 0, 520)
Main.Position = UDim2.new(0.05,0,0.18,0)
Main.BackgroundColor3 = Color3.fromRGB(18,18,18)
Main.BorderSizePixel = 0
Instance.new("UICorner", Main).CornerRadius = UDim.new(0,14)

local Stroke = Instance.new("UIStroke", Main)
Stroke.Color = Color3.fromRGB(90,90,90)

local Title = Instance.new("TextLabel", Main)
Title.Size = UDim2.new(1,0,0,50)
Title.BackgroundTransparency = 1
Title.Text = "⚡ Line Hub"
Title.Font = Enum.Font.GothamBold
Title.TextSize = 24
Title.TextColor3 = Color3.fromRGB(255,255,255)

-- ================= BUTTON SYSTEM =================
local function Button(text, y, callback)
    local b = Instance.new("TextButton", Main)
    b.Size = UDim2.new(0.88,0,0,38)
    b.Position = UDim2.new(0.06,0,0,y)
    b.Text = text.." [OFF]"
    b.Font = Enum.Font.GothamMedium
    b.TextSize = 14
    b.TextColor3 = Color3.new(1,1,1)
    b.BackgroundColor3 = Color3.fromRGB(35,35,35)
    b.BorderSizePixel = 0
    Instance.new("UICorner", b).CornerRadius = UDim.new(0,10)

    local state = false

    b.MouseButton1Click:Connect(function()
        state = not state
        b.Text = text.." ["..(state and "ON" or "OFF").."]"
        b.BackgroundColor3 = state and Color3.fromRGB(55,80,55) or Color3.fromRGB(35,35,35)
        callback(state)
    end)
end

-- ================= FLY (WASD) =================
local Fly = false
local FlySpeed = 65
local BV, BG
local keys = {W=false,A=false,S=false,D=false}

UserInputService.InputBegan:Connect(function(i,g)
    if g then return end
    if keys[i.KeyCode.Name] ~= nil then keys[i.KeyCode.Name] = true end
end)
UserInputService.InputEnded:Connect(function(i)
    if keys[i.KeyCode.Name] ~= nil then keys[i.KeyCode.Name] = false end
end)

RunService.RenderStepped:Connect(function()
    if Fly and HRP then
        local dir = Vector3.zero
        if keys.W then dir += Camera.CFrame.LookVector end
        if keys.S then dir -= Camera.CFrame.LookVector end
        if keys.A then dir -= Camera.CFrame.RightVector end
        if keys.D then dir += Camera.CFrame.RightVector end
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

-- ================= TELEPORT CLICK =================
local TPClick = false
UserInputService.InputBegan:Connect(function(i)
    if TPClick and i.UserInputType == Enum.UserInputType.MouseButton1 then
        local mouse = LocalPlayer:GetMouse()
        if mouse.Hit then
            HRP.CFrame = mouse.Hit + Vector3.new(0,3,0)
        end
    end
end)

-- ================= SPEED =================
local function ToggleSpeed(state)
    Humanoid.WalkSpeed = state and 50 or 16
end

-- ================= JUMP =================
local function ToggleJump(state)
    Humanoid.JumpPower = state and 120 or 50
end

-- ================= NOCLIP =================
local Noclip = false
RunService.Stepped:Connect(function()
    if Noclip and Character then
        for _,v in pairs(Character:GetDescendants()) do
            if v:IsA("BasePart") then
                v.CanCollide = false
            end
        end
    end
end)

-- ================= ESP =================
local ESP = false
local function ToggleESP(state)
    ESP = state
    for _,plr in pairs(Players:GetPlayers()) do
        if plr ~= LocalPlayer and plr.Character then
            if ESP then
                local h = Instance.new("Highlight", plr.Character)
                h.Name = "LineESP"
                h.FillTransparency = 1
                h.OutlineColor = Color3.fromRGB(255,60,60)
            else
                if plr.Character:FindFirstChild("LineESP") then
                    plr.Character.LineESP:Destroy()
                end
            end
        end
    end
end

-- ================= ANTI KNOCKBACK (RoVibes) =================
local AntiKB = false
RunService.Heartbeat:Connect(function()
    if AntiKB and HRP then
        HRP.AssemblyLinearVelocity = Vector3.zero
        HRP.AssemblyAngularVelocity = Vector3.zero
    end
end)

-- ================= BUTTONS =================
Button("✈️ Fly", 70, ToggleFly)
Button("📍 Teleport Click", 115, function(s) TPClick = s end)
Button("🏃 Speed", 160, ToggleSpeed)
Button("🦘 Jump", 205, ToggleJump)
Button("🚫 Noclip", 250, function(s) Noclip = s end)
Button("👁️ ESP", 295, ToggleESP)
Button("🛡️ Anti-Knockback (RoVibes)", 340, function(s) AntiKB = s end)

-- ================= TOGGLE GUI =================
UserInputService.InputBegan:Connect(function(i,g)
    if not g and i.KeyCode == Enum.KeyCode.RightShift then
        Main.Visible = not Main.Visible
    end
end)
