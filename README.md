-- HUB UNIVERSAL OTIMIZADO PARA XENO
local Player = game.Players.LocalPlayer
local Mouse = Player:GetMouse()
local UserInputService = game:GetService("UserInputService")
local RunService = game:GetService("RunService")

-- Criando a Interface manualmente para garantir que apareça no Xeno
local ScreenGui = Instance.new("ScreenGui")
ScreenGui.Name = "XenoHub_Gemini"
ScreenGui.Parent = Player:WaitForChild("PlayerGui")
ScreenGui.ResetOnSpawn = false

local MainFrame = Instance.new("Frame")
MainFrame.Name = "MainFrame"
MainFrame.Parent = ScreenGui
MainFrame.BackgroundColor3 = Color3.fromRGB(25, 25, 25)
MainFrame.BorderSizePixel = 0
MainFrame.Position = UDim2.new(0.5, -125, 0.5, -175)
MainFrame.Size = UDim2.new(0, 250, 0, 400)
MainFrame.Active = true
MainFrame.Draggable = true

local Title = Instance.new("TextLabel")
Title.Parent = MainFrame
Title.Size = UDim2.new(1, 0, 0, 35)
Title.BackgroundColor3 = Color3.fromRGB(40, 40, 40)
Title.Text = "💎 GEMINI HUB | [B] FECHAR"
Title.TextColor3 = Color3.fromRGB(255, 255, 255)
Title.TextSize = 14

local Content = Instance.new("ScrollingFrame")
Content.Parent = MainFrame
Content.Position = UDim2.new(0, 5, 0, 40)
Content.Size = UDim2.new(1, -10, 1, -45)
Content.BackgroundTransparency = 1
Content.CanvasSize = UDim2.new(0, 0, 1.5, 0)
Content.ScrollBarThickness = 4

local Layout = Instance.new("UIListLayout")
Layout.Parent = Content
Layout.Padding = UDim.new(0, 5)

-- FUNÇÕES TIPO TOGGLE/BOTÃO
local function CreateButton(text, color, callback)
    local btn = Instance.new("TextButton")
    btn.Parent = Content
    btn.Size = UDim2.new(1, 0, 0, 30)
    btn.BackgroundColor3 = color
    btn.Text = text
    btn.TextColor3 = Color3.fromRGB(255, 255, 255)
    btn.Font = Enum.Font.SourceSansBold
    btn.MouseButton1Click:Connect(callback)
    
    local corner = Instance.new("UICorner")
    corner.CornerRadius = UDim.new(0, 6)
    corner.Parent = btn
end

-- VARIÁVEIS DE ESTADO
local States = {
    AntiAFK = false,
    AntiFling = false,
    Noclip = false,
    ClickTP = false,
    Target = nil
}

-- [ SISTEMA DE BUSCA COM FOTO ]
local Photo = Instance.new("ImageLabel")
Photo.Parent = Content
Photo.Size = UDim2.new(0, 60, 0, 60)
Photo.BackgroundTransparency = 1
Photo.Image = "rbxassetid://0" -- Vazio no início

local Search = Instance.new("TextBox")
Search.Parent = Content
Search.Size = UDim2.new(1, 0, 0, 30)
Search.PlaceholderText = "Nick do Jogador + ENTER"
Search.Text = ""
Search.FocusLost:Connect(function(enter)
    if enter then
        for _, v in pairs(game.Players:GetPlayers()) do
            if v.Name:lower():sub(1, #Search.Text) == Search.Text:lower() then
                States.Target = v
                Search.Text = "Alvo: " .. v.DisplayName
                Photo.Image = "rbxthumb://type=AvatarHeadShot&id=" .. v.UserId .. "&w=150&h=150"
                break
            end
        end
    end
end)

-- [ BOTÕES ]
CreateButton("View / Unview Alvo", Color3.fromRGB(60, 100, 200), function()
    local cam = workspace.CurrentCamera
    if cam.CameraSubject == Player.Character.Humanoid then
        if States.Target and States.Target.Character then
            cam.CameraSubject = States.Target.Character.Humanoid
        end
    else
        cam.CameraSubject = Player.Character.Humanoid
    end
end)

CreateButton("Teleport ao Alvo", Color3.fromRGB(60, 100, 200), function()
    if States.Target and States.Target.Character then
        Player.Character.HumanoidRootPart.CFrame = States.Target.Character.HumanoidRootPart.CFrame
    end
end)

CreateButton("Anti-Fling (On/Off)", Color3.fromRGB(180, 50, 50), function()
    States.AntiFling = not States.AntiFling
end)

CreateButton("Anti-AFK (On/Off)", Color3.fromRGB(50, 180, 50), function()
    States.AntiAFK = not States.AntiAFK
end)

CreateButton("Click TP (CTRL+Click)", Color3.fromRGB(150, 50, 150), function()
    States.ClickTP = not States.ClickTP
end)

CreateButton("Speed x100", Color3.fromRGB(70, 70, 70), function()
    Player.Character.Humanoid.WalkSpeed = 100
end)

-- [ LOOPS E EVENTOS ]

-- Tecla B para fechar/abrir
UserInputService.InputBegan:Connect(function(input, gpe)
    if not gpe and input.KeyCode == Enum.KeyCode.B then
        MainFrame.Visible = not MainFrame.Visible
    end
end)

-- Click TP Logic
Mouse.Button1Down:Connect(function()
    if States.ClickTP and UserInputService:IsKeyDown(Enum.KeyCode.LeftControl) then
        if Mouse.Target then
            Player.Character:MoveTo(Mouse.Hit.p)
        end
    end
end)

-- Anti-AFK Logic
Player.Idled:Connect(function()
    if States.AntiAFK then
        game:GetService("VirtualUser"):CaptureController()
        game:GetService("VirtualUser"):ClickButton2(Vector2.new())
    end
end)

-- Heartbeat Loop (Anti-Fling)
RunService.Heartbeat:Connect(function()
    if States.AntiFling then
        for _, v in pairs(game.Players:GetPlayers()) do
            if v ~= Player and v.Character then
                for _, part in pairs(v.Character:GetDescendants()) do
                    if part:IsA("BasePart") then
                        part.CanCollide = false
                        part.Velocity = Vector3.new(0,0,0)
                    end
                end
            end
        end
    end
end)

print("Xeno Hub Carregado!")
