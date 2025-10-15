--[[
	SNAP ESP V6 - FINAL BUILD
	-------------------------------------------------------------
	Autor: [Seu Nome]
	Data: [Data Atual]

	CHANGELOG (V6):
	- CORREÇÃO DEFINITIVA: O problema de sobreposição do texto do nome foi resolvido
	  com a reestruturação do layout do item da lista (usando UIListLayout).
	- RESPONSIVIDADE: A interface agora se adapta a diferentes resoluções,
	  com limites de tamanho mínimo e máximo para uma experiência consistente.
	- ANIMAÇÃO: Animação de escala (zoom) suave para abrir e fechar o painel.
	- MEMÓRIA DE POSIÇÃO: O painel reabre na última posição em que foi deixado.
	- DESIGN: Implementado o design moderno e limpo da V5.
	- CÓDIGO: Reformatado e comentado para máxima clareza e manutenibilidade.
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
	BOX_COLOR = Color3.fromRGB(0, 255, 127),
	THICKNESS = 2
}

-- ===================================================================
-- VARIÁVEIS DE ESTADO
-- ===================================================================
local isUiVisible = false
local espTargets = {}
local espDrawings = {}
local lastUIPosition = UDim2.new(0.5, 0, 0.5, 0) -- Posição inicial no centro

-- ===================================================================
-- 1. CRIAÇÃO DA INTERFACE GRÁFICA (GUI)
-- ===================================================================
-- Paleta de Cores
local COLORS = { Background = Color3.fromRGB(40, 42, 46), Item = Color3.fromRGB(54, 57, 63), Green = Color3.fromRGB(88, 207, 102), Red = Color3.fromRGB(224, 80, 80), Gray = Color3.fromRGB(80, 82, 86), Text = Color3.fromRGB(255, 255, 255) }

-- ScreenGui Principal
local screenGui = Instance.new("ScreenGui")
screenGui.Name = "ESP_ControlPanel_GUI"
screenGui.ResetOnSpawn = false
screenGui.Parent = localPlayer:WaitForChild("PlayerGui")

-- Janela Principal (MainFrame)
local mainFrame = Instance.new("Frame")
mainFrame.Name = "MainFrame"
mainFrame.AnchorPoint = Vector2.new(0.5, 0.5)
mainFrame.Position = lastUIPosition
mainFrame.BackgroundColor3 = COLORS.Background
mainFrame.BorderSizePixel = 0
mainFrame.Visible = isUiVisible
mainFrame.ClipsDescendants = true
mainFrame.Parent = screenGui
Instance.new("UICorner", mainFrame).CornerRadius = UDim.new(0, 10)

-- Lógica de Tamanho Responsivo
mainFrame.Size = UDim2.fromScale(0, 0.45)
local aspectRatio = Instance.new("UIAspectRatioConstraint", mainFrame)
aspectRatio.AspectRatio = 280 / 420
aspectRatio.DominantAxis = Enum.DominantAxis.Height
local sizeConstraint = Instance.new("UISizeConstraint", mainFrame)
sizeConstraint.MinSize = Vector2.new(240, 360)
sizeConstraint.MaxSize = Vector2.new(350, 525)

-- Container do Título
local titleContainer = Instance.new("Frame", mainFrame)
titleContainer.Name = "TitleContainer"; titleContainer.Size = UDim2.fromScale(1, 0.12); titleContainer.BackgroundTransparency = 1
local titleLayout = Instance.new("UIListLayout", titleContainer); titleLayout.FillDirection = Enum.FillDirection.Horizontal; titleLayout.VerticalAlignment = Enum.VerticalAlignment.Center; titleLayout.SortOrder = Enum.SortOrder.LayoutOrder; titleLayout.Padding = UDim.new(0, 8)
local titlePadding = Instance.new("UIPadding", titleContainer); titlePadding.PaddingLeft = UDim.new(0, 15); titlePadding.PaddingRight = UDim.new(0, 15)
local titleLabel = Instance.new("TextLabel", titleContainer); titleLabel.Name = "Title"; titleLabel.Size = UDim2.new(1, -145, 1, 0); titleLabel.Text = "SNAP ESP"; titleLabel.Font = Enum.Font.GothamBold; titleLabel.TextColor3 = COLORS.Text; titleLabel.TextXAlignment = Enum.TextXAlignment.Left; titleLabel.BackgroundTransparency = 1; titleLabel.LayoutOrder = 1; titleLabel.TextScaled = true
local buttonSize = UDim2.new(0, 40, 0, 32)
local toggleAllOnButton = Instance.new("TextButton", titleContainer); toggleAllOnButton.Name = "ToggleAllOn"; toggleAllOnButton.Size = buttonSize; toggleAllOnButton.Text = "ON"; toggleAllOnButton.Font = Enum.Font.GothamBold; toggleAllOnButton.TextColor3 = COLORS.Text; toggleAllOnButton.BackgroundColor3 = COLORS.Green; toggleAllOnButton.LayoutOrder = 2; toggleAllOnButton.TextScaled = true; Instance.new("UICorner", toggleAllOnButton).CornerRadius = UDim.new(0, 8)
local toggleAllOffButton = Instance.new("TextButton", titleContainer); toggleAllOffButton.Name = "ToggleAllOff"; toggleAllOffButton.Size = buttonSize; toggleAllOffButton.Text = "OFF"; toggleAllOffButton.Font = Enum.Font.GothamBold; toggleAllOffButton.TextColor3 = COLORS.Text; toggleAllOffButton.BackgroundColor3 = COLORS.Red; toggleAllOffButton.LayoutOrder = 3; toggleAllOffButton.TextScaled = true; Instance.new("UICorner", toggleAllOffButton).CornerRadius = UDim.new(0, 8)
local refreshButton = Instance.new("TextButton", titleContainer); refreshButton.Name = "RefreshButton"; refreshButton.Size = buttonSize; refreshButton.Text = "R"; refreshButton.Font = Enum.Font.GothamBold; refreshButton.TextColor3 = COLORS.Text; refreshButton.BackgroundColor3 = COLORS.Gray; refreshButton.LayoutOrder = 4; refreshButton.TextScaled = true; Instance.new("UICorner", refreshButton).CornerRadius = UDim.new(0, 8)

-- Área da Lista de Jogadores (ScrollingFrame)
local scrollingFrame = Instance.new("ScrollingFrame", mainFrame); scrollingFrame.Name = "PlayerList"; scrollingFrame.Size = UDim2.new(1, 0, 0.88, 0); scrollingFrame.Position = UDim2.fromScale(0, 0.12); scrollingFrame.BackgroundColor3 = COLORS.Background; scrollingFrame.BorderSizePixel = 0; scrollingFrame.CanvasSize = UDim2.new(0, 0, 0, 0); scrollingFrame.ScrollBarImageColor3 = Color3.fromRGB(200, 200, 200); scrollingFrame.ScrollBarThickness = 6
local listPadding = Instance.new("UIPadding", scrollingFrame); listPadding.PaddingLeft = UDim.new(0, 15); listPadding.PaddingRight = UDim.new(0, 15); listPadding.PaddingTop = UDim.new(0, 15); listPadding.PaddingBottom = UDim.new(0, 15)
local uiListLayout = Instance.new("UIListLayout", scrollingFrame); uiListLayout.Padding = UDim.new(0, 8); uiListLayout.SortOrder = Enum.SortOrder.LayoutOrder

-- ------------------- TEMPLATE DE JOGADOR - LÓGICA DE LAYOUT CORRIGIDA -------------------
local playerTemplate = Instance.new("Frame")
playerTemplate.Name = "PlayerTemplate"
playerTemplate.Size = UDim2.new(1, 0, 0, 50)
playerTemplate.BackgroundColor3 = COLORS.Item
playerTemplate.BorderSizePixel = 0
Instance.new("UICorner", playerTemplate).CornerRadius = UDim.new(0, 8)

-- Layout Horizontal para organizar os itens (Ícone, Nome, Botão)
local itemLayout = Instance.new("UIListLayout", playerTemplate)
itemLayout.FillDirection = Enum.FillDirection.Horizontal
itemLayout.VerticalAlignment = Enum.VerticalAlignment.Center
itemLayout.SortOrder = Enum.SortOrder.LayoutOrder
itemLayout.Padding = UDim.new(0, 10)

-- Padding interno para dar respiro
local itemPadding = Instance.new("UIPadding", playerTemplate)
itemPadding.PaddingLeft = UDim.new(0, 5)

-- Ícone do Jogador (tamanho fixo)
local playerIcon = Instance.new("ImageLabel", playerTemplate)
playerIcon.Name = "Icon"
playerIcon.LayoutOrder = 1
playerIcon.Size = UDim2.new(0, 40, 0, 40)
playerIcon.BackgroundTransparency = 1
Instance.new("UICorner", playerIcon).CornerRadius = UDim.new(1, 0)

-- Nome do Jogador (tamanho dinâmico)
local playerNameLabel = Instance.new("TextLabel", playerTemplate)
playerNameLabel.Name = "PlayerName"
playerNameLabel.LayoutOrder = 2
playerNameLabel.Size = UDim2.new(1, -110, 1, 0) -- Ocupa 100% do espaço menos o espaço dos outros itens
playerNameLabel.Font = Enum.Font.Gotham
playerNameLabel.TextColor3 = COLORS.Text
playerNameLabel.TextXAlignment = Enum.TextXAlignment.Left
playerNameLabel.BackgroundTransparency = 1
playerNameLabel.TextScaled = true -- Agora funciona perfeitamente pois o container é dinâmico

-- Botão Toggle (tamanho fixo)
local espToggleButton = Instance.new("TextButton", playerTemplate)
espToggleButton.Name = "ESPToggle"
espToggleButton.LayoutOrder = 3
espToggleButton.Size = UDim2.new(0, 45, 0, 30)
espToggleButton.Font = Enum.Font.GothamBold
espToggleButton.TextScaled = true
Instance.new("UICorner", espToggleButton).CornerRadius = UDim.new(0, 8)
-- -----------------------------------------------------------------------------------

-- (O restante do script permanece funcionalmente igual, pois já estava robusto)
-- ===================================================================
-- 2. LÓGICA DA INTERFACE E DOS JOGADORES
-- ===================================================================
local function updateToggleButton(button, isEnabled) if isEnabled then button.Text = "ON"; button.BackgroundColor3 = COLORS.Green else button.Text = "OFF"; button.BackgroundColor3 = COLORS.Red end end
local function populatePlayerList()
	for _, child in ipairs(scrollingFrame:GetChildren()) do if child:IsA("Frame") then child:Destroy() end end
	local playerCount = 0
	for _, player in ipairs(Players:GetPlayers()) do if player == localPlayer then continue end; playerCount = playerCount + 1; local playerFrame = playerTemplate:Clone(); playerFrame.Name = player.Name; playerFrame.PlayerName.Text = string.format("%s (%s)", player.DisplayName, player.Name); local content, isReady = Players:GetUserThumbnailAsync(player.UserId, Enum.ThumbnailType.HeadShot, Enum.ThumbnailSize.Size48x48); if isReady then playerFrame.Icon.Image = content end; if espTargets[player] == nil then espTargets[player] = false end; updateToggleButton(playerFrame.ESPToggle, espTargets[player]); playerFrame.ESPToggle.MouseButton1Click:Connect(function() espTargets[player] = not espTargets[player]; updateToggleButton(playerFrame.ESPToggle, espTargets[player]) end); playerFrame.Parent = scrollingFrame
	end
	local itemHeight = playerTemplate.Size.Y.Offset; local padding = uiListLayout.Padding.Offset; scrollingFrame.CanvasSize = UDim2.fromOffset(0, (itemHeight * playerCount) + (padding * (playerCount + 1)))
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
local tweenInfo = TweenInfo.new(0.3, Enum.EasingStyle.Back, Enum.EasingDirection.Out); local animator = { Size = Instance.new("NumberValue") }; animator.Size.Value = 0
animator.Size.Changed:Connect(function(value) mainFrame.Size = UDim2.fromScale(0.45 * value, 0.45 * value) end)
UserInputService.InputBegan:Connect(function(input, gameProcessed) if gameProcessed or input.KeyCode ~= CONFIG.TOGGLE_UI_KEY then return end; isUiVisible = not isUiVisible; local targetValue = isUiVisible and 1 or 0; if isUiVisible then mainFrame.Visible = true; mainFrame.Position = lastUIPosition; populatePlayerList() end; local tween = TweenService:Create(animator.Size, tweenInfo, { Value = targetValue }); tween:Play(); if not isUiVisible then tween.Completed:Connect(function() mainFrame.Visible = false end) end end)
toggleAllOnButton.MouseButton1Click:Connect(function() for _, player in ipairs(Players:GetPlayers()) do if player ~= localPlayer then espTargets[player] = true end end; populatePlayerList() end)
toggleAllOffButton.MouseButton1Click:Connect(function() for _, player in ipairs(Players:GetPlayers()) do if player ~= localPlayer then espTargets[player] = false end end; populatePlayerList() end)
refreshButton.MouseButton1Click:Connect(populatePlayerList)
Players.PlayerAdded:Connect(function(player) task.wait(1); if mainFrame.Visible then populatePlayerList() end end)
Players.PlayerRemoving:Connect(function(player) if espTargets[player] then espTargets[player] = nil end; if espDrawings[player] then for _, line in pairs(espDrawings[player]) do line:Remove() end; espDrawings[player] = nil end; if mainFrame.Visible then populatePlayerList() end end)
RunService.RenderStepped:Connect(updateEsp)
populatePlayerList()
