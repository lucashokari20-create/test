--[[
  Qr HUB com ESP, painel arrastável
  Ao fechar, aparece só Qr HUB onde estava o painel
  Botão Qr HUB abre o painel no mesmo local, ambos com animação de clique/círculo
--]]

local Players = game:GetService("Players")
local LocalPlayer = Players.LocalPlayer
local RunService = game:GetService("RunService")
local TweenService = game:GetService("TweenService")

local espEnabled = false
local panelClosed = false

local screenGui = Instance.new("ScreenGui")
screenGui.Name = "ESP_GUI"
screenGui.Parent = LocalPlayer:WaitForChild("PlayerGui")

-- Configurações do painel
local panelWidth, panelHeight = 400, 230
local cornerRadius = 20

local defaultPanelPosition = nil

local function getCenteredPosition()
    local viewportSize = workspace.CurrentCamera.ViewportSize
    return UDim2.new(0, (viewportSize.X - panelWidth) / 2, 0, (viewportSize.Y - panelHeight) / 2)
end

-- Painel principal
local panel = Instance.new("Frame")
panel.Name = "QrHubPanel"
panel.Size = UDim2.new(0, panelWidth, 0, panelHeight)
panel.Position = getCenteredPosition()
panel.BackgroundColor3 = Color3.fromRGB(35, 35, 70)
panel.Active = true
panel.Parent = screenGui

local corner = Instance.new("UICorner")
corner.CornerRadius = UDim.new(0, cornerRadius)
corner.Parent = panel

-- Título "Qr HUB" centralizado e maior
local title = Instance.new("TextLabel")
title.Name = "QrHubTitle"
title.Text = "Qr HUB"
title.TextColor3 = Color3.fromRGB(80, 170, 255)
title.BackgroundTransparency = 1
title.Font = Enum.Font.SourceSansBold
title.TextSize = 38
title.Size = UDim2.new(1, 0, 0, 60)
title.Position = UDim2.new(0, 0, 0, 0)
title.TextXAlignment = Enum.TextXAlignment.Center
title.Parent = panel

-- Botão X azul maior no topo à direita
local closeButton = Instance.new("TextButton")
closeButton.Name = "CloseButton"
closeButton.Text = "X"
closeButton.Font = Enum.Font.SourceSansBold
closeButton.TextSize = 36
closeButton.Size = UDim2.new(0, 60, 0, 60)
closeButton.Position = UDim2.new(1, -65, 0, 0)
closeButton.BackgroundColor3 = Color3.fromRGB(50, 120, 255)
closeButton.TextColor3 = Color3.new(1,1,1)
closeButton.Parent = panel
local closeCorner = Instance.new("UICorner")
closeCorner.CornerRadius = UDim.new(0, 18)
closeCorner.Parent = closeButton

-- Botão ESP centralizado no meio do painel
local espButton = Instance.new("TextButton")
espButton.Name = "ToggleESP"
espButton.Size = UDim2.new(0, 240, 0, 70)
espButton.Position = UDim2.new(0.5, -120, 0.5, -35)
espButton.Font = Enum.Font.SourceSansBold
espButton.TextSize = 32
espButton.Text = "Ativar ESP"
espButton.TextColor3 = Color3.new(1, 1, 1)
espButton.BackgroundColor3 = Color3.fromRGB(90, 180, 255)
espButton.Parent = panel
local espCorner = Instance.new("UICorner")
espCorner.CornerRadius = UDim.new(0, 22)
espCorner.Parent = espButton

-- Botão Qr HUB (aparece só quando o painel está fechado)
local qrHubButton = Instance.new("TextButton")
qrHubButton.Name = "QrHubButton"
qrHubButton.Text = "Qr HUB"
qrHubButton.Font = Enum.Font.SourceSansBold
qrHubButton.TextSize = 38
qrHubButton.Size = UDim2.new(0, 170, 0, 60)
qrHubButton.BackgroundTransparency = 1
qrHubButton.Position = UDim2.new(0, panel.Position.X.Offset, 0, panel.Position.Y.Offset)
qrHubButton.TextColor3 = Color3.fromRGB(80, 170, 255)
qrHubButton.Visible = false
qrHubButton.Parent = screenGui

-- Table para GUIs dos jogadores
local playerLabels = {}

local function createLabel(player)
    local label = Instance.new("TextLabel")
    label.Name = player.Name .. "_ESPLabel"
    label.BackgroundTransparency = 0.5
    label.BackgroundColor3 = Color3.new(0, 0, 0)
    label.TextColor3 = Color3.new(1, 1, 1)
    label.Font = Enum.Font.SourceSansBold
    label.TextSize = 16
    label.Size = UDim2.new(0, 200, 0, 30)
    label.Visible = false
    label.Parent = screenGui
    playerLabels[player] = label
end

local function updateESP()
    if not espEnabled or panelClosed then
        for _, label in pairs(playerLabels) do
            label.Visible = false
        end
        return
    end

    if not LocalPlayer.Character or not LocalPlayer.Character:FindFirstChild("HumanoidRootPart") then return end
    local myPos = LocalPlayer.Character.HumanoidRootPart.Position

    for _, player in ipairs(Players:GetPlayers()) do
        if player ~= LocalPlayer and player.Character and player.Character:FindFirstChild("HumanoidRootPart") then
            local targetPos = player.Character.HumanoidRootPart.Position
            local dist = (myPos - targetPos).Magnitude

            if not playerLabels[player] then
                createLabel(player)
            end

            local label = playerLabels[player]
            if dist <= 250 then
                label.Visible = true
                label.Text = string.format("%s | Distância: %.1f", player.Name, dist)
                local vector, onScreen = workspace.CurrentCamera:WorldToViewportPoint(targetPos)
                if onScreen then
                    label.Position = UDim2.new(0, vector.X - 100, 0, vector.Y - 15)
                else
                    label.Visible = false
                end
            else
                label.Visible = false
            end
        elseif playerLabels[player] then
            playerLabels[player].Visible = false
        end
    end
