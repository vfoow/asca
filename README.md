local game = game:GetService("Game")

local player = game.Players.LocalPlayer

local character = player.Character

script.Parent.MouseButton1Down:Connect()

game:GetService("RunService"):BindToRenderStep("Fly", 2, script.Parent)

script.Fly:Connect(function()

local humanoid = player.Humanoid

local old\_pos = humanoid.Position

local old\_cframe = humanoid.CFrame
