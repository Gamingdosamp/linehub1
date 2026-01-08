--[[ 
    CLONE KSX PANEL - Versão 4.5.2
    Funcionalidades: Drag, Tecla B para abrir, Notificações e Comandos
]]

local UserInputService = game:GetService("UserInputService")
local TweenService = game:GetService("TweenService")
local RunService = game:GetService("RunService")
local Players = game:GetService("Players")
local Player = Players.LocalPlayer

-- CONFIGURAÇÃO DE CORES
local COLORS = {
    Background = Color3.fromRGB(35, 43, 53),
    Sidebar = Color3.fromRGB(25, 30, 38),
    Accent = Color3.fromRGB(50, 60, 75),
    Text = Color3.fromRGB(240, 240, 240)
}

-- INTERFACE PRINCIPAL
local ScreenGui = Instance.new("ScreenGui")
ScreenGui.Name = "KSX_Panel_Clone"
ScreenGui.Parent = Player:WaitForChild("PlayerGui")
ScreenGui.ResetOnSpawn = false

local MainFrame = Instance.new("Frame")
MainFrame.Name = "MainFrame"
MainFrame.Size = UDim2.new(0, 500, 0, 300)
MainFrame.Position = UDim2.new(0.5, -250, 0.5, -150)
MainFrame.BackgroundColor3 = COLORS.Background
MainFrame.BorderSizePixel = 0
MainFrame.Parent = ScreenGui
MainFrame.Visible = true

-- NOTIFICAÇÃO
local function Notify(msg)
    local n = Instance.new("TextLabel")
    n.Size = UDim2.new(0, 250, 0, 30)
    n.Position = UDim2.new(0.5, -125, 0.1, 0)
    n.BackgroundColor3 = Color3.fromRGB(0, 0, 0)
    n.BackgroundTransparency = 0.5
    n.TextColor3 = Color3.new(1, 1, 1)
    n.Text = "[KSX]: " .. msg
    n.Parent = ScreenGui
    
    task.wait(2)
    TweenService:Create(n, TweenInfo.new(0.5), {TextTransparency = 1, BackgroundTransparency = 1}):Play()
    game:GetService("Debris"):AddItem(n, 0.5)
end

-- TÍTULO
local Title = Instance.new("TextLabel")
Title.Size = UDim2.new(1, -100, 0, 30)
Title.Position = UDim2.new(0, 100, 0, 0)
Title.Text = "ksx's Panel v4.5.2"
Title.TextColor3 = COLORS.Text
Title.BackgroundColor3 = COLORS.Accent
Title.Parent = MainFrame

-- SIDEBAR (Menu Lateral)
local Sidebar = Instance.new("Frame")
Sidebar.Size = UDim2.new(0, 100, 1, 0)
Sidebar.BackgroundColor3 = COLORS.Sidebar
Sidebar.Parent = MainFrame

local Layout = Instance.new("UIListLayout")
Layout.Parent = Sidebar

local function CreateTabBtn(name)
    local btn = Instance.new("TextButton")
    btn.Size = UDim2.new(1, 0, 0, 30)
    btn.BackgroundColor3 = COLORS.Sidebar
    btn.Text = name
    btn.TextColor3 = COLORS.Text
    btn.BorderSizePixel = 0
    btn.Parent = Sidebar
    return btn
end

local tabs = {"Home", "VIP", "Emphasis", "Character", "Target", "Animations", "Misc"}
for _, name in pairs(tabs) do CreateTabBtn(name) end

-- CONTAINER DE BOTÕES (GRID)
local Container = Instance.new("ScrollingFrame")
Container.Size = UDim2.new(1, -110, 1, -40)
Container.Position = UDim2.new(0, 105, 0, 35)
Container.BackgroundTransparency = 1
Container.CanvasSize = UDim2.new(0, 0, 2, 0)
Container.Parent = MainFrame

local Grid = Instance.new("UIGridLayout")
Grid.CellSize = UDim2.new(0, 120, 0, 30)
Grid.Parent = Container

-- FUNÇÕES DE LOGICA
local function AddFunction(name, callback)
    local btn = Instance.new("TextButton")
    btn.Text = name
    btn.BackgroundColor3 = COLORS.Accent
    btn.TextColor3 = COLORS.Text
    btn.Parent = Container
    btn.MouseButton1Click:Connect(function()
        callback()
        Notify(name .. " Executado")
    end)
end

-- TECLA B PARA ABRIR/FECHAR
UserInputService.InputBegan:Connect(function(input, gpe)
    if not gpe and input.KeyCode == Enum.KeyCode.B then
        MainFrame.Visible = not MainFrame.Visible
    end
end)

-- SISTEMA DE ARRASTAR
local dragging, dragInput, dragStart, startPos
MainFrame.InputBegan:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseButton1 then
        dragging = true; dragStart = input.Position; startPos = MainFrame.Position
    end
end)
UserInputService.InputChanged:Connect(function(input)
    if dragging and input.UserInputType == Enum.UserInputType.MouseMovement then
        local delta = input.Position - dragStart
        MainFrame.Position = UDim2.new(startPos.X.Scale, startPos.X.Offset + delta.X, startPos.Y.Scale, startPos.Y.Offset + delta.Y)
    end
end)
UserInputService.InputEnded:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseButton1 then dragging = false end
end)

--- LISTA DE FUNÇÕES (Baseado nas suas imagens) ---

-- Aba Character
AddFunction("Invisible", function()
    local c = Player.Character
    if c then for _, v in pairs(c:GetDescendants()) do if v:IsA("BasePart") or v:IsA("Decal") then v.Transparency = 1 end end end
end)

AddFunction("NoClip", function()
    RunService.Stepped:Connect(function()
        if Player.Character then
            for _, v in pairs(Player.Character:GetDescendants()) do
                if v:IsA("BasePart") then v.CanCollide = false end
            end
        end
    end)
end)

AddFunction("Fly", function() Notify("Fly ativado (Requer script de voo)") end)

-- Aba Target (Simulação das interações)
AddFunction("Fling", function() Notify("Aguardando alvo...") end)
AddFunction("Bang", function() Notify("Animação Bang iniciada") end)

-- Aba Misc
AddFunction("Anti-AFK", function()
    local vu = game:GetService("VirtualUser")
    Player.Idled:Connect(function() vu:Button2Down(Vector2.new(0,0),workspace.CurrentCamera.CFrame) task.wait(1) vu:Button2Up(Vector2.new(0,0),workspace.CurrentCamera.CFrame) end)
end)

AddFunction("Rejoin", function()
    game:GetService("TeleportService"):Teleport(game.PlaceId, Player)
end)

AddFunction("Infinite Premium", function() Notify("Premium Ativado Visualmente") end)

-- Exemplo de inputs de WalkSpeed/JumpPower
AddFunction("Speed 100", function() Player.Character.Humanoid.WalkSpeed = 100 end)
AddFunction("Jump 100", function() Player.Character.Humanoid.JumpPower = 100 end)

Notify("Painel Carregado! Aperte 'B' para abrir.")
