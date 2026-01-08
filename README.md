-- Reiniciar variáveis para evitar conflitos
local Player = game:GetService("Players").LocalPlayer
local UIS = game:GetService("UserInputService")
local CoreGui = game:GetService("CoreGui")

-- Remover versões antigas do painel se existirem
if CoreGui:FindFirstChild("KSX_FIXED") then
    CoreGui.KSX_FIXED:Destroy()
end

-- Criar Interface Segura
local ScreenGui = Instance.new("ScreenGui")
ScreenGui.Name = "KSX_FIXED"
ScreenGui.Parent = CoreGui
ScreenGui.ResetOnSpawn = false

-- Notificação simples na tela
local function Notify(text)
    local msg = Instance.new("TextLabel")
    msg.Size = UDim2.new(0, 200, 0, 40)
    msg.Position = UDim2.new(0.5, -100, 0.1, 20)
    msg.BackgroundColor3 = Color3.fromRGB(20, 20, 20)
    msg.TextColor3 = Color3.new(1, 1, 1)
    msg.Text = "[KSX]: " .. text
    msg.Parent = ScreenGui
    msg.BorderSizePixel = 0
    task.wait(2)
    msg:Destroy()
end

-- Janela Principal (Baseada no seu print inicial)
local Main = Instance.new("Frame")
Main.Size = UDim2.new(0, 450, 0, 300)
Main.Position = UDim2.new(0.5, -225, 0.5, -150)
Main.BackgroundColor3 = Color3.fromRGB(35, 43, 53)
Main.BorderSizePixel = 0
Main.Active = true
Main.Draggable = true -- Ativando nativo para máxima compatibilidade
Main.Parent = ScreenGui

-- Barra Lateral
local Sidebar = Instance.new("Frame")
Sidebar.Size = UDim2.new(0, 100, 1, 0)
Sidebar.BackgroundColor3 = Color3.fromRGB(25, 30, 38)
Sidebar.BorderSizePixel = 0
Sidebar.Parent = Main

-- Título
local Title = Instance.new("TextLabel")
Title.Size = UDim2.new(1, -100, 0, 40)
Title.Position = UDim2.new(0, 100, 0, 0)
Title.Text = "ksx's Panel v4.5.2"
Title.TextColor3 = Color3.new(1, 1, 1)
Title.BackgroundTransparency = 1
Title.Parent = Main

-- Container de Botões (Grid)
local Container = Instance.new("ScrollingFrame")
Container.Size = UDim2.new(1, -110, 1, -50)
Container.Position = UDim2.new(0, 105, 0, 45)
Container.BackgroundTransparency = 1
Container.Parent = Main

local Grid = Instance.new("UIGridLayout")
Grid.CellSize = UDim2.new(0, 105, 0, 30)
Grid.Parent = Container

-- Função para criar botões rápido
local function Add(name, func)
    local b = Instance.new("TextButton")
    b.Text = name
    b.BackgroundColor3 = Color3.fromRGB(50, 60, 75)
    b.TextColor3 = Color3.new(1, 1, 1)
    b.Parent = Container
    b.MouseButton1Click:Connect(function()
        pcall(func)
        Notify(name .. " ativado")
    end)
end

-- === LISTA DE TODAS AS FUNÇÕES ===
Add("Invisivel", function() 
    for _, v in pairs(Player.Character:GetDescendants()) do 
        if v:IsA("BasePart") or v:IsA("Decal") then v.Transparency = 1 end 
    end 
end)

Add("NoClip", function()
    game:GetService("RunService").Stepped:Connect(function()
        for _, v in pairs(Player.Character:GetDescendants()) do
            if v:IsA("BasePart") then v.CanCollide = false end
        end
    end)
end)

Add("Anti-AFK", function()
    Player.Idled:Connect(function()
        game:GetService("VirtualUser"):CaptureController()
        game:GetService("VirtualUser"):ClickButton2(Vector2.new())
    end)
end)

Add("Speed 100", function() Player.Character.Humanoid.WalkSpeed = 100 end)
Add("Jump 100", function() Player.Character.Humanoid.JumpPower = 100 end)
Add("Rejoin", function() game:GetService("TeleportService"):Teleport(game.PlaceId, Player) end)

-- SISTEMA ABRIR/FECHAR COM TECLA 'B'
UIS.InputBegan:Connect(function(input, gpe)
    if not gpe and input.KeyCode == Enum.KeyCode.B then
        Main.Visible = not Main.Visible
    end
end)

Notify("Painel Carregado! Use 'B' para abrir.")
