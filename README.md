-- LINE HUB | SCRIPT FINAL COMPLETO
-- JOGO PRÓPRIO | LocalScript

---------------- SERVICES ----------------
local Players = game:GetService("Players")
local UIS = game:GetService("UserInputService")
local RunService = game:GetService("RunService")
local TweenService = game:GetService("TweenService")
local Camera = workspace.CurrentCamera

---------------- PLAYER ----------------
local lp = Players.LocalPlayer
local char = lp.Character or lp.CharacterAdded:Wait()
local hum = char:WaitForChild("Humanoid")
local root = char:WaitForChild("HumanoidRootPart")

lp.CharacterAdded:Connect(function(c)
	char = c
	hum = c:WaitForChild("Humanoid")
	root = c:WaitForChild("HumanoidRootPart")
	task.wait(0.3)
	applySavedAnimations(c)
end)

---------------- GUI ----------------
local gui = Instance.new("ScreenGui", game.CoreGui)
gui.Name = "LineHub"
gui.ResetOnSpawn = false

local main = Instance.new("Frame", gui)
main.Size = UDim2.fromOffset(540, 380)
main.Position = UDim2.fromScale(0.3, 0.3)
main.BackgroundColor3 = Color3.fromRGB(18,18,22)
main.Active = true
main.Draggable = true
Instance.new("UICorner", main).CornerRadius = UDim.new(0,12)

local title = Instance.new("TextLabel", main)
title.Size = UDim2.new(1,0,0,40)
title.BackgroundTransparency = 1
title.Text = "LINE HUB"
title.Font = Enum.Font.GothamBold
title.TextSize = 22
title.TextColor3 = Color3.fromRGB(0,200,255)

---------------- SIDE MENU ----------------
local side = Instance.new("Frame", main)
side.Size = UDim2.new(0,130,1,-40)
side.Position = UDim2.fromOffset(0,40)
side.BackgroundColor3 = Color3.fromRGB(25,25,30)

local pages = Instance.new("Folder", main)

local function newPage(name)
	local f = Instance.new("Frame", pages)
	f.Name = name
	f.Size = UDim2.new(1,-140,1,-50)
	f.Position = UDim2.fromOffset(140,50)
	f.BackgroundTransparency = 1
	f.Visible = false
	return f
end

local function animateButton(btn)
	btn.MouseEnter:Connect(function()
		TweenService:Create(btn,TweenInfo.new(0.15),{BackgroundColor3=Color3.fromRGB(50,50,65)}):Play()
	end)
	btn.MouseLeave:Connect(function()
		TweenService:Create(btn,TweenInfo.new(0.15),{BackgroundColor3=Color3.fromRGB(35,35,45)}):Play()
	end)
end

local function newTab(text,page,y)
	local b = Instance.new("TextButton", side)
	b.Size = UDim2.new(1,0,0,36)
	b.Position = UDim2.fromOffset(0,y)
	b.BackgroundColor3 = Color3.fromRGB(30,30,35)
	b.Text = text
	b.Font = Enum.Font.Gotham
	b.TextSize = 14
	b.TextColor3 = Color3.new(1,1,1)
	animateButton(b)

	b.MouseButton1Click:Connect(function()
		for _,p in pairs(pages:GetChildren()) do p.Visible = false end
		page.Visible = true
	end)
end

---------------- PAGES ----------------
local home = newPage("Home")
local character = newPage("Character")
local target = newPage("Target")
local animations = newPage("Animations")
local misc = newPage("Misc")
home.Visible = true

newTab("Home",home,10)
newTab("Character",character,50)
newTab("Target",target,90)
newTab("Animations",animations,130)
newTab("Misc",misc,170)

---------------- UTIL BUTTON ----------------
local function button(parent,text,y)
	local b = Instance.new("TextButton", parent)
	b.Size = UDim2.fromOffset(220,36)
	b.Position = UDim2.fromOffset(0,y)
	b.BackgroundColor3 = Color3.fromRGB(35,35,45)
	b.Text = text.." : OFF"
	b.Font = Enum.Font.Gotham
	b.TextSize = 14
	b.TextColor3 = Color3.new(1,1,1)
	Instance.new("UICorner", b)
	animateButton(b)
	return b
end

---------------- ANTI KNOCKBACK ----------------
local antiKB = false
local safeCF

RunService.Heartbeat:Connect(function()
	if antiKB and root then
		if not safeCF then safeCF = root.CFrame end
		if (root.Position - safeCF.Position).Magnitude > 4 then
			root.CFrame = safeCF
		else
			safeCF = root.CFrame
		end
	end
end)

local antiBtn = button(character,"Anti Knockback",0)
antiBtn.MouseButton1Click:Connect(function()
	antiKB = not antiKB
	antiBtn.Text = "Anti Knockback : "..(antiKB and "ON" or "OFF")
end)

---------------- FLY ----------------
local fly = false
local flySpeed = 70
local bv,bg

RunService.RenderStepped:Connect(function()
	if fly and root then
		local dir = Vector3.zero
		if UIS:IsKeyDown(Enum.KeyCode.W) then dir += Camera.CFrame.LookVector end
		if UIS:IsKeyDown(Enum.KeyCode.S) then dir -= Camera.CFrame.LookVector end
		if UIS:IsKeyDown(Enum.KeyCode.A) then dir -= Camera.CFrame.RightVector end
		if UIS:IsKeyDown(Enum.KeyCode.D) then dir += Camera.CFrame.RightVector end
		if UIS:IsKeyDown(Enum.KeyCode.Space) then dir += Vector3.new(0,1,0) end
		if UIS:IsKeyDown(Enum.KeyCode.LeftControl) then dir -= Vector3.new(0,1,0) end
		bv.Velocity = dir * flySpeed
		bg.CFrame = Camera.CFrame
	end
end)

