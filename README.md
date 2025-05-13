--// Serviço de Input para detectar teclas
local UserInputService = game:GetService("UserInputService")
--// Serviço de GUI para criar a interface
local GuiService = game:GetService("GuiService")
--// Serviço de Players para acessar outros jogadores
local Players = game:GetService("Players")
--// O jogador local
local LocalPlayer = Players.LocalPlayer
--// A câmera do jogador
local Camera = workspace.CurrentCamera

--// Variáveis para as cores customizáveis
local corForaDoCampo = Color3.new(1, 0, 0) -- Vermelho
local corDentroDoCampo = Color3.new(1, 1, 0) -- Amarelo
local espAtivo = false
local painelVisivel = false

--// Criar o painel
local espPanel = Instance.new("ScreenGui")
espPanel.Name = "ESPanel"
espPanel.Parent = GuiService:WaitForChild("CoreGui")

local ativarBotao = Instance.new("TextButton")
ativarBotao.Size = UDim2.new(0.2, 0, 0.1, 0)
ativarBotao.Position = UDim2.new(0.1, 0, 0.1, 0)
ativarBotao.Text = "Ativar ESP"
ativarBotao.Parent = espPanel

--// Frame para as opções de cores (removido por enquanto para simplificar)
--// Se você quiser reativar a customização, precisará readicionar essa parte

espPanel.Visible = false -- Inicialmente invisível

--// Função para verificar se um vetor está dentro do campo de visão
local function estaNoCampoDeVisao(pontoMundo)
    local direcaoParaPonto = (pontoMundo - Camera.CFrame.Position).Unit
    local direcaoDaCamera = Camera.CFrame.LookVector
    local angulo = acos(direcaoParaPonto:Dot(direcaoDaCamera))
    local limiteAngulo = math.rad(60)
    return angulo < limiteAngulo
end

--// Função para mudar a cor de uma parte do corpo
local function mudarCorParteCorpo(parte, dentroDoCampo)
    if parte:IsA("BasePart") and parte.Name ~= "HumanoidRootPart" then
        parte.Color = dentroDoCampo and corDentroDoCampo or corForaDoCampo
    end
end

--// Loop para atualizar as cores das partes do corpo dos personagens
game:GetService("RunService").RenderStepped:Connect(function()
    if espAtivo then
        for _, jogador in ipairs(Players:GetPlayers()) do
            if jogador ~= LocalPlayer and jogador.Character and jogador.Character:FindFirstChild("Humanoid") then
                local personagem = jogador.Character
                local humanoid = personagem:FindFirstChild("Humanoid")
                local pontoDeReferencia = personagem:FindFirstChild("HumanoidRootPart") and personagem.HumanoidRootPart.Position
                if pontoDeReferencia then
                    local visivel = estaNoCampoDeVisao(pontoDeReferencia)
                    for _, parte in ipairs(personagem:GetChildren()) do
                        --// Verifica se a parte é uma parte do corpo padrão
                        if humanoid:GetBodyPartRbxType(parte) ~= Enum.BodyPartRbxType.Other then
                            mudarCorParteCorpo(parte, visivel)
                        end
                    end
                else
                    --// Se não houver HumanoidRootPart, colore todas as partes do corpo de vermelho
                    for _, parte in ipairs(personagem:GetChildren()) do
                        if humanoid:GetBodyPartRbxType(parte) ~= Enum.BodyPartRbxType.Other then
                            mudarCorParteCorpo(parte, false)
                        end
                    end
                end
            end
        end
    end
end)

--// Evento para o botão de ativar/desativar
ativarBotao.MouseButton1Click:Connect(function()
    espAtivo = not espAtivo
    ativarBotao.Text = espAtivo and "Desativar ESP" or "Ativar ESP"
    --// Resetar as cores quando desativa
    if not espAtivo then
        for _, jogador in ipairs(Players:GetPlayers()) do
            if jogador ~= LocalPlayer and jogador.Character and jogador.Character:FindFirstChild("Humanoid") then
                local humanoid = jogador.Character:FindFirstChild("Humanoid")
                for _, parte in ipairs(jogador.Character:GetChildren()) do
                    if humanoid:GetBodyPartRbxType(parte) ~= Enum.BodyPartRbxType.Other then
                        --// Aqui você precisaria restaurar a cor original da parte
                        --// Isso é mais complexo e exigiria armazenar a cor original
                    end
                end
            end
        end
    end
end)

--// Detectar a tecla F4 para mostrar/esconder o painel
UserInputService.InputBegan:Connect(function(input, gameProcessedEvent)
    if not gameProcessedEvent and input.KeyCode == Enum.KeyCode.F4 then
        painelVisivel = not painelVisivel
        espPanel.Visible = painelVisivel
    end
end)
