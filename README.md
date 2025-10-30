-- SNAP ESP - BUILD "GLASS" v2.7 (com fallback para Studio)
-- Autor: snap & Assistente AI
-- Observação: Detecta Drawing; se não existir usa UI Frames como fallback.

local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local UserInputService = game:GetService("UserInputService")
local Workspace = game:GetService("Workspace")
local TweenService = game:GetService("TweenService")

local localPlayer = Players.LocalPlayer
local camera = Workspace.CurrentCamera
local playerGui = localPlayer:WaitForChild("PlayerGui")

local CONFIG = {
	TOGGLE_UI_KEY = Enum.KeyCode.LeftAlt,
	BOX_COLOR = Color3.fromRGB(0, 200, 255),
	THICKNESS = 2,
	MAX_WALKSPEED = 50
}

local isUiVisible = false
local espTargets = {}
local espDrawings = {}
local lastUIPosition = UDim2.new(0.5, 0, 0.5, 0)

local gameDefaultWalkSpeed = 16
local currentWalkSpeedValue = 16

-- GUI base (igual ao anterior)
local COLORS = { Background = Color3.fromRGB(20, 22, 30), Primary = Color3.fromRGB(35, 38, 50), Secondary = Color3.fromRGB(28, 30, 40), Stroke = Color3.fromRGB(80, 85, 100), Accent = Color3.fromRGB(0, 200, 255), Red = Color3.fromRGB(255, 80, 80), Text = Color3.fromRGB(255, 255, 255), SubText = Color3.fromRGB(170, 175, 190) }
local FONT_SETTINGS = { Font = Enum.Font.GothamSemibold, TitleSize = 18, RegularSize = 14, SmallSize = 12 }

if playerGui:FindFirstChild("SNAP_ESP_GUI") then playerGui.SNAP_ESP_GUI:Destroy() end
local screenGui = Instance.new("ScreenGui"); screenGui.Name = "SNAP_ESP_GUI"; screenGui.ResetOnSpawn = false; screenGui.Parent = playerGui
local blurEffect = Instance.new("BlurEffect", camera); blurEffect.Size = 0; blurEffect.Enabled = false
local mainFrame = Instance.new("Frame"); mainFrame.Name = "MainFrame"; mainFrame.Size = UDim2.new(0, 300, 0, 450); mainFrame.AnchorPoint = Vector2.new(0.5, 0.5); mainFrame.Position = lastUIPosition; mainFrame.BackgroundColor3 = COLORS.Background; mainFrame.BackgroundTransparency = 0.2; mainFrame.BorderSizePixel = 0; mainFrame.Visible = isUiVisible; mainFrame.ClipsDescendants = true; mainFrame.Parent = screenGui
Instance.new("UICorner", mainFrame).CornerRadius = UDim.new(0, 8)
Instance.new("UIStroke", mainFrame).Color = COLORS.Stroke

local titleContainer = Instance.new("Frame", mainFrame); titleContainer.Name = "TitleContainer"; titleContainer.Size = UDim2.new(1, 0, 0, 45); titleContainer.BackgroundTransparency = 1
Instance.new("UIPadding", titleContainer).PaddingLeft = UDim.new(0, 15)
local titleLabel = Instance.new("TextLabel", titleContainer); titleLabel.Name = "Title"; titleLabel.Size = UDim2.new(1, 0, 1, 0); titleLabel.Position = UDim2.fromScale(0, 0.5); titleLabel.AnchorPoint = Vector2.new(0, 0.5); titleLabel.Text = "SNAP ESP"; titleLabel.Font = FONT_SETTINGS.Font; titleLabel.TextSize = FONT_SETTINGS.TitleSize; titleLabel.TextColor3 = COLORS.Text; titleLabel.TextXAlignment = Enum.TextXAlignment.Left; titleLabel.BackgroundTransparency = 1

local contentContainer = Instance.new("Frame", mainFrame); contentContainer.Name = "ContentContainer"; contentContainer.Size = UDim2.new(1, 0, 1, -45); contentContainer.Position = UDim2.new(0, 0, 0, 45); contentContainer.BackgroundTransparency = 1
local contentLayout = Instance.new("UIListLayout", contentContainer); contentLayout.Padding = UDim.new(0, 10); contentLayout.SortOrder = Enum.SortOrder.LayoutOrder
local contentPadding = Instance.new("UIPadding", contentContainer); contentPadding.PaddingLeft = UDim.new(0, 15); contentPadding.PaddingRight = UDim.new(0, 15); contentPadding.PaddingTop = UDim.new(0, 10)

