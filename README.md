-- Serviços
local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local Debris = game:GetService("Debris")
local Lighting = game:GetService("Lighting")
local UIS = game:GetService("UserInputService")
local player = Players.LocalPlayer

-- GUI Principal
local screenGui = Instance.new("ScreenGui")
screenGui.Name = "CustomGUI"
screenGui.ResetOnSpawn = false
screenGui.Parent = player:WaitForChild("PlayerGui")

-- Variáveis para o ESP Lock Time
local activeLockTimeEsp = true
local lteInstances = {}
local plotName = nil

-- Notificação
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

-- Drag
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

-- Painel
local frame = Instance.new("Frame")
frame.Size = UDim2.new(0, 180, 0, 210)
frame.Position = UDim2.new(0, 20, 0.5, -105)
frame.BackgroundColor3 = Color3.fromRGB(30, 30, 30)
frame.BorderSizePixel = 0
frame.Visible = true
frame.Active = true
frame.Parent = screenGui
Instance.new("UICorner", frame).CornerRadius = UDim.new(0, 10)

-- Botão Toggle
local toggleButton = Instance.new("TextButton")
toggleButton.Size = UDim2.new(0, 40, 0, 40)
toggleButton.Position = UDim2.new(0, 10, 0, 10)
toggleButton.BackgroundColor3 = Color3.fromRGB(0, 0, 0)
toggleButton.Text = "≡"
toggleButton.TextColor3 = Color3.fromRGB(255, 0, 0)
toggleButton.Font = Enum.Font.GothamBold
toggleButton.TextSize = 24
toggleButton.Active = true
toggleButton.Visible = true
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
local humanoid
local speedEnabled = true
local speedValue = 40

local function getRootPart()
	return player.Character and player.Character:FindFirstChild("HumanoidRootPart") or nil
end

local function atualizarVelocidade()
	if humanoid then
		humanoid.WalkSpeed = speedEnabled and speedValue or 16
	end
end

local function ativarGodMode()
	local character = player.Character
	if not character then return end
	local humanoid = character:FindFirstChildOfClass("Humanoid")
	if not humanoid then return end
	humanoid.HealthChanged:Connect(function(health)
		if health < 1 then
			humanoid.Health = humanoid.MaxHealth
		end
	end)
end

local function onCharacterAdded(character)
	humanoid = character:WaitForChild("Humanoid")
	atualizarVelocidade()
	humanoid:GetPropertyChangedSignal("WalkSpeed"):Connect(atualizarVelocidade)
	UIS.JumpRequest:Connect(function()
		if humanoid then
			humanoid:ChangeState(Enum.HumanoidStateType.Jumping)
		end
	end)
	ativarGodMode()
end

player.CharacterAdded:Connect(onCharacterAdded)
if player.Character then onCharacterAdded(player.Character) end

-- Criar botão
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
	return btn
end

-- Velocidade
criarBotao(frame, "⚡ Velocidade", 20, function()
	speedEnabled = not speedEnabled
	atualizarVelocidade()
	showNotification(speedEnabled and "Velocidade ativada!" or "Velocidade desativada!")
end)

-- ESP LockTime
criarBotao(frame, "🔒 ESP LockTime", 60, function()
	activeLockTimeEsp = not activeLockTimeEsp
	if not activeLockTimeEsp then
		for _, gui in pairs(lteInstances) do if gui then gui:Destroy() end end
		lteInstances = {}
		showNotification("ESP LockTime desativado")
	else
		showNotification("ESP LockTime ativado")
	end
end)

-- Botão Pulo Infinito
criarBotao(frame, "🦘 Pulo Infinito", 100, function()
	showNotification("Pulo Infinito já está ativo!")
end)

-- Anti-Lag e Delay (Ativado automaticamente)
local function ativarAntiLag()
	Lighting.GlobalShadows = false
	Lighting.FogEnd = 1000000
	settings().Rendering.QualityLevel = Enum.QualityLevel.Level01

	for _, obj in pairs(workspace:GetDescendants()) do
		if obj:IsA("Decal") or obj:IsA("Texture") then
			obj:Destroy()
		elseif obj:IsA("ParticleEmitter") or obj:IsA("Trail") then
			obj.Enabled = false
		elseif obj:IsA("BasePart") then
			obj.CastShadow = false
		end
	end
	showNotification("Anti-Lag ativado automaticamente!")
end

-- Chamar anti-lag no início
ativarAntiLag()

-- ESP LockTime atualizado sempre
RunService.Heartbeat:Connect(function()
	if not activeLockTimeEsp then return end

	for _, plot in pairs(workspace.Plots:GetChildren()) do
		local billboardName = "LockTimeESP" .. plot.Name
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
		end
	end
end)
