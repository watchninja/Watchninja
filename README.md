-- Serviços
local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local Debris = game:GetService("Debris")
local player = Players.LocalPlayer

-- GUI Principal
local screenGui = Instance.new("ScreenGui")
screenGui.Name = "CustomGUI"
screenGui.ResetOnSpawn = false
screenGui.Parent = player:WaitForChild("PlayerGui")

-- 🔐 Variáveis da Key
local chaveCorreta = "watch"
local acessoLiberado = false

-- Função de notificação
local function showNotification(text)
	local note = Instance.new("TextLabel")
	note.Size = UDim2.new(0, 200, 0, 35)
	note.Position = UDim2.new(1, -210, 1, -60)
	note.BackgroundColor3 = Color3.fromRGB(0, 0, 0)
	note.BackgroundTransparency = 0.3
	note.Text = text
	note.TextColor3 = Color3.new(1, 1, 1)
	note.TextScaled = true
	note.Font = Enum.Font.GothamBold
	note.ZIndex = 20
	note.Parent = screenGui
	local corner = Instance.new("UICorner")
	corner.CornerRadius = UDim.new(0, 8)
	corner.Parent = note
	Debris:AddItem(note, 2)
end

-- Drag UI
local function enableDrag(frame)
	local dragging, dragInput, startPos, startInputPos
	local function update(input)
		local delta = input.Position - startInputPos
		frame.Position = UDim2.new(startPos.X.Scale, startPos.X.Offset + delta.X, startPos.Y.Scale, startPos.Y.Offset + delta.Y)
	end
	frame.InputBegan:Connect(function(input)
		if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then
			dragging = true
			startInputPos = input.Position
			startPos = frame.Position
			input.Changed:Connect(function()
				if input.UserInputState == Enum.UserInputState.End then
					dragging = false
				end
			end)
		end
	end)
	frame.InputChanged:Connect(function(input)
		if dragging and (input.UserInputType == Enum.UserInputType.MouseMovement or input.UserInputType == Enum.UserInputType.Touch) then
			update(input)
		end
	end)
end

-- Painel principal
local frame = Instance.new("Frame")
frame.Size = UDim2.new(0, 180, 0, 210)
frame.Position = UDim2.new(0, 20, 0.5, -105)
frame.BackgroundColor3 = Color3.fromRGB(30, 30, 30)
frame.BorderSizePixel = 0
frame.Visible = false
frame.Active = true
frame.Parent = screenGui
Instance.new("UICorner", frame).CornerRadius = UDim.new(0, 10)

-- Toggle Button
local toggleButton = Instance.new("TextButton")
toggleButton.Size = UDim2.new(0, 40, 0, 40)
toggleButton.Position = UDim2.new(0, 10, 0, 10)
toggleButton.BackgroundColor3 = Color3.fromRGB(0, 0, 0)
toggleButton.Text = "≡"
toggleButton.TextColor3 = Color3.fromRGB(255, 0, 0)
toggleButton.Font = Enum.Font.GothamBold
toggleButton.TextSize = 24
toggleButton.Active = true
toggleButton.Visible = false
toggleButton.Parent = screenGui
Instance.new("UICorner", toggleButton).CornerRadius = UDim.new(1, 0)
enableDrag(toggleButton)
toggleButton.MouseButton1Click:Connect(function()
	frame.Visible = not frame.Visible
end)

-- Créditos
local creditLabel = Instance.new("TextLabel")
creditLabel.Size = UDim2.new(1, 0, 0, 20)
creditLabel.Position = UDim2.new(0, 0, 1, -20)
creditLabel.BackgroundTransparency = 1
creditLabel.Text = "Créditos: watchninja"
creditLabel.TextColor3 = Color3.fromRGB(255, 255, 255)
creditLabel.Font = Enum.Font.Gotham
creditLabel.TextSize = 14
creditLabel.TextXAlignment = Enum.TextXAlignment.Center
creditLabel.Parent = frame