local walkspeedFrame = Instance.new("Frame", contentContainer); walkspeedFrame.Name = "WalkSpeedSection"; walkspeedFrame.LayoutOrder = 1; walkspeedFrame.Size = UDim2.new(1, 0, 0, 80); walkspeedFrame.BackgroundColor3 = COLORS.Primary; walkspeedFrame.BorderSizePixel = 0; Instance.new("UICorner", walkspeedFrame).CornerRadius = UDim.new(0, 6); Instance.new("UIStroke", walkspeedFrame).Color = COLORS.Stroke
local walkspeedMainLayout = Instance.new("UIListLayout", walkspeedFrame); walkspeedMainLayout.Padding = UDim.new(0, 10); walkspeedMainLayout.HorizontalAlignment = Enum.HorizontalAlignment.Center; walkspeedMainLayout.VerticalAlignment = Enum.VerticalAlignment.Center
local walkspeedPadding = Instance.new("UIPadding", walkspeedFrame); walkspeedPadding.PaddingLeft = UDim.new(0, 10); walkspeedPadding.PaddingRight = UDim.new(0, 10)
local slider = Instance.new("Frame", walkspeedFrame); slider.Name = "Slider"; slider.Size = UDim2.new(1, 0, 0, 10); slider.BackgroundColor3 = COLORS.Background; slider.BorderSizePixel = 0; Instance.new("UICorner", slider).CornerRadius = UDim.new(1, 0); Instance.new("UIStroke", slider).Color = COLORS.Stroke
local sliderFill = Instance.new("Frame", slider); sliderFill.Name = "Fill"; sliderFill.Size = UDim2.fromScale(0, 1); sliderFill.BackgroundColor3 = COLORS.Accent; sliderFill.BorderSizePixel = 0; Instance.new("UICorner", sliderFill).CornerRadius = UDim.new(1, 0)
local sliderHandle = Instance.new("TextButton", slider); sliderHandle.Name = "Handle"; sliderHandle.Text = ""; sliderHandle.Size = UDim2.new(0, 12, 0, 24); sliderHandle.Position = UDim2.fromScale(0, 0.5); sliderHandle.AnchorPoint = Vector2.new(0.5, 0.5); sliderHandle.BackgroundColor3 = COLORS.Text; sliderHandle.BorderSizePixel = 0; Instance.new("UICorner", sliderHandle).CornerRadius = UDim.new(0, 4)
local bottomControls = Instance.new("Frame", walkspeedFrame); bottomControls.Name = "BottomControls"; bottomControls.Size = UDim2.new(1, 0, 0, 28); bottomControls.BackgroundTransparency = 1
local bottomLayout = Instance.new("UIListLayout", bottomControls); bottomLayout.FillDirection = Enum.FillDirection.Horizontal; bottomLayout.VerticalAlignment = Enum.VerticalAlignment.Center; bottomLayout.SortOrder = Enum.SortOrder.LayoutOrder
local walkspeedLabel = Instance.new("TextLabel", bottomControls); walkspeedLabel.LayoutOrder = 1; walkspeedLabel.Name = "Label"; walkspeedLabel.Size = UDim2.new(0, 80, 1, 0); walkspeedLabel.Text = "WalkSpeed"; walkspeedLabel.Font = FONT_SETTINGS.Font; walkspeedLabel.TextSize = FONT_SETTINGS.RegularSize; walkspeedLabel.TextColor3 = COLORS.SubText; walkspeedLabel.TextXAlignment = Enum.TextXAlignment.Left; walkspeedLabel.BackgroundTransparency = 1
local spacer = Instance.new("Frame", bottomControls); spacer.LayoutOrder = 2; spacer.Name = "Spacer"; spacer.Size = UDim2.new(1, -165, 1, 0); spacer.BackgroundTransparency = 1
local rightControls = Instance.new("Frame", bottomControls); rightControls.LayoutOrder = 3; rightControls.Name = "RightControls"; rightControls.Size = UDim2.new(0, 80, 1, 0); rightControls.BackgroundTransparency = 1
local rightLayout = Instance.new("UIListLayout", rightControls); rightLayout.FillDirection = Enum.FillDirection.Horizontal; rightLayout.VerticalAlignment = Enum.VerticalAlignment.Center; rightLayout.HorizontalAlignment = Enum.HorizontalAlignment.Right; rightLayout.Padding = UDim.new(0, 5)
local speedInput = Instance.new("TextBox", rightControls); speedInput.LayoutOrder = 1; speedInput.Name = "Input"; speedInput.Size = UDim2.new(0, 45, 1, 0); speedInput.BackgroundColor3 = COLORS.Secondary; speedInput.Font = FONT_SETTINGS.Font; speedInput.Text = tostring(gameDefaultWalkSpeed); speedInput.TextColor3 = COLORS.Text; speedInput.TextSize = FONT_SETTINGS.RegularSize; speedInput.ClearTextOnFocus = false; speedInput.TextXAlignment = Enum.TextXAlignment.Center; Instance.new("UICorner", speedInput).CornerRadius = UDim.new(0, 4); Instance.new("UIStroke", speedInput).Color = COLORS.Stroke
local resetButton = Instance.new("TextButton", rightControls); resetButton.LayoutOrder = 2; resetButton.Name = "Reset"; resetButton.Size = UDim2.new(0, 30, 1, 0); resetButton.BackgroundColor3 = COLORS.Secondary; resetButton.Font = FONT_SETTINGS.Font; resetButton.Text = "R"; resetButton.TextColor3 = COLORS.Text; resetButton.TextSize = FONT_SETTINGS.RegularSize; Instance.new("UICorner", resetButton).CornerRadius = UDim.new(0, 4); Instance.new("UIStroke", resetButton).Color = COLORS.Stroke

