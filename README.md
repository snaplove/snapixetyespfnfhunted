--[[
	SNAP ESP V3 - por [Seu Nome]
	-------------------------------------------------------------
	FUNCIONALIDADES:
	- Pressione 'K' para abrir/fechar a interface com uma animação suave.
	- Interface de usuário arrastável pela barra de título.
	- Lista de jogadores com scroll funcional e nomes completos (Apelido e @Nick).
	- ESP individual para cada jogador, com padrão "OFF".
	- Botões "ON Todos" e "OFF Todos" para controle rápido.
	- Desenho do ESP preciso usando GetBoundingBox(), visível através de paredes.
]]

-- Serviços
local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local UserInputService = game:GetService("UserInputService")
local Workspace = game:GetService("Workspace")
local TweenService = game:GetService("TweenService")

-- Jogador local e Câmera
local localPlayer = Players.LocalPlayer
local camera = Workspace.CurrentCamera

-- ================== CONFIGURAÇÃO ==================
local CONFIG = {
	TOGGLE_UI_KEY = Enum.KeyCode.K,
	BOX_COLOR = Color3.fromRGB(0, 255, 127),
	THICKNESS = 2
}
-- ================================================

-- Variáveis de Estado
local isUiVisible = false
local espTargets = {}
local espDrawings = {}

-- ===================================================================
-- 1. CRIAÇÃO DA INTERFACE GRÁFICA (GUI)
-- ===================================================================
local screenGui = Instance.new("ScreenGui")
screenGui.Name = "ESP_ControlPanel_GUI"
screenGui.ResetOnSpawn = false

local mainFrame = Instance.new("Frame")
mainFrame.Name = "MainFrame"
mainFrame.Size = UDim2.new(0, 250, 0, 400)
mainFrame.BackgroundColor3 = Color3.fromRGB(35, 37, 40)
mainFrame.BorderSizePixel = 0
mainFrame.Visible = isUiVisible
mainFrame.ClipsDescendants = true
mainFrame.Parent = screenGui

local corner = Instance.new("UICorner", mainFrame)
corner.CornerRadius = UDim.new(0, 8)

local titleLabel = Instance.new("TextLabel")
titleLabel.Name = "Title"
titleLabel.Size = UDim2.new(1, 0, 0, 40)
titleLabel.Text = "SNAP ESP V3"
titleLabel.Font = Enum.Font.SourceSansBold
titleLabel.TextSize = 18
titleLabel.TextColor3 = Color3.fromRGB(255, 255, 255)
titleLabel.BackgroundColor3 = Color3.fromRGB(25, 26, 28)
titleLabel.Parent = mainFrame
local titleCorner = Instance.new("UICorner", titleLabel)
titleCorner.CornerRadius = UDim.new(0, 8)

local toggleAllOnButton = Instance.new("TextButton")
toggleAllOnButton.Name = "ToggleAllOn"; toggleAllOnButton.Size = UDim2.new(0, 30, 0, 30); toggleAllOnButton.Position = UDim2.new(1, -110, 0, 5)
toggleAllOnButton.Text = "ON"; toggleAllOnButton.Font = Enum.Font.SourceSansBold; toggleAllOnButton.TextSize = 14
toggleAllOnButton.TextColor3 = Color3.fromRGB(255, 255, 255); toggleAllOnButton.BackgroundColor3 = Color3.fromRGB(70, 180, 80)
toggleAllOnButton.Parent = titleLabel; Instance.new("UICorner", toggleAllOnButton)

local toggleAllOffButton = Instance.new("TextButton")
toggleAllOffButton.Name = "ToggleAllOff"; toggleAllOffButton.Size = UDim2.new(0, 30, 0, 30); toggleAllOffButton.Position = UDim2.new(1, -75, 0, 5)
toggleAllOffButton.Text = "OFF"; toggleAllOffButton.Font = Enum.Font.SourceSansBold; toggleAllOffButton.TextSize = 14
toggleAllOffButton.TextColor3 = Color3.fromRGB(255, 255, 255); toggleAllOffButton.BackgroundColor3 = Color3.fromRGB(180, 70, 70)
toggleAllOffButton.Parent = titleLabel; Instance.new("UICorner", toggleAllOffButton)

