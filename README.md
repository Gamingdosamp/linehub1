-- Tentativa de forçar a renderização acima de tudo
local function CreateUI()
    local sg = Instance.new("ScreenGui")
    -- Tentamos colocar no CoreGui, se falhar, vai pro PlayerGui
    local success, err = pcall(function()
        sg.Parent = game:GetService("CoreGui")
    end)
    if not success then
        sg.Parent = game.Players.LocalPlayer:WaitForChild("PlayerGui")
    end
    
    sg.Name = "GeminiUltra"
    sg.DisplayOrder = 999999 -- Força ficar na frente de tudo

    local main = Instance.new("Frame", sg)
    main.Size = UDim2.new(0, 220, 0, 300)
    main.Position = UDim2.new(0.5, -110, 0.5, -150)
    main.BackgroundColor3 = Color3.fromRGB(20, 20, 20)
    main.BorderSizePixel = 2
    main.Active = true
    main.Draggable = true

    local title = Instance.new("TextLabel", main)
    title.Size = UDim2.new(1, 0, 0, 30)
    title.Text = "GEMINI HUB [B]"
    title.BackgroundColor3 = Color3.fromRGB(100, 0, 0)
    title.TextColor3 = Color3.new(1,1,1)

    local btnContainer = Instance.new("ScrollingFrame", main)
    btnContainer.Size = UDim2.new(1, -10, 1, -40)
    btnContainer.Position = UDim2.new(0, 5, 0, 35)
    btnContainer.BackgroundTransparency = 1
    btnContainer.CanvasSize = UDim2.new(0,0,2,0)

    local layout = Instance.new("UIListLayout", btnContainer)
    layout.Padding = UDim.new(0, 5)

    -- Função de Botão Simples
    local function AddBtn(name, fn)
        local b = Instance.new("TextButton", btnContainer)
        b.Size = UDim2.new(1, 0, 0, 30)
        b.Text = name
        b.BackgroundColor3 = Color3.fromRGB(50,50,50)
        b.TextColor3 = Color3.new(1,1,1)
        b.MouseButton1Click:Connect(fn)
    end

    -- FUNÇÕES
    AddBtn("Speed 100", function() game.Players.LocalPlayer.Character.Humanoid.WalkSpeed = 100 end)
    AddBtn("Jump 150", function() game.Players.LocalPlayer.Character.Humanoid.JumpPower = 150 end)
    
    local afk = false
    AddBtn("Anti-AFK: OFF", function(self) 
        afk = not afk 
        print("Anti-AFK:", afk)
    end)

    -- Tecla B para fechar
    game:GetService("UserInputService").InputBegan:Connect(function(i)
        if i.KeyCode == Enum.KeyCode.B then
            main.Visible = not main.Visible
        end
    end)

    -- Anti-AFK Loop
    game.Players.LocalPlayer.Idled:Connect(function()
        if afk then
            game:GetService("VirtualUser"):Button2Down(Vector2.new(0,0), workspace.CurrentCamera.CFrame)
            wait(1)
            game:GetService("VirtualUser"):Button2Up(Vector2.new(0,0), workspace.CurrentCamera.CFrame)
        end
    end)
end

-- Executa a criação
pcall(CreateUI)
warn("Gemini Hub tentou carregar!")