local playersFrame = Instance.new("Frame", contentContainer); playersFrame.Name = "PlayersSection"; playersFrame.LayoutOrder = 2; playersFrame.Size = UDim2.new(1, 0, 1, -110); playersFrame.BackgroundColor3 = COLORS.Primary; playersFrame.BorderSizePixel = 0; Instance.new("UICorner", playersFrame).CornerRadius = UDim.new(0, 6); Instance.new("UIStroke", playersFrame).Color = COLORS.Stroke
local playersHeader = Instance.new("Frame", playersFrame); playersHeader.Name = "Header"; playersHeader.Size = UDim2.new(1, 0, 0, 35); playersHeader.BackgroundTransparency = 1; Instance.new("UIPadding", playersHeader).PaddingLeft = UDim.new(0, 10); Instance.new("UIPadding", playersHeader).PaddingRight = UDim.new(0, 10)
local playersTitle = Instance.new("TextLabel", playersHeader); playersTitle.Name = "Title"; playersTitle.Size = UDim2.new(1, -100, 1, 0); playersTitle.AnchorPoint = Vector2.new(0, 0.5); playersTitle.Position = UDim2.fromScale(0, 0.5); playersTitle.Text = "Players"; playersTitle.Font = FONT_SETTINGS.Font; playersTitle.TextSize = FONT_SETTINGS.RegularSize; playersTitle.TextColor3 = COLORS.Text; playersTitle.TextXAlignment = Enum.TextXAlignment.Left; playersTitle.BackgroundTransparency = 1

local toggleAllOnButton = Instance.new("TextButton", playersHeader); toggleAllOnButton.Name = "ToggleAllOn"; toggleAllOnButton.Size = UDim2.new(0, 40, 0, 24); toggleAllOnButton.AnchorPoint = Vector2.new(1, 0.5); toggleAllOnButton.Position = UDim2.new(1, -60, 0.5, 0); toggleAllOnButton.BackgroundColor3 = COLORS.Accent; toggleAllOnButton.Font = FONT_SETTINGS.Font; toggleAllOnButton.Text = "ALL"; toggleAllOnButton.TextColor3 = COLORS.Text; toggleAllOnButton.TextSize = FONT_SETTINGS.SmallSize; Instance.new("UICorner", toggleAllOnButton).CornerRadius = UDim.new(0, 4); Instance.new("UIStroke", toggleAllOnButton).Color = COLORS.Stroke
local toggleAllOffButton = Instance.new("TextButton", playersHeader); toggleAllOffButton.Name = "ToggleAllOff"; toggleAllOffButton.Size = UDim2.new(0, 40, 0, 24); toggleAllOffButton.AnchorPoint = Vector2.new(1, 0.5); toggleAllOffButton.Position = UDim2.new(1, -10, 0.5, 0); toggleAllOffButton.BackgroundColor3 = COLORS.Red; toggleAllOffButton.Font = FONT_SETTINGS.Font; toggleAllOffButton.Text = "NONE"; toggleAllOffButton.TextColor3 = COLORS.Text; toggleAllOffButton.TextSize = FONT_SETTINGS.SmallSize; Instance.new("UICorner", toggleAllOffButton).CornerRadius = UDim.new(0, 4); Instance.new("UIStroke", toggleAllOffButton).Color = COLORS.Stroke

local scrollingFrame = Instance.new("ScrollingFrame", playersFrame); scrollingFrame.Name = "PlayerList"; scrollingFrame.Size = UDim2.new(1, 0, 1, -35); scrollingFrame.Position = UDim2.new(0, 0, 0, 35); scrollingFrame.BackgroundTransparency = 1; scrollingFrame.BorderSizePixel = 0; scrollingFrame.CanvasSize = UDim2.new(0, 0, 0, 0); scrollingFrame.ScrollBarImageColor3 = COLORS.Accent; scrollingFrame.ScrollBarThickness = 4
local listPadding = Instance.new("UIPadding", scrollingFrame); listPadding.PaddingLeft = UDim.new(0, 10); listPadding.PaddingRight = UDim.new(0, 10); listPadding.PaddingTop = UDim.new(0, 5)
local uiListLayout = Instance.new("UIListLayout", scrollingFrame); uiListLayout.Padding = UDim.new(0, 5); uiListLayout.SortOrder = Enum.SortOrder.LayoutOrder
local playerTemplate = Instance.new("Frame"); playerTemplate.Name = "PlayerTemplate"; playerTemplate.Size = UDim2.new(1, 0, 0, 50); playerTemplate.BackgroundTransparency = 1
local playerIcon = Instance.new("ImageLabel", playerTemplate); playerIcon.Name = "Icon"; playerIcon.Size = UDim2.new(0, 36, 0, 36); playerIcon.AnchorPoint = Vector2.new(0, 0.5); playerIcon.Position = UDim2.new(0, 0, 0.5, 0); playerIcon.BackgroundTransparency = 1; Instance.new("UICorner", playerIcon).CornerRadius = UDim.new(1, 0); Instance.new("UIStroke", playerIcon).Color = COLORS.Stroke
local textContainer = Instance.new("Frame", playerTemplate); textContainer.Name = "TextContainer"; textContainer.Size = UDim2.new(1, -100, 1, 0); textContainer.Position = UDim2.new(0, 46, 0.5, 0); textContainer.AnchorPoint = Vector2.new(0, 0.5); textContainer.BackgroundTransparency = 1
local textLayout = Instance.new("UIListLayout", textContainer); textLayout.FillDirection = Enum.FillDirection.Vertical; textLayout.HorizontalAlignment = Enum.HorizontalAlignment.Left; textLayout.SortOrder = Enum.SortOrder.LayoutOrder
local displayNameLabel = Instance.new("TextLabel", textContainer); displayNameLabel.Name = "DisplayName"; displayNameLabel.Size = UDim2.new(1, 0, 0, 16); displayNameLabel.Font = FONT_SETTINGS.Font; displayNameLabel.TextColor3 = COLORS.Text; displayNameLabel.TextSize = FONT_SETTINGS.RegularSize; displayNameLabel.TextXAlignment = Enum.TextXAlignment.Left; displayNameLabel.BackgroundTransparency = 1
local userNameLabel = Instance.new("TextLabel", textContainer); userNameLabel.Name = "UserName"; userNameLabel.Size = UDim2.new(1, 0, 0, 12); userNameLabel.Font = FONT_SETTINGS.Font; userNameLabel.TextColor3 = COLORS.SubText; userNameLabel.TextSize = FONT_SETTINGS.SmallSize; userNameLabel.TextXAlignment = Enum.TextXAlignment.Left; userNameLabel.BackgroundTransparency = 1
local espToggleButton = Instance.new("TextButton", playerTemplate); espToggleButton.Name = "ESPToggle"; espToggleButton.Size = UDim2.new(0, 45, 0, 28); espToggleButton.AnchorPoint = Vector2.new(1, 0.5); espToggleButton.Position = UDim2.new(1, 0, 0.5, 0); espToggleButton.Font = FONT_SETTINGS.Font; espToggleButton.TextSize = FONT_SETTINGS.RegularSize; Instance.new("UICorner", espToggleButton).CornerRadius = UDim.new(0, 6); Instance.new("UIStroke", espToggleButton).Color = COLORS.Stroke

