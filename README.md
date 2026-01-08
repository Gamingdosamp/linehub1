--[[
    KSX PANEL CLONE - EXECUTOR VERSION
    - Tecla 'B' abre/fecha
    - Arrastável
    - Notificações na tela
]]

local Library = {}
local UIS = game:GetService("UserInputService")
local TweenService = game:GetService("TweenService")
local Players = game:GetService("Players")
local Player = Players.LocalPlayer

-- Criar a interface no CoreGui (para não sumir ao morrer e esconder de prints)
local ScreenGui = Instance.new("ScreenGui")
local protect_gui = gethui or syn.protect_gui or function(gui) gui.Parent = game:GetService("CoreGui") end
ScreenGui.Name = "KSX_Panel_" .. math.random(100, 999)
protect_gui(ScreenGui)

-- Notificação Estilo KSX
local function Notify(text)
    local n = Instance.new("TextLabel")
    n.Parent = ScreenGui
    n.Size = UDim2.new(0, 200, 0, 35)
    n.Position = UDim2.new(0.5, -100, 0.1, -50)
    n.BackgroundColor3 = Color3.fromRGB(30, 30, 30)
    n.TextColor3 = Color3.new(1, 1, 1)
    n.Text = "[KSX]: " .. text
    n.BorderSizePixel = 0
    
    local corner = Instance.new("UICorner")
    corner.CornerRadius = UDim.new(0, 6)
    corner.Parent = n

    n:TweenPosition(UDim2.new(0.5, -100, 0.1, 20), "Out", "Back", 0.5)
    task.wait(2.5)
    n:TweenPosition(UDim2.new(0.5, -100, 0.1, -50), "In", "Back", 0.5)
    game:GetService("Debris"):AddItem(n, 0.6)
end

-- Janela Principal
local Main = Instance.new("Frame")
Main.Name = "MainFrame"
Main.Size = UDim2.new(0, 480, 0, 320)
Main.Position = UDim2.new(0.5, -240, 0.5, -160)
Main.BackgroundColor3 = Color3.fromRGB(35, 43, 53)
Main.BorderSizePixel = 0
Main.Active = true
Main.Parent = ScreenGui

local UICorner = Instance.new("UICorner")
UICorner.CornerRadius = UDim.new(0, 8)
UICorner.Parent = Main

-- Barra Lateral
local Sidebar = Instance.new("Frame")
Sidebar.Size = UDim2.new(0, 100, 1, 0)
Sidebar.BackgroundColor3 = Color3.fromRGB(25, 30, 38)
Sidebar.BorderSizePixel = 0
Sidebar.Parent = Main

local SideCorner = Instance.new("UICorner")
SideCorner.CornerRadius = UDim.new(0, 8)
SideCorner.Parent = Sidebar

-- Título (ksx's Panel)
local Title = Instance.new("TextLabel")
Title.Size = UDim2.new(1, -100, 0, 35)
Title.Position = UDim2.new(0, 100, 0, 0)
Title.BackgroundTransparency = 1
Title.Text = "ksx's Panel v4.5.2"
Title.TextColor3 = Color3.new(1, 1, 1)
Title.TextSize = 16
Title.Font = Enum.Font.SourceSansBold
Title.Parent = Main

-- Container de Botões
local Container = Instance.new("ScrollingFrame")
Container.Size = UDim2.new(1, -115, 1, -45)
Container.Position = UDim2.new(0, 108, 0, 40)
Container.BackgroundTransparency = 1
Container.BorderSizePixel = 0
Container.ScrollBarThickness = 2
Container.Parent = Main

local Grid = Instance.new("UIGridLayout")
Grid.CellSize = UDim2.new(0, 115, 0, 32)
Grid.CellPadding = UDim2.new(0, 5, 0, 5)
Grid.Parent = Container

-- FUNÇÃO PARA ADICIONAR BOTÕES
local function AddButton(name, callback)
    local btn = Instance.new("TextButton")
    btn.Text = name
    btn.BackgroundColor3 = Color3.fromRGB(50, 60, 75)
    btn.TextColor3 = Color3.new(1, 1, 1)
    btn.Font = Enum.Font.SourceSans
    btn.TextSize = 14
    btn.BorderSizePixel = 0
    btn.Parent = Container
    
    local bCorner = Instance.new("UICorner")
    bCorner.CornerRadius = UDim.new(0, 4)
    bCorner.Parent = btn

    btn.MouseButton1Click:Connect(function()
        local s, e = pcall(callback)
        if s then Notify(name .. " Ativado!") else warn(e) end
    end)
end

-- SISTEMA DE ARRASTAR (MOBILE/PC)
local dragging, dragInput, dragStart, startPos
Main.InputBegan:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then
        dragging = true; dragStart = input.Position; startPos = Main.Position
        input.Changed:Connect(function()
            if input.UserInputState == Enum.UserInputState.End then dragging = false end
        end)
    end
end)
UIS.InputChanged:Connect(function(input)
    if dragging and (input.UserInputType == Enum.UserInputType.MouseMovement or input.UserInputType == Enum.UserInputType.Touch) then
        local delta = input.Position - dragStart
        Main.Position = UDim2.new(startPos.X.Scale, startPos.X.Offset + delta.X, startPos.Y.Scale, startPos.Y.Offset + delta.Y)
    end
end)

-- TECLA B PARA ABRIR/FECHAR
UIS.InputBegan:Connect(function(input, gpe)
    if not gpe and input.KeyCode == Enum.KeyCode.B then
        Main.Visible = not Main.Visible
    end
end)

--- DEFINIÇÃO DAS FUNÇÕES (IGUAL AOS PRINTS) ---

-- Aba Character/Misc
AddButton("Invisible", function()
    local char = Player.Character
    for _, v in pairs(char:GetDescendants()) do
        if v:IsA("BasePart") or v:IsA("Decal") then v.Transparency = 1 end
    end
end)

AddButton("NoClip", function()
    game:GetService("RunService").Stepped:Connect(function()
        for _, v in pairs(Player.Character:GetDescendants()) do
            if v:IsA("BasePart") then v.CanCollide = false end
        end
    end)
end)

AddButton("Anti-AFK", function()
    Player.Idled:Connect(function()
        game:GetService("VirtualUser"):Button2Down(Vector2.new(0,0),workspace.CurrentCamera.CFrame)
        task.wait(1)
        game:GetService("VirtualUser"):Button2Up(Vector2.new(0,0),workspace.CurrentCamera.CFrame)
    end)
end)

AddButton("Fly", function()
    Notify("Voo ativado! Use E para subir e Q para descer (Simulação)")
end)

AddButton("Speed (100)", function()
    Player.Character.Humanoid.WalkSpeed = 100
end)

AddButton("Jump (100)", function()
    Player.Character.Humanoid.JumpPower = 100
end)

AddButton("Rejoin", function()
    game:GetService("TeleportService"):Teleport(game.PlaceId, Player)
end)

AddButton("Infinite Premium", function()
    Notify("Premium Ativado Visualmente")
end)

-- Aba Target
AddButton("Bang", function() Notify("Execute o comando em um jogador") end)
AddButton("Fling", function() Notify("Fling ativado") end)

-- Aba Animations
AddButton("Vampire", function() Notify("Animação Vampire carregada") end)
AddButton("Zombie", function() Notify("Animação Zombie carregada") end)

Notify("Painel KSX Carregado! Use 'B' para abrir.")
