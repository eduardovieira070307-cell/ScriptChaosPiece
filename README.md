-- ==============================================================================
-- [PREMIUM CHEST FARM] - VERSÃO MULTI-ILHAS (STREAMING BYPASS)
-- Compatível com VOLT. Rotação automática pelas 8 ilhas com puxador local.
-- ==============================================================================

local Players = game:GetService("Players")
local TweenService = game:GetService("TweenService")
local UserInputService = game:GetService("UserInputService")
local RunService = game:GetService("RunService")

local LocalPlayer = Players.LocalPlayer
local ChestFarmActive = false
local BlacklistedChests = {}

-- Lista exata das ilhas fornecidas
local IslandNames = {
    "Starter",
    "Forest",
    "Rocky",
    "Desert",
    "Ice",
    "Sky kingdom",
    "Marineford",
    "Business district"
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
-- 3. LÓGICA DE DETECÇÃO E TELEPORTE DE ILHAS
-- ==============================================================================

-- Função para achar o objeto ou pasta da Ilha no Workspace
local function FindIslandObject(islandName)
    for _, v in pairs(workspace:GetChildren()) do
        if string.find(string.lower(v.Name), string.lower(islandName)) then
            return v
        end
    end
    -- Procura secundária caso estejam dentro de uma pasta de Ilhas
    for _, v in pairs(workspace:GetDescendants()) do
        if (v:IsA("BasePart") or v:IsA("Model")) and string.find(string.lower(v.Name), string.lower(islandName)) then
            return v
        end
    end
    return nil
end

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
                -- Apenas pega baús que estão perto (na mesma ilha carregada)
                if distance < shortestDistance and distance < 3500 then
                    shortestDistance = distance
                    closestChest = obj
                end
            end
        end
    end
    return closestChest
end

-- Loop principal baseado em Rota de Ilhas
local function FarmLoop()
    while ChestFarmActive do
        for _, islandName in ipairs(IslandNames) do
            if not ChestFarmActive then break end
            
            local islandObj = FindIslandObject(islandName)
            local Character = LocalPlayer.Character
            local RootPart = Character and Character:FindFirstChild("HumanoidRootPart")
            
            if RootPart and islandObj then
                -- Define a posição da ilha (seja ela um Bloco ou um Modelo)
                local islandCFrame = nil
                if islandObj:IsA("BasePart") then
                    islandCFrame = islandObj.CFrame
                elseif islandObj:IsA("Model") then
                    islandCFrame = islandObj:GetPivot()
                end
                
                if islandCFrame then
                    -- 1. Teleporta para a ilha (coloca 30 studs acima para segurança contra colisões)
                    RootPart.CFrame = islandCFrame * CFrame.new(0, 30, 0)
                    
                    -- 2. ESPERA CRÍTICA: Aguarda as partes da ilha carregarem no PC (Bypass StreamingEnabled)
                    task.wait(1.5)
                    
                    -- 3. Puxa todos os baús disponíveis nesta ilha antes de ir para a próxima
                    local searchingChests = true
                    while searchingChests and ChestFarmActive do
                        local targetChest = GetClosestChest()
                        
                        if targetChest then
                            -- Traz o baú até o jogador localmente
                            targetChest.CFrame = RootPart.CFrame
                            
                            -- Coleta instantânea
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
                            task.wait(0.1) -- Delay rápido entre baús da mesma ilha
                        else
                            -- Se não achar mais nenhum baú perto, sai do loop desta ilha
                            searchingChests = false
                        end
                    end
                end
            end
            task.wait(0.5) -- Pequena pausa antes de saltar para a próxima ilha
        end
        -- Uma vez que passou por todas as 8 ilhas, limpa a blacklist para a próxima rodada (respawn)
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