local notificationFrame = Instance.new("Frame", screenGui); notificationFrame.Name = "Notification"; notificationFrame.Size = UDim2.new(0, 120, 0, 40); notificationFrame.AnchorPoint = Vector2.new(1, 0); notificationFrame.Position = UDim2.new(1, -15, 0, -50); notificationFrame.BackgroundColor3 = COLORS.Primary; notificationFrame.BackgroundTransparency = 1; Instance.new("UICorner", notificationFrame).CornerRadius = UDim.new(0, 6)
local notificationStroke = Instance.new("UIStroke", notificationFrame); notificationStroke.Color = COLORS.Stroke; notificationStroke.Transparency = 1
local notificationLayout = Instance.new("UIListLayout", notificationFrame); notificationLayout.FillDirection = Enum.FillDirection.Horizontal; notificationLayout.VerticalAlignment = Enum.VerticalAlignment.Center; notificationLayout.Padding = UDim.new(0, 8); Instance.new("UIPadding", notificationFrame).PaddingLeft = UDim.new(0, 12)
local notificationIcon = Instance.new("TextLabel", notificationFrame); notificationIcon.Name = "Icon"; notificationIcon.Size = UDim2.new(0, 20, 1, 0); notificationIcon.Font = Enum.Font.SourceSans; notificationIcon.TextSize = 20; notificationIcon.TextColor3 = COLORS.Text; notificationIcon.TextTransparency = 1; notificationIcon.BackgroundTransparency = 1
local notificationLabel = Instance.new("TextLabel", notificationFrame); notificationLabel.Name = "Label"; notificationLabel.Size = UDim2.new(1, -32, 1, 0); notificationLabel.Font = FONT_SETTINGS.Font; notificationLabel.TextSize = FONT_SETTINGS.RegularSize; notificationLabel.TextColor3 = COLORS.Text; notificationLabel.TextTransparency = 1; notificationLabel.BackgroundTransparency = 1; notificationLabel.TextXAlignment = Enum.TextXAlignment.Left

-- Notificação e utilitárias
local currentNotificationTween
local function showNotification(icon, message, duration)
	if currentNotificationTween and currentNotificationTween.PlaybackState == Enum.PlaybackState.Playing then
		currentNotificationTween:Cancel()
	end
	notificationIcon.Text = icon
	notificationLabel.Text = message
	notificationFrame.Position = UDim2.new(1, -15, 0, -50)
	notificationFrame.BackgroundTransparency = 1
	notificationStroke.Transparency = 1
	notificationIcon.TextTransparency = 1
	notificationLabel.TextTransparency = 1
	local tweenInfo = TweenInfo.new(0.3, Enum.EasingStyle.Quad, Enum.EasingDirection.Out)
	currentNotificationTween = TweenService:Create(notificationFrame, tweenInfo, { Position = UDim2.new(1, -15, 0, 15), BackgroundTransparency = 0.2 })
	local strokeTween = TweenService:Create(notificationStroke, tweenInfo, { Transparency = 0 })
	local textTween = TweenService:Create(notificationIcon, tweenInfo, { TextTransparency = 0 })
	local textTween2 = TweenService:Create(notificationLabel, tweenInfo, { TextTransparency = 0 })
	currentNotificationTween:Play(); strokeTween:Play(); textTween:Play(); textTween2:Play()
	task.delay(duration or 2, function()
		if not notificationFrame or not notificationFrame.Parent then return end
		local tweenOut = TweenService:Create(notificationFrame, tweenInfo, { Position = UDim2.new(1, -15, 0, -50), BackgroundTransparency = 1 })
		local strokeTweenOut = TweenService:Create(notificationStroke, tweenInfo, { Transparency = 1 })
		local textTweenOut = TweenService:Create(notificationIcon, tweenInfo, { TextTransparency = 1 })
		local textTween2Out = TweenService:Create(notificationLabel, tweenInfo, { TextTransparency = 1 })
		tweenOut:Play(); strokeTweenOut:Play(); textTweenOut:Play(); textTween2Out:Play()
	end)
