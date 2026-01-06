-- HUB UNIVERSAL SEM DEPENDÊNCIA EXTERNA (VERSÃO ROBUSTA)
local ScreenGui = Instance.new("ScreenGui")
local MainFrame = Instance.new("Frame")
local Title = Instance.new("TextLabel")
local Content = Instance.new("ScrollingFrame")
local UIListLayout = Instance.new("UIListLayout")

-- Configurações da Janela Principal
ScreenGui.Name = "GeminiHub"
ScreenGui.Parent = game:GetService("CoreGui")
ScreenGui.ResetOnSpawn = false

MainFrame.Name = "MainFrame"
MainFrame.Parent = ScreenGui
MainFrame.BackgroundColor3 = Color3.fromRGB(30, 30, 30)
MainFrame.Position = UDim2.new(0.3, 0, 0.3, 0)
MainFrame.Size = UDim2.new(0, 250, 0, 350)
MainFrame.Active = true
MainFrame.Draggable = true -- Deixa o painel arrastável

Title.Name = "Title"
Title.Parent = MainFrame
Title.Size = UDim2.new(1, 0, 0, 30)
Title.BackgroundColor3 = Color3.fromRGB(45, 45, 45)
Title.Text = "💎 PREMIUM HUB (B p/ fechar)"
Title.TextColor3 = Color3.fromRGB(255, 255, 255)
Title.TextSize = 14

Content.Name = "Content"
Content.Parent = MainFrame
Content.Position = UDim2.new(0, 0, 0, 30)
Content.Size = UDim2.new(1, 0, 1, -30)
Content.BackgroundTransparency = 1
Content.CanvasSize = UDim2.new(0, 0, 2, 0)

UIListLayout.Parent = Content
UIListLayout.Padding = UDim.new(0, 5)

-- Funções de Atalho (Abrir/Fechar com B)
game:GetService("UserInputService").InputBegan:Connect(function(input)
    if input.KeyCode == Enum.KeyCode.B then
        MainFrame.Visible = not MainFrame.Visible
    end
end)

-- FUNÇÃO: CRIAR BOTÕES FACILMENTE
local function CreateButton(txt, callback)
    local btn = Instance.new("TextButton")
    btn.Size = UDim2.new(0.9, 0, 0, 35)
    btn.Parent = Content
    btn.BackgroundColor3 = Color3.fromRGB(60, 60, 60)
    btn.Text = txt
    btn.TextColor3 = Color3.fromRGB(255, 255, 255)
    btn.Position = UDim2.new(0.05, 0, 0, 0)
    
    btn.MouseButton1Click:Connect(callback)
    return btn
end

-- [ SISTEMA DE BUSCA DE JOGADOR COM FOTO ]
local TargetName = ""
local TargetPlayer = nil
local PlayerIcon = Instance.new("ImageLabel")
PlayerIcon.Size = UDim2.new(0, 50, 0, 50)
PlayerIcon.Parent = Content
PlayerIcon.BackgroundTransparency = 1

local InputBox = Instance.new("TextBox")
InputBox.Size = UDim2.new(0.9, 0, 0, 35)
InputBox.Parent = Content
InputBox.PlaceholderText = "Nick do Jogador..."
InputBox.Text = ""

InputBox.FocusLost:Connect(function()
    for _, v in pairs(game.Players:GetPlayers()) do
        if v.Name:lower():sub(1, #InputBox.Text) == InputBox.Text:lower() then
            TargetPlayer = v
            InputBox.Text = v.DisplayName
            PlayerIcon.Image = "rbxthumb://type=AvatarHeadShot&id=" .. v.UserId .. "&w=150&h=150"
            break
        end
    end
end)

-- [ BOTÕES DE FUNÇÃO ]

CreateButton("Velocidade (100)", function()
    game.Players.LocalPlayer.Character.Humanoid.WalkSpeed = 100
end)

CreateButton("Pulo (150)", function()
    game.Players.LocalPlayer.Character.Humanoid.JumpPower = 150
end)

CreateButton("Alternar View (Mesmo Botão)", function()
    local cam = workspace.CurrentCamera
    if cam.CameraSubject == game.Players.LocalPlayer.Character.Humanoid then
        if TargetPlayer and TargetPlayer.Character then
            cam.CameraSubject = TargetPlayer.Character.Humanoid
        end
    else
        cam.CameraSubject = game.Players.LocalPlayer.Character.Humanoid
    end
end)

CreateButton("Teleportar ao Alvo", function()
    if TargetPlayer and TargetPlayer.Character then
        game.Players.LocalPlayer.Character.HumanoidRootPart.CFrame = TargetPlayer.Character.HumanoidRootPart.CFrame
    end
end)

-- [ ANTI-AFK ]
local antiAfkEnabled = false
CreateButton("Ativar/Desativar Anti-AFK", function()
    antiAfkEnabled = not antiAfkEnabled
    print("Anti-AFK: " .. tostring(antiAfkEnabled))
end)

game.Players.LocalPlayer.Idled:Connect(function()
    if antiAfkEnabled then
        game:GetService("VirtualUser"):CaptureController()
        game:GetService("VirtualUser"):ClickButton2(Vector2.new())
    end
end)

-- [ ANTI-FLING ]
local antiFling = false
CreateButton("Anti-Fling (Rovibes)", function()
    antiFling = not antiFling
end)

game:GetService("RunService").Heartbeat:Connect(function()
    if antiFling then
        for _, v in pairs(game.Players:GetPlayers()) do
            if v ~= game.Players.LocalPlayer and v.Character then
                for _, p in pairs(v.Character:GetDescendants()) do
                    if p:IsA("BasePart") then
                        p.CanCollide = false
                        p.Velocity = Vector3.new(0,0,0)
                    end
                end
            end
        end
    end
end)

print("Script Carregado com Sucesso!")
