--[[
Script Local para Roblox: Voo com Interface GUI
Como usar:
1. Execute este script em um LocalScript (StarterPlayerScripts, por exemplo).
2. Um botão aparecerá na tela. Clique para ativar/desativar o voo.
--]]

-- Configuração da Interface
local player = game.Players.LocalPlayer

-- Cria o ScreenGui
local screenGui = Instance.new("ScreenGui")
screenGui.Name = "FlyGui"
screenGui.ResetOnSpawn = false
screenGui.Parent = player:WaitForChild("PlayerGui")

-- Cria o botão
local button = Instance.new("TextButton")
button.Size = UDim2.new(0, 120, 0, 40)
button.Position = UDim2.new(0, 20, 0, 20)
button.BackgroundColor3 = Color3.fromRGB(60, 120, 255)
button.Text = "Ativar Voo"
button.TextSize = 18
button.TextColor3 = Color3.new(1, 1, 1)
button.Parent = screenGui

-- Variáveis de voo
local flying = false
local flySpeed = 50
local bodyGyro = nil
local bodyVelocity = nil

-- Função para ativar/desativar voo
local function setFlying(state)
    local char = player.Character or player.CharacterAdded:Wait()
    local hrp = char:FindFirstChild("HumanoidRootPart")
    local humanoid = char:FindFirstChildOfClass("Humanoid")
    if not hrp or not humanoid then return end

    if state and not flying then
        flying = true
        button.Text = "Desativar Voo"

        -- Cria BodyGyro e BodyVelocity
        bodyGyro = Instance.new("BodyGyro")
        bodyGyro.P = 9e4
        bodyGyro.MaxTorque = Vector3.new(9e9, 9e9, 9e9)
        bodyGyro.CFrame = hrp.CFrame
        bodyGyro.Parent = hrp

        bodyVelocity = Instance.new("BodyVelocity")
        bodyVelocity.Velocity = Vector3.new(0, 0, 0)
        bodyVelocity.MaxForce = Vector3.new(9e9, 9e9, 9e9)
        bodyVelocity.Parent = hrp

        -- Loop de voo
        spawn(function()
            while flying and player.Character and humanoid.Health > 0 do
                local cam = workspace.CurrentCamera
                local moveDir = Vector3.new()
                if userInput:IsKeyDown(Enum.KeyCode.W) then
                    moveDir = moveDir + cam.CFrame.LookVector
                end
                if userInput:IsKeyDown(Enum.KeyCode.S) then
                    moveDir = moveDir - cam.CFrame.LookVector
                end
                if userInput:IsKeyDown(Enum.KeyCode.A) then
                    moveDir = moveDir - cam.CFrame.RightVector
                end
                if userInput:IsKeyDown(Enum.KeyCode.D) then
                    moveDir = moveDir + cam.CFrame.RightVector
                end
                if userInput:IsKeyDown(Enum.KeyCode.Space) then
                    moveDir = moveDir + cam.CFrame.UpVector
                end
                if userInput:IsKeyDown(Enum.KeyCode.LeftControl) or userInput:IsKeyDown(Enum.KeyCode.LeftShift) then
                    moveDir = moveDir - cam.CFrame.UpVector
                end

                if moveDir.Magnitude > 0 then
                    moveDir = moveDir.Unit
                end
                bodyVelocity.Velocity = moveDir * flySpeed
                bodyGyro.CFrame = cam.CFrame
                wait()
            end
        end)

    elseif not state and flying then
        flying = false
        button.Text = "Ativar Voo"
        if bodyGyro then bodyGyro:Destroy() end
        if bodyVelocity then bodyVelocity:Destroy() end
    end
end

-- Input de teclado para movimento
local userInput = game:GetService("UserInputService")

-- Clique do botão
button.MouseButton1Click:Connect(function()
    setFlying(not flying)
end)

-- Se morrer, desativa voo
player.CharacterAdded:Connect(function(char)
    setFlying(false)
end)
