-- Serviços
local Players = game:GetService("Players")
local UIS = game:GetService("UserInputService")
local RunService = game:GetService("RunService")
local Debris = game:GetService("Debris")
local player = Players.LocalPlayer

-- GUI Principal
local screenGui = Instance.new("ScreenGui")
screenGui.Name = "CustomGUI"
screenGui.ResetOnSpawn = false
screenGui.Parent = player:WaitForChild("PlayerGui")

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
	note.Parent = screenGui

	local corner = Instance.new("UICorner")
	corner.CornerRadius = UDim.new(0, 8)
	corner.Parent = note

	Debris:AddItem(note, 2)
end

-- Função de arrastar UI
local function enableDrag(frame)
	local dragging, dragInput, startPos, startInputPos

	local function update(input)
		local delta = input.Position - startInputPos
		frame.Position = UDim2.new(startPos.X.Scale, startPos.X.Offset + delta.X,
								   startPos.Y.Scale, startPos.Y.Offset + delta.Y)
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

local corner = Instance.new("UICorner")
corner.CornerRadius = UDim.new(0, 10)
corner.Parent = frame

-- Botão de toggle
local toggleButton = Instance.new("TextButton")
toggleButton.Size = UDim2.new(0, 40, 0, 40)
toggleButton.Position = UDim2.new(0, 10, 0, 10)
toggleButton.BackgroundColor3 = Color3.fromRGB(0, 0, 0)
toggleButton.Text = "≡"
toggleButton.TextColor3 = Color3.fromRGB(255, 0, 0)
toggleButton.Font = Enum.Font.GothamBold
toggleButton.TextSize = 24
toggleButton.Active = true
toggleButton.Parent = screenGui

local iconCorner = Instance.new("UICorner")
iconCorner.CornerRadius = UDim.new(1, 0)
iconCorner.Parent = toggleButton

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

-- Função para criar botões
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

	local btnCorner = Instance.new("UICorner")
	btnCorner.CornerRadius = UDim.new(0, 6)
	btnCorner.Parent = btn

	btn.MouseButton1Click:Connect(callback)
end

-- Funções de movimentação
local altura = 150
local plataformaAerea
local humanoid
local speedEnabled = false
local speedValue = 45
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

	humanoid:GetPropertyChangedSignal("WalkSpeed"):Connect(function()
		atualizarVelocidade()
	end)
end

player.CharacterAdded:Connect(onCharacterAdded)
if player.Character then
	onCharacterAdded(player.Character)
else
	player.CharacterAdded:Wait()
end

-- Botões do painel principal
criarBotao(frame, "⚡ Velocidade", 20, function()
	speedEnabled = not speedEnabled
	atualizarVelocidade()
	showNotification(speedEnabled and "Velocidade ativada!" or "Velocidade desativada!")
end)

-- Botão SUBIR
local frameSubir = Instance.new("TextButton")
frameSubir.Size = UDim2.new(0, 90, 0, 30)
frameSubir.Position = UDim2.new(1, -200, 1, -100)
frameSubir.BackgroundColor3 = Color3.fromRGB(0, 0, 0)
frameSubir.Text = "↑ Subir"
frameSubir.TextColor3 = Color3.fromRGB(255, 0, 0)
frameSubir.Font = Enum.Font.GothamBold
frameSubir.TextSize = 16
frameSubir.Parent = screenGui

Instance.new("UICorner", frameSubir).CornerRadius = UDim.new(0, 6)
enableDrag(frameSubir)

frameSubir.MouseButton1Click:Connect(function()
	local rootPart = getRootPart()
	if rootPart then
		local destino = rootPart.Position + Vector3.new(0, altura, 0)
		local result = workspace:Raycast(rootPart.Position, Vector3.new(0, altura, 0), RaycastParams.new())
		if result then
			showNotification("Algo está acima!")
		end
		if plataformaAerea then plataformaAerea:Destroy() end

		plataformaAerea = Instance.new("Part")
		plataformaAerea.Size = Vector3.new(20, 4, 20)
		plataformaAerea.Position = destino - Vector3.new(0, 2, 0)
		plataformaAerea.Anchored = true
		plataformaAerea.Transparency = 1
		plataformaAerea.CanCollide = true
		plataformaAerea.Name = "PlataformaAerea"
		plataformaAerea.Parent = workspace

		rootPart.CFrame = CFrame.new(destino, destino + rootPart.CFrame.LookVector)
		showNotification("Teleportado para cima!")
	end
end)

-- Botão DESCER
local frameDescer = Instance.new("TextButton")
frameDescer.Size = UDim2.new(0, 90, 0, 30)
frameDescer.Position = UDim2.new(1, -100, 1, -100)
frameDescer.BackgroundColor3 = Color3.fromRGB(0, 0, 0)
frameDescer.Text = "↓ Descer"
frameDescer.TextColor3 = Color3.fromRGB(255, 0, 0)
frameDescer.Font = Enum.Font.GothamBold
frameDescer.TextSize = 16
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

