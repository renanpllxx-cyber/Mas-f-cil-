local button = script.Parent
local player = game.Players.LocalPlayer
local camera = workspace.CurrentCamera
local runService = game:GetService("RunService")
local replicatedStorage = game:GetService("ReplicatedStorage")

-- Cria o evento de rede se não existir
local toggleEvent = replicatedStorage:FindFirstChild("ToggleRadarEvent")
if not toggleEvent then
    toggleEvent = Instance.new("RemoteEvent")
    toggleEvent.Name = "ToggleRadarEvent"
    toggleEvent.Parent = replicatedStorage
end

local ativo = false
local conexaoCamera = nil

-- Função para encontrar o inimigo mais próximo
local function encontrarAlvoMaisProximo()
    local menorDistancia = math.huge
    local alvoEscolhido = nil
    
    for _, outroJogador in ipairs(game.Players:GetPlayers()) do
        if outroJogador ~= player and outroJogador.Character and outroJogador.Character:FindFirstChild("HumanoidRootPart") then
            local char = outroJogador.Character
            local humanoid = char:FindFirstChildOfClass("Humanoid")
            
            if humanoid and humanoid.Health > 0 then
                local raiz = char.HumanoidRootPart
                local distancia = (raiz.Position - camera.CFrame.Position).Magnitude
                
                if distancia < menorDistancia then
                    menorDistancia = distancia
                    alvoEscolhido = raiz
                end
            end
        end
    end
    
    return alvoEscolhido
end

-- Botão de ativação
button.MouseButton1Click:Connect(function()
    ativo = not ativo
    
    if ativo then
        button.Text = "Habilidade: ATIVA"
        button.BackgroundColor3 = Color3.fromRGB(0, 255, 0)
        
        conexaoCamera = runService.RenderStepped:Connect(function()
            local alvo = encontrarAlvoMaisProximo()
            if alvo then
                local direcaoAlvo = CFrame.new(camera.CFrame.Position, alvo.Position)
                
                -- Mudamos o número 0.1 para 0.5 (quanto mais perto de 1, mais rápido ele gruda. Se colocar 1, ele gruda instantaneamente)
                camera.CFrame = camera.CFrame:Lerp(direcaoAlvo, 0.5)
            end
        end)
    else
        button.Text = "Habilidade: DESLIGADA"
        button.BackgroundColor3 = Color3.fromRGB(255, 0, 0)
        
        if conexaoCamera then
            conexaoCamera:Disconnect()
            conexaoCamera = nil
        end
    end
    
    toggleEvent:FireServer(ativo)
end)
