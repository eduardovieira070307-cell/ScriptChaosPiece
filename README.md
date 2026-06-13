-- ==============================================================================
-- [PREMIUM CHEST FARM UI & SCRIPT] - VERSÃO INSTANTÂNEA / TELEPORTE
-- ==============================================================================

local Players = game:GetService("Players")
local TweenService = game:GetService("TweenService")
local UserInputService = game:GetService("UserInputService")
local RunService = game:GetService("RunService")

local LocalPlayer = Players.LocalPlayer
local ChestFarmActive = false

-- ==============================================================================
-- 1. CRIAÇÃO DA INTERFACE GRÁFICA (UI)
-- ==============================================================================
local ScreenGui = Instance.new("ScreenGui")
ScreenGui.Name = "PremiumChestFarmGui"
ScreenGui.ResetOnSpawn = false
ScreenGui.Parent = game.CoreGui or LocalPlayer:WaitForChild("PlayerGui")

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
TitleLabel.Text = "✧ Fast Chest Farm ✧"
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
-- 3. LÓGICA DO CHEST FARM (TELEPORTE INSTANTÂNEO)
-- ==============================================================================

local function GetClosestChest()
    local Character = LocalPlayer.Character
    if not Character or not Character:FindFirstChild("HumanoidRootPart") then return nil end
    local RootPart = Character.HumanoidRootPart

    local closestChest = nil
    local shortestDistance = math.huge

    for _, obj in pairs(workspace:GetDescendants()) do
        if obj.Name == "Chest" and obj:IsA("BasePart") then
            local distance = (obj.Position - RootPart.Position).Magnitude
            if distance < shortestDistance then
                shortestDistance = distance
                closestChest = obj
            end
        end
    end

    return closestChest
end

local function FarmLoop()
    while ChestFarmActive do
        local Character = LocalPlayer.Character
        local Humanoid = Character and Character:FindFirstChild("Humanoid")
        local RootPart = Character and Character:FindFirstChild("HumanoidRootPart")

        if Character and Humanoid and RootPart and Humanoid.Health > 0 then
            local targetChest = GetClosestChest()

            if targetChest then
                -- 1. Teleporte instantâneo para o baú (com um pequeno ajuste de altura para não bugar no chão)
                RootPart.CFrame = targetChest.CFrame * CFrame.new(0, 1, 0)
                
                -- 2. Tenta coletar o baú remotamente (bypass de tempo)
                -- Se o baú for de encostar (Touch):
                pcall(function()
                    if firetouchinterest then
                        firetouchinterest(RootPart, targetChest, 0)
                        firetouchinterest(RootPart, targetChest, 1)
                    end
                end)

                -- Se o baú usar o sistema de segurar tecla (ProximityPrompt):
                pcall(function()
                    local prompt = targetChest:FindFirstChildWhichIsA("ProximityPrompt", true)
                    if prompt and fireproximityprompt then
                        fireproximityprompt(prompt)
                    end
                end)

                -- Aguarda uma fração de segundo para o servidor registrar a coleta e o baú sumir
                task.wait(0.1) 
            else
                task.wait(0.5) -- Se não achar baú, procura de novo em meio segundo
            end
        else
            task.wait(0.5)
        end
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

ToggleButton.MouseButton1Click:Connect(function()
    ChestFarmActive = not ChestFarmActive

    local targetColor = ChestFarmActive and Color3.fromRGB(60, 200, 100) or Color3.fromRGB(230, 60, 60)
    local targetText = ChestFarmActive and "Chest Farm: ON" or "Chest Farm: OFF"
    
    TweenService:Create(ToggleButton, TweenInfo.new(0.3, Enum.EasingStyle.Quad, Enum.EasingDirection.Out), {
        BackgroundColor3 = targetColor
    }):Play()
    ToggleButton.Text = targetText

    if ChestFarmActive then
        task.spawn(FarmLoop)
    end
end)