local refreshButton = Instance.new("TextButton")
refreshButton.Name = "RefreshButton"; refreshButton.Size = UDim2.new(0, 30, 0, 30); refreshButton.Position = UDim2.new(1, -35, 0, 5)
refreshButton.Text = "R"; refreshButton.Font = Enum.Font.SourceSansBold; refreshButton.TextSize = 18
refreshButton.TextColor3 = Color3.fromRGB(255, 255, 255); refreshButton.BackgroundColor3 = Color3.fromRGB(50, 52, 56)
refreshButton.Parent = titleLabel; Instance.new("UICorner", refreshButton)

local scrollingFrame = Instance.new("ScrollingFrame")
scrollingFrame.Name = "PlayerList"; scrollingFrame.Size = UDim2.new(1, -10, 1, -45); scrollingFrame.Position = UDim2.new(0, 5, 0, 40)
scrollingFrame.BackgroundColor3 = mainFrame.BackgroundColor3; scrollingFrame.BorderSizePixel = 0
scrollingFrame.CanvasSize = UDim2.new(0, 0, 0, 0); scrollingFrame.ScrollBarImageColor3 = Color3.fromRGB(255, 255, 255)
scrollingFrame.ScrollBarThickness = 5; scrollingFrame.Parent = mainFrame

local uiListLayout = Instance.new("UIListLayout", scrollingFrame)
uiListLayout.Padding = UDim.new(0, 5); uiListLayout.SortOrder = Enum.SortOrder.LayoutOrder

local playerTemplate = Instance.new("Frame")
playerTemplate.Name = "PlayerTemplate"; playerTemplate.Size = UDim2.new(1, 0, 0, 45)
playerTemplate.BackgroundColor3 = Color3.fromRGB(50, 52, 56); playerTemplate.BorderSizePixel = 0
Instance.new("UICorner", playerTemplate)

local playerIcon = Instance.new("ImageLabel", playerTemplate)
playerIcon.Name = "Icon"; playerIcon.Size = UDim2.new(0, 35, 0, 35); playerIcon.Position = UDim2.new(0, 5, 0.5, -17.5)
playerIcon.BackgroundTransparency = 1; Instance.new("UICorner", playerIcon)

local playerNameLabel = Instance.new("TextLabel", playerTemplate)
playerNameLabel.Name = "PlayerName"; playerNameLabel.Size = UDim2.new(1, -95, 1, 0); playerNameLabel.Position = UDim2.new(0, 45, 0, 0)
playerNameLabel.Text = "Nickname"; playerNameLabel.Font = Enum.Font.SourceSans; playerNameLabel.TextSize = 16
playerNameLabel.TextColor3 = Color3.fromRGB(255, 255, 255); playerNameLabel.TextXAlignment = Enum.TextXAlignment.Left
playerNameLabel.BackgroundTransparency = 1

local espToggleButton = Instance.new("TextButton", playerTemplate)
espToggleButton.Name = "ESPToggle"; espToggleButton.Size = UDim2.new(0, 40, 0, 25); espToggleButton.Position = UDim2.new(1, -45, 0.5, -12.5)
espToggleButton.Font = Enum.Font.SourceSansBold; espToggleButton.TextSize = 14
Instance.new("UICorner", espToggleButton)

screenGui.Parent = localPlayer:WaitForChild("PlayerGui")

-- ===================================================================
-- 2. LÓGICA DA INTERFACE E DOS JOGADORES
-- ===================================================================
local function updateToggleButton(button, isEnabled)
	if isEnabled then button.Text = "ON"; button.BackgroundColor3 = Color3.fromRGB(70, 180, 80)
	else button.Text = "OFF"; button.BackgroundColor3 = Color3.fromRGB(180, 70, 70) end
