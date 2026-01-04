-- LINE HUB ULTRA | FINAL + CLICK TP (CTRL)
-- Toggle GUI: B

-- ================= SERVICES =================
local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local UIS = game:GetService("UserInputService")

local LP = Players.LocalPlayer
local Camera = workspace.CurrentCamera
local Mouse = LP:GetMouse()

-- ================= CLEAN GUI =================
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

local Char, Humanoid, HRP = GetChar()
LP.CharacterAdded:Connect(function()
    Char, Humanoid, HRP = GetChar()
end)

-- ================= GUI =================
local Gui = Instance.new("ScreenGui")
Gui.Name = "LineHub"
Gui.ResetOnSpawn = false
Gui.Parent = LP:WaitForChild("PlayerGui")

local Main = Instance.new("Frame", Gui)
Main.Size = UDim2.new(0, 430, 0, 580)
Main.Position = UDim2.new(0.05,0,0.18,0)
Main.BackgroundColor3 = Color3.fromRGB(15,15,15)
Main.BorderSizePixel = 0
Main.Active = true
Main.Draggable = true
Instance.new("UICorner", Main).CornerRadius = UDim.new(0,16)

local Title = Instance.new("TextLabel", Main)
Title.Size = UDim2.new(1,0,0,50)
Title.BackgroundTransparency = 1
Title.Text = "⚡ LINE HUB"
Title.Font = Enum.Font.GothamBold
Title.TextSize = 26
Title.TextColor3 = Color3.new(1,1,1)

-- ================= TABS =================
local Tabs = Instance.new("Frame", Main)
Tabs.Size = UDim2.new(1,0,0,40)
Tabs.Position = UDim2.new(0,0,0,50)
Tabs.BackgroundTransparency = 1

local Pages = {}
local CurrentPage

local function NewPage()
    local p = Instance.new("Frame", Main)
    p.Size = UDim2.new(1,0,1,-90)
    p.Position = UDim2.new(0,0,0,90)
    p.BackgroundTransparency = 1
    p.Visible = false
    return p
end

local function NewTab(text, x, page)
    local b = Instance.new("TextButton", Tabs)
    b.Size = UDim2.new(0,130,1,0)
    b.Position = UDim2.new(0,x,0,0)
    b.Text = text
    b.Font = Enum.Font.GothamMedium
    b.TextSize = 14
    b.TextColor3 = Color3.new(1,1,1)
    b.BackgroundColor3 = Color3.fromRGB(30,30,30)
    b.BorderSizePixel = 0
    Instance.new("UICorner", b)

    b.MouseButton1Click:Connect(function()
        if CurrentPage then CurrentPage.Visible = false end
        page.Visible = true
        CurrentPage = page
    end)
end

local PageMain   = NewPage()
local PagePlayer = NewPage()
local PageExtra  = NewPage()

NewTab("Main", 10, PageMain)
NewTab("Player", 150, PagePlayer)
NewTab("Extra", 290, PageExtra)

PageMain.Visible = true
CurrentPage = PageMain

-- ================= UI HELPERS =================
local function Toggle(parent, text, y, cb)
    local b = Instance.new("TextButton", parent)
    b.Size = UDim2.new(0.9,0,0,36)
    b.Position = UDim2.new(0.05,0,0,y)
    b.Text = text.." [OFF]"
    b.Font = Enum.Font.GothamMedium
    b.TextSize = 14
    b.TextColor3 = Color3.new(1,1,1)
    b.BackgroundColor3 = Color3.fromRGB(32,32,32)
    b.BorderSizePixel = 0
    Instance.new("UICorner", b)

    local on = false
    b.MouseButton1Click:Connect(function()
        on = not on
        b.Text = text.." ["..(on and "ON" or "OFF").."]"
        b.BackgroundColor3 = on and Color3.fromRGB(60,100,60) or Color3.fromRGB(32,32,32)
        cb(on)
    end)
end

local function Box(parent, ph, y)
    local t = Instance.new("TextBox", parent)
    t.Size = UDim2.new(0.9,0,0,30)
    t.Position = UDim2.new(0.05,0,0,y)
    t.PlaceholderText = ph
    t.Text = ""
    t.Font = Enum.Font.Gotham
    t.TextSize = 14
    t.TextColor3 = Color3.new(1,1,1)
    t.BackgroundColor3 = Color3.fromRGB(26,26,26)
    Instance.new("UICorner", t)
    return t
end

-- ================= SPEED =================
local SpeedOn = false
local SpeedValue = 16
local SpeedBox = Box(PageMain, "Speed (16 - 150)", 10)

SpeedBox.FocusLost:Connect(function()
    local v = tonumber(SpeedBox.Text)
    if v then SpeedValue = math.clamp(v,16,150) end
end)

RunService.Heartbeat:Connect(function()
    if SpeedOn and Humanoid then
        Humanoid.WalkSpeed = SpeedValue
    end
end)

Toggle(PageMain, "🏃 Speed Custom", 50, function(v)
    SpeedOn = v
    if not v and Humanoid then Humanoid.WalkSpeed = 16 end
end)

-- ================= FLY =================
local Fly = false
local FlySpeed = 90
local BV, BG
local keys = {}

UIS.InputBegan:Connect(function(i,g) if not g then keys[i.KeyCode] = true end end)
UIS.InputEnded:Connect(function(i) keys[i.KeyCode] = false end)

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

Toggle(PageMain, "✈️ Fly Avançado", 100, function(v)
    Fly = v
    if v then
        BV = Instance.new("BodyVelocity", HRP)
        BV.MaxForce = Vector3.new(1e9,1e9,1e9)
        BG = Instance.new("BodyGyro", HRP)
        BG.MaxTorque = Vector3.new(1e9,1e9,1e9)
        BG.P = 6000
    else
        if BV then BV:Destroy() end
        if BG then BG:Destroy() end
    end
end)

-- ================= ANTI KNOCKBACK =================
local AntiKB = false
local lastCF
local lock = false

RunService.Heartbeat:Connect(function()
    if HRP and not lock then
        lastCF = HRP.CFrame
    end
end)

RunService.RenderStepped:Connect(function()
    if AntiKB and HRP and HRP.AssemblyLinearVelocity.Magnitude > 40 then
        lock = true
        HRP.AssemblyLinearVelocity = Vector3.zero
        HRP.CFrame = lastCF
        task.delay(0.06,function() lock = false end)
    end
end)

Toggle(PageMain, "🛡️ Anti-Knockback Invisível", 150, function(v)
    AntiKB = v
end)

-- ================= CLICK TELEPORT =================
local ClickTP = false

Mouse.Button1Down:Connect(function()
    if not ClickTP then return end
    if not UIS:IsKeyDown(Enum.KeyCode.LeftControl) then return end
    if HRP and Mouse.Hit then
        HRP.CFrame = CFrame.new(Mouse.Hit.Position + Vector3.new(0,3,0))
    end
end)

Toggle(PageMain, "🖱️ Click Teleport (CTRL)", 200, function(v)
    ClickTP = v
end)

-- ================= TOGGLE GUI =================
UIS.InputBegan:Connect(function(i,g)
    if not g and i.KeyCode == Enum.KeyCode.B then
        Main.Visible = not Main.Visible
    end
end)
