-- LINE HUB | FINAL FIX
-- Toggle GUI: B

-- SERVICES
local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local UIS = game:GetService("UserInputService")
local LP = Players.LocalPlayer
local Camera = workspace.CurrentCamera

-- CHARACTER
local function GetChar()
    local c = LP.Character or LP.CharacterAdded:Wait()
    return c, c:WaitForChild("Humanoid"), c:WaitForChild("HumanoidRootPart")
end

local Character, Humanoid, HRP = GetChar()
LP.CharacterAdded:Connect(function()
    Character, Humanoid, HRP = GetChar()
end)

-- ================= GUI =================
local Gui = Instance.new("ScreenGui", game.CoreGui)
Gui.Name = "LineHub"

local Main = Instance.new("Frame", Gui)
Main.Size = UDim2.new(0, 380, 0, 520)
Main.Position = UDim2.new(0.05,0,0.18,0)
Main.BackgroundColor3 = Color3.fromRGB(18,18,18)
Main.BorderSizePixel = 0
Main.Active = true
Main.Draggable = true

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
SpeedBox.PlaceholderText = "Speed (ex: 30, 50, 80)"
Spee
