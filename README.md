-- ==============================================================================
-- [PREMIUM CHEST FARM] - VERSÃO COORDENADAS EXATAS & COOLDOWN
-- Totalmente compatível com VOLT. Rotação controlada por tempo mínimo (8s).
-- ==============================================================================

local Players = game:GetService("Players")
local TweenService = game:GetService("TweenService")
local UserInputService = game:GetService("UserInputService")
local RunService = game:GetService("RunService")

local LocalPlayer = Players.LocalPlayer
local ChestFarmActive = false
local BlacklistedChests = {}

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
MainFrame.Size = UDim2.new(0, 300, 0, 150)
MainFrame.Position = UDim2.new(0.5, -150, 0.5, -75)
MainFrame.BackgroundColor3 = Color3.fromRGB(25, 25, 30)
MainFrame.BorderSizePixel = 0
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

-- ==============================================================================
-- 2. SISTEMA DE ARRASTE (DRAGGABLE)
-- ==============================================================================
local dragging = false
local dragInput, dragStart, startPos

MainFrame.InputBegan:Connect(function(input)
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

MainFrame.InputChanged:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseMovement or input.UserInputType == Enum.UserInputType.Touch then
        dragInput = input
    end
end)

UserInputService.InputChanged:Connect(function(input)
    if input == dragInput and dragging then
        local delta = input.Position - dragStart
        local targetPos = UDim2.new(startPos.X.Scale, startPos.X.Offset + delta.X, startPos.Y.Scale, startPos.Y.Offset + delta.Y)
        TweenService:Create(MainFrame, TweenInfo.new(0.1, Enum.EasingStyle.Quad, Enum.EasingDirection.Out), {Position = targetPos}):Play()
    end
end)

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
                -- Raio limite baseado no tamanho médio de uma ilha grande (1500 studs)
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
                -- Marca o tempo exato de entrada na ilha
                local islandStartTime = tick()
                
                -- 1. Teleporte instantâneo para a coordenada exata fornecida
                RootPart.CFrame = islandCFrame
                
                -- 2. Aguarda um momento inicial para renderização da ilha
                task.wait(1.5)
                
                -- 3. Loop de puxar baús locais na ilha atual
                local searchingChests = true
                while searchingChests and ChestFarmActive do
                    local targetChest = GetClosestChest()
                    
                    if targetChest then
                        -- Puxa localmente
                        targetChest.CFrame = RootPart.CFrame
                        
                        -- Coletas nativas e remotas simultâneas
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
                
                -- 4. CONTROLE DE COOLDOWN SEGURO (Mínimo de 8 segundos por ilha)
                local timeSpentOnIsland = tick() - islandStartTime
                if timeSpentOnIsland < 8 then
                    -- Calcula o tempo restante para completar os 8 segundos e aguarda
                    task.wait(8 - timeSpentOnIsland)
                end
            end
        end
        -- Reset da blacklist após passar pelas 8 ilhas para permitir coletar o respawn
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
