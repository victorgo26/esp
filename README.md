--// Serviços
local UserInputService = game:GetService("UserInputService")
local GuiService = game:GetService("GuiService")
local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local Workspace = game:GetService("Workspace")

--// Jogador local e câmera
local LocalPlayer = Players.LocalPlayer
local Camera = Workspace.CurrentCamera
local Mouse = LocalPlayer:GetMouse()

--// Variáveis de controle
local corVisivel = Color3.new(1, 1, 0) -- Amarelo padrão para visível
local corOculto = Color3.new(1, 0, 0)  -- Vermelho padrão para oculto
local espAtivo = false
local painelVisivel = false
local aimbotAtivo = false
local alvoAimbot = "Cabeça" -- Padrão para mirar na cabeça

--// Painel
local espPanel = Instance.new("ScreenGui")
espPanel.Name = "ESPanel"
espPanel.Parent = GuiService:WaitForChild("CoreGui")
espPanel.DisplayOrder = 10 -- Para garantir que fique na frente

local ativarESPBotao = Instance.new("TextButton")
ativarESPBotao.Size = UDim2.new(0.2, 0, 0.1, 0)
ativarESPBotao.Position = UDim2.new(0.1, 0, 0.1, 0)
ativarESPBotao.Text = "Ativar Contorno: OFF"
ativarESPBotao.Parent = espPanel

local corVisivelBotao = Instance.new("TextButton")
corVisivelBotao.Size = UDim2.new(0.2, 0, 0.1, 0)
corVisivelBotao.Position = UDim2.new(0.1, 0, 0.25, 0)
corVisivelBotao.Text = "Cor Visível"
corVisivelBotao.BackgroundColor3 = corVisivel
corVisivelBotao.Parent = espPanel

local corOcultoBotao = Instance.new("TextButton")
corOcultoBotao.Size = UDim2.new(0.2, 0, 0.1, 0)
corOcultoBotao.Position = UDim2.new(0.1, 0, 0.4, 0)
corOcultoBotao.Text = "Cor Oculto"
corOcultoBotao.BackgroundColor3 = corOculto
corOcultoBotao.Parent = espPanel

local ativarAimbotBotao = Instance.new("TextButton")
ativarAimbotBotao.Size = UDim2.new(0.2, 0, 0.1, 0)
ativarAimbotBotao.Position = UDim2.new(0.4, 0, 0.1, 0)
ativarAimbotBotao.Text = "Ativar Aimbot: OFF"
ativarAimbotBotao.Parent = espPanel

local selecionarAlvo = Instance.new("TextButton")
selecionarAlvo.Size = UDim2.new(0.2, 0, 0.1, 0)
selecionarAlvo.Position = UDim2.new(0.4, 0, 0.25, 0)
selecionarAlvo.Text = "Alvo: Cabeça"
selecionarAlvo.Parent = espPanel

espPanel.Visible = false

--// Funções
local function estaNoCampoDeVisao(pontoMundo)
    local direcaoParaPonto = (pontoMundo - Camera.CFrame.Position).Unit
    local direcaoDaCamera = Camera.CFrame.LookVector
    local angulo = acos(direcaoParaPonto:Dot(direcaoDaCamera))
    local limiteAngulo = math.rad(60)
    return angulo < limiteAngulo
end

local function estaOcluso(pontoOrigem, pontoAlvo, ignorar)
    local direcao = (pontoAlvo - pontoOrigem).Unit
    local distancia = (pontoAlvo - pontoOrigem).Magnitude
    local raycastParams = RaycastParams.new()
    raycastParams.FilterDescendantsInstances = {LocalPlayer.Character, ignorar}
    raycastParams.FilterType = Enum.RaycastFilterType.Exclude
    local raycastResult = Workspace:Raycast(pontoOrigem, direcao * distancia, raycastParams)
    return raycastResult ~= nil
end

local function aplicarCorOclusao(personagem, ocluso)
    if personagem and personagem:FindFirstChild("Humanoid") then
        local corAplicar = ocluso and corOculto or corVisivel
        for _, parte in ipairs(personagem:GetChildren()) do
            if parte:IsA("BasePart") and parte.Name ~= "HumanoidRootPart" and personagem:FindFirstChild("Humanoid"):GetBodyPartRbxType(parte) ~= Enum.BodyPartRbxType.Other then
                parte.Color = corAplicar
            end
        end
    end
end

