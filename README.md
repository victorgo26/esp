-- ESTE É UM EXEMPLO CONCEITUAL E NÃO FUNCIONARÁ DIRETAMENTE NO ROBLOX SEM UM EXPLOIT

local Players = game:GetService("Players")
local LocalPlayer = Players.LocalPlayer
local UserInputService = game:GetService("UserInputService")
local Workspace = game:GetService("Workspace")
local Camera = Workspace.CurrentCamera

local espGui = Instance.new("ScreenGui")
espGui.Name = "DetailedESP"
espGui.Parent = LocalPlayer.PlayerGui

local playerBoxes = {} -- Tabela para armazenar as caixas de cada jogador
local espEnabled = false
local configPanelVisible = false
local configPanel = nil
local visibleColor = Color3.new(0, 1, 0) -- Cor padrão para visível
local hiddenColor = Color3.new(1, 0, 0) -- Cor padrão para não visível

local function worldToViewport(worldPos)
    return Camera:WorldToViewportPoint(worldPos)
end

local function createBoundingBox()
    local frame = Instance.new("Frame")
    frame.BackgroundColor3 = Color3.new(1, 1, 1)
    frame.BackgroundTransparency = 0.8
    frame.BorderSizePixel = 1
    frame.BorderColor3 = Color3.new(0, 0, 0)
    frame.Size = UDim2.new(0, 50, 0, 20) -- Tamanho inicial
    frame.AnchorPoint = Vector2.new(0.5, 0.5)
    frame.Visible = false
    frame.Parent = espGui
    return frame
end

local function updateBoxColor(box, isVisible)
    if box then
        box.BackgroundColor3 = isVisible and visibleColor or hiddenColor
    end
end

local function updateESPVisibility()
    for _, playerTable in pairs(playerBoxes) do
        for _, box in pairs(playerTable) do
            box.Visible = espEnabled
        end
    end
end

local function createConfigPanel()
    local panel = Instance.new("Frame")
    panel.Name = "ESPConfigPanel"
    panel.Size = UDim2.new(0, 200, 0, 150)
    panel.Position = UDim2.new(0.5, -100, 0.5, -75)
    panel.AnchorPoint = Vector2.new(0.5, 0.5)
    panel.BackgroundColor3 = Color3.new(0.2, 0.2, 0.2)
    panel.BorderSizePixel = 2
    panel.BorderColor3 = Color3.new(0, 0, 0)
    panel.Visible = false
    panel.Parent = espGui

    -- Título
    local title = Instance.new("TextLabel")
    title.Size = UDim2.new(1, 0, 0, 30)
    title.Position = UDim2.new(0, 0, 0, 0)
    title.BackgroundColor3 = Color3.new(0.1, 0.1, 0.1)
    title.TextColor3 = Color3.new(1, 1, 1)
    title.Text = "Configurações do ESP"
    title.Font = Enum.Font.SourceSansBold
    title.TextSize = 16
    title.AnchorPoint = Vector2.new(0, 0)
    title.Parent = panel

    -- Botão de Ativar/Desativar
    local toggleButton = Instance.new("TextButton")
    toggleButton.Size = UDim2.new(0.8, 0, 0, 30)
    toggleButton.Position = UDim2.new(0.1, 0, 0.2, 0)
    toggleButton.BackgroundColor3 = espEnabled and Color3.new(0, 0.5, 0) or Color3.new(0.5, 0, 0)
    toggleButton.TextColor3 = Color3.new(1, 1, 1)
    toggleButton.Text = espEnabled and "Desativar ESP" or "Ativar ESP"
    toggleButton.Font = Enum.Font.SourceSans
    toggleButton.TextSize = 14
    toggleButton.AnchorPoint = Vector2.new(0.5, 0)
    toggleButton.Parent = panel
    toggleButton.MouseButton1Click:Connect(function()
        espEnabled = not espEnabled
        toggleButton.Text = espEnabled and "Desativar ESP" or "Ativar ESP"
        toggleButton.BackgroundColor3 = espEnabled and Color3.new(0, 0.5, 0) or Color3.new(0.5, 0, 0)
        updateESPVisibility()
    end)

    -- Seção de Cor Visível
    local visibleColorLabel = Instance.new("TextLabel")
    visibleColorLabel.Size = UDim2.new(0.4, 0, 0, 20)
    visibleColorLabel.Position = UDim2.new(0.1, 0, 0.5, 0)
    visibleColorLabel.BackgroundColor3 = Color3.new(0.3, 0.3, 0.3)
    visibleColorLabel.TextColor3 = Color3.new(1, 1, 1)
    visibleColorLabel.Text = "Cor Visível:"
    visibleColorLabel.Font = Enum.Font.SourceSans
    visibleColorLabel.TextSize = 12
    visibleColorLabel.AnchorPoint = Vector2.new(0, 0)
    visibleColorLabel.Parent = panel

    local visibleColorPicker = Instance.new("ColorPicker")
    visibleColorPicker.Size = UDim2.new(0.4, 0, 0, 20)
    visibleColorPicker.Position = UDim2.new(0.5, 0, 0.5, 0)
    visibleColorPicker.Color = visibleColor
    visibleColorPicker.AnchorPoint = Vector2.new(0, 0)
    visibleColorPicker.Parent = panel
    visibleColorPicker.ColorChanged:Connect(function(newColor)
        visibleColor = newColor
        for player, playerTable in pairs(playerBoxes) do
            if player.Character then
                for partName, box in pairs(playerTable) do
                    local part = player.Character:FindFirstChild(partName)
                    if part then
                        local raycastParams = RaycastParams.new()
                        raycastParams.FilterDescendantsInstances = {LocalPlayer.Character}
                        raycastParams.FilterType = Enum.RaycastFilterType.Blacklist
                        local raycastResult = Workspace:Raycast(Camera.CFrame.Position, (part.Position - Camera.CFrame.Position).Unit * 1000, raycastParams)
                        updateBoxColor(box, raycastResult and raycastResult.Instance == part)
                    end
                end
            end
        end
    end)

    -- Seção de Cor Não Visível
    local hiddenColorLabel = Instance.new("TextLabel")
    hiddenColorLabel.Size = UDim2.new(0.4, 0, 0, 20)
    hiddenColorLabel.Position = UDim2.new(0.1, 0, 0.7, 0)
    hiddenColorLabel.BackgroundColor3 = Color3.new(0.3, 0.3, 0.3)
    hiddenColorLabel.TextColor3 = Color3.new(1, 1, 1)
    hiddenColorLabel.Text = "Cor Não Visível:"
    hiddenColorLabel.Font = Enum.Font.SourceSans
    hiddenColorLabel.TextSize = 12
    hiddenColorLabel.AnchorPoint = Vector2.new(0, 0)
    hiddenColorLabel.Parent = panel

    local hiddenColorPicker = Instance.new("ColorPicker")
    hiddenColorPicker.Size = UDim2.new(0.4, 0, 0, 20)
    hiddenColorPicker.Position = UDim2.new(0.5, 0, 0.7, 0)
    hiddenColorPicker.Color = hiddenColor
    hiddenColorPicker.AnchorPoint = Vector2.new(0, 0)
    hiddenColorPicker.Parent = panel
    hiddenColorPicker.ColorChanged:Connect(function(newColor)
        hiddenColor = newColor
        for player, playerTable in pairs(playerBoxes) do
            if player.Character then
                for partName, box in pairs(playerTable) do
                    local part = player.Character:FindFirstChild(partName)
                    if part then
                        local raycastParams = RaycastParams.new()
                        raycastParams.FilterDescendantsInstances = {LocalPlayer.Character}
                        raycastParams.FilterType = Enum.RaycastFilterType.Blacklist
                        local raycastResult = Workspace:Raycast(Camera.CFrame.Position, (part.Position - Camera.CFrame.Position).Unit * 1000, raycastParams)
                        updateBoxColor(box, raycastResult and raycastResult.Instance == part)
                    end
                end
            end
        end
    end)

    return panel
