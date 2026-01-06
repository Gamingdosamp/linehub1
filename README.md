local Rayfield = loadstring(game:HttpGet('https://sirius.menu/rayfield'))()

local Window = Rayfield:CreateWindow({
   Name = "💎 Premium Universal Hub v2",
   LoadingTitle = "Carregando Interface...",
   LoadingSubtitle = "por Gemini AI",
   ConfigurationSaving = {
      Enabled = true,
      FolderName = "GeminiHub",
      FileName = "Config"
   }
})

-- Variáveis de Controle
local Plrs = game:GetService("Players")
local LocalPlayer = Plrs.LocalPlayer
local RunService = game:GetService("RunService")
local Camera = workspace.CurrentCamera
local VirtualUser = game:GetService("VirtualUser")

local FlyEnabled = false
local AntiFlingEnabled = false
local AntiAFKEnabled = false
local SpeedValue = 16
local JumpValue = 50
local ViewTarget = nil

-- Conexão Anti-AFK (Impedir desconexão por ociosidade)
LocalPlayer.Idled:Connect(function()
    if AntiAFKEnabled then
        VirtualUser:CaptureController()
        VirtualUser:ClickButton2(Vector2.new())
    end
end)

-- Funções de Suporte
local function GetPlayer(String)
    for _, v in pairs(Plrs:GetPlayers()) do
        if v.Name:lower():sub(1, #String) == String:lower() or v.DisplayName:lower():sub(1, #String) == String:lower() then
            return v
        end
    end
end

-- Aba Principal: Movimentação
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
      SpeedValue = Value
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
      JumpValue = Value
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

-- Aba Jogadores: View e Teleport
local PlayerTab = Window:CreateTab("Jogadores", 4483362458)

local PlayerImageLabel = PlayerTab:CreateLabel("Aguardando nick...", 4483362458)

PlayerTab:CreateInput({
   Name = "Nome do Jogador",
   PlaceholderText = "Escreva o nick aqui...",
   RemoveTextAfterFocusLost = false,
   Callback = function(Text)
      local target = GetPlayer(Text)
      if target then
         ViewTarget = target
         PlayerImageLabel:Set("Alvo Selecionado: " .. target.DisplayName)
         Rayfield:Notify({Title = "Sucesso", Content = "Jogador " .. target.Name .. " encontrado!", Duration = 3})
      else
         PlayerImageLabel:Set("Jogador não encontrado.")
      end
   end,
})

PlayerTab:CreateButton({
   Name = "View / Unview (Alternar)",
   Callback = function()
      if Camera.CameraSubject == LocalPlayer.Character.Humanoid then
         if ViewTarget and ViewTarget.Character and ViewTarget.Character:FindFirstChild("Humanoid") then
            Camera.CameraSubject = ViewTarget.Character.Humanoid
            Rayfield:Notify({Title = "View", Content = "Observando: " .. ViewTarget.DisplayName, Duration = 3})
         else
            Rayfield:Notify({Title = "Erro", Content = "Selecione um jogador válido primeiro.", Duration = 3})
         end
      else
         Camera.CameraSubject = LocalPlayer.Character.Humanoid
         Rayfield:Notify({Title = "View", Content = "Câmera resetada para o seu personagem.", Duration = 3})
      end
   end,
})

PlayerTab:CreateButton({
   Name = "Teleportar ao Alvo",
   Callback = function()
      if ViewTarget and ViewTarget.Character and ViewTarget.Character:FindFirstChild("HumanoidRootPart") then
         LocalPlayer.Character.HumanoidRootPart.CFrame = ViewTarget.Character.HumanoidRootPart.CFrame
      else
         Rayfield:Notify({Title = "Erro", Content = "Não foi possível teleportar.", Duration = 3})
      end
   end,
})

-- Aba Segurança
local SafetyTab = Window:CreateTab("Segurança", 4483362458)

SafetyTab:CreateToggle({
   Name = "Anti-AFK (Evitar Kick)",
   CurrentValue = false,
   Callback = function(Value)
      AntiAFKEnabled = Value
      if Value then
         Rayfield:Notify({Title = "Anti-AFK", Content = "Ativado! Você não será desconectado.", Duration = 3})
      else
         Rayfield:Notify({Title = "Anti-AFK", Content = "Desativado.", Duration = 3})
      end
   end,
})

SafetyTab:CreateToggle({
   Name = "Anti-Fling (Física Estática)",
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

Rayfield:LoadConfiguration()