local flyBtn = button(character,"Fly",44)
flyBtn.MouseButton1Click:Connect(function()
	fly = not fly
	flyBtn.Text = "Fly : "..(fly and "ON" or "OFF")
	if fly then
		bv = Instance.new("BodyVelocity", root)
		bg = Instance.new("BodyGyro", root)
		bv.MaxForce = Vector3.new(1e5,1e5,1e5)
		bg.MaxTorque = Vector3.new(1e5,1e5,1e5)
	else
		if bv then bv:Destroy() end
		if bg then bg:Destroy() end
	end
end)

---------------- SPEED ----------------
local speedOn = false
local speedVal = 30
RunService.RenderStepped:Connect(function()
	hum.WalkSpeed = speedOn and speedVal or 16
end)

local speedBtn = button(character,"Speed",88)
speedBtn.MouseButton1Click:Connect(function()
	speedOn = not speedOn
	speedBtn.Text = "Speed : "..(speedOn and "ON" or "OFF")
end)

---------------- CLICK TP ----------------
local clickTP = false
local mouse = lp:GetMouse()

mouse.Button1Down:Connect(function()
	if clickTP and UIS:IsKeyDown(Enum.KeyCode.LeftControl) then
		if mouse.Hit then
			root.CFrame = mouse.Hit + Vector3.new(0,3,0)
		end
	end
end)

local clickBtn = button(misc,"Click TP (CTRL)",0)
clickBtn.MouseButton1Click:Connect(function()
	clickTP = not clickTP
	clickBtn.Text = "Click TP : "..(clickTP and "ON" or "OFF")
end)

---------------- TARGET ----------------
local function getPlayer(name)
	for _,p in pairs(Players:GetPlayers()) do
		if p.Name:lower():sub(1,#name)==name:lower() then
			return p
		end
	end
end

local box = Instance.new("TextBox", target)
box.Size = UDim2.fromOffset(220,36)
box.PlaceholderText = "Nome do player"
box.BackgroundColor3 = Color3.fromRGB(35,35,45)
box.TextColor3 = Color3.new(1,1,1)
box.Font = Enum.Font.Gotham
box.TextSize = 14
Instance.new("UICorner", box)

local tpBtn = button(target,"Teleport",44)
tpBtn.MouseButton1Click:Connect(function()
	local p = getPlayer(box.Text)
	if p and p.Character then
		root.CFrame = p.Character.HumanoidRootPart.CFrame * CFrame.new(0,0,3)
	end
end)

local viewBtn = button(target,"View",88)
viewBtn.MouseButton1Click:Connect(function()
	local p = getPlayer(box.Text)
	if p and p.Character then
		Camera.CameraSubject = p.Character.Humanoid
	end
end)

local backBtn = button(target,"Voltar View",132)
backBtn.MouseButton1Click:Connect(function()
	Camera.CameraSubject = hum
end)

---------------- ANIMATIONS (SAVE SYSTEM) ----------------
local savedAnimations = {}

function applySavedAnimations(character)
	local animate = character:FindFirstChild("Animate")
	if not animate then return end
	for t,id in pairs(savedAnimations) do
		local f = animate:FindFirstChild(t)
		if f then
			for _,a in pairs(f:GetChildren()) do
				if a:IsA("Animation") then
					a.AnimationId = "rbxassetid://"..id
				end
			end
		end
	end
end

local animList = {
	{n="Zombie Idle",t="idle",id=616158929},
	{n="Stylish Walk",t="walk",id=616168032},
	{n="Robot Walk",t="walk",id=616013216},
	{n="Ninja Run",t="run",id=656118852},
}

local scroll = Instance.new("ScrollingFrame", animations)
scroll.Size = UDim2.new(0,260,1,0)
scroll.CanvasSize = UDim2.fromOffset(0,0)
scroll.ScrollBarThickness = 6
scroll.BackgroundTransparency = 1

local layout = Instance.new("UIListLayout", scroll)
layout.Padding = UDim.new(0,6)

layout:GetPropertyChangedSignal("AbsoluteContentSize"):Connect(function()
	scroll.CanvasSize = UDim2.fromOffset(0,layout.AbsoluteContentSize.Y+10)
end)

for _,a in ipairs(animList) do
	local b = Instance.new("TextButton", scroll)
	b.Size = UDim2.new(1,-10,0,34)
	b.BackgroundColor3 = Color3.fromRGB(35,35,45)
	b.Text = a.n
	b.Font = Enum.Font.Gotham
	b.TextSize = 14
	b.TextColor3 = Color3.new(1,1,1)
	Instance.new("UICorner", b)
	animateButton(b)

	b.MouseButton1Click:Connect(function()
		local animate = char:FindFirstChild("Animate")
		if not animate then return end
		local f = animate:FindFirstChild(a.t)
		if not f then return end
		for _,an in pairs(f:GetChildren()) do
			if an:IsA("Animation") then
				an.AnimationId = "rbxassetid://"..a.id
			end
		end
		savedAnimations[a.t] = a.id
	end)
end

---------------- TOGGLE B ----------------
UIS.InputBegan:Connect(function(i,gp)
	if not gp and i.KeyCode==Enum.KeyCode.B then
		main.Visible = not main.Visible
	end
end)