-- Variáveis
local altura = 150
local plataformaAerea
local humanoid
local speedEnabled = false
local speedValue = 40
local voando = false

local function getRootPart()
	return player.Character and player.Character:FindFirstChild("HumanoidRootPart") or nil
end

local function atualizarVelocidade()
	if humanoid then
		humanoid.WalkSpeed = speedEnabled and speedValue or 16
	end
end

local function onCharacterAdded(character)
	humanoid = character:WaitForChild("Humanoid")
	atualizarVelocidade()
	humanoid:GetPropertyChangedSignal("WalkSpeed"):Connect(atualizarVelocidade)
end

player.CharacterAdded:Connect(onCharacterAdded)
if player.Character then onCharacterAdded(player.Character) end

-- Botão de velocidade
local function criarBotao(pai, texto, posY, callback)
	local btn = Instance.new("TextButton")
	btn.Size = UDim2.new(1, -20, 0, 30)
	btn.Position = UDim2.new(0, 10, 0, posY)
	btn.Text = texto
	btn.BackgroundColor3 = Color3.fromRGB(0, 0, 0)
	btn.TextColor3 = Color3.fromRGB(255, 0, 0)
	btn.Font = Enum.Font.GothamBold
	btn.TextSize = 16
	btn.Parent = pai
	Instance.new("UICorner", btn).CornerRadius = UDim.new(0, 6)
	btn.MouseButton1Click:Connect(callback)
end

criarBotao(frame, "⚡ Velocidade", 20, function()
	speedEnabled = not speedEnabled
	atualizarVelocidade()
	showNotification(speedEnabled and "Velocidade ativada!" or "Velocidade desativada!")
end)

-- Botão UNIFICADO: "Sair e Subir"
local sairESubirBtn = Instance.new("TextButton")
sairESubirBtn.Size = UDim2.new(0, 160, 0, 30)
sairESubirBtn.Position = UDim2.new(1, -300, 1, -220)
sairESubirBtn.BackgroundColor3 = Color3.fromRGB(0, 0, 0)
sairESubirBtn.Text = "🚀 Sair e Subir"
sairESubirBtn.TextColor3 = Color3.fromRGB(0, 255, 255)
sairESubirBtn.Font = Enum.Font.GothamBold
sairESubirBtn.TextSize = 14
sairESubirBtn.Visible = false
sairESubirBtn.Parent = screenGui
Instance.new("UICorner", sairESubirBtn).CornerRadius = UDim.new(0, 6)
enableDrag(sairESubirBtn)

sairESubirBtn.MouseButton1Click:Connect(function()
	local rootPart = getRootPart()
	if not rootPart then return end

	voando = true
	local duration = 3
	local speed = 40
	if rootPart:FindFirstChild("FlightVelocity") then
		rootPart.FlightVelocity:Destroy()
	end
	local bodyVelocity = Instance.new("BodyVelocity")
	bodyVelocity.Name = "FlightVelocity"
	bodyVelocity.Velocity = (rootPart.CFrame.RightVector + Vector3.new(0, 0.2, 0)).Unit * speed
	bodyVelocity.MaxForce = Vector3.new(1, 1, 1) * 1e5
	bodyVelocity.Parent = rootPart
	showNotification("Voando para a direita...")

	task.delay(duration, function()
		if bodyVelocity and bodyVelocity.Parent then
			bodyVelocity:Destroy()
			voando = false
			showNotification("Você saiu da base!")

			-- Subir
			local destino = rootPart.Position + Vector3.new(0, altura, 0)
			if plataformaAerea then plataformaAerea:Destroy() end
			plataformaAerea = Instance.new("Part")
			plataformaAerea.Size = Vector3.new(20, 4, 20)
			plataformaAerea.Position = destino - Vector3.new(0, 2, 0)
			plataformaAerea.Anchored = true
			plataformaAerea.Transparency = 1
			plataformaAerea.CanCollide = true
			plataformaAerea.Parent = workspace
			rootPart.CFrame = CFrame.new(destino, destino + rootPart.CFrame.LookVector)
			showNotification("Teleportado para cima!")
		end
	end)
end)