end

local function applyHoverEffect(button, baseColor, hoverColor)
	button.MouseEnter:Connect(function()
		TweenService:Create(button, TweenInfo.new(0.2), { BackgroundColor3 = hoverColor }):Play()
	end)
	button.MouseLeave:Connect(function()
		TweenService:Create(button, TweenInfo.new(0.2), { BackgroundColor3 = baseColor }):Play()
	end)
end

local function updateToggleButton(button, isEnabled)
	local targetColor = isEnabled and COLORS.Accent or COLORS.Red
	button.Text = isEnabled and "ON" or "OFF"
	button.TextColor3 = isEnabled and COLORS.Background or COLORS.Text
	TweenService:Create(button, TweenInfo.new(0.15), { BackgroundColor3 = targetColor }):Play()
end

local function applyDynamicHoverEffect(button, player)
	button.MouseEnter:Connect(function()
		local isEnabled = espTargets[player]
		local baseColor = isEnabled and COLORS.Accent or COLORS.Red
		local hoverColor = baseColor:Lerp(Color3.new(1, 1, 1), 0.2)
		TweenService:Create(button, TweenInfo.new(0.2), { BackgroundColor3 = hoverColor }):Play()
	end)
	button.MouseLeave:Connect(function()
		local isEnabled = espTargets[player]
		local baseColor = isEnabled and COLORS.Accent or COLORS.Red
		TweenService:Create(button, TweenInfo.new(0.2), { BackgroundColor3 = baseColor }):Play()
	end)
end

local function populatePlayerList()
	for _, child in ipairs(scrollingFrame:GetChildren()) do
		if child:IsA("Frame") or child:IsA("TextLabel") then
			child:Destroy()
		end
	end

	local playerCount = 0
	for _, player in ipairs(Players:GetPlayers()) do
		if player ~= localPlayer then
			playerCount = playerCount + 1
			local playerFrame = playerTemplate:Clone()
			playerFrame.Name = player.Name
			playerFrame.TextContainer.DisplayName.Text = player.DisplayName
			playerFrame.TextContainer.UserName.Text = "(@" .. player.Name .. ")"

			local ok, content = pcall(function()
				return Players:GetUserThumbnailAsync(player.UserId, Enum.ThumbnailType.HeadShot, Enum.ThumbnailSize.Size48x48)
			end)
			if ok and content then
				playerFrame.Icon.Image = content
			end

			if espTargets[player] == nil then espTargets[player] = false end
			local isEnabled = espTargets[player]
			playerFrame.ESPToggle.Text = isEnabled and "ON" or "OFF"
			playerFrame.ESPToggle.BackgroundColor3 = isEnabled and COLORS.Accent or COLORS.Red
			playerFrame.ESPToggle.TextColor3 = isEnabled and COLORS.Background or COLORS.Text

			playerFrame.ESPToggle.MouseButton1Click:Connect(function()
				espTargets[player] = not espTargets[player]
				updateToggleButton(playerFrame.ESPToggle, espTargets[player])
			end)

			applyDynamicHoverEffect(playerFrame.ESPToggle, player)
			playerFrame.Parent = scrollingFrame
		end
	end

	local itemHeight = playerTemplate.Size.Y.Offset
	local padding = uiListLayout.Padding.Offset
	local totalHeight = (itemHeight * playerCount) + (padding * math.max(0, playerCount - 1))
	scrollingFrame.CanvasSize = UDim2.fromOffset(0, totalHeight)
end

-- ---------- DRAWING DETECTION E FALBACK UI ----------
local HAS_DRAWING = false
if typeof(Drawing) == "table" or typeof(Drawing) == "userdata" then
	if type(Drawing.new) == "function" then
		HAS_DRAWING = true
	end
end

-- pasta de UI para fallback
local espUiFolder = Instance.new("Folder", screenGui)
espUiFolder.Name = "ESP_UI_FALLBACK"

if not HAS_DRAWING then
	pcall(function()
		if showNotification then showNotification("⚠️", "Drawing API não disponível. Usando fallback UI.", 3) else warn("Drawing API não disponível. Usando fallback UI.") end
	end)
end

