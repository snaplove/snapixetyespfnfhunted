--[[
	SNAP ESP - FINAL BUILD (Enxuta)
	-------------------------------------------------------------
	Autor: [Seu Nome] & Assistente AI
	Data: [Data Atual]

	DESCRIÇÃO:
	Uma ferramenta de ESP (Extra Sensory Perception) completa, estável e focada.
	Este build remove todas as funcionalidades secundárias para a máxima leveza
	e confiabilidade.

	CHANGELOG (Build Enxuta):
	- REMOVIDO: Sistema de notificações foi completamente retirado.
	- CÓDIGO OTIMIZADO: Script mais leve e com inicialização mais rápida.
	- ESTABILIDADE MÁXIMA: Focado exclusivamente na funcionalidade principal do
	  painel de controle e do ESP.
]]

-- ===================================================================
-- SERVIÇOS E VARIÁVEIS GLOBAIS
-- ===================================================================
local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local UserInputService = game:GetService("UserInputService")
local Workspace = game:GetService("Workspace")
local TweenService = game:GetService("TweenService")

local localPlayer = Players.LocalPlayer
local camera = Workspace.CurrentCamera

-- ===================================================================
-- CONFIGURAÇÃO PRINCIPAL
-- ===================================================================
local CONFIG = {
	TOGGLE_UI_KEY = Enum.KeyCode.K,
	BOX_COLOR = Color3.fromRGB(0, 230, 230),
	THICKNESS = 2
}

-- ===================================================================
-- VARIÁVEIS DE ESTADO
-- ===================================================================
local isUiVisible = false
local espTargets = {}
local espDrawings = {}
local lastUIPosition = UDim2.new(0.5, 0, 0.5, 0)

