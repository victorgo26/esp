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
local corDentroDoCampo = Color3.new(1, 0, 0) -- Vermelho por padrão
local corForaDoCampo = Color3.new(0, 0, 1) -- Azul por padrão
local espAtivo = false
local painelVisivel = false

--// Criar o painel
local espPanel = Instance.new("ScreenGui")
espPanel.Name = "ESPanel"
espPanel.Parent = GuiService:WaitForChild("CoreGui") -- CoreGui para interfaces que persistem

local ativarBotao = Instance.new("TextButton")
ativarBotao.Size = UDim2.new(0.2, 0, 0.1, 0)
ativarBotao.Position = UDim2.new(0.1, 0, 0.1, 0)
ativarBotao.Text = "Ativar ESP"
ativarBotao.Parent = espPanel

--// Frame para as opções de cores
local coresFrame = Instance.new("Frame")
coresFrame.Size = UDim2.new(0.4, 0, 0.2, 0)
coresFrame.Position = UDim2.new(0.1, 0, 0.25, 0)
coresFrame.BackgroundColor3 = Color3.new(0.2, 0.2, 0.2)
coresFrame.Parent = espPanel

local textoCorDentro = Instance.new("TextLabel")
textoCorDentro.Size = UDim2.new(0.45, 0, 0.3, 0)
textoCorDentro.Position = UDim2.new(0.05, 0, 0.1, 0)
textoCorDentro.Text = "Cor Dentro do Campo:"
textoCorDentro.TextColor3 = Color3.new(1, 1, 1)
textoCorDentro.BackgroundTransparency = 1
textoCorDentro.Parent = coresFrame

--// (Aqui você adicionaria elementos interativos para selecionar a cor de 'dentroDoCampo')
--// Por exemplo, botões ou seletores de cor personalizados

local textoCorFora = Instance.new("TextLabel")
textoCorFora.Size = UDim2.new(0.45, 0, 0.3, 0)
textoCorFora.Position = UDim2.new(0.05, 0, 0.6, 0)
textoCorFora.Text = "Cor Fora do Campo:"
textoCorFora.TextColor3 = Color3.new(1, 1, 1)
textoCorFora.BackgroundTransparency = 1
textoCorFora.Parent = coresFrame

--// (Aqui você adicionaria elementos interativos para selecionar a cor de 'foraDoCampo')
--// Por exemplo, botões ou seletores de cor personalizados

espPanel.Visible = false -- Inicialmente invisível

--// Função para verificar se um vetor está dentro do campo de visão
local function estaNoCampoDeVisao(pontoMundo)
    local direcaoParaPonto = (pontoMundo - Camera.CFrame.Position).Unit
    local direcaoDaCamera = Camera.CFrame.LookVector
    local angulo = acos(direcaoParaPonto:Dot(direcaoDaCamera))
    --// Ajuste este valor para controlar a largura do campo de visão
    local limiteAngulo = math.rad(60)
    return angulo < limiteAngulo
end

--// Função para mudar a cor de um personagem
local function mudarCorPersonagem(personagem, dentroDoCampo)
    if personagem and personagem:FindFirstChild("HumanoidRootPart") then
        for _, parte in ipairs(personagem:GetDescendants()) do
            if parte:IsA("BasePart") and parte.Name ~= "HumanoidRootPart" then
                parte.Color = dentroDoCampo and corDentroDoCampo or corForaDoCampo
            end
        end
    end
end

--// Loop para atualizar as cores dos personagens
game:GetService("RunService").RenderStepped:Connect(function()
    if espAtivo then
        for _, jogador in ipairs(Players:GetPlayers()) do
            if jogador ~= LocalPlayer and jogador.Character then
                local personagem = jogador.Character
                local pontoDeReferencia = personagem:FindFirstChild("HumanoidRootPart") and personagem.HumanoidRootPart.Position
                if pontoDeReferencia then
                    local visivel = estaNoCampoDeVisao(pontoDeReferencia)
                    mudarCorPersonagem(personagem, visivel)
                else
                    mudarCorPersonagem(personagem, false) -- Se não houver HumanoidRootPart, considera fora
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
            if jogador ~= LocalPlayer and jogador.Character then
                --// Aqui você precisaria restaurar a cor original do personagem
                --// Isso é mais complexo e exigiria armazenar a cor original
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