local function createUIBoxForPlayer(player)
	local container = Instance.new("Frame", espUiFolder)
	container.Name = "ESP_UI_" .. player.Name
	container.BackgroundTransparency = 1
	container.ZIndex = 1000

	local top = Instance.new("Frame", container)
	top.Name = "Top"; top.BackgroundColor3 = CONFIG.BOX_COLOR; top.BorderSizePixel = 0; top.Visible = false; top.ZIndex = 1000
	local bottom = Instance.new("Frame", container)
	bottom.Name = "Bottom"; bottom.BackgroundColor3 = CONFIG.BOX_COLOR; bottom.BorderSizePixel = 0; bottom.Visible = false; bottom.ZIndex = 1000
	local left = Instance.new("Frame", container)
	left.Name = "Left"; left.BackgroundColor3 = CONFIG.BOX_COLOR; left.BorderSizePixel = 0; left.Visible = false; left.ZIndex = 1000
	local right = Instance.new("Frame", container)
	right.Name = "Right"; right.BackgroundColor3 = CONFIG.BOX_COLOR; right.BorderSizePixel = 0; right.Visible = false; right.ZIndex = 1000

	return {
		type = "ui",
		Top = top,
		Bottom = bottom,
		Left = left,
		Right = right,
		_container = container
	}
end

local function getOrCreateDrawings(player)
	if espDrawings[player] then return espDrawings[player] end

	if HAS_DRAWING then
		local ok, newDrawings = pcall(function()
			return {
				type = "drawing",
				Top = Drawing.new("Line"),
				Bottom = Drawing.new("Line"),
				Left = Drawing.new("Line"),
				Right = Drawing.new("Line")
			}
		end)
		if not ok or type(newDrawings) ~= "table" then
			warn("Falha ao criar Drawings. Mudando para fallback UI.")
			HAS_DRAWING = false
			espDrawings[player] = createUIBoxForPlayer(player)
			return espDrawings[player]
		end

		for _, line in pairs(newDrawings) do
			if type(line) == "table" or type(line) == "userdata" then
				line.Color = CONFIG.BOX_COLOR
				line.Thickness = CONFIG.THICKNESS
				line.ZIndex = 2
				line.Visible = false
			end
		end
		espDrawings[player] = newDrawings
		espDrawings[player].type = "drawing"
		return espDrawings[player]
	else
		espDrawings[player] = createUIBoxForPlayer(player)
		return espDrawings[player]
	end
end

local function removeDrawingsForPlayer(player)
	local d = espDrawings[player]
	if not d then return end
	if d.type == "drawing" then
		for _, v in pairs({d.Top, d.Bottom, d.Left, d.Right}) do
			pcall(function() if v and v.Remove then v:Remove() end end)
		end
	else
		pcall(function()
			if d._container then d._container:Destroy() end
		end)
	end
	espDrawings[player] = nil
end

-- ---------- BOUNDING BOX (manual) ----------
local function getManualBoundingBox(model)
	local min, max = Vector3.new(math.huge, math.huge, math.huge), Vector3.new(-math.huge, -math.huge, -math.huge)
	local hasParts = false
	for _, descendant in ipairs(model:GetDescendants()) do
		if descendant:IsA("BasePart") then
			hasParts = true
			local cf, size = descendant.CFrame, descendant.Size
			local halfSize = size / 2
			local corners = {
				(cf * CFrame.new( halfSize.X,  halfSize.Y,  halfSize.Z)).Position, (cf * CFrame.new( halfSize.X,  halfSize.Y, -halfSize.Z)).Position,
				(cf * CFrame.new( halfSize.X, -halfSize.Y,  halfSize.Z)).Position, (cf * CFrame.new( halfSize.X, -halfSize.Y, -halfSize.Z)).Position,
				(cf * CFrame.new(-halfSize.X,  halfSize.Y,  halfSize.Z)).Position, (cf * CFrame.new(-halfSize.X,  halfSize.Y, -halfSize.Z)).Position,
				(cf * CFrame.new(-halfSize.X, -halfSize.Y,  halfSize.Z)).Position, (cf * CFrame.new(-halfSize.X, -halfSize.Y, -halfSize.Z)).Position
			}
			for _, corner in ipairs(corners) do
				min = Vector3.new(math.min(min.X, corner.X), math.min(min.Y, corner.Y), math.min(min.Z, corner.Z))
				max = Vector3.new(math.max(max.X, corner.X), math.max(max.Y, corner.Y), math.max(max.Z, corner.Z))
			end
		end
	end
	if not hasParts then return nil, nil end
	local center = (min + max) / 2
	local size = max - min
	return CFrame.new(center), size
end