-- ===================================================================
-- 1. CRIAÇÃO DA INTERFACE GRÁFICA (GUI) - TEMA "NEBULA"
-- ===================================================================
local COLORS = { BackgroundStart = Color3.fromRGB(25, 25, 40), BackgroundEnd = Color3.fromRGB(45, 30, 60), Item = Color3.fromRGB(35, 35, 55), Stroke = Color3.fromRGB(120, 100, 160), Accent = Color3.fromRGB(0, 230, 230), Red = Color3.fromRGB(255, 80, 120), Text = Color3.fromRGB(255, 255, 255), SubText = Color3.fromRGB(180, 180, 200) }
local screenGui = Instance.new("ScreenGui"); screenGui.Name = "ESP_ControlPanel_GUI"; screenGui.ResetOnSpawn = false; screenGui.Parent = localPlayer:WaitForChild("PlayerGui")
local mainFrame = Instance.new("Frame"); mainFrame.Name = "MainFrame"; mainFrame.AnchorPoint = Vector2.new(0.5, 0.5); mainFrame.Position = lastUIPosition; mainFrame.BackgroundColor3 = COLORS.BackgroundEnd; mainFrame.BorderSizePixel = 0; mainFrame.Visible = isUiVisible; mainFrame.ClipsDescendants = true; mainFrame.Parent = screenGui
Instance.new("UICorner", mainFrame).CornerRadius = UDim.new(0, 8); local gradient = Instance.new("UIGradient", mainFrame); gradient.Color = ColorSequence.new({ColorSequenceKeypoint.new(0, COLORS.BackgroundStart), ColorSequenceKeypoint.new(1, COLORS.BackgroundEnd)}); gradient.Rotation = 90; Instance.new("UIStroke", mainFrame).Color = COLORS.Stroke
local BASE_SIZE_SCALE = 0.5; mainFrame.Size = UDim2.fromScale(0, BASE_SIZE_SCALE); local aspectRatio = Instance.new("UIAspectRatioConstraint", mainFrame); aspectRatio.AspectRatio = 280 / 450; aspectRatio.DominantAxis = Enum.DominantAxis.Height; local sizeConstraint = Instance.new("UISizeConstraint", mainFrame); sizeConstraint.MinSize = Vector2.new(260, 400); sizeConstraint.MaxSize = Vector2.new(350, 550)
local titleContainer = Instance.new("Frame", mainFrame); titleContainer.Name = "TitleContainer"; titleContainer.Size = UDim2.new(1, 0, 0, 50); titleContainer.BackgroundTransparency = 1
local titleLayout = Instance.new("UIListLayout", titleContainer); titleLayout.FillDirection = Enum.FillDirection.Horizontal; titleLayout.VerticalAlignment = Enum.VerticalAlignment.Center; titleLayout.SortOrder = Enum.SortOrder.LayoutOrder; titleLayout.Padding = UDim.new(0, 8)
local titlePadding = Instance.new("UIPadding", titleContainer); titlePadding.PaddingLeft = UDim.new(0, 15); titlePadding.PaddingRight = UDim.new(0, 15)
local titleLabel = Instance.new("TextLabel", titleContainer); titleLabel.Name = "Title"; titleLabel.Size = UDim2.new(1, -145, 1, 0); titleLabel.Text = "SNAP ESP"; titleLabel.Font = Enum.Font.SciFi; titleLabel.TextSize = 22; titleLabel.TextColor3 = COLORS.Text; titleLabel.TextXAlignment = Enum.TextXAlignment.Left; titleLabel.BackgroundTransparency = 1; titleLabel.LayoutOrder = 1
local buttonSize = UDim2.new(0, 40, 0, 32)
local toggleAllOnButton = Instance.new("TextButton", titleContainer); toggleAllOnButton.Name = "ToggleAllOn"; toggleAllOnButton.Size = buttonSize; toggleAllOnButton.Text = "ON"; toggleAllOnButton.Font = Enum.Font.SciFi; toggleAllOnButton.TextSize = 16; toggleAllOnButton.TextColor3 = COLORS.Text; toggleAllOnButton.BackgroundColor3 = COLORS.Accent; toggleAllOnButton.LayoutOrder = 2; Instance.new("UICorner", toggleAllOnButton).CornerRadius = UDim.new(0, 6); Instance.new("UIStroke", toggleAllOnButton).Color = COLORS.Stroke
local toggleAllOffButton = Instance.new("TextButton", titleContainer); toggleAllOffButton.Name = "ToggleAllOff"; toggleAllOffButton.Size = buttonSize; toggleAllOffButton.Text = "OFF"; toggleAllOffButton.Font = Enum.Font.SciFi; toggleAllOffButton.TextSize = 16; toggleAllOffButton.TextColor3 = COLORS.Text; toggleAllOffButton.BackgroundColor3 = COLORS.Red; toggleAllOffButton.LayoutOrder = 3; Instance.new("UICorner", toggleAllOffButton).CornerRadius = UDim.new(0, 6); Instance.new("UIStroke", toggleAllOffButton).Color = COLORS.Stroke
local refreshButton = Instance.new("TextButton", titleContainer); refreshButton.Name = "RefreshButton"; refreshButton.Size = buttonSize; refreshButton.Text = "R"; refreshButton.Font = Enum.Font.SciFi; refreshButton.TextSize = 20; refreshButton.TextColor3 = COLORS.Text; refreshButton.BackgroundColor3 = COLORS.Item; refreshButton.LayoutOrder = 4; Instance.new("UICorner", refreshButton).CornerRadius = UDim.new(0, 6); Instance.new("UIStroke", refreshButton).Color = COLORS.Stroke
local scrollingFrame = Instance.new("ScrollingFrame", mainFrame); scrollingFrame.Name = "PlayerList"; scrollingFrame.Size = UDim2.new(1, 0, 1, -50); scrollingFrame.Position = UDim2.new(0, 0, 0, 50); scrollingFrame.BackgroundTransparency = 1; scrollingFrame.BorderSizePixel = 0; scrollingFrame.CanvasSize = UDim2.new(0, 0, 0, 0); scrollingFrame.ScrollBarImageColor3 = COLORS.Accent; scrollingFrame.ScrollBarThickness = 4
local listPadding = Instance.new("UIPadding", scrollingFrame); listPadding.PaddingLeft = UDim.new(0, 15); listPadding.PaddingRight = UDim.new(0, 15); listPadding.PaddingTop = UDim.new(0, 10); listPadding.PaddingBottom = UDim.new(0, 10)
local uiListLayout = Instance.new("UIListLayout", scrollingFrame); uiListLayout.Padding = UDim.new(0, 8); uiListLayout.SortOrder = Enum.SortOrder.LayoutOrder
local playerTemplate = Instance.new("Frame"); playerTemplate.Name = "PlayerTemplate"; playerTemplate.Size = UDim2.new(1, 0, 0, 55); playerTemplate.BackgroundColor3 = COLORS.Item; playerTemplate.BorderSizePixel = 0; Instance.new("UICorner", playerTemplate).CornerRadius = UDim.new(0, 6); Instance.new("UIStroke", playerTemplate).Color = COLORS.Stroke
local itemLayout = Instance.new("UIListLayout", playerTemplate); itemLayout.FillDirection = Enum.FillDirection.Horizontal; itemLayout.VerticalAlignment = Enum.VerticalAlignment.Center; itemLayout.SortOrder = Enum.SortOrder.LayoutOrder; itemLayout.Padding = UDim.new(0, 10)
local itemPadding = Instance.new("UIPadding", playerTemplate); itemPadding.PaddingLeft = UDim.new(0, 8); itemPadding.PaddingRight = UDim.new(0, 8)
local playerIcon = Instance.new("ImageLabel", playerTemplate); playerIcon.Name = "Icon"; playerIcon.LayoutOrder = 1; playerIcon.Size = UDim2.new(0, 40, 0, 40); playerIcon.BackgroundTransparency = 1; Instance.new("UICorner", playerIcon).CornerRadius = UDim.new(1, 0); Instance.new("UIStroke", playerIcon).Color = COLORS.Stroke
local textContainer = Instance.new("Frame", playerTemplate); textContainer.Name = "TextContainer"; textContainer.LayoutOrder = 2; textContainer.Size = UDim2.new(1, -115, 1, 0); textContainer.BackgroundTransparency = 1
local textLayout = Instance.new("UIListLayout", textContainer); textLayout.FillDirection = Enum.FillDirection.Vertical; textLayout.HorizontalAlignment = Enum.HorizontalAlignment.Left; textLayout.SortOrder = Enum.SortOrder.LayoutOrder
local displayNameLabel = Instance.new("TextLabel", textContainer); displayNameLabel.Name = "DisplayName"; displayNameLabel.Size = UDim2.new(1, 0, 0, 18); displayNameLabel.Font = Enum.Font.SciFi; displayNameLabel.TextColor3 = COLORS.Text; displayNameLabel.TextXAlignment = Enum.TextXAlignment.Left; displayNameLabel.BackgroundTransparency = 1; displayNameLabel.TextScaled = true
local userNameLabel = Instance.new("TextLabel", textContainer); userNameLabel.Name = "UserName"; userNameLabel.Size = UDim2.new(1, 0, 0, 14); userNameLabel.Font = Enum.Font.SciFi; userNameLabel.TextColor3 = COLORS.SubText; userNameLabel.TextXAlignment = Enum.TextXAlignment.Left; userNameLabel.BackgroundTransparency = 1; userNameLabel.TextScaled = true
local espToggleButton = Instance.new("TextButton", playerTemplate); espToggleButton.Name = "ESPToggle"; espToggleButton.LayoutOrder = 3; espToggleButton.Size = UDim2.new(0, 45, 0, 30); espToggleButton.Font = Enum.Font.SciFi; espToggleButton.TextSize = 16; Instance.new("UICorner", espToggleButton).CornerRadius = UDim.new(0, 6); Instance.new("UIStroke", espToggleButton).Color = COLORS.Stroke