end

RunService.RenderStepped:Connect(updateESP)

Players.PlayerRemoving:Connect(function(player)
    if playerLabels[player] then
        playerLabels[player]:Destroy()
        playerLabels[player] = nil
    end
end)

-- Animação de clique para botões (escala + círculo menor)
local function playButtonAnimation(button)
    local originalSize = button.Size
    local shrinkTween = TweenService:Create(button, TweenInfo.new(0.12, Enum.EasingStyle.Quad, Enum.EasingDirection.Out), {Size = UDim2.new(originalSize.X.Scale, originalSize.X.Offset-20, originalSize.Y.Scale, originalSize.Y.Offset-12)})
    local growTween = TweenService:Create(button, TweenInfo.new(0.14, Enum.EasingStyle.Quad, Enum.EasingDirection.Out), {Size = originalSize})

    -- Ripple effect (menor)
    local ripple = Instance.new("Frame")
    ripple.BackgroundTransparency = 0.4
    ripple.BackgroundColor3 = Color3.new(1, 1, 1)
    ripple.Size = UDim2.new(0, 0, 0, 0)
    ripple.AnchorPoint = Vector2.new(0.5, 0.5)
    ripple.Position = UDim2.new(0.5, 0, 0.5, 0)
    ripple.ZIndex = button.ZIndex + 2
    ripple.Parent = button

    local rippleCorner = Instance.new("UICorner")
    rippleCorner.CornerRadius = UDim.new(1,0)
    rippleCorner.Parent = ripple

    local maxRipple = math.min(button.AbsoluteSize.X, button.AbsoluteSize.Y) * 0.38
    local rippleTween = TweenService:Create(ripple, TweenInfo.new(0.22, Enum.EasingStyle.Quad, Enum.EasingDirection.Out), {BackgroundTransparency = 1, Size = UDim2.new(0, maxRipple, 0, maxRipple)})
    
    shrinkTween:Play()
    shrinkTween.Completed:Connect(function()
        growTween:Play()
    end)
    ripple.Size = UDim2.new(0, 0, 0, 0)
    ripple.BackgroundTransparency = 0.35
    rippleTween:Play()
    rippleTween.Completed:Connect(function()
        ripple:Destroy()
    end)
end

espButton.MouseButton1Click:Connect(function()
    playButtonAnimation(espButton)
    espEnabled = not espEnabled
    if espEnabled then
        espButton.Text = "Desativar ESP"
        espButton.BackgroundColor3 = Color3.fromRGB(30, 80, 160) -- azul escuro
    else
        espButton.Text = "Ativar ESP"
        espButton.BackgroundColor3 = Color3.fromRGB(90, 180, 255) -- azul claro
    end
end)

closeButton.MouseButton1Click:Connect(function()
    playButtonAnimation(closeButton)
    panelClosed = true
    espEnabled = false
    for _, label in pairs(playerLabels) do
        label.Visible = false
    end
    espButton.Visible = false
    closeButton.Visible = false
    panel.Visible = false

    -- Salva onde estava o painel
    defaultPanelPosition = panel.Position
    -- Mostra botão Qr HUB exatamente ali
    qrHubButton.Position = UDim2.new(0, panel.Position.X.Offset, 0, panel.Position.Y.Offset)
    qrHubButton.Visible = true
end)

-- Botão Qr HUB (abre o painel de volta no mesmo local)
qrHubButton.MouseButton1Click:Connect(function()
    playButtonAnimation(qrHubButton)
    panelClosed = false
    espButton.Visible = true
    closeButton.Visible = true
    panel.Visible = true

    -- Volta painel para o local salvo
    if defaultPanelPosition then
        panel.Position = defaultPanelPosition
    end
    qrHubButton.Visible = false
end)

-- Painel arrastável
local dragging, dragInput, dragStart, startPos

local function update(input)
    local delta = input.Position - dragStart
    panel.Position = UDim2.new(
        startPos.X.Scale, startPos.X.Offset + delta.X,
        startPos.Y.Scale, startPos.Y.Offset + delta.Y
    )
end

panel.InputBegan:Connect(function(input)
    if panelClosed then return end
    if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then
        dragging = true
        dragStart = input.Position
        startPos = panel.Position

        input.Changed:Connect(function()
            if input.UserInputState == Enum.UserInputState.End then
                dragging = false
            end
        end)
    end
end)

panel.InputChanged:Connect(function(input)
    if panelClosed then return end
    if dragging and (input.UserInputType == Enum.UserInputType.MouseMovement or input.UserInputType == Enum.UserInputType.Touch) then
        update(input)
    end
end)

workspace.CurrentCamera:GetPropertyChangedSignal("ViewportSize"):Connect(function()
    if not dragging and not panelClosed then
        panel.Position = getCenteredPosition()
    elseif panelClosed and defaultPanelPosition then
        qrHubButton.Position = UDim2.new(0, defaultPanelPosition.X.Offset, 0, defaultPanelPosition.Y.Offset)
    end
end)
