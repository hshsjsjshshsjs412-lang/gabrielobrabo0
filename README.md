-- Serviços principais
local Players = game:GetService("Players")
local RunService = game:GetService("RunService")

-- CONFIGURAÇÃO DA IMAGEM
-- Substitua os números abaixo pelo ID que você copiou do site
local MINHA_FOTO_ID = "rbxassetid:137516505042910" 

local LocalPlayer = Players.LocalPlayer
local ESP_ATIVADO = false

-- 1. CRIANDO A INTERFACE COM SUA FOTO
local screenGui = Instance.new("ScreenGui")
screenGui.Name = "PainelTestePrivado"
screenGui.Parent = LocalPlayer:WaitForChild("PlayerGui")

local botaoFoto = Instance.new("ImageButton")
botaoFoto.Name = "IconeToggle"
botaoFoto.Size = UDim2.new(0, 70, 0, 70) -- Tamanho do ícone
botaoFoto.Position = UDim2.new(0, 10, 0.4, 0) -- Posição na lateral esquerda
botaoFoto.Image = MINHA_FOTO_ID
botaoFoto.BackgroundColor3 = Color3.fromRGB(255, 255, 255)
botaoFoto.BorderSizePixel = 3
botaoFoto.BorderColor3 = Color3.fromRGB(255, 0, 0) -- Começa vermelho (desativado)
botaoFoto.Parent = screenGui

-- Arredondar a foto para ficar um ícone bonito
local uiCorner = Instance.new("UICorner")
uiCorner.CornerRadius = UDim.new(1, 0) -- Deixa o ícone circular
uiCorner.Parent = botaoFoto

-- 2. FUNÇÃO DE ESP (WALLHACK E DISTÂNCIA)
local function criarVisual(targetPlayer)
    if targetPlayer == LocalPlayer then return end

    local function atualizarPersonagem(char)
        -- Highlight (Linhas ao redor do corpo)
        local highlight = Instance.new("Highlight")
        highlight.Name = "LinhaESP"
        highlight.FillTransparency = 0.5
        highlight.OutlineColor = Color3.fromRGB(255, 255, 255)
        highlight.Parent = char
        highlight.Enabled = ESP_ATIVADO

        -- Billboard (Texto da distância)
        local billboard = Instance.new("BillboardGui")
        billboard.Name = "InfoESP"
        billboard.Size = UDim2.new(0, 100, 0, 50)
        billboard.AlwaysOnTop = true
        billboard.StudsOffset = Vector3.new(0, 3, 0)
        billboard.Parent = char:WaitForChild("Head")
        billboard.Enabled = ESP_ATIVADO

        local texto = Instance.new("TextLabel")
        texto.Size = UDim2.new(1, 0, 1, 0)
        texto.BackgroundTransparency = 1
        texto.TextColor3 = Color3.fromRGB(255, 255, 255)
        texto.TextStrokeTransparency = 0
        texto.Font = Enum.Font.SourceSansBold
        texto.TextSize = 16
        texto.Parent = billboard

        -- Loop de atualização de distância
        RunService.RenderStepped:Connect(function()
            if char:FindFirstChild("HumanoidRootPart") and LocalPlayer.Character:FindFirstChild("HumanoidRootPart") then
                local distStuds = (char.HumanoidRootPart.Position - LocalPlayer.Character.HumanoidRootPart.Position).Magnitude
                local distMetros = math.floor(distStuds / 20) -- Conversão aproximada para metros
                texto.Text = targetPlayer.Name .. "\n" .. distMetros .. "m"
            end
            
            -- Sincroniza com o botão
            highlight.Enabled = ESP_ATIVADO
            billboard.Enabled = ESP_ATIVADO
        end)
    end

    targetPlayer.CharacterAdded:Connect(atualizarPersonagem)
    if targetPlayer.Character then atualizarPersonagem(targetPlayer.Character) end
end

-- 3. LOGICA DO CLIQUE NA FOTO
botaoFoto.MouseButton1Click:Connect(function()
    ESP_ATIVADO = not ESP_ATIVADO
    
    if ESP_ATIVADO then
        botaoFoto.BorderColor3 = Color3.fromRGB(0, 255, 0) -- Verde se ligado
        print("Ambiente de Teste: ESP Ativado")
    else
        botaoFoto.BorderColor3 = Color3.fromRGB(255, 0, 0) -- Vermelho se desligado
        print("Ambiente de Teste: ESP Desativado")
    end
end)

-- Inicializar para todos no servidor privado
for _, p in pairs(Players:GetPlayers()) do
    criarVisual(p)
end
Players.PlayerAdded:Connect(criarVisual)
