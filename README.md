-- Serviços
local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local Debris = game:GetService("Debris")
local Lighting = game:GetService("Lighting")
local UIS = game:GetService("UserInputService")
local TweenService = game:GetService("TweenService")

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

-- ==== Código do TP Delivery, corrigido para não sumir após morte ====

local random = Random.new()
local void = CFrame.new(0, -3.4028234663852886e+38, 0)

local function TP(position)
	local hrp = player.Character and player.Character:FindFirstChild("HumanoidRootPart")
	if hrp and typeof(position) == "CFrame" then
		hrp.CFrame = position + Vector3.new(
			random:NextNumber(-0.0001, 0.0001),
			random:NextNumber(-0.0001, 0.0001),
			random:NextNumber(-0.0001, 0.0001)
		)
	end
	RunService.Heartbeat:Wait()
end

local function FindDelivery()
	local plots = workspace:FindFirstChild("Plots")
	if not plots then return nil end
	for _, plot in pairs(plots:GetChildren()) do
		local sign = plot:FindFirstChild("PlotSign")
		if sign then
			local yourBase = sign:FindFirstChild("YourBase")
			if yourBase and yourBase.Enabled then
				local hitbox = plot:FindFirstChild("DeliveryHitbox")
				if hitbox then return hitbox end
			end
		end
	end
	return nil
end

local function AindaComPet()
	local char = player.Character
	if not char then return false end
	for _, obj in pairs(char:GetChildren()) do
		if obj:IsA("Tool") and obj.Name:lower():find("pet") then
			return true
		end
	end
	return false
end

local function ProtegerComForceField()
	local char = player.Character
	if char and not char:FindFirstChildOfClass("ForceField") then
		local ff = Instance.new("ForceField", char)
		ff.Visible = false
	end
end

local function TeleportToDeliveryBox()
	local hitbox = FindDelivery()
	if not hitbox then
		warn("DeliveryHitbox not found")
		return false, nil
	end

	local hrp = player.Character and player.Character:FindFirstChild("HumanoidRootPart")
	if not hrp then return false, hitbox end

	local flyTarget = hitbox.Position + Vector3.new(0, 8, 0)
	local flyDistance = (hrp.Position - flyTarget).Magnitude
	local flyTime = math.clamp(flyDistance / 40, 1.5, 4)

	local tweenInfo = TweenInfo.new(flyTime, Enum.EasingStyle.Sine, Enum.EasingDirection.Out)
	local tween = TweenService:Create(hrp, tweenInfo, { CFrame = CFrame.new(flyTarget) })
	tween:Play()
	tween.Completed:Wait()

	task.wait(0.2)

	for i = 1, 3 do
		TP(hitbox.CFrame * CFrame.new(0, -3, 0))
	end

	task.wait(0.2)

	local distance = (hrp.Position - hitbox.Position).Magnitude
	if distance <= 30 then
		return true, hitbox
	else
		return false, hitbox
	end
end

-- Função para criar o GUI do botão "ROUBAR PET"
local function createTPDeliveryGui()
	-- Remove GUI antiga se existir
	local oldGui = player.PlayerGui:FindFirstChild("TPDeliveryOnly")
	if oldGui then
		oldGui:Destroy()
	end

	local ScreenGui2 = Instance.new("ScreenGui")
	ScreenGui2.Name = "TPDeliveryOnly"
	ScreenGui2.ResetOnSpawn = false
	ScreenGui2.Parent = player.PlayerGui

	local Button = Instance.new("TextButton", ScreenGui2)
	Button.Size = UDim2.new(0, 180, 0, 50)
	Button.Position = UDim2.new(0, 30, 0, 5) -- posição mais pra cima para não sobrepor o painel
	Button.BackgroundColor3 = Color3.fromRGB(255, 0, 0)
	Button.TextColor3 = Color3.fromRGB(255, 255, 255)
	Button.Font = Enum.Font.GothamBold
	Button.TextSize = 18
	Button.Text = "ROUBAR PET"

	local UICorner = Instance.new("UICorner", Button)
	UICorner.CornerRadius = UDim.new(0, 12)

	local Stroke = Instance.new("UIStroke", Button)
	Stroke.Color = Color3.fromRGB(0, 80, 200)
	Stroke.Thickness = 2

	local spamming = false
	local spamThread

	Button.MouseButton1Click:Connect(function()
		spamming = not spamming
		if spamming then
			Button.Text = "PARAR"
			spamThread = task.spawn(function()
				while spamming do
					local comPet = AindaComPet()
					local success, hitbox = TeleportToDeliveryBox()

					if comPet or (success and AindaComPet()) then
						print("Pet detectado na mão. Iniciando entrega...")
						ProtegerComForceField()

						local start = tick()
						while AindaComPet() and tick() - start < 3 do
							local delivery = FindDelivery()
							if delivery then
								TP(delivery.CFrame * CFrame.new(0, -3, 0))
							end
							task.wait(0.05)
						end

						if not AindaComPet() then
							print("Pet entregue com sucesso!")
							spamming = false
							Button.Text = "ROUBAR PET"
							break
						else
							print("Entrega falhou. Retentando...")
						end
					else
						print("Tentando pegar pet...")
					end

					task.wait(0.05)
				end
			end)
		else
			Button.Text = "ROUBAR PET"
		end
	end)

	-- Efeito rainbow do botão
	spawn(function()
		local hue = 0
		while true do
			hue = (hue + 0.005) % 1
			local color = Color3.fromHSV(hue, 1, 1)
			Button.BackgroundColor3 = color
			task.wait(0.03)
		end
	end)

	return ScreenGui2, Button
end

-- Cria o GUI do botão "ROUBAR PET" na inicialização
createTPDeliveryGui()

-- Recria o GUI toda vez que o personagem spawna para garantir visibilidade
player.CharacterAdded:Connect(function()
	task.wait(1) -- espera PlayerGui estabilizar
	createTPDeliveryGui()
end)
