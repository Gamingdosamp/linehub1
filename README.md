-- Usando Orion Lib por ser mais leve e estável
local OrionLib = loadstring(game:HttpGet(('https://raw.githubusercontent.com/shlexware/Orion/main/source')))()

local Window = OrionLib:MakeWindow({
    Name = "💎 Premium Universal Hub", 
    HidePremium = false, 
    SaveConfig = true, 
    ConfigFolder = "GeminiHub",
    IntroText = "Carregando Gemini Hub..."
})

-- Variáveis
local Plrs = game:GetService("Players")
local LP = Plrs.LocalPlayer
local Mouse = LP:GetMouse()
local UserInputService = game:GetService("UserInputService")
local RunService = game:GetService("RunService")
local VirtualUser = game:GetService("VirtualUser")

local AntiAFK = false
local AntiFling = false
local ClickTP = false
local TargetPlayer = nil

-- [ FUNÇÃO ABRIR/FECHAR NA TECLA B ]
UserInputService.InputBegan:Connect(function(input, gameProcessed)
    if not gameProcessed and input.KeyCode == Enum.KeyCode.B then
        if game:GetService("CoreGui"):FindFirstChild("Orion") then
            local gui = game:GetService("CoreGui").Orion
            gui.Enabled = not gui.Enabled
        end
    end
end)

-- [ FUNÇÃO ANTI-AFK ]
LP.Idled:Connect(function()
    if AntiAFK then
        VirtualUser:CaptureController()
        VirtualUser:ClickButton2(Vector2.new())
    end
end)

-- [ FUNÇÃO CLICK TP (CTRL + CLICK) ]
Mouse.Button1Down:Connect(function()
    if ClickTP and UserInputService:IsKeyDown(Enum.KeyCode.LeftControl) then
        if Mouse.Target then
            LP.Character:MoveTo(Mouse.Hit.p)
        end
    end
end)

-- [ ABA MOVIMENTAÇÃO ]
local Tab1 = Window:MakeTab({ Name = "Movimentação", Icon = "rbxassetid://4483362458" })

Tab1:AddSlider({
    Name = "Velocidade (Speed)",
    Min = 16, Max = 300, Default = 16, Color = Color3.fromRGB(255,255,255),
    Callback = function(Value) LP.Character.Humanoid.WalkSpeed = Value end    
})

Tab1:AddToggle({
    Name = "Click TP (Segure CTRL + Clique)",
    Default = false,
    Callback = function(Value) ClickTP = Value end
})

Tab1:AddToggle({
    Name = "Noclip",
    Default = false,
    Callback = function(Value)
        _G.Noclip = Value
        RunService.Stepped:Connect(function()
            if _G.Noclip and LP.Character then
                for _, v in pairs(LP.Character:GetDescendants()) do
                    if v:IsA("BasePart") then v.CanCollide = false end
                end
            end
        end)
    end
})

-- [ ABA JOGADORES ]
local Tab2 = Window:MakeTab({ Name = "Jogadores", Icon = "rbxassetid://4483362458" })

Tab2:AddTextbox({
    Name = "Nome do Jogador",
    Default = "",
    TextDisappear = false,
    Callback = function(Text)
        for _, v in pairs(Plrs:GetPlayers()) do
            if v.Name:lower():sub(1, #Text) == Text:lower() then
                TargetPlayer = v
                OrionLib:MakeNotification({
                    Name = "Alvo Encontrado!",
                    Content = "Selecionado: " .. v.DisplayName,
                    Image = "rbxthumb://type=AvatarHeadShot&id=" .. v.UserId .. "&w=150&h=150",
                    Time = 5
                })
                break
            end
        end
    end	  
})

Tab2:AddButton({
    Name = "View / Unview (Alternar)",
    Callback = function()
        if workspace.CurrentCamera.CameraSubject == LP.Character.Humanoid then
            if TargetPlayer and TargetPlayer.Character then
                workspace.CurrentCamera.CameraSubject = TargetPlayer.Character.Humanoid
            end
        else
            workspace.CurrentCamera.CameraSubject = LP.Character.Humanoid
        end
    end
})

Tab2:AddButton({
    Name = "Teleportar ao Alvo",
    Callback = function()
        if TargetPlayer and TargetPlayer.Character then
            LP.Character.HumanoidRootPart.CFrame = TargetPlayer.Character.HumanoidRootPart.CFrame
        end
    end
})

-- [ ABA SEGURANÇA ]
local Tab3 = Window:MakeTab({ Name = "Segurança", Icon = "rbxassetid://4483362458" })

Tab3:AddToggle({
    Name = "Anti-AFK",
    Default = false,
    Callback = function(Value) AntiAFK = Value end
})

Tab3:AddToggle({
    Name = "Anti-Fling (Anti-Rovibes)",
    Default = false,
    Callback = function(Value)
        AntiFling = Value
        RunService.Heartbeat:Connect(function()
            if AntiFling and LP.Character then
                for _, v in pairs(Plrs:GetPlayers()) do
                    if v ~= LP and v.Character then
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
    end
})

OrionLib:Init()