-- BOTÃO: ➡️ Sair da Base (com BodyVelocity reduzida)
local sairBaseBtn = Instance.new("TextButton")
sairBaseBtn.Size = UDim2.new(0, 160, 0, 30)
sairBaseBtn.Position = UDim2.new(1, -300, 1, -220)
sairBaseBtn.BackgroundColor3 = Color3.fromRGB(0, 0, 0)
sairBaseBtn.Text = "➡️ Sair da Base"
sairBaseBtn.TextColor3 = Color3.fromRGB(0, 255, 0)
sairBaseBtn.Font = Enum.Font.GothamBold
sairBaseBtn.TextSize = 14
sairBaseBtn.Parent = screenGui

Instance.new("UICorner", sairBaseBtn).CornerRadius = UDim.new(0, 6)
enableDrag(sairBaseBtn)

sairBaseBtn.MouseButton1Click:Connect(function()
	local rootPart = getRootPart()
	if not rootPart then return end

	voando = true
	local duration = 3
	local speed = 40 -- Reduzido para estabilidade

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
			showNotification("Você saiu da base!")
			voando = false
		end
	end)
end)

-- BOTÃO: 🛑 Parar Voo
local pararVooBtn = Instance.new("TextButton")
pararVooBtn.Size = UDim2.new(0, 160, 0, 30)
pararVooBtn.Position = UDim2.new(1, -300, 1, -180)
pararVooBtn.BackgroundColor3 = Color3.fromRGB(0, 0, 0)
pararVooBtn.Text = "🛑 Parar Voo"
pararVooBtn.TextColor3 = Color3.fromRGB(255, 50, 50)
pararVooBtn.Font = Enum.Font.GothamBold
pararVooBtn.TextSize = 14
pararVooBtn.Parent = screenGui

Instance.new("UICorner", pararVooBtn).CornerRadius = UDim.new(0, 6)
enableDrag(pararVooBtn)

pararVooBtn.MouseButton1Click:Connect(function()
	if voando then
		voando = false
		local rootPart = getRootPart()
		if rootPart and rootPart:FindFirstChild("FlightVelocity") then
			rootPart.FlightVelocity:Destroy()
			showNotification("Voo cancelado!")
		end
	end
end)

-- === ESP Lock Time Automática ===
local lteInstances = {}
local plotName = nil

local function updatelock()
	for _, plot in pairs(workspace.Plots:GetChildren()) do
		local billboardName = "LockTimeESP_" .. plot.Name
		local timeLabel = plot:FindFirstChild("Purchases", true)
			and plot.Purchases:FindFirstChild("PlotBlock", true)
			and plot.Purchases.PlotBlock.Main:FindFirstChild("BillboardGui", true)
			and plot.Purchases.PlotBlock.Main.BillboardGui:FindFirstChild("RemainingTime", true)

		if timeLabel and timeLabel:IsA("TextLabel") then
			local existing = lteInstances[plot.Name]
			local isUnlocked = timeLabel.Text == "0s"
			local displayText = isUnlocked and "Unlocked" or ("Lock: " .. timeLabel.Text)

			local color = Color3.fromRGB(255, 255, 0)
			if plot.Name == plotName then
				color = Color3.fromRGB(0, 255, 0)
			elseif isUnlocked then
				color = Color3.fromRGB(255, 0, 0)
			end

			if not existing then
				local gui = Instance.new("BillboardGui")
				gui.Name = billboardName
				gui.Size = UDim2.new(0, 120, 0, 25)
				gui.StudsOffset = Vector3.new(0, 4, 0)
				gui.AlwaysOnTop = true
				gui.Adornee = plot.Purchases.PlotBlock.Main
				gui.Parent = plot

				local label = Instance.new("TextLabel")
				label.Size = UDim2.new(1, 0, 1, 0)
				label.BackgroundTransparency = 1
				label.TextScaled = true
				label.TextColor3 = color
				label.TextStrokeTransparency = 0
				label.TextStrokeColor3 = Color3.new(0, 0, 0)
				label.Font = Enum.Font.SourceSansBold
				label.Text = displayText
				label.Parent = gui

				lteInstances[plot.Name] = gui
			else
				local label = existing:FindFirstChildOfClass("TextLabel")
				if label then
					label.Text = displayText
					label.TextColor3 = color
				end
			end
		else
			if lteInstances[plot.Name] then
				lteInstances[plot.Name]:Destroy()
				lteInstances[plot.Name] = nil
			end
		end
	end
end

RunService.Heartbeat:Connect(function(dt)
	accumulatedTime = (accumulatedTime or 0) + dt
	if accumulatedTime >= 1 then
		accumulatedTime = 0
		updatelock()
	end
end)