end

UserInputService.InputBegan:Connect(function(input, gameProcessedEvent)
    if not gameProcessedEvent and input.KeyCode == Enum.KeyCode.F4 then
        if not configPanel then
            configPanel = createConfigPanel()
        end
        configPanelVisible = not configPanelVisible
        configPanel.Visible = configPanelVisible
    end
end)

game:GetService("RunService").RenderStepped:Connect(function()
    if not espEnabled then return end

    for _, player in ipairs(Players:GetPlayers()) do
        if player ~= LocalPlayer then
            local character = player.Character
            if character then
                if not playerBoxes[player] then
                    playerBoxes[player] = {}
                end

                local playerBoxTable = playerBoxes[player]
                local bodyPartsToTrack = {"Head", "Torso", "Left Arm", "Right Arm", "Left Leg", "Right Leg"}

                for _, partName in ipairs(bodyPartsToTrack) do
                    local part = character:FindFirstChild(partName)
                    if part then
                        local viewportPoint, onScreen = worldToViewport(part.Position)

                        if onScreen then
                            if not playerBoxTable[partName] then
                                playerBoxTable[partName] = createBoundingBox()
                            end

                            local box = playerBoxTable[partName]

                            local size = Camera:WorldToViewportPoint(part.CFrame:ToWorldSpace(part.Size)):ToObjectSpace(viewportPoint)
                            box.Size = UDim2.new(0, math.abs(size.X), 0, math.abs(size.Y))
                            box.Position = UDim2.new(0, viewportPoint.X, 0, viewportPoint.Y)
                            box.Visible = espEnabled

                            -- Verificar visibilidade com Raycast (simplificado)
                            local raycastParams = RaycastParams.new()
                            raycastParams.FilterDescendantsInstances = {LocalPlayer.Character}
                            raycastParams.FilterType = Enum.RaycastFilterType.Blacklist

                            local raycastResult = Workspace:Raycast(Camera.CFrame.Position, (part.Position - Camera.CFrame.Position).Unit * 1000, raycastParams)
                            updateBoxColor(box, raycastResult and raycastResult.Instance == part)
                        else
                            if playerBoxTable[partName] then
                                playerBoxTable[partName].Visible = false
                            end
                        end
                    else
                        -- Remover a caixa se a parte não existir mais
                        if playerBoxTable[partName] then
                            playerBoxTable[partName]:Destroy()
                            playerBoxTable[partName] = nil
                        end
                    end
                end
            else
                -- Remover todas as caixas se o personagem não existir
                if playerBoxes[player] then
                    for _, box in pairs(playerBoxes[player]) do
                        box:Destroy()
                    end
                    playerBoxes[player] = nil
                end
            end
        else
            -- Remover as caixas do jogador local
            if playerBoxes[player] then
                for _, box in pairs(playerBoxes[player]) do
                    box:Destroy()
                end
                playerBoxes[player] = nil
            end
        end
    end
end)

Players.PlayerRemoving:Connect(function(player)
    if playerBoxes[player] then
        for _, box in pairs(playerBoxes[player]) do
            box:Destroy()
        end
        playerBoxes[player] = nil
    end
end)
