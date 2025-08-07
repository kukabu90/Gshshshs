-- Crie uma nova ScreenGui para o painel
local espGui = Instance.new("ScreenGui")
espGui.Name = "ESPGUI"
espGui.Parent = game.Players.LocalPlayer:WaitForChild("PlayerGui")

-- Crie o Frame que vai ser o mini painel
local panelFrame = Instance.new("Frame")
panelFrame.Name = "PanelFrame"
local originalSize = UDim2.new(0, 200, 0, 80)
local minimizedSize = UDim2.new(0, 50, 0, 80)
panelFrame.Size = originalSize
panelFrame.Position = UDim2.new(0.5, -100, 0.5, -40)
panelFrame.BackgroundColor3 = Color3.new(0.2, 0.2, 0.2)
panelFrame.BorderSizePixel = 1
panelFrame.Active = true
panelFrame.Draggable = true
panelFrame.Parent = espGui

-- Crie o botão "Allan" para o ESP
local espButton = Instance.new("TextButton")
espButton.Name = "ESPButton"
espButton.Size = UDim2.new(0, 130, 0, 40)
espButton.Position = UDim2.new(0, 10, 0.5, -20)
espButton.BackgroundColor3 = Color3.new(0.3, 0.3, 0.3)
espButton.Text = "Allan"
espButton.TextColor3 = Color3.new(1, 1, 1)
espButton.Font = Enum.Font.SourceSansBold
espButton.TextSize = 20
espButton.Parent = panelFrame

-- Crie o botão de minimizar
local minimizeButton = Instance.new("TextButton")
minimizeButton.Name = "MinimizeButton"
minimizeButton.Size = UDim2.new(0, 40, 0, 40)
minimizeButton.Position = UDim2.new(1, -50, 0.5, -20)
minimizeButton.BackgroundColor3 = Color3.new(0.3, 0.3, 0.3)
minimizeButton.Text = "–" -- Use um hífen para minimizar
minimizeButton.TextColor3 = Color3.new(1, 1, 1)
minimizeButton.Font = Enum.Font.SourceSansBold
minimizeButton.TextSize = 20
minimizeButton.Parent = panelFrame

-- Variáveis de estado
local espAtivado = false
local isMinimized = false

-- Tabela para armazenar os Highlights criados
local playerHighlights = {}

-- Função para ativar o ESP
local function ativarESP()
    espAtivado = true
    espButton.Text = "Desligar" -- Mudei o texto para manter o nome "Allan" no botão principal
    espButton.BackgroundColor3 = Color3.new(0.7, 0.2, 0.2)

    for _, player in ipairs(game.Players:GetPlayers()) do
        if player ~= game.Players.LocalPlayer then
            local character = player.Character
            if character and character:FindFirstChild("HumanoidRootPart") then
                local highlight = Instance.new("Highlight")
                highlight.Name = "PlayerESP"
                highlight.FillColor = Color3.new(1, 0, 0)
                highlight.OutlineColor = Color3.new(1, 0, 0)
                highlight.DepthMode = Enum.HighlightDepthMode.AlwaysOnTop
                highlight.Adornee = character
                highlight.Parent = character
                playerHighlights[player.UserId] = highlight
            end
        end
    end
end

-- Função para desativar o ESP
local function desativarESP()
    espAtivado = false
    espButton.Text = "Allan"
    espButton.BackgroundColor3 = Color3.new(0.3, 0.3, 0.3)

    for _, highlight in pairs(playerHighlights) do
        if highlight.Parent then
            highlight:Destroy()
        end
    end
    playerHighlights = {}
end

-- Função para minimizar/restaurar o painel
local function toggleMinimize()
    isMinimized = not isMinimized

    if isMinimized then
        panelFrame:TweenSize(minimizedSize, Enum.EasingDirection.Out, Enum.EasingStyle.Quad, 0.3, true)
        espButton.Visible = false
        minimizeButton.Position = UDim2.new(0, 5, 0.5, -20)
        minimizeButton.Text = "+"
    else
        panelFrame:TweenSize(originalSize, Enum.EasingDirection.Out, Enum.EasingStyle.Quad, 0.3, true)
        espButton.Visible = true
        minimizeButton.Position = UDim2.new(1, -50, 0.5, -20)
        minimizeButton.Text = "–"
    end
end

-- Conectar a função ao clique dos botões
espButton.MouseButton1Click:Connect(function()
    if espAtivado then
        desativarESP()
    else
        ativarESP()
    end
end)

minimizeButton.MouseButton1Click:Connect(toggleMinimize)

-- Conectar eventos para adicionar/remover Highlights quando os jogadores entram ou saem
game.Players.PlayerAdded:Connect(function(player)
    if espAtivado and player ~= game.Players.LocalPlayer then
        player.CharacterAdded:Connect(function(character)
            local highlight = Instance.new("Highlight")
            highlight.Name = "PlayerESP"
            highlight.FillColor = Color3.new(1, 0, 0)
            highlight.OutlineColor = Color3.new(1, 0, 0)
            highlight.DepthMode = Enum.HighlightDepthMode.AlwaysOnTop
            highlight.Adornee = character
            highlight.Parent = character
            playerHighlights[player.UserId] = highlight
        end)
    end
end)

game.Players.PlayerRemoving:Connect(function(player)
    if espAtivado then
        if playerHighlights[player.UserId] then
            playerHighlights[player.UserId]:Destroy()
            playerHighlights[player.UserId] = nil
        end
    end
end)
