--[[
 KSX Panel – Script Completo (LocalScript)
 Coloque em StarterPlayer > StarterPlayerScripts
 Tecla B abre/fecha | Painel movível
 Inclui: Home (Ping/Rejoin), Speed configurável, Fly avançado,
 AntiFling/AntiKnockback estável, Click TP (CTRL), View toggle,
 Lista de Animações (reseta ao reset), Target por nome com avatar
 OBS: Código client-side
]]

-- Serviços
local Players = game:GetService("Players")
local UIS = game:GetService("UserInputService")
local RS = game:GetService("RunService")
local TS = game:GetService("TeleportService")
local Http = game:GetService("HttpService")
local lp = Players.LocalPlayer

-- Util
local function getChar()
    return lp.Character or lp.CharacterAdded:Wait()
end

-- UI Base
local gui = Instance.new("ScreenGui", lp:WaitForChild("PlayerGui"))
gui.Name = "KSXPanel"

gui.ResetOnSpawn = false

local main = Instance.new("Frame", gui)
main.Size = UDim2.fromOffset(520, 360)
main.Position = UDim2.fromScale(0.05, 0.2)
main.BackgroundColor3 = Color3.fromRGB(20,20,20)
main.BorderSizePixel = 0
main.Visible = true

-- Drag
local dragging, dragStart, startPos
main.InputBegan:Connect(function(i)
    if i.UserInputType == Enum.UserInputType.MouseButton1 then
        dragging = true; dragStart = i.Position; startPos = main.Position
    end
end)
UIS.InputEnded:Connect(function(i) if i.UserInputType==Enum.UserInputType.MouseButton1 then dragging=false end end)
UIS.InputChanged:Connect(function(i)
    if dragging and i.UserInputType==Enum.UserInputType.MouseMovement then
        local delta = i.Position - dragStart
        main.Position = UDim2.new(startPos.X.Scale, startPos.X.Offset+delta.X, startPos.Y.Scale, startPos.Y.Offset+delta.Y)
    end
end)

-- Toggle B
UIS.InputBegan:Connect(function(i,gp)
    if not gp and i.KeyCode==Enum.KeyCode.B then
        main.Visible = not main.Visible
    end
end)

-- Header
local header = Instance.new("TextLabel", main)
header.Size = UDim2.new(1,0,0,40)
header.Text = "ksx's Panel"
header.TextColor3 = Color3.new(1,1,1)
header.BackgroundColor3 = Color3.fromRGB(30,30,30)
header.BorderSizePixel = 0
header.Font = Enum.Font.GothamBold
header.TextSize = 16

-- Tabs
local tabs = {"Home","Character","Target","Animations","Misc"}
local tabButtons = {}
local pages = {}

local tabBar = Instance.new("Frame", main)
tabBar.Position = UDim2.fromOffset(0,40)
tabBar.Size = UDim2.fromOffset(120,320)
tabBar.BackgroundColor3 = Color3.fromRGB(25,25,25)
tabBar.BorderSizePixel = 0

local content = Instance.new("Frame", main)
content.Position = UDim2.fromOffset(120,40)
content.Size = UDim2.fromOffset(400,320)
content.BackgroundColor3 = Color3.fromRGB(18,18,18)
content.BorderSizePixel = 0

local function makePage(name)
    local p = Instance.new("Frame", content)
    p.Size = UDim2.fromScale(1,1)
    p.Visible = false
    pages[name]=p
    return p
end

for i,t in ipairs(tabs) do
    local b = Instance.new("TextButton", tabBar)
    b.Size = UDim2.fromOffset(120, 40)
    b.Position = UDim2.fromOffset(0, (i-1)*40)
    b.Text = t
    b.Font = Enum.Font.Gotham
    b.TextSize = 14
    b.TextColor3 = Color3.new(1,1,1)
    b.BackgroundColor3 = Color3.fromRGB(35,35,35)
    b.BorderSizePixel = 0
    tabButtons[t]=b
    makePage(t)
    b.MouseButton1Click:Connect(function()
        for _,pg in pairs(pages) do pg.Visible=false end
        pages[t].Visible=true
    end)
end
pages.Home.Visible = true

