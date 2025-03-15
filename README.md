--[[  
🔥 TROLL HUB BROOKHAVEN 🔥  
💻 Feito por LUA Programming GOD  
]]--

-- Proteção simples
pcall(function() game.CoreGui.TrollHub:Destroy() end)

-- Criar a interface principal
local gui = Instance.new("ScreenGui")
gui.Name = "TrollHub"
gui.Parent = game.CoreGui

local frame = Instance.new("Frame")
frame.Parent = gui
frame.BackgroundColor3 = Color3.fromRGB(20, 20, 20)
frame.Position = UDim2.new(0, 50, 0, 100)
frame.Size = UDim2.new(0, 350, 0, 600)
frame.Active = true
frame.Draggable = true

local title = Instance.new("TextLabel")
title.Parent = frame
title.Text = "🔥 TROLL HUB 🔥"
title.TextColor3 = Color3.fromRGB(255, 255, 255)
title.Size = UDim2.new(1, 0, 0, 50)
title.BackgroundColor3 = Color3.fromRGB(40, 40, 40)
title.Font = Enum.Font.GothamBlack
title.TextSize = 24

-- Função para criar botões
function criarBotao(texto, posY, callback)
    local button = Instance.new("TextButton")
    button.Parent = frame
    button.Text = texto
    button.Size = UDim2.new(0, 300, 0, 40)
    button.Position = UDim2.new(0, 25, 0, posY)
    button.BackgroundColor3 = Color3.fromRGB(30, 30, 30)
    button.TextColor3 = Color3.fromRGB(255, 255, 255)
    button.Font = Enum.Font.Gotham
    button.TextSize = 18
    button.MouseButton1Click:Connect(callback)
end

--------------------------------------------------
-- BOTÕES E FUNÇÕES DA HUB
--------------------------------------------------

-- Botão 1 - Virar Gigante Neon
criarBotao("👾 Virar Gigante Neon", 60, function()
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
        Text = "Você virou um monstro gigante! 😈",
        Duration = 4
    })
end)

-- Botão 2 - Super Speed & Jump
criarBotao("⚡ Super Speed + Jump", 110, function()
    local char = game.Players.LocalPlayer.Character
    local humanoid = char:FindFirstChildOfClass("Humanoid")

    humanoid.WalkSpeed = 150
    humanoid.JumpPower = 200

    game.StarterGui:SetCore("SendNotification", {
        Title = "Troll Hub",
        Text = "Velocidade e pulo ativados! 🏃‍♂️",
        Duration = 4
    })
end)

-- Botão 3 - Explosões infinitas
criarBotao("💣 Explosões Troll", 160, function()
    spawn(function()
        while true do
            local explosion = Instance.new("Explosion")
            explosion.Position = game.Players.LocalPlayer.Character.PrimaryPart.Position + Vector3.new(0, 5, 0)
            explosion.BlastRadius = 5
            explosion.BlastPressure = 0
            explosion.Parent = workspace
            wait(1)
        end
    end)

    game.StarterGui:SetCore("SendNotification", {
        Title = "Troll Hub",
        Text = "Explosões infinitas ativadas! 💥",
        Duration = 4
    })
end)

-- Botão 4 - Spamar Clones Pretos
criarBotao("🧟‍♂️ Spamar Clones Pretos", 210, function()
    local player = game.Players.LocalPlayer
    local char = player.Character or player.CharacterAdded:Wait()

    spawn(function()
        while true do
            local clone = char:Clone()
            clone.Parent = workspace
            clone:MoveTo(char.PrimaryPart.Position + Vector3.new(math.random(-5, 5), 0, math.random(-5, 5)))
            
            for _, v in pairs(clone:GetChildren()) do
                if v:IsA("BasePart") then
                    v.BrickColor = BrickColor.new("Really black")
                    v.Material = Enum.Material.SmoothPlastic
                end
            end

            wait(3)
            clone:Destroy()
        end
    end)

    game.StarterGui:SetCore("SendNotification", {
        Title = "Troll Hub",
        Text = "Clones pretos invadindo! 👻",
        Duration = 4
    })
end)

-- Caixa de texto para ID de Música
local musicBox = Instance.new("TextBox")
musicBox.Parent = frame
musicBox.PlaceholderText = "Digite o ID da Música 🎶"
musicBox.Position = UDim2.new(0, 25, 0, 270)
musicBox.Size = UDim2.new(0, 300, 0, 40)
musicBox.BackgroundColor3 = Color3.fromRGB(30, 30, 30)
musicBox.TextColor3 = Color3.fromRGB(255, 255, 255)
musicBox.Font = Enum.Font.Gotham
musicBox.TextSize = 16

-- Botão 5 - Tocar Música Global
criarBotao("🎵 Tocar Música no Servidor", 320, function()
    local musicId = tonumber(musicBox.Text)

    if musicId then
        local sound = Instance.new("Sound", workspace)
        sound.SoundId = "rbxassetid://"..musicId
        sound.Looped = true
        sound.Volume = 10
        sound:Play()

        game.StarterGui:SetCore("SendNotification", {
            Title = "Troll Hub",
            Text = "Tocando música para todos! 🎶",
            Duration = 4
        })
    else
        game.StarterGui:SetCore("SendNotification", {
            Title = "Erro",
            Text = "Digite um ID de música válido!",
            Duration = 4
        })
    end
end)

-- Botão 6 - Mudar Skybox Global
criarBotao("🌌 Mudar Skybox para TODOS", 370, function()
    local skyId = "118489197285736"

    local sky = Instance.new("Sky")
    sky.SkyboxBk = "rbxassetid://"..skyId
    sky.SkyboxDn = "rbxassetid://"..skyId
    sky.SkyboxFt = "rbxassetid://"..skyId
    sky.SkyboxLf = "rbxassetid://"..skyId
    sky.SkyboxRt = "rbxassetid://"..skyId
    sky.SkyboxUp = "rbxassetid://"..skyId

    -- Limpar outras skybox se tiver
    for _, v in pairs(game.Lighting:GetChildren()) do
        if v:IsA("Sky") then
            v:Destroy()
        end
    end

    sky.Parent = game.Lighting

    game.Lighting.Brightness = 0.2
    game.Lighting.FogEnd = 500
    game.Lighting.FogColor = Color3.fromRGB(0, 0, 0)

    game.StarterGui:SetCore("SendNotification", {
        Title = "Troll Hub",
        Text = "Skybox alterada com sucesso! 🌌",
        Duration = 4
    })
end)

--------------------------------------------------
-- FIM DA HUB
--------------------------------------------------

print("🔥 TROLL HUB CARREGADA COM SUCESSO! 🔥")
