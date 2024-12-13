-- Script básico de voo com menu e controle por nick
local player = game.Players.LocalPlayer
local character = player.Character or player.CharacterAdded:Wait()
local humanoid = character:WaitForChild("Humanoid")
local rootPart = character:WaitForChild("HumanoidRootPart")

local flyingPlayers = {} -- Lista de jogadores que podem voar
local speed = 50

-- Criar a interface gráfica
local screenGui = Instance.new("ScreenGui", player:WaitForChild("PlayerGui"))

-- Função para criar botões no menu
local function createButton(parent, text, position, callback)
    local button = Instance.new("TextButton", parent)
    button.Size = UDim2.new(0, 200, 0, 50)
    button.Position = position
    button.Text = text
    button.BackgroundColor3 = Color3.fromRGB(0, 170, 255)
    button.TextScaled = true
    button.TextColor3 = Color3.new(1, 1, 1)
    button.Font = Enum.Font.SourceSansBold
    button.MouseButton1Click:Connect(callback)
    return button
end

-- Função para criar caixas de texto
local function createTextBox(parent, placeholderText, position, callback)
    local textBox = Instance.new("TextBox", parent)
    textBox.Size = UDim2.new(0, 200, 0, 50)
    textBox.Position = position
    textBox.PlaceholderText = placeholderText
    textBox.Text = ""
    textBox.BackgroundColor3 = Color3.fromRGB(255, 255, 255)
    textBox.TextScaled = true
    textBox.TextColor3 = Color3.new(0, 0, 0)
    textBox.Font = Enum.Font.SourceSans
    textBox.FocusLost:Connect(function(enterPressed)
        if enterPressed and callback then
            callback(textBox.Text)
        end
    end)
    return textBox
end

-- Botão para adicionar um jogador na lista de voo
local selectedPlayer = ""
createTextBox(screenGui, "Digite o nick do jogador", UDim2.new(0.5, -100, 0.4, 0), function(input)
    selectedPlayer = input
end)

createButton(screenGui, "Permitir Voo", UDim2.new(0.5, -100, 0.5, 0), function()
    if selectedPlayer ~= "" and not flyingPlayers[selectedPlayer] then
        flyingPlayers[selectedPlayer] = true
        print(selectedPlayer .. " agora pode voar.")
    end
end)

-- Função de voo
local function toggleFlight(targetPlayer)
    local targetCharacter = game.Players:FindFirstChild(targetPlayer) and game.Players[targetPlayer].Character
    if not targetCharacter then return end

    local rootPart = targetCharacter:FindFirstChild("HumanoidRootPart")
    if not rootPart then return end

    if flyingPlayers[targetPlayer] then
        -- Ativar voo
        local bodyVelocity = Instance.new("BodyVelocity", rootPart)
        bodyVelocity.Name = "FlightVelocity"
        bodyVelocity.MaxForce = Vector3.new(1e4, 1e4, 1e4)
        bodyVelocity.Velocity = Vector3.zero

        local bodyGyro = Instance.new("BodyGyro", rootPart)
        bodyGyro.Name = "FlightGyro"
        bodyGyro.MaxTorque = Vector3.new(1e4, 1e4, 1e4)
        bodyGyro.CFrame = rootPart.CFrame

        print(targetPlayer .. " está voando.")
    else
        -- Desativar voo
        local bodyVelocity = rootPart:FindFirstChild("FlightVelocity")
        if bodyVelocity then
            bodyVelocity:Destroy()
        end

        local bodyGyro = rootPart:FindFirstChild("FlightGyro")
        if bodyGyro then
            bodyGyro:Destroy()
        end

        print(targetPlayer .. " não está mais voando.")
    end
end

-- Monitorar e atualizar o estado de voo
game:GetService("RunService").RenderStepped:Connect(function()
    for playerName, canFly in pairs(flyingPlayers) do
        if canFly then
            local targetCharacter = game.Players:FindFirstChild(playerName) and game.Players[playerName].Character
            local rootPart = targetCharacter and targetCharacter:FindFirstChild("HumanoidRootPart")
            local humanoid = targetCharacter and targetCharacter:FindFirstChild("Humanoid")

            if rootPart and humanoid then
                local moveDirection = humanoid.MoveDirection
                local bodyVelocity = rootPart:FindFirstChild("FlightVelocity")
                if bodyVelocity then
                    bodyVelocity.Velocity = Vector3.new(moveDirection.X * speed, bodyVelocity.Velocity.Y, moveDirection.Z * speed)
                end
            end
        end
    end
end)

-- Detectar teclas para ajustar voo
local userInputService = game:GetService("UserInputService")
userInputService.InputBegan:Connect(function(input, gameProcessedEvent)
    if gameProcessedEvent then return end

    for playerName, canFly in pairs(flyingPlayers) do
        if canFly then
            local targetCharacter = game.Players:FindFirstChild(playerName) and game.Players[playerName].Character
            local rootPart = targetCharacter and targetCharacter:FindFirstChild("HumanoidRootPart")

            if rootPart then
                local bodyVelocity = rootPart:FindFirstChild("FlightVelocity")
                if input.KeyCode == Enum.KeyCode.Space and bodyVelocity then
                    bodyVelocity.Velocity = Vector3.new(bodyVelocity.Velocity.X, speed, bodyVelocity.Velocity.Z)
                elseif input.KeyCode == Enum.KeyCode.LeftShift and bodyVelocity then
                    bodyVelocity.Velocity = Vector3.new(bodyVelocity.Velocity.X, -speed, bodyVelocity.Velocity.Z)
                end
            end
        end
    end
end)