-- HOME
local home = pages.Home
local nameLbl = Instance.new("TextLabel", home)
nameLbl.Text = "Olá! "..lp.Name.." | Pressione [B]"
nameLbl.Size = UDim2.fromOffset(380,30)
nameLbl.Position = UDim2.fromOffset(10,10)
nameLbl.TextColor3 = Color3.new(1,1,1)
nameLbl.BackgroundTransparency = 1
nameLbl.Font = Enum.Font.Gotham
nameLbl.TextSize = 14

local pingLbl = Instance.new("TextLabel", home)
pingLbl.Position = UDim2.fromOffset(10,50)
pingLbl.Size = UDim2.fromOffset(200,24)
pingLbl.BackgroundTransparency=1
pingLbl.TextColor3 = Color3.fromRGB(255,200,0)
pingLbl.Font = Enum.Font.Gotham
pingLbl.TextSize = 13

RS.Heartbeat:Connect(function()
    local stats = lp:FindFirstChild("NetworkClient")
    pingLbl.Text = "Ping: "..math.floor((lp:GetNetworkPing()*1000)).."ms"
end)

local rejoin = Instance.new("TextButton", home)
rejoin.Text = "Rejoin (mesmo servidor)"
rejoin.Size = UDim2.fromOffset(220,30)
rejoin.Position = UDim2.fromOffset(10,90)
rejoin.BackgroundColor3 = Color3.fromRGB(45,45,45)
rejoin.TextColor3 = Color3.new(1,1,1)
rejoin.BorderSizePixel=0
rejoin.MouseButton1Click:Connect(function()
    TS:TeleportToPlaceInstance(game.PlaceId, game.JobId, lp)
end)

-- CHARACTER (Speed/Fly/AntiFling)
local char = pages.Character

-- Speed
local speedOn=false; local speedVal=16
local speedBox = Instance.new("TextBox", char)
speedBox.PlaceholderText = "Speed (ex: 30)"
speedBox.Size = UDim2.fromOffset(140,28)
speedBox.Position = UDim2.fromOffset(10,10)
speedBox.Text=""

local speedBtn = Instance.new("TextButton", char)
speedBtn.Text="Toggle Speed"
speedBtn.Size = UDim2.fromOffset(140,28)
speedBtn.Position = UDim2.fromOffset(160,10)

speedBtn.MouseButton1Click:Connect(function()
    speedOn = not speedOn
    speedVal = tonumber(speedBox.Text) or 16
end)

RS.Stepped:Connect(function()
    local hum = getChar():FindFirstChildOfClass("Humanoid")
    if hum and speedOn then hum.WalkSpeed = speedVal else if hum then hum.WalkSpeed = 16 end end
end)

-- Fly avançado
local fly=false; local bv; local bg
local flyBtn = Instance.new("TextButton", char)
flyBtn.Text="Toggle Fly"
flyBtn.Size = UDim2.fromOffset(140,28)
flyBtn.Position = UDim2.fromOffset(10,50)

flyBtn.MouseButton1Click:Connect(function()
    fly = not fly
    local hrp = getChar():WaitForChild("HumanoidRootPart")
    if fly then
        bv = Instance.new("BodyVelocity", hrp)
        bv.MaxForce = Vector3.new(1e5,1e5,1e5)
        bg = Instance.new("BodyGyro", hrp)
        bg.MaxTorque = Vector3.new(1e5,1e5,1e5)
    else
        if bv then bv:Destroy() end
        if bg then bg:Destroy() end
    end
end)

RS.RenderStepped:Connect(function()
    if fly and bv and bg then
        local cam = workspace.CurrentCamera
        local move = Vector3.zero
        if UIS:IsKeyDown(Enum.KeyCode.W) then move += cam.CFrame.LookVector end
        if UIS:IsKeyDown(Enum.KeyCode.S) then move -= cam.CFrame.LookVector end
        if UIS:IsKeyDown(Enum.KeyCode.A) then move -= cam.CFrame.RightVector end
        if UIS:IsKeyDown(Enum.KeyCode.D) then move += cam.CFrame.RightVector end
        bv.Velocity = move.Unit * 60
        bg.CFrame = cam.CFrame
    end
end)

-- AntiFling / AntiKnockback estável
local antiOn=false
local antiBtn = Instance.new("TextButton", char)
antiBtn.Text="Toggle AntiFling"
antiBtn.Size = UDim2.fromOffset(140,28)
antiBtn.Position = UDim2.fromOffset(160,50)