-- ---------- UPDATE ESP (suporta drawing e ui fallback) ----------
local function updateEsp()
	-- primeiro esconde tudo
	for _, d in pairs(espDrawings) do
		if d then
			if d.type == "drawing" then
				pcall(function()
					if d.Top then d.Top.Visible = false end
					if d.Bottom then d.Bottom.Visible = false end
					if d.Left then d.Left.Visible = false end
					if d.Right then d.Right.Visible = false end
				end)
			else
				pcall(function()
					if d.Top then d.Top.Visible = false end
					if d.Bottom then d.Bottom.Visible = false end
					if d.Left then d.Left.Visible = false end
					if d.Right then d.Right.Visible = false end
				end)
			end
		end
	end

	for player, enabled in pairs(espTargets) do
		if enabled then
			local character = player.Character
			local humanoid = character and character:FindFirstChildOfClass("Humanoid")
			local primaryPart = character and character.PrimaryPart
			if player.Parent and humanoid and humanoid.Health > 0 and primaryPart then
				local _, size = getManualBoundingBox(character)
				if size then
					local cframe = primaryPart.CFrame
					local corners = {}
					local pointsOnScreen = 0
					local minX, maxX = math.huge, -math.huge
					local minY, maxY = math.huge, -math.huge
					local halfSize = size / 2

					local signs = { -1, 1 }
					for _, sx in ipairs(signs) do
						for _, sy in ipairs(signs) do
							for _, sz in ipairs(signs) do
								local worldPos = (cframe * CFrame.new(halfSize.X * sx, halfSize.Y * sy, halfSize.Z * sz)).Position
								table.insert(corners, worldPos)
							end
						end
					end

					for _, pos3D in ipairs(corners) do
						local pos2D, onScreen = camera:WorldToScreenPoint(pos3D)
						if onScreen and pos2D.Z > 0 then
							pointsOnScreen = pointsOnScreen + 1
							minX = math.min(minX, pos2D.X)
							maxX = math.max(maxX, pos2D.X)
							minY = math.min(minY, pos2D.Y)
							maxY = math.max(maxY, pos2D.Y)
						end
					end

					if pointsOnScreen > 0 then
						local lines = getOrCreateDrawings(player)
						if not lines then
							-- sem desenho possível
						else
							local topLeftX = minX
							local topLeftY = minY
							local boxWidth = math.max(1, maxX - minX)
							local boxHeight = math.max(1, maxY - minY)
							if lines.type == "drawing" then
								pcall(function()
									lines.Top.From = Vector2.new(topLeftX, topLeftY)
									lines.Top.To = Vector2.new(topLeftX + boxWidth, topLeftY)
									lines.Bottom.From = Vector2.new(topLeftX, topLeftY + boxHeight)
									lines.Bottom.To = Vector2.new(topLeftX + boxWidth, topLeftY + boxHeight)
									lines.Left.From = Vector2.new(topLeftX, topLeftY)
									lines.Left.To = Vector2.new(topLeftX, topLeftY + boxHeight)
									lines.Right.From = Vector2.new(topLeftX + boxWidth, topLeftY)
									lines.Right.To = Vector2.new(topLeftX + boxWidth, topLeftY + boxHeight)
									lines.Top.Visible = true
									lines.Bottom.Visible = true
									lines.Left.Visible = true
									lines.Right.Visible = true
								end)
							else
								-- UI fallback: posiciona / redimensiona os frames em screenGui (pixels)
								pcall(function()
									local thickness = math.max(1, CONFIG.THICKNESS)
									lines.Top.Position = UDim2.fromOffset(topLeftX, topLeftY)
									lines.Top.Size = UDim2.fromOffset(boxWidth, thickness)
									lines.Bottom.Position = UDim2.fromOffset(topLeftX, topLeftY + boxHeight - thickness)
									lines.Bottom.Size = UDim2.fromOffset(boxWidth, thickness)
									lines.Left.Position = UDim2.fromOffset(topLeftX, topLeftY)
									lines.Left.Size = UDim2.fromOffset(thickness, boxHeight)
									lines.Right.Position = UDim2.fromOffset(topLeftX + boxWidth - thickness, topLeftY)
									lines.Right.Size = UDim2.fromOffset(thickness, boxHeight)

									lines.Top.BackgroundColor3 = CONFIG.BOX_COLOR
									lines.Bottom.BackgroundColor3 = CONFIG.BOX_COLOR
									lines.Left.BackgroundColor3 = CONFIG.BOX_COLOR
									lines.Right.BackgroundColor3 = CONFIG.BOX_COLOR

									lines.Top.Visible = true
									lines.Bottom.Visible = true
									lines.Left.Visible = true
									lines.Right.Visible = true
								end)
							end
						end
					end
				end
			end
		end
	end
end

-- ---------- DRAGGABLE / SLIDER / WALKSPEED ----------
local function makeDraggable(guiObject, dragHandle)
	local dragging = false
	local dragStart = nil
	local startPos = nil
	dragHandle.InputBegan:Connect(function(input)
		if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then
			dragging = true
			dragStart = input.Position
			startPos = guiObject.Position
			input.Changed:Connect(function()
				if input.UserInputState == Enum.UserInputState.End then
					dragging = false
					lastUIPosition = guiObject.Position
				end
			end)
		end
	end)
	UserInputService.InputChanged:Connect(function(input)
		if (input.UserInputType == Enum.UserInputType.MouseMovement or input.UserInputType == Enum.UserInputType.Touch) and dragging and dragStart and startPos then
			local delta = input.Position - dragStart
			guiObject.Position = UDim2.new(startPos.X.Scale, startPos.X.Offset + delta.X, startPos.Y.Scale, startPos.Y.Offset + delta.Y)
		end
	end)
end

local function setWalkSpeed(speed)
	local clampedSpeed = math.clamp(tonumber(speed) or gameDefaultWalkSpeed, 0, CONFIG.MAX_WALKSPEED)
	currentWalkSpeedValue = clampedSpeed
end

