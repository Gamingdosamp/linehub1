local Rayfield = loadstring(game:HttpGet('https://sirius.menu/rayfield'))()

-- Configuração da Janela (Arrastável por padrão)
local Window = Rayfield:CreateWindow({
   Name = "💎 Premium Universal Hub v3",
   LoadingTitle = "Iniciando Sistema Avançado...",
   LoadingSubtitle = "por Gemini AI",
   ConfigurationSaving = {
      Enabled = true,
      FolderName = "GeminiHub",
      FileName = "Config"
   },
   KeybindSource = "B" -- Tecla padrão para abrir/fechar o menu
})

-- Variáveis de Controle
local Plrs = game:GetService("Players")
local LocalPlayer = Plrs.LocalPlayer
local RunService = game:GetService("RunService")
local UserInputService = game:GetService("UserInputService")
local Camera = workspace.CurrentCamera
local VirtualUser = game:GetService("VirtualUser")
local Mouse = LocalPlayer:GetMouse()

local AntiFlingEnabled = false
local AntiAFKEnabled = false
local ClickTPEnabled = false
local ViewTarget = nil

-- [ FUNÇÃO ANTI-AFK ]
LocalPlayer.Idled:Connect(function()
    if AntiAFKEnabled then
        VirtualUser:CaptureController()
        VirtualUser:ClickButton2(Vector2.new())
    end
end)

-- [ FUNÇÃO CLICK TELEPORT (CTRL + CLICK) ]
Mouse.Button1Down:Connect(function()
    if ClickTPEnabled and UserInputService:IsKeyDown(Enum.KeyCode.LeftControl) then
        if Mouse.Target then
            LocalPlayer.Character:MoveTo(Mouse.Hit.p)
        end
    end
end)

-- [ FUNÇÃO AUXILIAR BUSCA JOGADOR ]
local function GetPlayer(String)
    for _, v in pairs(Plrs:GetPlayers()) do
        if v.Name:lower():sub(1, #String) == String:lower() or v.DisplayName:lower():sub(1, #String) == String:lower() then
            return v
        end
    end
end

-- ==========================================
-- ABA: MOVIMENTAÇÃO
-- ==========================================
local MainTab = Window:CreateTab("Movimentação", 4483362458)

MainTab:CreateSlider({
   Name = "Velocidade (Speed)",
   Range = {16, 300},
   Increment = 1,
   CurrentValue = 16,
   Callback = function(Value)
      if LocalPlayer.Character and LocalPlayer.Character:FindFirstChild("Humanoid") then
         LocalPlayer.Character.Humanoid.WalkSpeed = Value
      end
   end,
})

MainTab:CreateSlider({
   Name = "Pulo (Jump)",
   Range = {50, 500},
   Increment = 1,
   CurrentValue = 50,
   Callback = function(Value)
      if LocalPlayer.Character and LocalPlayer.Character:FindFirstChild("Humanoid") then
         LocalPlayer.Character.Humanoid.JumpPower = Value
      end
   end,
})

MainTab:CreateToggle({
   Name = "Click TP (Segurar CTRL + Clique)",
   CurrentValue = false,
   Callback = function(Value)
      ClickTPEnabled = Value
      if Value then
         Rayfield:Notify({Title = "Click TP", Content = "Ativado! Segure CTRL e clique em um lugar.", Duration = 3})
      end
   end,
})

MainTab:CreateToggle({
   Name = "Noclip (Atravessar Paredes)",
   CurrentValue = false,
   Callback = function(Value)
      _G.Noclip = Value
      RunService.Stepped:Connect(function()
         if _G.Noclip and LocalPlayer.Character then
            for _, part in pairs(LocalPlayer.Character:GetDescendants()) do
               if part:IsA("BasePart") then part.CanCollide = false end
            end
         end
      end)
   end,
})

-- ==========================================
-- ABA: JOGADORES (VIEW/TP)
-- ==========================================
local PlayerTab = Window:CreateTab("Jogadores", 4483362458)

local PlayerImageLabel = PlayerTab:CreateLabel("Aguardando nick...", 4483362458)

PlayerTab:CreateInput({
   Name = "Nome do Jogador",
   PlaceholderText = "Escreva o nick ou começo dele...",
   RemoveTextAfterFocusLost = false,
   Callback = function(Text)
      local target = GetPlayer(Text)
      if target then
         ViewTarget = target
         PlayerImageLabel:Set("Alvo: " .. target.DisplayName)
         
         -- Sistema de confirmação visual via Notificação (Rayfield suporta ícones)
         local userId = target.UserId
         local thumb = "rbxthumb://type=AvatarHeadShot&id=" .. userId .. "&w=150&h=150"
         
         Rayfield:Notify({
            Title = "Jogador Identificado",
            Content = "Nome: " .. target.Name,
            Image = thumb,
            Duration = 5
         })
      else
         PlayerImageLabel:Set("Não encontrado.")
      end
   end,
})

PlayerTab:CreateButton({
   Name = "View / Unview (Alternar Mesma Tecla)",
   Callback = function()
      if Camera.CameraSubject == LocalPlayer.Character.Humanoid then
         if ViewTarget and ViewTarget.Character and ViewTarget.Character:FindFirstChild("Humanoid") then
            Camera.CameraSubject = ViewTarget.Character.Humanoid
            Rayfield:Notify({Title = "View", Content = "Observando: " .. ViewTarget.DisplayName, Duration = 3})
         end
      else
         Camera.CameraSubject = LocalPlayer.Character.Humanoid
         Rayfield:Notify({Title = "View", Content = "Câmera resetada.", Duration = 3})
      end
   end,
})

PlayerTab:CreateButton({
   Name = "Teleportar ao Alvo",
   Callback = function()
      if ViewTarget and ViewTarget.Character and ViewTarget.Character:FindFirstChild("HumanoidRootPart") then
         LocalPlayer.Character.HumanoidRootPart.CFrame = ViewTarget.Character.HumanoidRootPart.CFrame
      end
   end,
})

-- ==========================================
-- ABA: SEGURANÇA (ANTI-FLING / AFK)
-- ==========================================
local SafetyTab = Window:CreateTab("Segurança", 4483362458)

SafetyTab:CreateToggle({
   Name = "Anti-AFK (Impedir Kick)",
   CurrentValue = false,
   Callback = function(Value)
      AntiAFKEnabled = Value
   end,
})

SafetyTab:CreateToggle({
   Name = "Anti-Fling (Anti-Rovibes)",
   CurrentValue = false,
   Callback = function(Value)
      AntiFlingEnabled = Value
      if Value then
         RunService.Heartbeat:Connect(function()
            if AntiFlingEnabled and LocalPlayer.Character then
               for _, v in pairs(Plrs:GetPlayers()) do
                  if v ~= LocalPlayer and v.Character then
                     for _, part in pairs(v.Character:GetDescendants()) do
                        if part:IsA("BasePart") then
                           part.CanCollide = false
                           part.Velocity = Vector3.new(0,0,0)
                           part.RotVelocity = Vector3.new(0,0,0)
                        end
                     end
                  end
               end
            end
         end)
      end
   end,
})

-- Notificação Inicial
Rayfield:Notify({
   Title = "Painel Carregado",
   Content = "Pressione 'B' para abrir ou fechar o menu!",
   Duration = 5
})

Rayfield:LoadConfiguration()