end

local function populatePlayerList()
	for _, child in ipairs(scrollingFrame:GetChildren()) do if child:IsA("Frame") then child:Destroy() end end
	local playerCount = 0
	for _, player in ipairs(Players:GetPlayers()) do
		if player == localPlayer then continue end
		playerCount = playerCount + 1
		local playerFrame = playerTemplate:Clone()
		playerFrame.Name = player.Name
		playerFrame.PlayerName.Text = string.format("%s (%s)", player.DisplayName, player.Name)
		if #playerFrame.PlayerName.Text > 20 then playerFrame.PlayerName.TextSize = 13 end
		local content, isReady = Players:GetUserThumbnailAsync(player.UserId, Enum.ThumbnailType.HeadShot, Enum.ThumbnailSize.Size48x48)
		if isReady then playerFrame.Icon.Image = content end
		if espTargets[player] == nil then espTargets[player] = false end
		updateToggleButton(playerFrame.ESPToggle, espTargets[player])
		playerFrame.ESPToggle.MouseButton1Click:Connect(function()
			espTargets[player] = not espTargets[player]
			updateToggleButton(playerFrame.ESPToggle, espTargets[player])
		end)
		playerFrame.Parent = scrollingFrame
	end
	local itemHeight = playerTemplate.Size.Y.Offset; local padding = uiListLayout.Padding.Offset
	scrollingFrame.CanvasSize = UDim2.fromOffset(0, (itemHeight * playerCount) + (padding * (playerCount - 1)))
end

-- ===================================================================
-- 3. LÓGICA DO ESP (DESENHO)
-- ===================================================================
local function getOrCreateDrawings(player)
	if espDrawings[player] then return espDrawings[player] end
	local newDrawings = { Top = Drawing.new("Line"), Bottom = Drawing.new("Line"), Left = Drawing.new("Line"), Right = Drawing.new("Line") }
	for _, line in pairs(newDrawings) do
		line.Color = CONFIG.BOX_COLOR; line.Thickness = CONFIG.THICKNESS; line.ZIndex = 2; line.Visible = false
	end
	espDrawings[player] = newDrawings; return newDrawings
end

local function hideDrawings(playerDrawings)
	if not playerDrawings then return end
	for _, line in pairs(playerDrawings) do line.Visible = false end
end

local function updateEsp()
	for _, drawings in pairs(espDrawings) do hideDrawings(drawings) end
	for player, isEnabled in pairs(espTargets) do
		if not isEnabled then continue end
		local character = player.Character
		if not (character and character.PrimaryPart and player.Parent) then continue end
		local humanoid = character:FindFirstChildOfClass("Humanoid")
		if not (humanoid and humanoid.Health > 0) then continue end
		local cframe, size = character:GetBoundingBox()
		local corners, minX, maxX, minY, maxY = {}, math.huge, -math.huge, math.huge, -math.huge
		local pointsOnScreen = 0; local halfSize = size / 2
		for x = -1, 1, 2 do for y = -1, 1, 2 do for z = -1, 1, 2 do table.insert(corners, cframe * Vector3.new(halfSize.X * x, halfSize.Y * y, halfSize.Z * z)) end end end
		for _, pos3D in ipairs(corners) do
			local pos2D, onScreen = camera:WorldToScreenPoint(pos3D)
			if onScreen and pos2D.Z > 0 then
				pointsOnScreen = pointsOnScreen + 1; minX = math.min(minX, pos2D.X); maxX = math.max(maxX, pos2D.X)
				minY = math.min(minY, pos2D.Y); maxY = math.max(maxY, pos2D.Y)
			end
		end
		if pointsOnScreen > 0 then
			local lines = getOrCreateDrawings(player)
			local topLeft = Vector2.new(minX, minY); local boxWidth = maxX - minX; local boxHeight = maxY - minY
			lines.Top.From = topLeft; lines.Top.To = Vector2.new(topLeft.X + boxWidth, topLeft.Y)
			lines.Bottom.From = Vector2.new(topLeft.X, topLeft.Y + boxHeight); lines.Bottom.To = Vector2.new(topLeft.X + boxWidth, topLeft.Y + boxHeight)
			lines.Left.From = topLeft; lines.Left.To = Vector2.new(topLeft.X, topLeft.Y + boxHeight)
			lines.Right.From = Vector2.new(topLeft.X + boxWidth, topLeft.Y); lines.Right.To = Vector2.new(topLeft.X + boxWidth, topLeft.Y + boxHeight)
			for _, line in pairs(lines) do line.Visible = true end
		end
	end
