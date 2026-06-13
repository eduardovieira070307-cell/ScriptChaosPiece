-- ==============================================================================
-- [PREMIUM CHEST FARM] - VERSÃO DEFINITIVA (UI ANIMADA + HOTKEY)
-- Compatível com VOLT. Rotação controlada, Cooldown (8s) e UI Premium.
-- ==============================================================================

local Players = game:GetService("Players")
local TweenService = game:GetService("TweenService")
local UserInputService = game:GetService("UserInputService")
local RunService = game:GetService("RunService")

local LocalPlayer = Players.LocalPlayer
local ChestFarmActive = false
local BlacklistedChests = {}

-- Configuração da Tecla de Atalho (Esconder/Mostrar UI)
local TOGGLE_KEY = Enum.KeyCode.RightControl

-- Coordenadas exatas fornecidas para cada uma das 8 ilhas
local IslandPositions = {
    CFrame.new(-189.54974365234375, 13.674964904785156, -609.3209228515625), -- 1. Starter
    CFrame.new(-856.2052612304688, 11.592161178588867, 662.95654296875),    -- 2. Forest
    CFrame.new(1372.12255859375, 3.61570405960083, 1822.9290771484375),     -- 3. Rocky
    CFrame.new(3403.5087890625, 16.919490814208984, 402.89752197265625),    -- 4. Desert
    CFrame.new(-397.72119140625, 36.68341064453125, -3268.46484375),        -- 5. Ice
    CFrame.new(-273.8543395996094, 1582.9114990234375, 2272.558349609375),  -- 6. Sky kingdom
    CFrame.new(2544.197021484375, 57.05317687988281, -4897.49267578125),    -- 7. Marineford
    CFrame.new(2897.0458984375, 83.3616943359375, -2853.43603515625)         -- 8. Business district
}

-- ==============================================================================
-- 1. CRIAÇÃO DA INTERFACE GRÁFICA (UI)
-- ==============================================================================
local ScreenGui = Instance.new("ScreenGui")
ScreenGui.Name = "PremiumChestFarmGui"
ScreenGui.ResetOnSpawn = false

local success = pcall(function() ScreenGui.Parent = game.CoreGui end)
if not success then ScreenGui.Parent = LocalPlayer:WaitForChild("PlayerGui") end

local MainFrame = Instance.new("Frame")
MainFrame.Name = "MainFrame"
-- AnchorPoint centralizado para a animação de zoom sair do meio
MainFrame.AnchorPoint = Vector2.new(0.5, 0.5) 
MainFrame.Position = UDim2.new(0.5, 0, 0.5, 0)
MainFrame.Size = UDim2.new(0, 0, 0, 0) -- Começa invisível/tamanho zero para animar
MainFrame.BackgroundColor3 = Color3.fromRGB(25, 25, 30)
MainFrame.BorderSizePixel = 0
MainFrame.ClipsDescendants = true -- Fundamental para o sistema de minimizar
MainFrame.Parent = ScreenGui

local MainCorner = Instance.new("UICorner")
MainCorner.CornerRadius = UDim.new(0, 12)
MainCorner.Parent = MainFrame

local MainStroke = Instance.new("UIStroke")
MainStroke.Color = Color3.fromRGB(60, 60, 70)
MainStroke.Thickness = 1.5
MainStroke.Parent = MainFrame

local TitleLabel = Instance.new("TextLabel")
TitleLabel.Name = "TitleLabel"
TitleLabel.Size = UDim2.new(1, 0, 0, 40)
TitleLabel.BackgroundTransparency = 1
TitleLabel.Text = "✧ Island Master Farm ✧"
TitleLabel.TextColor3 = Color3.fromRGB(255, 255, 255)
TitleLabel.Font = Enum.Font.GothamBold
TitleLabel.TextSize = 18
TitleLabel.Parent = MainFrame

