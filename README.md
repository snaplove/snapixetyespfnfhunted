--[[
	SNAP ESP V6 - por [Seu Nome]
	-------------------------------------------------------------
	NOVIDADES (V6):
	- INTERFACE RESPONSIVA: A UI agora se adapta a diferentes resoluções de tela.
	- Usa Scale + UIAspectRatioConstraint para manter a forma.
	- Usa UISizeConstraint para limitar o tamanho máximo e mínimo (não fica gigante).
	- Textos agora usam TextScaled para se ajustarem perfeitamente.
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
local lastUIPosition

-- ===================================================================
-- 1. CRIAÇÃO DA INTERFACE GRÁFICA (GUI) - VERSÃO RESPONSIVA
-- ===================================================================
local COLORS = { Background = Color3.fromRGB(40, 42, 46), Item = Color3.fromRGB(54, 57, 63), Green = Color3.fromRGB(88, 207, 102), Red = Color3.fromRGB(224, 80, 80), Gray = Color3.fromRGB(80, 82, 86), Text = Color3.fromRGB(255, 255, 255) }
local screenGui = Instance.new("ScreenGui"); screenGui.Name = "ESP_ControlPanel_GUI"; screenGui.ResetOnSpawn = false

local mainFrame = Instance.new("Frame")
mainFrame.Name = "MainFrame"
mainFrame.AnchorPoint = Vector2.new(0.5, 0.5)
lastUIPosition = UDim2.new(0.5, 0, 0.5, 0)
mainFrame.Position = lastUIPosition
mainFrame.BackgroundColor3 = COLORS.Background
mainFrame.BorderSizePixel = 0
mainFrame.Visible = isUiVisible
mainFrame.ClipsDescendants = true
mainFrame.Parent = screenGui
Instance.new("UICorner", mainFrame).CornerRadius = UDim.new(0, 10)

-- ------------------- MUDANÇA PRINCIPAL: LÓGICA DE TAMANHO RESPONSIVO -------------------
-- Tamanho base em Scale (45% da altura da tela)
mainFrame.Size = UDim2.fromScale(0, 0.45) 

-- Força a proporção (280 de largura / 420 de altura = 0.667)
local aspectRatio = Instance.new("UIAspectRatioConstraint", mainFrame)
aspectRatio.AspectRatio = 280 / 420
aspectRatio.DominantAxis = Enum.DominantAxis.Height

-- Limita o tamanho máximo e mínimo em pixels
local sizeConstraint = Instance.new("UISizeConstraint", mainFrame)
sizeConstraint.MinSize = Vector2.new(240, 360) -- Tamanho mínimo
sizeConstraint.MaxSize = Vector2.new(350, 525) -- Tamanho máximo
-- ------------------------------------------------------------------------------------

local titleContainer = Instance.new("Frame", mainFrame)
titleContainer.Name = "TitleContainer"; titleContainer.Size = UDim2.fromScale(1, 0.12); titleContainer.BackgroundTransparency = 1
local titleLayout = Instance.new("UIListLayout", titleContainer); titleLayout.FillDirection = Enum.FillDirection.Horizontal; titleLayout.VerticalAlignment = Enum.VerticalAlignment.Center; titleLayout.SortOrder = Enum.SortOrder.LayoutOrder; titleLayout.Padding = UDim.new(0, 8)
local titlePadding = Instance.new("UIPadding", titleContainer); titlePadding.PaddingLeft = UDim.new(0, 15); titlePadding.PaddingRight = UDim.new(0, 15)

local titleLabel = Instance.new("TextLabel", titleContainer)
titleLabel.Name = "Title"; titleLabel.Size = UDim2.new(1, -145, 1, 0); titleLabel.Text = "SNAP ESP"
titleLabel.Font = Enum.Font.GothamBold; titleLabel.TextColor3 = COLORS.Text; titleLabel.TextXAlignment = Enum.TextXAlignment.Left
titleLabel.BackgroundTransparency = 1; titleLabel.LayoutOrder = 1; titleLabel.TextScaled = true -- Texto escalável

local buttonSize = UDim2.new(0, 40, 0, 32)
local toggleAllOnButton = Instance.new("TextButton", titleContainer)
toggleAllOnButton.Name = "ToggleAllOn"; toggleAllOnButton.Size = buttonSize; toggleAllOnButton.Text = "ON"; toggleAllOnButton.Font = Enum.Font.GothamBold
toggleAllOnButton.TextColor3 = COLORS.Text; toggleAllOnButton.BackgroundColor3 = COLORS.Green; toggleAllOnButton.LayoutOrder = 2; toggleAllOnButton.TextScaled = true
Instance.new("UICorner", toggleAllOnButton).CornerRadius = UDim.new(0, 8)

local toggleAllOffButton = Instance.new("TextButton", titleContainer)
toggleAllOffButton.Name = "ToggleAllOff"; toggleAllOffButton.Size = buttonSize; toggleAllOffButton.Text = "OFF"; toggleAllOffButton.Font = Enum.Font.GothamBold
toggleAllOffButton.TextColor3 = COLORS.Text; toggleAllOffButton.BackgroundColor3 = COLORS.Red; toggleAllOffButton.LayoutOrder = 3; toggleAllOffButton.TextScaled = true
Instance.new("UICorner", toggleAllOffButton).CornerRadius = UDim.new(0, 8)

local refreshButton = Instance.new("TextButton", titleContainer)
refreshButton.Name = "RefreshButton"; refreshButton.Size = buttonSize; refreshButton.Text = "R"; refreshButton.Font = Enum.Font.GothamBold
refreshButton.TextColor3 = COLORS.Text; refreshButton.BackgroundColor3 = COLORS.Gray; refreshButton.LayoutOrder = 4; refreshButton.TextScaled = true
Instance.new("UICorner", refreshButton).CornerRadius = UDim.new(0, 8)

local scrollingFrame = Instance.new("ScrollingFrame", mainFrame)
scrollingFrame.Name = "PlayerList"; scrollingFrame.Size = UDim2.new(1, 0, 0.88, 0); scrollingFrame.Position = UDim2.fromScale(0, 0.12)
scrollingFrame.BackgroundColor3 = COLORS.Background; scrollingFrame.BorderSizePixel = 0; scrollingFrame.CanvasSize = UDim2.new(0, 0, 0, 0)
scrollingFrame.ScrollBarImageColor3 = Color3.fromRGB(200, 200, 200); scrollingFrame.ScrollBarThickness = 6
local listPadding = Instance.new("UIPadding", scrollingFrame); listPadding.PaddingLeft = UDim.new(0, 15); listPadding.PaddingRight = UDim.new(0, 15); listPadding.PaddingTop = UDim.new(0, 15); listPadding.PaddingBottom = UDim.new(0, 15)
local uiListLayout = Instance.new("UIListLayout", scrollingFrame); uiListLayout.Padding = UDim.new(0, 8); uiListLayout.SortOrder = Enum.SortOrder.LayoutOrder

local playerTemplate = Instance.new("Frame"); playerTemplate.Name = "PlayerTemplate"; playerTemplate.Size = UDim2.new(1, 0, 0, 50)
playerTemplate.BackgroundColor3 = COLORS.Item; playerTemplate.BorderSizePixel = 0; Instance.new("UICorner", playerTemplate).CornerRadius = UDim.new(0, 8)
local playerIcon = Instance.new("ImageLabel", playerTemplate); playerIcon.Name = "Icon"; playerIcon.Size = UDim2.new(0, 40, 0, 40); playerIcon.Position = UDim2.new(0, 5, 0.5, -20); playerIcon.BackgroundTransparency = 1; Instance.new("UICorner", playerIcon).CornerRadius = UDim.new(1, 0)
local playerNameLabel = Instance.new("TextLabel", playerTemplate); playerNameLabel.Name = "PlayerName"; playerNameLabel.Size = UDim2.new(1, -110, 1, 0); playerNameLabel.Position = UDim2.new(0, 55, 0, 0); playerNameLabel.Font = Enum.Font.Gotham; playerNameLabel.TextColor3 = COLORS.Text; playerNameLabel.TextXAlignment = Enum.TextXAlignment.Left; playerNameLabel.BackgroundTransparency = 1; playerNameLabel.TextScaled = true
local espToggleButton = Instance.new("TextButton", playerTemplate); espToggleButton.Name = "ESPToggle"; espToggleButton.Size = UDim2.new(0, 45, 0, 30); espToggleButton.Position = UDim2.new(1, -50, 0.5, -15); espToggleButton.Font = Enum.Font.GothamBold; espToggleButton.TextScaled = true; Instance.new("UICorner", espToggleButton).CornerRadius = UDim.new(0, 8)
screenGui.Parent = localPlayer:WaitForChild("PlayerGui")

-- ===================================================================
-- 2. LÓGICA DA INTERFACE E DOS JOGADORES
-- ===================================================================
local function updateToggleButton(button, isEnabled) if isEnabled then button.Text = "ON"; button.BackgroundColor3 = COLORS.Green else button.Text = "OFF"; button.BackgroundColor3 = COLORS.Red end end
local function populatePlayerList()
	for _, child in ipairs(scrollingFrame:GetChildren()) do if child:IsA("Frame") then child:Destroy() end end
	local playerCount = 0
	for _, player in ipairs(Players:GetPlayers()) do
		if player == localPlayer then continue end; playerCount = playerCount + 1
		local playerFrame = playerTemplate:Clone(); playerFrame.Name = player.Name
		playerFrame.PlayerName.Text = string.format("%s (%s)", player.DisplayName, player.Name)
		local content, isReady = Players:GetUserThumbnailAsync(player.UserId, Enum.ThumbnailType.HeadShot, Enum.ThumbnailSize.Size48x48); if isReady then playerFrame.Icon.Image = content end
		if espTargets[player] == nil then espTargets[player] = false end
		updateToggleButton(playerFrame.ESPToggle, espTargets[player])
		playerFrame.ESPToggle.MouseButton1Click:Connect(function() espTargets[player] = not espTargets[player]; updateToggleButton(playerFrame.ESPToggle, espTargets[player]) end)
		playerFrame.Parent = scrollingFrame
	end
	local itemHeight = playerTemplate.Size.Y.Offset; local padding = uiListLayout.Padding.Offset
	scrollingFrame.CanvasSize = UDim2.fromOffset(0, (itemHeight * playerCount) + (padding * (playerCount + 1)))
end

-- ===================================================================
-- 3. LÓGICA DO ESP (DESENHO)
-- ===================================================================
local function getOrCreateDrawings(player) if espDrawings[player] then return espDrawings[player] end; local newDrawings = { Top = Drawing.new("Line"), Bottom = Drawing.new("Line"), Left = Drawing.new("Line"), Right = Drawing.new("Line") }; for _, line in pairs(newDrawings) do line.Color = CONFIG.BOX_COLOR; line.Thickness = CONFIG.THICKNESS; line.ZIndex = 2; line.Visible = false end; espDrawings[player] = newDrawings; return newDrawings end
local function updateEsp() for _, drawings in pairs(espDrawings) do for _, line in pairs(drawings) do line.Visible = false end end; for player, isEnabled in pairs(espTargets) do if not isEnabled then continue end; local character = player.Character; if not (character and character.PrimaryPart and player.Parent) then continue end; local humanoid = character:FindFirstChildOfClass("Humanoid"); if not (humanoid and humanoid.Health > 0) then continue end; local cframe, size = character:GetBoundingBox(); local corners, minX, maxX, minY, maxY = {}, math.huge, -math.huge, math.huge, -math.huge; local pointsOnScreen = 0; local halfSize = size / 2; for x = -1, 1, 2 do for y = -1, 1, 2 do for z = -1, 1, 2 do table.insert(corners, cframe * Vector3.new(halfSize.X * x, halfSize.Y * y, halfSize.Z * z)) end end end; for _, pos3D in ipairs(corners) do local pos2D, onScreen = camera:WorldToScreenPoint(pos3D); if onScreen and pos2D.Z > 0 then pointsOnScreen = pointsOnScreen + 1; minX = math.min(minX, pos2D.X); maxX = math.max(maxX, pos2D.X); minY = math.min(minY, pos2D.Y); maxY = math.max(maxY, pos2D.Y) end end; if pointsOnScreen > 0 then local lines = getOrCreateDrawings(player); local topLeft = Vector2.new(minX, minY); local boxWidth = maxX - minX; local boxHeight = maxY - minY; lines.Top.From = topLeft; lines.Top.To = Vector2.new(topLeft.X + boxWidth, topLeft.Y); lines.Bottom.From = Vector2.new(topLeft.X, topLeft.Y + boxHeight); lines.Bottom.To = Vector2.new(topLeft.X + boxWidth, topLeft.Y + boxHeight); lines.Left.From = topLeft; lines.Left.To = Vector2.new(topLeft.X, topLeft.Y + boxHeight); lines.Right.From = Vector2.new(topLeft.X + boxWidth, topLeft.Y); lines.Right.To = Vector2.new(topLeft.X + boxWidth, topLeft.Y + boxHeight); for _, line in pairs(lines) do line.Visible = true end end end end

-- ===================================================================
-- 4. LÓGICA PARA ARRASTAR A JANELA
-- ===================================================================
local function makeDraggable(guiObject, dragHandle) local dragging = false; local dragStart = nil; local startPos = nil; dragHandle.InputBegan:Connect(function(input) if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then dragging = true; dragStart = input.Position; startPos = guiObject.Position; input.Changed:Connect(function() if input.UserInputState == Enum.UserInputState.End then dragging = false; lastUIPosition = guiObject.Position end end) end end); UserInputService.InputChanged:Connect(function(input) if (input.UserInputType == Enum.UserInputType.MouseMovement or input.UserInputType == Enum.UserInputType.Touch) and dragging then local delta = input.Position - dragStart; guiObject.Position = UDim2.new(startPos.X.Scale, startPos.X.Offset + delta.X, startPos.Y.Scale, startPos.Y.Offset + delta.Y) end end) end
makeDraggable(mainFrame, titleContainer)

-- ===================================================================
-- 5. CONEXÕES DE EVENTOS
-- ===================================================================
local tweenInfo = TweenInfo.new(0.3, Enum.EasingStyle.Back, Enum.EasingDirection.Out)
UserInputService.InputBegan:Connect(function(input, gameProcessed)
	if gameProcessed or input.KeyCode ~= CONFIG.TOGGLE_UI_KEY then return end; isUiVisible = not isUiVisible
	local targetSize = isUiVisible and UDim2.fromScale(1, 1) or UDim2.fromScale(0, 0)
	mainFrame.Size = UDim2.fromScale(0,0) -- Animação de tamanho agora usa Scale
	if isUiVisible then
		populatePlayerList()
		mainFrame.Position = lastUIPosition
		mainFrame.Visible = true
		local tween = TweenService:Create(mainFrame, tweenInfo, { Size = mainFrame.AbsoluteSize }) -- Truque para animar para o tamanho responsivo
		mainFrame:GetPropertyChangedSignal("AbsoluteSize"):Wait()
		tween:Play()
	else
		local tween = TweenService:Create(mainFrame, tweenInfo, { Size = UDim2.fromScale(0, 0) })
		tween:Play(); tween.Completed:Connect(function() mainFrame.Visible = false end)
	end
end)
toggleAllOnButton.MouseButton1Click:Connect(function() for _, player in ipairs(Players:GetPlayers()) do if player ~= localPlayer then espTargets[player] = true end end; populatePlayerList() end)
toggleAllOffButton.MouseButton1Click:Connect(function() for _, player in ipairs(Players:GetPlayers()) do if player ~= localPlayer then espTargets[player] = false end end; populatePlayerList() end)
refreshButton.MouseButton1Click:Connect(populatePlayerList)
Players.PlayerAdded:Connect(function(player) task.wait(1); if mainFrame.Visible then populatePlayerList() end end)
Players.PlayerRemoving:Connect(function(player) if espTargets[player] then espTargets[player] = nil end; if espDrawings[player] then for _, line in pairs(espDrawings[player]) do line:Remove() end; espDrawings[player] = nil end; if mainFrame.Visible then populatePlayerList() end end)
RunService.RenderStepped:Connect(updateEsp)
populatePlayerList()