local function makeSliderDraggable(handle, track)
	handle.InputBegan:Connect(function(input)
		if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then
			local dragging = true
			local connection
			connection = UserInputService.InputChanged:Connect(function(subInput)
				if (subInput.UserInputType == Enum.UserInputType.MouseMovement or subInput.UserInputType == Enum.UserInputType.Touch) and dragging then
					local mousePos = UserInputService:GetMouseLocation()
					local relativePos = mousePos.X - track.AbsolutePosition.X
					local percentage = math.clamp(relativePos / math.max(1, track.AbsoluteSize.X), 0, 1)
					local base = math.min(gameDefaultWalkSpeed, CONFIG.MAX_WALKSPEED)
					local newSpeed = base + (percentage * (CONFIG.MAX_WALKSPEED - base))
					setWalkSpeed(newSpeed)
				end
			end)
			local inputEndedConn
			inputEndedConn = UserInputService.InputEnded:Connect(function(endInput)
				if endInput.UserInputType == Enum.UserInputType.MouseButton1 or endInput.UserInputType == Enum.UserInputType.Touch then
					dragging = false
					if connection and connection.Disconnect then connection:Disconnect() end
					if inputEndedConn and inputEndedConn.Disconnect then inputEndedConn:Disconnect() end
				end
			end)
		end
	end)
end

local function syncAndForceWalkSpeed()
	if localPlayer.Character and localPlayer.Character:FindFirstChildOfClass("Humanoid") then
		local humanoid = localPlayer.Character:FindFirstChildOfClass("Humanoid")
		if humanoid then
			if humanoid.WalkSpeed < currentWalkSpeedValue then
				humanoid.WalkSpeed = currentWalkSpeedValue
			end

			local realSpeed = humanoid.WalkSpeed
			local speedText = tostring(math.floor(realSpeed))
			if speedInput.Text ~= speedText then
				speedInput.Text = speedText
			end

			local denom = (CONFIG.MAX_WALKSPEED - gameDefaultWalkSpeed)
			local percentage = 0
			if denom ~= 0 then
				percentage = (realSpeed - gameDefaultWalkSpeed) / denom
			end
			percentage = math.clamp(percentage, 0, 1)

			pcall(function()
				TweenService:Create(sliderFill, TweenInfo.new(0.1), { Size = UDim2.fromScale(percentage, 1) }):Play()
				TweenService:Create(sliderHandle, TweenInfo.new(0.1), { Position = UDim2.fromScale(percentage, 0.5) }):Play()
			end)
		end
	end
end

-- ---------- CONEXÕES E INICIALIZAÇÃO ----------
makeDraggable(mainFrame, titleContainer)
makeSliderDraggable(sliderHandle, slider)

UserInputService.InputBegan:Connect(function(input, gameProcessed)
	if gameProcessed then return end
	if input.UserInputType == Enum.UserInputType.Keyboard and input.KeyCode == CONFIG.TOGGLE_UI_KEY then
		isUiVisible = not isUiVisible
		mainFrame.Visible = isUiVisible
		blurEffect.Enabled = isUiVisible
		local blurSize = isUiVisible and 12 or 0
		TweenService:Create(blurEffect, TweenInfo.new(0.3), {Size = blurSize}):Play()
		showNotification("🔨", isUiVisible and "ON" or "OFF", 2)
		if isUiVisible then
			mainFrame.Position = lastUIPosition
			populatePlayerList()
		end
	end
end)

toggleAllOnButton.MouseButton1Click:Connect(function()
	for _, player in ipairs(Players:GetPlayers()) do
		if player ~= localPlayer then espTargets[player] = true end
	end
	populatePlayerList()
end)
toggleAllOffButton.MouseButton1Click:Connect(function()
	for _, player in ipairs(Players:GetPlayers()) do
		if player ~= localPlayer then espTargets[player] = false end
	end
	populatePlayerList()
end)

speedInput.FocusLost:Connect(function(enterPressed)
	if enterPressed then setWalkSpeed(speedInput.Text) end
end)

resetButton.MouseButton1Click:Connect(function()
	setWalkSpeed(gameDefaultWalkSpeed)
end)

applyHoverEffect(toggleAllOnButton, COLORS.Accent, COLORS.Accent:Lerp(Color3.new(1,1,1), 0.2))
applyHoverEffect(toggleAllOffButton, COLORS.Red, COLORS.Red:Lerp(Color3.new(1,1,1), 0.2))
applyHoverEffect(resetButton, COLORS.Secondary, COLORS.Stroke)

Players.PlayerAdded:Connect(function(player)
	task.wait(1)
	if mainFrame.Visible then populatePlayerList() end
end)

Players.PlayerRemoving:Connect(function(player)
	if espTargets[player] then espTargets[player] = nil end
	removeDrawingsForPlayer(player)
	if mainFrame.Visible then populatePlayerList() end
end)

localPlayer.CharacterAdded:Connect(function(char)
	task.wait(1)
	local humanoid = char:WaitForChild("Humanoid")
	if humanoid then
		gameDefaultWalkSpeed = humanoid.WalkSpeed or gameDefaultWalkSpeed
		setWalkSpeed(currentWalkSpeedValue)
	end
end)

RunService.RenderStepped:Connect(function()
	updateEsp()
	syncAndForceWalkSpeed()
end)

populatePlayerList()

local function initialize()
	local char = localPlayer.Character or localPlayer.CharacterAdded:Wait()
	local humanoid = char:WaitForChild("Humanoid")
	gameDefaultWalkSpeed = humanoid.WalkSpeed or gameDefaultWalkSpeed
	currentWalkSpeedValue = gameDefaultWalkSpeed
	speedInput.Text = tostring(math.floor(gameDefaultWalkSpeed))
end
initialize()