--// Aimbot Function
local function aimAtTarget(targetPart)
    if targetPart and aimbotAtivo then
        local targetPosition = targetPart.WorldPosition
        local currentCameraPosition = Camera.CFrame.Position
        local lookVector = (targetPosition - currentCameraPosition).Unit
        Camera.CFrame = CFrame.lookAt(currentCameraPosition, targetPosition)
    end
end

--// Loop de renderização
RunService.RenderStepped:Connect(function()
    if espAtivo or aimbotAtivo then
        local closestTarget = nil
        local shortestDistance = math.huge

        for _, jogador in ipairs(Players:GetPlayers()) do
            if jogador ~= LocalPlayer and jogador.Character and jogador.Character:FindFirstChild("Humanoid") then
                local personagem = jogador.Character
                local pontoDeReferencia = personagem:FindFirstChild("HumanoidRootPart") and personagem.HumanoidRootPart.Position

                if pontoDeReferencia and estaNoCampoDeVisao(pontoDeReferencia) then
                    --// Contorno
                    local ocluso = estaOcluso(Camera.CFrame.Position, pontoDeReferencia, personagem)
                    aplicarCorOclusao(personagem, ocluso)

                    --// Aimbot Targetting
                    if aimbotAtivo then
                        local targetPart = personagem:FindFirstChild(alvoAimbot == "Cabeça" and "Head" or "HumanoidRootPart")
                        if targetPart then
                            local distance = (Camera.CFrame.Position - targetPart.WorldPosition).Magnitude
                            if distance < shortestDistance then
                                closestTarget = targetPart
                                shortestDistance = distance
                            end
                        end
                    end
                else
                    aplicarCorOclusao(personagem, true) -- Fora do campo de visão é considerado ocluso
                end
            end
        end

        --// Aimbot aiming
        if aimbotAtivo and closestTarget then
            aimAtTarget(closestTarget)
        end
    end
end)

--// Eventos dos botões
ativarESPBotao.MouseButton1Click:Connect(function()
    espAtivo = not espAtivo
    ativarESPBotao.Text = "Ativar Contorno: " .. (espAtivo and "ON" or "OFF")
    --// Reset de cores ao desativar (simplificado)
    if not espAtivo then
        for _, jogador in ipairs(Players:GetPlayers()) do
            if jogador ~= LocalPlayer and jogador.Character and jogador.Character:FindFirstChild("Humanoid") then
                for _, parte in ipairs(jogador.Character:GetChildren()) do
                    if parte:IsA("BasePart") and personagem:FindFirstChild("Humanoid"):GetBodyPartRbxType(parte) ~= Enum.BodyPartRbxType.Other then
                        parte.Color = parte.BrickColor.Color
                    end
                end
            end
        end
    end
end)

corVisivelBotao.MouseButton1Click:Connect(function()
    --// Aqui você implementaria uma lógica mais avançada para selecionar cores
    --// Por exemplo, abrir um seletor de cores ou usar um input de texto
    --// Para simplificar, vamos alternar entre algumas cores predefinidas
    if corVisivel == Color3.new(1, 1, 0) then
        corVisivel = Color3.new(0, 1, 0) -- Verde
    else
        corVisivel = Color3.new(1, 1, 0) -- Amarelo
    end
    corVisivelBotao.BackgroundColor3 = corVisivel
end)

corOcultoBotao.MouseButton1Click:Connect(function()
    --// Similar ao botão de cor visível, implemente uma seleção de cores mais robusta
    if corOculto == Color3.new(1, 0, 0) then
        corOculto = Color3.new(0, 0, 1) -- Azul
    else
        corOculto = Color3.new(1, 0, 0) -- Vermelho
    end
    corOcultoBotao.BackgroundColor3 = corOculto
end)

ativarAimbotBotao.MouseButton1Click:Connect(function()
    aimbotAtivo = not aimbotAtivo
    ativarAimbotBotao.Text = "Ativar Aimbot: " .. (aimbotAtivo and "ON" or "OFF")
end)

selecionarAlvo.MouseButton1Click:Connect(function()
    if alvoAimbot == "Cabeça" then
        alvoAimbot = "HumanoidRootPart"
        selecionarAlvo.Text = "Alvo: Corpo"
    else
        alvoAimbot = "Cabeça"
        selecionarAlvo.Text = "Alvo: Cabeça"
    end
end)

--// Tecla F4 para o painel
UserInputService.InputBegan:Connect(function(input, gameProcessedEvent)
    if not gameProcessedEvent and input.KeyCode == Enum.KeyCode.F4 then
        painelVisivel = not painelVisivel
        espPanel.Visible = painelVisivel
    end
end)
