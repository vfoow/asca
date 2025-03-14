-- // Troll Hub Completo com Música Global // --

-- Função rápida para criar botões
function createButton(name, position, parent, callback)
    local button = Instance.new("TextButton")
    button.Name = name
    button.Parent = parent
    button.BackgroundColor3 = Color3.fromRGB(30, 30, 30)
    button.Position = position
    button.Size = UDim2.new(0, 200, 0, 50)
    button.Font = Enum.Font.GothamBold
    button.Text = name
    button.TextColor3 = Color3.fromRGB(255, 255, 255)
    button.TextSize = 18
    button.MouseButton1Click:Connect(callback)
end

-- Criar ScreenGui
local screenGui = Instance.new("ScreenGui")
screenGui.Parent = game.CoreGui
screenGui.Name = "TrollHub"

-- Criar o Frame
local mainFrame = Instance.new("Frame")
mainFrame.Parent = screenGui
mainFrame.BackgroundColor3 = Color3.fromRGB(20, 20, 20)
mainFrame.Position = UDim2.new(0, 50, 0, 100)
mainFrame.Size = UDim2.new(0, 300, 0, 400)
mainFrame.Active = true
mainFrame.Draggable = true

-- Título
local title = Instance.new("TextLabel")
title.Parent = mainFrame
title.BackgroundColor3 = Color3.fromRGB(40, 40, 40)
title.Size = UDim2.new(1, 0, 0, 50)
title.Font = Enum.Font.GothamBold
title.Text = "🔥 Troll Hub 🔥"
title.TextColor3 = Color3.fromRGB(255, 255, 255)
title.TextSize = 22

-- // Botão 1: Crescer e virar neon
createButton("Gigante Neon", UDim2.new(0, 50, 0, 60), mainFrame, function()
    local player = game.Players.LocalPlayer
    local char = player.Character or player.CharacterAdded:Wait()

    for _, part in pairs(char:GetChildren()) do
        if part:IsA("BasePart") then
            part.Size = part.Size * 3
            part.Material = Enum.Material.Neon
            part.BrickColor = BrickColor.new("Really red")
        end
    end
    game.StarterGui:SetCore("SendNotification", {
        Title = "Troll Hub",
        Text = "Você virou um Gigante Neon! 🔴",
        Duration = 3
    })
end)

-- // Botão 2: Super velocidade e pulo
createButton("Super Speed & Jump", UDim2.new(0, 50, 0, 120), mainFrame, function()
    local player = game.Players.LocalPlayer
    local char = player.Character or player.CharacterAdded:Wait()
    local humanoid = char:WaitForChild("Humanoid")

    humanoid.WalkSpeed = 100
    humanoid.JumpPower = 200

    game.StarterGui:SetCore("SendNotification", {
        Title = "Troll Hub",
        Text = "Super velocidade e pulo ativados! ⚡",
        Duration = 3
    })
end)

-- // Botão 3: Clones troll
createButton("Spamar Clones", UDim2.new(0, 50, 0, 180), mainFrame, function()
    local player = game.Players.LocalPlayer
    local char = player.Character or player.CharacterAdded:Wait()

    spawn(function()
        while true do
            local clone = char:Clone()
            clone.Parent = workspace
            clone:MoveTo(char.PrimaryPart.Position + Vector3.new(math.random(-5,5), 0, math.random(-5,5)))
            for _,v in pairs(clone:GetChildren()) do
                if v:IsA("BasePart") then
                    v.BrickColor = BrickColor.new("Really black")
                    v.Material = Enum.Material.SmoothPlastic
                end
            end
            wait(5)
            clone:Destroy()
        end
    end)

    game.StarterGui:SetCore("SendNotification", {
        Title = "Troll Hub",
        Text = "Clones começaram a aparecer! 🤖",
        Duration = 3
    })
end)

-- // Botão 4: Explosões troll
createButton("Explosões Troll", UDim2.new(0, 50, 0, 240), mainFrame, function()
    local player = game.Players.LocalPlayer
    local char = player.Character or player.CharacterAdded:Wait()

    spawn(function()
        while true do
            local explosion = Instance.new("Explosion")
            explosion.Position = char.PrimaryPart.Position
            explosion.BlastRadius = 5
            explosion.BlastPressure = 0
            explosion.Parent = workspace
            wait(2)
        end
    end)

    game.StarterGui:SetCore("SendNotification", {
        Title = "Troll Hub",
        Text = "Explosões sem parar! 💥",
        Duration = 3
    })
end)

-- // Botão 5 + Caixa de Texto: Música Global
-- Caixa de Texto para colocar o ID
local musicIdBox = Instance.new("TextBox")
musicIdBox.Parent = mainFrame
musicIdBox.BackgroundColor3 = Color3.fromRGB(30, 30, 30)
musicIdBox.Position = UDim2.new(0, 50, 0, 300)
musicIdBox.Size = UDim2.new(0, 200, 0, 40)
musicIdBox.Font = Enum.Font.Gotham
musicIdBox.PlaceholderText = "Digite o ID da música"
musicIdBox.Text = ""
musicIdBox.TextColor3 = Color3.fromRGB(255, 255, 255)
musicIdBox.TextSize = 16

-- Botão para tocar a música
createButton("Tocar Música 🎵", UDim2.new(0, 50, 0, 350), mainFrame, function()
    local musicId = tonumber(musicIdBox.Text)

    if musicId then
        -- Cria um Sound e toca globalmente
        local sound = Instance.new("Sound", workspace)
        sound.SoundId = "rbxassetid://"..musicId
        sound.Looped = true
        sound.Volume = 10 -- Volume alto pra geral ouvir
        sound:Play()

        game.StarterGui:SetCore("SendNotification", {
            Title = "Troll Hub",
            Text = "Música tocando para o servidor! 🎶",
            Duration = 3
        })
    else
        game.StarterGui:SetCore("SendNotification", {
            Title = "Erro",
            Text = "ID inválido! Digite um número válido.",
            Duration = 3
        })
    end
end)

print("🔥 Troll Hub com Música Global carregado! 🔥")