-- ===================================================================
-- 2. LÓGICA DA INTERFACE E DOS JOGADORES
-- ===================================================================
local function applyHoverEffect(button, baseColor, hoverColor) button.MouseEnter:Connect(function() TweenService:Create(button, TweenInfo.new(0.2), { BackgroundColor3 = hoverColor }):Play() end); button.MouseLeave:Connect(function() TweenService:Create(button, TweenInfo.new(0.2), { BackgroundColor3 = baseColor }):Play() end) end
local function updateToggleButton(button, isEnabled) local targetColor = isEnabled and COLORS.Accent or COLORS.Red; button.Text = isEnabled and "ON" or "OFF"; TweenService:Create(button, TweenInfo.new(0.15), { BackgroundColor3 = targetColor }):Play() end
local function applyDynamicHoverEffect(button, player) button.MouseEnter:Connect(function() local isEnabled = espTargets[player]; local hoverColor = (isEnabled and COLORS.Accent or COLORS.Red):Lerp(Color3.new(1, 1, 1), 0.3); TweenService:Create(button, TweenInfo.new(0.2), { BackgroundColor3 = hoverColor }):Play() end); button.MouseLeave:Connect(function() local isEnabled = espTargets[player]; local baseColor = isEnabled and COLORS.Accent or COLORS.Red; TweenService:Create(button, TweenInfo.new(0.2), { BackgroundColor3 = baseColor }):Play() end) end
local function populatePlayerList() for _, child in ipairs(scrollingFrame:GetChildren()) do if child:IsA("Frame") then child:Destroy() end end; local playerCount = 0; for _, player in ipairs(Players:GetPlayers()) do if player == localPlayer then continue end; playerCount = playerCount + 1; local playerFrame = playerTemplate:Clone(); playerFrame.Name = player.Name; playerFrame.TextContainer.DisplayName.Text = player.DisplayName; playerFrame.TextContainer.UserName.Text = "(@" .. player.Name .. ")"; local content, isReady = Players:GetUserThumbnailAsync(player.UserId, Enum.ThumbnailType.HeadShot, Enum.ThumbnailSize.Size48x48); if isReady then playerFrame.Icon.Image = content end; if espTargets[player] == nil then espTargets[player] = false end; local isEnabled = espTargets[player]; playerFrame.ESPToggle.Text = isEnabled and "ON" or "OFF"; playerFrame.ESPToggle.BackgroundColor3 = isEnabled and COLORS.Accent or COLORS.Red; playerFrame.ESPToggle.MouseButton1Click:Connect(function() espTargets[player] = not espTargets[player]; updateToggleButton(playerFrame.ESPToggle, espTargets[player]) end); applyDynamicHoverEffect(playerFrame.ESPToggle, player); playerFrame.Parent = scrollingFrame end; local itemHeight = playerTemplate.Size.Y.Offset; local padding = uiListLayout.Padding.Offset; scrollingFrame.CanvasSize = UDim2.fromOffset(0, (itemHeight * playerCount) + (padding * (playerCount + 1))) end