-- Botão de Minimizar
local MinimizeBtn = Instance.new("TextButton")
MinimizeBtn.Name = "MinimizeBtn"
MinimizeBtn.Size = UDim2.new(0, 30, 0, 30)
MinimizeBtn.Position = UDim2.new(1, -35, 0, 5)
MinimizeBtn.BackgroundTransparency = 1
MinimizeBtn.Text = "-"
MinimizeBtn.TextColor3 = Color3.fromRGB(200, 200, 200)
MinimizeBtn.Font = Enum.Font.GothamBold
MinimizeBtn.TextSize = 22
MinimizeBtn.Parent = MainFrame

local ToggleButton = Instance.new("TextButton")
ToggleButton.Name = "ToggleButton"
ToggleButton.Size = UDim2.new(0, 200, 0, 45)
ToggleButton.Position = UDim2.new(0.5, -100, 0.6, -10)
ToggleButton.BackgroundColor3 = Color3.fromRGB(230, 60, 60)
ToggleButton.Text = "Chest Farm: OFF"
ToggleButton.TextColor3 = Color3.fromRGB(255, 255, 255)
ToggleButton.Font = Enum.Font.GothamSemibold
ToggleButton.TextSize = 16
ToggleButton.AutoButtonColor = false
ToggleButton.Parent = MainFrame

local ButtonCorner = Instance.new("UICorner")
ButtonCorner.CornerRadius = UDim.new(0, 8)
ButtonCorner.Parent = ToggleButton

-- Dica de Atalho (Aparece embaixo do botão)
local HintLabel = Instance.new("TextLabel")
HintLabel.Name = "HintLabel"
HintLabel.Size = UDim2.new(1, 0, 0, 20)
HintLabel.Position = UDim2.new(0, 0, 1, -25)
HintLabel.BackgroundTransparency = 1
HintLabel.Text = "Aperte 'RightControl' para esconder a UI"
HintLabel.TextColor3 = Color3.fromRGB(150, 150, 150)
HintLabel.Font = Enum.Font.Gotham
HintLabel.TextSize = 11
HintLabel.Parent = MainFrame

-- ==============================================================================
-- 2. SISTEMA DE ARRASTE, ANIMAÇÕES E HOTKEY
-- ==============================================================================
local dragging = false
local dragInput, dragStart, startPos

-- Arrastar pela barra de título
TitleLabel.InputBegan:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then
        dragging = true
        dragStart = input.Position
        startPos = MainFrame.Position
        
        input.Changed:Connect(function()
            if input.UserInputState == Enum.UserInputState.End then
                dragging = false
            end
        end)
    end
end)

UserInputService.InputChanged:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseMovement or input.UserInputType == Enum.UserInputType.Touch then
        dragInput = input
    end
    
    if input == dragInput and dragging then
        local delta = input.Position - dragStart
        local targetPos = UDim2.new(startPos.X.Scale, startPos.X.Offset + delta.X, startPos.Y.Scale, startPos.Y.Offset + delta.Y)
        TweenService:Create(MainFrame, TweenInfo.new(0.1, Enum.EasingStyle.Quad, Enum.EasingDirection.Out), {Position = targetPos}):Play()
    end
end)

-- Sistema de Minimizar
local isMinimized = false
MinimizeBtn.Activated:Connect(function()
    isMinimized = not isMinimized
    local targetSize = isMinimized and UDim2.new(0, 300, 0, 40) or UDim2.new(0, 300, 0, 150)
    
    TweenService:Create(MainFrame, TweenInfo.new(0.3, Enum.EasingStyle.Back, Enum.EasingDirection.Out), {Size = targetSize}):Play()
end)

-- Sistema de Hotkey (Esconder tudo)
UserInputService.InputBegan:Connect(function(input, gameProcessed)
    if not gameProcessed and input.KeyCode == TOGGLE_KEY then
        ScreenGui.Enabled = not ScreenGui.Enabled
    end
end)

