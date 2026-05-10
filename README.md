local Player = game:GetService("Players").LocalPlayer
local CoreGui = game:GetService("CoreGui")
local RunService = game:GetService("RunService")
local UserInputService = game:GetService("UserInputService")

-- Configurações
local FORCA_IMPULSO = 250 -- Aumentado para um impulso maior
local DISTANCIA_PE = -3.2
local wAtivo = false
local fAtivo = false

-- Interface Mobile
local ScreenGui = Instance.new("ScreenGui")
ScreenGui.Name = "MobileControls"
ScreenGui.Parent = CoreGui
ScreenGui.ResetOnSpawn = false

-- Função para criar botões estilosos para Mobile
local function criarBotao(nome, pos, cor)
    local btn = Instance.new("TextButton")
    btn.Name = nome
    btn.Size = UDim2.new(0, 70, 0, 70)
    btn.Position = pos
    btn.BackgroundColor3 = cor
    btn.BackgroundTransparency = 0.4
    btn.Text = nome
    btn.TextColor3 = Color3.new(1, 1, 1)
    btn.Font = Enum.Font.SourceSansBold
    btn.TextSize = 30
    btn.Parent = ScreenGui
    
    -- Deixa o botão redondo
    local corner = Instance.new("UICorner")
    corner.CornerRadius = UDim.new(1, 0)
    corner.Parent = btn

    return btn
end

-- Posicionando os botões no canto inferior direito
local btnW = criarBotao("W", UDim2.new(0.8, 0, 0.6, 0), Color3.fromRGB(0, 170, 255))
local btnF = criarBotao("F", UDim2.new(0.9, 0, 0.75, 0), Color3.fromRGB(255, 85, 0))

-- Lógica de Input (Teclado)
UserInputService.InputBegan:Connect(function(input, proc)
    if proc then return end
    if input.KeyCode == Enum.KeyCode.W then wAtivo = true end
    if input.KeyCode == Enum.KeyCode.F then fAtivo = true end
end)

UserInputService.InputEnded:Connect(function(input)
    if input.KeyCode == Enum.KeyCode.W then wAtivo = false end
    if input.KeyCode == Enum.KeyCode.F then fAtivo = false end
end)

-- Lógica de Input (Mobile)
btnW.MouseButton1Down:Connect(function() wAtivo = true end)
btnW.MouseButton1Up:Connect(function() wAtivo = false end)
btnF.MouseButton1Down:Connect(function() fAtivo = true end)
btnF.MouseButton1Up:Connect(function() fAtivo = false end)

-- Função de Reciclagem
local function acharParte()
    for _, obj in pairs(game.Workspace:GetDescendants()) do
        if obj:IsA("Part") and obj.CanCollide and not obj:IsDescendantOf(Player.Character) then
            return obj
        end
    end
    return nil
end

local parteReciclada = acharParte()

-- Loop Principal de Física
RunService.RenderStepped:Connect(function()
    local char = Player.Character
    local hrp = char and char:FindFirstChild("HumanoidRootPart")
    local hum = char and char:FindFirstChildOfClass("Humanoid")
    
    if not hrp or not hum then return end
    if not parteReciclada or not parteReciclada.Parent then parteReciclada = acharParte() end

    -- Ação da Tecla W (Air Jump / Escada)
    if wAtivo then
        hum:Move(Vector3.new(0, 0, -1), true) -- Força andar para frente
        parteReciclada.Size = Vector3.new(6, 0.5, 6)
        parteReciclada.Transparency = 1
        parteReciclada.Anchored = true
        parteReciclada.CanCollide = true
        parteReciclada.CFrame = hrp.CFrame * CFrame.new(0, DISTANCIA_PE, 0)
        hum:ChangeState(Enum.HumanoidStateType.Jumping)
    end

    -- Ação da Tecla F (Super Dash)
    if fAtivo then
        -- Torna a parte minúscula para não atrapalhar
        parteReciclada.Size = Vector3.new(0.1, 0.1, 0.1)
        parteReciclada.Transparency = 1
        parteReciclada.Anchored = false -- Solta para não travar o boneco
        parteReciclada.CanCollide = false
        
        -- Aplica o Super Impulso na direção da câmera
        hrp.AssemblyLinearVelocity = hrp.CFrame.LookVector * FORCA_IMPULSO
    end
end)