-- ===================================================================
-- 3. LÓGICA DO ESP (DESENHO) - VERSÃO FINAL (ROBUSTA E CENTRALIZADA)
-- ===================================================================
local function getOrCreateDrawings(player) if espDrawings[player] then return espDrawings[player] end; local newDrawings = { Top = Drawing.new("Line"), Bottom = Drawing.new("Line"), Left = Drawing.new("Line"), Right = Drawing.new("Line") }; for _, line in pairs(newDrawings) do line.Color = CONFIG.BOX_COLOR; line.Thickness = CONFIG.THICKNESS; line.ZIndex = 2; line.Visible = false end; espDrawings[player] = newDrawings; return newDrawings end

-- [!] FUNÇÃO FINAL: Usa PrimaryPart para máxima compatibilidade e centraliza no torso.
local function updateEsp()
	-- Esconde todas as caixas antes de redesenhar
	for _, drawings in pairs(espDrawings) do
		for _, line in pairs(drawings) do
			line.Visible = false
		end
	end

	-- Itera por todos os alvos do ESP
	for player, isEnabled in pairs(espTargets) do
		if not isEnabled then continue end

		local character = player.Character
		if not (character and player.Parent) then continue end

		-- [MUDANÇA 1 - CONFIABILIDADE]: Usamos 'PrimaryPart' e 'Humanoid'.
		-- Esta é a forma mais segura de verificar se um personagem é válido.
		local rootPart = character.PrimaryPart
		local humanoid = character:FindFirstChildOfClass("Humanoid")

		-- Se qualquer uma dessas partes essenciais não existir, ou se o jogador estiver morto, pulamos.
		if not (rootPart and humanoid and humanoid.Health > 0) then
			continue
		end

		-- [MUDANÇA 2 - CENTRALIZAÇÃO]: Calculamos o centro da caixa.
		-- A posição do rootPart (pélvis) + um pequeno deslocamento para cima (1 stud)
		-- move o centro da caixa para o meio do torso, que fica visualmente perfeito.
		local boxCenter3D = rootPart.Position + Vector3.new(0, 0, 0)

		-- Convertemos o centro 3D para a posição 2D na tela
		local boxCenter2D, onScreen = camera:WorldToScreenPoint(boxCenter3D)

		if onScreen then
			local lines = getOrCreateDrawings(player)

			-- [MUDANÇA 3 - TAMANHO]: O tamanho da caixa é baseado na distância.
			local distance = (camera.CFrame.Position - rootPart.Position).Magnitude
			
			-- Fórmula para o tamanho. Você pode ajustar o '3000' para maior/menor.
			local boxHeight = math.clamp(5000 / distance, 20, 300) 
			-- A largura é metade da altura, uma proporção ideal para personagens R6.
			local boxWidth = boxHeight / 2 

			-- Posição do canto superior esquerdo, calculado a partir do centro da caixa
			local topLeft = Vector2.new(boxCenter2D.X - boxWidth / 2, boxCenter2D.Y - boxHeight / 2)

			-- Desenha as 4 linhas da caixa
			lines.Top.From = topLeft
			lines.Top.To = Vector2.new(topLeft.X + boxWidth, topLeft.Y)

			lines.Bottom.From = Vector2.new(topLeft.X, topLeft.Y + boxHeight)
			lines.Bottom.To = Vector2.new(topLeft.X + boxWidth, topLeft.Y + boxHeight)

			lines.Left.From = topLeft
			lines.Left.To = Vector2.new(topLeft.X, topLeft.Y + boxHeight)

			lines.Right.From = Vector2.new(topLeft.X + boxWidth, topLeft.Y)
			lines.Right.To = Vector2.new(topLeft.X + boxWidth, topLeft.Y + boxHeight)

			-- Torna todas as linhas da caixa visíveis
			for _, line in pairs(lines) do
				line.Visible = true
			end
		end
	end