antiBtn.MouseButton1Click:Connect(function()
    antiOn = not antiOn
end)

RS.Heartbeat:Connect(function()
    if antiOn then
        local hrp = getChar():FindFirstChild("HumanoidRootPart")
        if hrp then
            hrp.AssemblyLinearVelocity = Vector3.zero
            hrp.AssemblyAngularVelocity = Vector3.zero
        end
    end
end)

-- TARGET (View/Avatar/TP Click)
local tgt = pages.Target
local nameBox = Instance.new("TextBox", tgt)
nameBox.PlaceholderText="Nome do jogador"
nameBox.Size = UDim2.fromOffset(160,28)
nameBox.Position = UDim2.fromOffset(10,10)

local avatar = Instance.new("ImageLabel", tgt)
avatar.Size = UDim2.fromOffset(64,64)
avatar.Position = UDim2.fromOffset(180,10)
avatar.BackgroundTransparency=1

local function findPlayer(n)
    for _,p in pairs(Players:GetPlayers()) do
        if p.Name:lower():sub(1,#n)==n:lower() then return p end
    end
end

nameBox.FocusLost:Connect(function()
    local p = findPlayer(nameBox.Text)
    if p then
        local img = Players:GetUserThumbnailAsync(p.UserId, Enum.ThumbnailType.HeadShot, Enum.ThumbnailSize.Size180x180)
        avatar.Image = img
    end
end)

-- View toggle
local viewing=false; local oldCam
local viewBtn = Instance.new("TextButton", tgt)
viewBtn.Text="View"
viewBtn.Size = UDim2.fromOffset(100,28)
viewBtn.Position = UDim2.fromOffset(10,50)

viewBtn.MouseButton1Click:Connect(function()
    local p = findPlayer(nameBox.Text)
    if not p then return end
    if not viewing then
        viewing=true; oldCam = workspace.CurrentCamera.CameraSubject
        workspace.CurrentCamera.CameraSubject = p.Character:WaitForChild("Humanoid")
    else
        viewing=false
        workspace.CurrentCamera.CameraSubject = getChar():WaitForChild("Humanoid")
    end
end)

-- Click TP (CTRL)
local clickTP=false
local tpBtn = Instance.new("TextButton", tgt)
tpBtn.Text="Toggle Click TP (CTRL)"
tpBtn.Size = UDim2.fromOffset(200,28)
tpBtn.Position = UDim2.fromOffset(10,90)

tpBtn.MouseButton1Click:Connect(function() clickTP = not clickTP end)

UIS.InputBegan:Connect(function(i,gp)
    if clickTP and not gp and i.UserInputType==Enum.UserInputType.MouseButton1 and UIS:IsKeyDown(Enum.KeyCode.LeftControl) then
        local mouse = lp:GetMouse()
        if mouse.Hit then
            getChar():WaitForChild("HumanoidRootPart").CFrame = mouse.Hit + Vector3.new(0,3,0)
        end
    end
end)

-- ANIMATIONS
local anim = pages.Animations
local anims = {
    {"Default", 507766666},
    {"Ninja", 656118852},
    {"Zombie", 616006778},
    {"Levitation", 616006778},
}

local y=10; local tracks={}
for _,a in ipairs(anims) do
    local b = Instance.new("TextButton", anim)
    b.Text=a[1]
    b.Size=UDim2.fromOffset(160,26)
    b.Position=UDim2.fromOffset(10,y); y+=30
    b.MouseButton1Click:Connect(function()
        local hum = getChar():WaitForChild("Humanoid")
        for _,t in pairs(tracks) do t:Stop() end
        tracks={}
        local an = Instance.new("Animation")
        an.AnimationId = "rbxassetid://"..a[2]
        local tr = hum:LoadAnimation(an)
        tr:Play(); table.insert(tracks,tr)
    end)
end

lp.CharacterAdded:Connect(function()
    for _,t in pairs(tracks) do pcall(function() t:Stop() end) end
    tracks={}
end)

-- MISC
local misc = pages.Misc
local lbl = Instance.new("TextLabel", misc)
lbl.Text = "Tudo integrado ao painel KSX"
lbl.Size = UDim2.fromOffset(360,30)
lbl.Position = UDim2.fromOffset(10,10)
lbl.BackgroundTransparency=1
lbl.TextColor3=Color3.new(1,1,1)
