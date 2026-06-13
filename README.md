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
HintLabel.TextColor3 = Color3.fromRGB(150, 150,