end

-- ===================================================================
-- 4. LÓGICA PARA ARRASTAR A JANELA
-- ===================================================================
local function makeDraggable(guiObject, dragHandle) local dragging = false; local dragStart = nil; local startPos = nil; dragHandle.InputBegan:Connect(function(input) if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then dragging = true; dragStart = input.Position; startPos = guiObject.Position; input.Changed:Connect(function() if input.UserInputState == Enum.UserInputState.End then dragging = false; lastUIPosition = guiObject.Position end end) end end); UserInputService.InputChanged:Connect(function(input) if (input.UserInputType == Enum.UserInputType.MouseMovement or input.UserInputType == Enum.UserInputType.Touch) and dragging then local delta = input.Position - dragStart; guiObject.Position = UDim2.new(startPos.X.Scale, startPos.X.Offset + delta.X, startPos.Y.Scale, startPos.Y.Offset + delta.Y) end end) end
makeDraggable(mainFrame, titleContainer)

-- ===================================================================
-- 5. CONEXÕES DE EVENTOS
-- ===================================================================
UserInputService.InputBegan:Connect(function(input, gameProcessed) if gameProcessed or input.KeyCode ~= CONFIG.TOGGLE_UI_KEY then return end; isUiVisible = not isUiVisible; mainFrame.Visible = isUiVisible; if isUiVisible then mainFrame.Position = lastUIPosition; populatePlayerList() end end)
toggleAllOnButton.MouseButton1Click:Connect(function() for _, player in ipairs(Players:GetPlayers()) do if player ~= localPlayer then espTargets[player] = true end end; populatePlayerList() end)
toggleAllOffButton.MouseButton1Click:Connect(function() for _, player in ipairs(Players:GetPlayers()) do if player ~= localPlayer then espTargets[player] = false end end; populatePlayerList() end)
refreshButton.MouseButton1Click:Connect(populatePlayerList)
applyHoverEffect(toggleAllOnButton, COLORS.Accent, COLORS.Accent:Lerp(Color3.new(1,1,1), 0.3))
applyHoverEffect(toggleAllOffButton, COLORS.Red, COLORS.Red:Lerp(Color3.new(1,1,1), 0.3))
applyHoverEffect(refreshButton, COLORS.Item, COLORS.Item:Lerp(Color3.new(1,1,1), 0.3))
Players.PlayerAdded:Connect(function(player) task.wait(1); if mainFrame.Visible then populatePlayerList() end end)
Players.PlayerRemoving:Connect(function(player) if espTargets[player] then espTargets[player] = nil end; if espDrawings[player] then for _, line in pairs(espDrawings[player]) do line:Remove() end; espDrawings[player] = nil end; if mainFrame.Visible then populatePlayerList() end end)
RunService.RenderStepped:Connect(updateEsp)

-- ===================================================================
-- INICIALIZAÇÃO
-- ===================================================================
populatePlayerList()