end

-- ===================================================================
-- 4. LÓGICA PARA ARRASTAR A JANELA
-- ===================================================================
local function makeDraggable(guiObject, dragHandle)
	local dragging = false; local dragStart = nil; local startPos = nil
	dragHandle.InputBegan:Connect(function(input)
		if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then
			dragging = true; dragStart = input.Position; startPos = guiObject.Position
			input.Changed:Connect(function() if input.UserInputState == Enum.UserInputState.End then dragging = false end end)
		end
	end)
	UserInputService.InputChanged:Connect(function(input)
		if (input.UserInputType == Enum.UserInputType.MouseMovement or input.UserInputType == Enum.UserInputType.Touch) and dragging then
			local delta = input.Position - dragStart
			guiObject.Position = UDim2.new(startPos.X.Scale, startPos.X.Offset + delta.X, startPos.Y.Scale, startPos.Y.Offset + delta.Y)
		end
	end)
end
makeDraggable(mainFrame, titleLabel)

-- ===================================================================
-- 5. CONEXÕES DE EVENTOS
-- ===================================================================
local tweenInfo = TweenInfo.new(0.3, Enum.EasingStyle.Quint, Enum.EasingDirection.Out)
local positionClosed = UDim2.new(-0.5, 0, 0.5, 0); local positionOpen = UDim2.new(0, 100, 0.5, 0)
mainFrame.Position = positionClosed; mainFrame.AnchorPoint = Vector2.new(0, 0.5)

UserInputService.InputBegan:Connect(function(input, gameProcessed)
	if gameProcessed or input.KeyCode ~= CONFIG.TOGGLE_UI_KEY then return end
	isUiVisible = not isUiVisible
	local targetPosition = isUiVisible and positionOpen or positionClosed
	local tween = TweenService:Create(mainFrame, tweenInfo, { Position = targetPosition })
	if isUiVisible then mainFrame.Visible = true; populatePlayerList() end
	tween:Play()
	tween.Completed:Connect(function() if not isUiVisible then mainFrame.Visible = false end end)
end)

toggleAllOnButton.MouseButton1Click:Connect(function()
	for _, player in ipairs(Players:GetPlayers()) do if player ~= localPlayer then espTargets[player] = true end end
	populatePlayerList()
end)

toggleAllOffButton.MouseButton1Click:Connect(function()
	for _, player in ipairs(Players:GetPlayers()) do if player ~= localPlayer then espTargets[player] = false end end
	populatePlayerList()
end)

refreshButton.MouseButton1Click:Connect(populatePlayerList)

Players.PlayerAdded:Connect(function(player)
	task.wait(1)
	if mainFrame.Visible then populatePlayerList() end
end)

Players.PlayerRemoving:Connect(function(player)
	if espTargets[player] then espTargets[player] = nil end
	if espDrawings[player] then for _, line in pairs(espDrawings[player]) do line:Remove() end; espDrawings[player] = nil end
	if mainFrame.Visible then populatePlayerList() end
end)

RunService.RenderStepped:Connect(updateEsp)
populatePlayerList()
