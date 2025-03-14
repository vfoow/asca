-- // Troll Hub Completo com Música e Skybox Global ID Pronto! // --

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
mainFrame.Size = UDim2.new(0, 300, 0, 550)
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

-- // Botão 1: Crescer Neon
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
end)

-- // Botão 2: Super velocidade e pulo
createButton("Super Speed & Jump", UDim2.new(0, 50, 0, 120), mainFrame, function()
    local player = game.Players.LocalPlayer
    local char = player.Character or player.CharacterAdded:Wait()
    local humanoid = char:WaitForChild("Humanoid")

    humanoid.WalkSpeed = 100
    humanoid.JumpPower = 200
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
end)

-- // Botão 5 + Caixa de Texto: Música Global
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

createButton("Tocar Música 🎵", UDim2.new(0, 50, 0, 350), mainFrame, function()
    local musicId = tonumber(musicIdBox.Text)

    if musicId then
        local sound = Instance.new("Sound", workspace)
        sound.SoundId = "rbxassetid://"..musicId
        sound.Looped = true
        sound.Volume = 10
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

-- // Botão 6: Mudar Skybox com ID fixo
createButton("Mudar Skybox 🌌", UDim2.new(0, 50, 0, 410), mainFrame, function()
    local assetId = 118489197285736 -- Seu ID fixo aqui!

    local sky = Instance.new("Sky")

    sky.SkyboxBk = "rbxassetid://"..assetId
    sky.SkyboxDn = "rbxassetid://"..assetId
    sky.SkyboxFt = "rbxassetid://"..assetId
    sky.SkyboxLf = "rbxassetid://"..assetId
    sky.SkyboxRt = "rbxassetid://"..assetId
    sky.SkyboxUp = "rbxassetid://"..assetId

    for _,v in pairs(game.Lighting:GetChildren()) do
        if v:IsA("Sky") then
            v:Destroy()
        end
    end

    sky.Parent = game.Lighting

    game.Lighting.Brightness = 0.2
    game.Lighting.FogEnd = 1000
    game.Lighting.FogColor = Color3.fromRGB(0, 0, 0)

    game.StarterGui:SetCore("SendNotification", {
        Title = "Troll Hub",
        Text = "Skybox mudada com sucesso para todos! 🌌",
        Duration = 5
    })
end)

print("🔥 Troll Hub com Música + Skybox Global carregado! 🔥")