-- ANIMAÇÃO DE ENTRADA AO ABRIR O SCRIPT
TweenService:Create(MainFrame, TweenInfo.new(0.6, Enum.EasingStyle.Back, Enum.EasingDirection.Out), {
    Size = UDim2.new(0, 300, 0, 150)
}):Play()

-- ==============================================================================
-- 3. LÓGICA DO FARM COM ROTAÇÃO FIXA E CONTROLE DE COOLDOWN
-- ==============================================================================

local function GetClosestChest()
    local Character = LocalPlayer.Character
    if not Character or not Character:FindFirstChild("HumanoidRootPart") then return nil end
    local RootPart = Character.HumanoidRootPart

    local closestChest = nil
    local shortestDistance = math.huge

    for _, obj in pairs(workspace:GetDescendants()) do
        if obj:IsA("BasePart") and string.find(string.lower(obj.Name), "chest") then
            if not BlacklistedChests[obj] then
                local distance = (obj.Position - RootPart.Position).Magnitude
                if distance < shortestDistance and distance < 1500 then
                    shortestDistance = distance
                    closestChest = obj
                end
            end
        end
    end
    return closestChest
end

local function FarmLoop()
    while ChestFarmActive do
        for index, islandCFrame in ipairs(IslandPositions) do
            if not ChestFarmActive then break end
            
            local Character = LocalPlayer.Character
            local RootPart = Character and Character:FindFirstChild("HumanoidRootPart")
            
            if RootPart then
                local islandStartTime = tick()
                
                -- Teleporte e aguarda carregar mapa
                RootPart.CFrame = islandCFrame
                task.wait(1.5)
                
                -- Coleta os baús
                local searchingChests = true
                while searchingChests and ChestFarmActive do
                    local targetChest = GetClosestChest()
                    
                    if targetChest then
                        targetChest.CFrame = RootPart.CFrame
                        
                        pcall(function()
                            if firetouchinterest then
                                firetouchinterest(RootPart, targetChest, 0)
                                firetouchinterest(RootPart, targetChest, 1)
                            end
                        end)
                        pcall(function()
                            local prompt = targetChest:FindFirstChildWhichIsA("ProximityPrompt", true)
                            if prompt and fireproximityprompt then
                                fireproximityprompt(prompt)
                            end
                        end)
                        
                        BlacklistedChests[targetChest] = true
                        task.wait(0.1)
                    else
                        searchingChests = false
                    end
                end
                
                -- Controle de Cooldown (Mínimo de 8s)
                local timeSpentOnIsland = tick() - islandStartTime
                if timeSpentOnIsland < 8 then
                    task.wait(8 - timeSpentOnIsland)
                end
            end
        end
        BlacklistedChests = {}
        task.wait(2)
    end
end

-- ==============================================================================
-- 4. CONTROLE DO BOTÃO
-- ==============================================================================
local function AnimateButton(isHovering)
    if ChestFarmActive then return end
    local color = isHovering and Color3.fromRGB(240, 80, 80) or Color3.fromRGB(230, 60, 60)
    TweenService:Create(ToggleButton, TweenInfo.new(0.2), {BackgroundColor3 = color}):Play()
end

ToggleButton.MouseEnter:Connect(function() AnimateButton(true) end)
ToggleButton.MouseLeave:Connect(function() AnimateButton(false) end)

ToggleButton.Activated:Connect(function()
    ChestFarmActive = not ChestFarmActive

    local targetColor = ChestFarmActive and Color3.fromRGB(60, 200, 100) or Color3.fromRGB(230, 60, 60)
    local targetText = ChestFarmActive and "Chest Farm: ON" or "Chest Farm: OFF"
    
    TweenService:Create(ToggleButton, TweenInfo.new(0.3, Enum.EasingStyle.Quad, Enum.EasingDirection.Out), {
        BackgroundColor3 = targetColor
    }):Play()
    ToggleButton.Text = targetText

    if ChestFarmActive then
        BlacklistedChests = {}
        coroutine.wrap(FarmLoop)()
    end
end)