-- Botão de Descer
local frameDescer = Instance.new("TextButton")
frameDescer.Size = UDim2.new(0, 90, 0, 30)
frameDescer.Position = UDim2.new(1, -100, 1, -100)
frameDescer.BackgroundColor3 = Color3.fromRGB(0, 0, 0)
frameDescer.Text = "↓ Descer"
frameDescer.TextColor3 = Color3.fromRGB(255, 0, 0)
frameDescer.Font = Enum.Font.GothamBold
frameDescer.TextSize = 16
frameDescer.Visible = false
frameDescer.Parent = screenGui
Instance.new("UICorner", frameDescer).CornerRadius = UDim.new(0, 6)
enableDrag(frameDescer)

frameDescer.MouseButton1Click:Connect(function()
	local rootPart = getRootPart()
	if rootPart then
		rootPart.CFrame = rootPart.CFrame - Vector3.new(0, altura, 0)
		if plataformaAerea then plataformaAerea:Destroy() plataformaAerea = nil end
		showNotification("Você desceu!")
	end
end)

-- Tela de Key
local keyFrame = Instance.new("Frame")
keyFrame.Size = UDim2.new(0, 300, 0, 150)
keyFrame.Position = UDim2.new(0.5, -150, 0.5, -75)
keyFrame.BackgroundColor3 = Color3.fromRGB(20, 20, 20)
keyFrame.BorderSizePixel = 0
keyFrame.ZIndex = 10
keyFrame.Parent = screenGui
Instance.new("UICorner", keyFrame).CornerRadius = UDim.new(0, 10)

local title = Instance.new("TextLabel")
title.Size = UDim2.new(1, 0, 0, 30)
title.Position = UDim2.new(0, 0, 0, 10)
title.BackgroundTransparency = 1
title.Text = "🔑 Digite a chave de acesso"
title.TextColor3 = Color3.fromRGB(255, 255, 255)
title.Font = Enum.Font.GothamBold
title.TextSize = 18
title.Parent = keyFrame

local input = Instance.new("TextBox")
input.PlaceholderText = "Digite a chave"
input.Size = UDim2.new(0.8, 0, 0, 30)
input.Position = UDim2.new(0.1, 0, 0.4, 0)
input.Font = Enum.Font.Gotham
input.TextSize = 16
input.TextColor3 = Color3.fromRGB(255, 255, 255)
input.BackgroundColor3 = Color3.fromRGB(40, 40, 40)
input.BorderSizePixel = 0
input.Parent = keyFrame
Instance.new("UICorner", input).CornerRadius = UDim.new(0, 6)

local verificarBtn = Instance.new("TextButton")
verificarBtn.Text = "✅ Confirmar"
verificarBtn.Size = UDim2.new(0.8, 0, 0, 30)
verificarBtn.Position = UDim2.new(0.1, 0, 0.7, 0)
verificarBtn.Font = Enum.Font.GothamBold
verificarBtn.TextSize = 16
verificarBtn.BackgroundColor3 = Color3.fromRGB(0, 170, 0)
verificarBtn.TextColor3 = Color3.new(1, 1, 1)
verificarBtn.ZIndex = 11
verificarBtn.Parent = keyFrame
Instance.new("UICorner", verificarBtn).CornerRadius = UDim.new(0, 6)

verificarBtn.MouseButton1Click:Connect(function()
	if input.Text == chaveCorreta then
		keyFrame:Destroy()
		acessoLiberado = true
		frame.Visible = true
		toggleButton.Visible = true
		sairESubirBtn.Visible = true
		frameDescer.Visible = true
		showNotification("✅ Acesso Liberado!")
	else
		showNotification("❌ Chave incorreta.")
	end
end)
