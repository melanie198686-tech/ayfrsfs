-- ⚙️ CONFIG --------------------------------------------------
local ANIM_ID     = "rbxassetid://179224234"
local FRONT_DIST  = 2
local MOVE_DIST   = 5
local MOVE_SPEED  = 4
local MIN_SPEED   = 1
local MAX_SPEED   = 12
-- ------------------------------------------------------------

local Players      = game:GetService("Players")
local RunService   = game:GetService("RunService")
local UserInput    = game:GetService("UserInputService")
local TweenService = game:GetService("TweenService")

local LocalPlayer = Players.LocalPlayer
local parentGui = (gethui and gethui()) or game:GetService("CoreGui")

local function getHRP(plr)
	local char = plr and plr.Character
	if not char then return nil end
	return char:FindFirstChild("HumanoidRootPart")
end

local function tweenColor(obj, prop, color, time)
	TweenService:Create(obj, TweenInfo.new(time or 0.15), { [prop] = color }):Play()
end

local BRIGHT_TEXT = Color3.fromRGB(255, 255, 255)

local target = nil
local enabled = false
local selectedButton = nil
local track = nil
local moveSpeed = MOVE_SPEED

local function startHugs()
	local char = LocalPlayer.Character
	if not char then return end
	local hum = char:FindFirstChildOfClass("Humanoid")
	if not hum then return end
	local animator = hum:FindFirstChildOfClass("Animator") or hum
	local anim = Instance.new("Animation")
	anim.AnimationId = ANIM_ID
	track = animator:LoadAnimation(anim)
	track.Looped = true
	track:Play()
end

local function stopHugs()
	enabled = false
	if track then track:Stop(); track = nil end
	local myHRP = getHRP(LocalPlayer)
	local char  = LocalPlayer.Character
	local hum   = char and char:FindFirstChildOfClass("Humanoid")
	if myHRP then
		myHRP.AssemblyLinearVelocity = Vector3.zero
		myHRP.AssemblyAngularVelocity = Vector3.zero
		myHRP.Anchored = true
		task.wait(0.15)
		myHRP.AssemblyLinearVelocity = Vector3.zero
		myHRP.AssemblyAngularVelocity = Vector3.zero
		myHRP.Anchored = false
	end
	if hum then hum.Jump = true end
end

local t = 0
RunService.Heartbeat:Connect(function(dt)
	if not enabled or not target then return end
	local myHRP  = getHRP(LocalPlayer)
	local tgtHRP = getHRP(target)
	if not myHRP or not tgtHRP then return end
	t += dt * moveSpeed
	local forward = math.abs(math.sin(t)) * MOVE_DIST
	local pos = (tgtHRP.CFrame * CFrame.new(0, 0, -FRONT_DIST - forward)).Position
	myHRP.CFrame = CFrame.new(pos) * tgtHRP.CFrame.Rotation * CFrame.Angles(math.rad(-90), 0, 0)
	myHRP.AssemblyLinearVelocity = Vector3.zero
end)

local gui = Instance.new("ScreenGui")
gui.Name = "PendulumGetHugs"
gui.ResetOnSpawn = false
gui.ZIndexBehavior = Enum.ZIndexBehavior.Sibling
gui.DisplayOrder = 999
gui.Parent = parentGui

local EXPANDED  = UDim2.new(0, 230, 0, 348)
local COLLAPSED = UDim2.new(0, 230, 0, 36)

local main = Instance.new("Frame")
main.Size = EXPANDED
main.Position = UDim2.new(0, 40, 0, 80)
main.BackgroundColor3 = Color3.fromRGB(14, 20, 38)
main.BorderSizePixel = 0
main.Active = true
main.ClipsDescendants = true
main.Parent = gui

local mainCorner = Instance.new("UICorner")
mainCorner.CornerRadius = UDim.new(0, 8)
mainCorner.Parent = main

local mainStroke = Instance.new("UIStroke")
mainStroke.Thickness = 2
mainStroke.Color = Color3.fromRGB(60, 130, 255)
mainStroke.Parent = main

local titleBar = Instance.new("Frame")
titleBar.Size = UDim2.new(1, 0, 0, 36)
titleBar.BackgroundColor3 = Color3.fromRGB(28, 55, 110)
titleBar.BorderSizePixel = 0
titleBar.Parent = main

local titleCorner = Instance.new("UICorner")
titleCorner.CornerRadius = UDim.new(0, 8)
titleCorner.Parent = titleBar

local titlePatch = Instance.new("Frame")
titlePatch.Size = UDim2.new(1, 0, 0, 10)
titlePatch.Position = UDim2.new(0, 0, 1, -10)
titlePatch.BackgroundColor3 = Color3.fromRGB(28, 55, 110)
titlePatch.BorderSizePixel = 0
titlePatch.ZIndex = 0
titlePatch.Parent = titleBar

local titleGrad = Instance.new("UIGradient")
titleGrad.Rotation = 90
titleGrad.Color = ColorSequence.new({
	ColorSequenceKeypoint.new(0, Color3.fromRGB(40, 75, 150)),
	ColorSequenceKeypoint.new(1, Color3.fromRGB(20, 40, 85)),
})
titleGrad.Parent = titleBar

local titleText = Instance.new("TextLabel")
titleText.Size = UDim2.new(1, -48, 1, 0)
titleText.Position = UDim2.new(0, 12, 0, 0)
titleText.BackgroundTransparency = 1
titleText.Text = "✦ Pendulum Get Hugs"
titleText.TextColor3 = Color3.fromRGB(225, 235, 255)
titleText.TextXAlignment = Enum.TextXAlignment.Left
titleText.Font = Enum.Font.GothamBold
titleText.TextSize = 16
titleText.Parent = titleBar

local minBtn = Instance.new("TextButton")
minBtn.Size = UDim2.new(0, 26, 0, 26)
minBtn.Position = UDim2.new(1, -32, 0, 5)
minBtn.BackgroundColor3 = Color3.fromRGB(30, 60, 120)
minBtn.BorderSizePixel = 0
minBtn.Text = "–"
minBtn.TextColor3 = Color3.fromRGB(225, 235, 255)
minBtn.Font = Enum.Font.GothamBold
minBtn.TextSize = 22
minBtn.ZIndex = 3
minBtn.Parent = titleBar

local minCorner = Instance.new("UICorner")
minCorner.CornerRadius = UDim.new(0, 6)
minCorner.Parent = minBtn

local function makeCoolButton(parent, size, pos, text, baseTop, baseBottom, accentColor)
	local btn = Instance.new("TextButton")
	btn.Size = size
	btn.Position = pos
	btn.BackgroundColor3 = baseTop
	btn.BorderSizePixel = 0
	btn.Text = ""
	btn.AutoButtonColor = false
	btn.ClipsDescendants = true
	btn.Parent = parent
	local corner = Instance.new("UICorner")
	corner.CornerRadius = UDim.new(0, 6)
	corner.Parent = btn
	local grad = Instance.new("UIGradient")
	grad.Rotation = 90
	grad.Color = ColorSequence.new({
		ColorSequenceKeypoint.new(0, baseTop),
		ColorSequenceKeypoint.new(1, baseBottom),
	})
	grad.Parent = btn
	local stroke = Instance.new("UIStroke")
	stroke.Thickness = 1.5
	stroke.Color = accentColor
	stroke.Transparency = 0.2
	stroke.ApplyStrokeMode = Enum.ApplyStrokeMode.Border
	stroke.Parent = btn
	local textLabel = Instance.new("TextLabel")
	textLabel.BackgroundTransparency = 1
	textLabel.Size = UDim2.new(1, -16, 1, 0)
	textLabel.Position = UDim2.new(0, 12, 0, 0)
	textLabel.Text = text
	textLabel.TextColor3 = BRIGHT_TEXT
	textLabel.Font = Enum.Font.GothamBold
	textLabel.TextSize = 15
	textLabel.TextXAlignment = Enum.TextXAlignment.Center
	textLabel.ZIndex = 2
	textLabel.Parent = btn
	local txtStroke = Instance.new("UIStroke")
	txtStroke.Thickness = 2
	txtStroke.Color = Color3.fromRGB(6, 12, 25)
	txtStroke.Transparency = 0
	txtStroke.ApplyStrokeMode = Enum.ApplyStrokeMode.Contextual
	txtStroke.Parent = textLabel
	local accent = Instance.new("Frame")
	accent.Size = UDim2.new(0, 4, 1, -8)
	accent.Position = UDim2.new(0, 4, 0, 4)
	accent.BackgroundColor3 = accentColor
	accent.BorderSizePixel = 0
	accent.ZIndex = 2
	accent.Parent = btn
	local accentCorner = Instance.new("UICorner")
	accentCorner.CornerRadius = UDim.new(1, 0)
	accentCorner.Parent = accent
	return btn, stroke, accent, grad, textLabel
end

local toggle, toggleStroke, toggleAccent, _, toggleLabel = makeCoolButton(
	main, UDim2.new(1, -24, 0, 40), UDim2.new(0, 12, 0, 46), "◈  OFF",
	Color3.fromRGB(35, 55, 95), Color3.fromRGB(22, 35, 62), Color3.fromRGB(60, 130, 255))

toggle.MouseEnter:Connect(function() tweenColor(toggleStroke, "Transparency", 0) end)
toggle.MouseLeave:Connect(function() tweenColor(toggleStroke, "Transparency", 0.2) end)

toggle.MouseButton1Click:Connect(function()
	if not enabled then
		if not target then return end
		enabled = true
		toggleLabel.Text = "◈  ON"
		tweenColor(toggle, "BackgroundColor3", Color3.fromRGB(45, 110, 235))
		tweenColor(toggleAccent, "BackgroundColor3", Color3.fromRGB(180, 215, 255))
		startHugs()
	else
		toggleLabel.Text = "◈  OFF"
		tweenColor(toggle, "BackgroundColor3", Color3.fromRGB(35, 55, 95))
		tweenColor(toggleAccent, "BackgroundColor3", Color3.fromRGB(60, 130, 255))
		stopHugs()
	end
end)

local list = Instance.new("ScrollingFrame")
list.Size = UDim2.new(1, -24, 1, -140)
list.Position = UDim2.new(0, 12, 0, 92)
list.BackgroundColor3 = Color3.fromRGB(10, 16, 30)
list.BorderSizePixel = 0
list.ScrollBarThickness = 4
list.ScrollBarImageColor3 = Color3.fromRGB(60, 130, 255)
list.CanvasSize = UDim2.new(0, 0, 0, 0)
list.Parent = main

local listCorner = Instance.new("UICorner")
listCorner.CornerRadius = UDim.new(0, 6)
listCorner.Parent = list

local layout = Instance.new("UIListLayout")
layout.Padding = UDim.new(0, 5)
layout.SortOrder = Enum.SortOrder.Name
layout.HorizontalAlignment = Enum.HorizontalAlignment.Center
layout.Parent = list

local listPad = Instance.new("UIPadding")
listPad.PaddingTop = UDim.new(0, 5)
listPad.Parent = list

layout:GetPropertyChangedSignal("AbsoluteContentSize"):Connect(function()
	list.CanvasSize = UDim2.new(0, 0, 0, layout.AbsoluteContentSize.Y + 10)
end)

local sliderLabel = Instance.new("TextLabel")
sliderLabel.Size = UDim2.new(1, -24, 0, 16)
sliderLabel.Position = UDim2.new(0, 12, 1, -44)
sliderLabel.BackgroundTransparency = 1
sliderLabel.Text = "Speed: " .. string.format("%.1f", moveSpeed)
sliderLabel.TextColor3 = BRIGHT_TEXT
sliderLabel.TextXAlignment = Enum.TextXAlignment.Left
sliderLabel.Font = Enum.Font.GothamBold
sliderLabel.TextSize = 13
sliderLabel.Parent = main

local sliderTrack = Instance.new("Frame")
sliderTrack.Size = UDim2.new(1, -24, 0, 8)
sliderTrack.Position = UDim2.new(0, 12, 1, -22)
sliderTrack.BackgroundColor3 = Color3.fromRGB(28, 42, 72)
sliderTrack.BorderSizePixel = 0
sliderTrack.Parent = main

local trackCorner = Instance.new("UICorner")
trackCorner.CornerRadius = UDim.new(1, 0)
trackCorner.Parent = sliderTrack

local sliderFill = Instance.new("Frame")
sliderFill.Size = UDim2.new((moveSpeed - MIN_SPEED) / (MAX_SPEED - MIN_SPEED), 0, 1, 0)
sliderFill.BackgroundColor3 = Color3.fromRGB(45, 110, 235)
sliderFill.BorderSizePixel = 0
sliderFill.Parent = sliderTrack

local fillCorner = Instance.new("UICorner")
fillCorner.CornerRadius = UDim.new(1, 0)
fillCorner.Parent = sliderFill

local knob = Instance.new("Frame")
knob.Size = UDim2.new(0, 16, 0, 16)
knob.AnchorPoint = Vector2.new(0.5, 0.5)
knob.Position = UDim2.new((moveSpeed - MIN_SPEED) / (MAX_SPEED - MIN_SPEED), 0, 0.5, 0)
knob.BackgroundColor3 = Color3.fromRGB(255, 255, 255)
knob.BorderSizePixel = 0
knob.ZIndex = 2
knob.Parent = sliderTrack

local knobCorner = Instance.new("UICorner")
knobCorner.CornerRadius = UDim.new(1, 0)
knobCorner.Parent = knob

local knobStroke = Instance.new("UIStroke")
knobStroke.Thickness = 2
knobStroke.Color = Color3.fromRGB(45, 110, 235)
knobStroke.Parent = knob

local draggingSlider = false
local function updateSlider(inputX)
	local rel = math.clamp((inputX - sliderTrack.AbsolutePosition.X) / sliderTrack.AbsoluteSize.X, 0, 1)
	moveSpeed = MIN_SPEED + rel * (MAX_SPEED - MIN_SPEED)
	sliderFill.Size = UDim2.new(rel, 0, 1, 0)
	knob.Position = UDim2.new(rel, 0, 0.5, 0)
	sliderLabel.Text = "Speed: " .. string.format("%.1f", moveSpeed)
end

sliderTrack.InputBegan:Connect(function(input)
	if input.UserInputType == Enum.UserInputType.MouseButton1
	or input.UserInputType == Enum.UserInputType.Touch then
		draggingSlider = true
		updateSlider(input.Position.X)
	end
end)
knob.InputBegan:Connect(function(input)
	if input.UserInputType == Enum.UserInputType.MouseButton1
	or input.UserInputType == Enum.UserInputType.Touch then
		draggingSlider = true
	end
end)

local minimized = false
minBtn.MouseButton1Click:Connect(function()
	minimized = not minimized
	if minimized then
		minBtn.Text = "+"
		toggle.Visible = false
		list.Visible = false
		sliderLabel.Visible = false
		sliderTrack.Visible = false
		TweenService:Create(main, TweenInfo.new(0.22, Enum.EasingStyle.Quad), { Size = COLLAPSED }):Play()
	else
		minBtn.Text = "–"
		TweenService:Create(main, TweenInfo.new(0.22, Enum.EasingStyle.Quad), { Size = EXPANDED }):Play()
		task.delay(0.22, function()
			if not minimized then
				toggle.Visible = true
				list.Visible = true
				sliderLabel.Visible = true
				sliderTrack.Visible = true
			end
		end)
	end
end)

local BTN_TOP, BTN_BOTTOM = Color3.fromRGB(30, 48, 82), Color3.fromRGB(20, 32, 56)
local SEL_TOP, SEL_BOTTOM = Color3.fromRGB(45, 110, 235), Color3.fromRGB(28, 65, 140)

local function refreshList()
	for _, c in ipairs(list:GetChildren()) do
		if c:IsA("TextButton") then c:Destroy() end
	end
	selectedButton = nil
	for _, plr in ipairs(Players:GetPlayers()) do
		if plr ~= LocalPlayer then
			local btn, stroke, accent, grad, lbl = makeCoolButton(
				list, UDim2.new(1, -6, 0, 32), UDim2.new(0, 0, 0, 0),
				plr.DisplayName, BTN_TOP, BTN_BOTTOM, Color3.fromRGB(60, 130, 255))
			lbl.TextSize = 13
			btn.MouseEnter:Connect(function()
				if btn ~= selectedButton then
					tweenColor(stroke, "Transparency", 0)
					tweenColor(btn, "BackgroundColor3", Color3.fromRGB(38, 62, 105))
				end
			end)
			btn.MouseLeave:Connect(function()
				if btn ~= selectedButton then
					tweenColor(stroke, "Transparency", 0.2)
					tweenColor(btn, "BackgroundColor3", BTN_TOP)
				end
			end)
			btn.MouseButton1Click:Connect(function()
				target = plr
				if selectedButton and selectedButton ~= btn then
					local pg = selectedButton:FindFirstChildOfClass("UIGradient")
					if pg then pg.Color = ColorSequence.new({
						ColorSequenceKeypoint.new(0, BTN_TOP),
						ColorSequenceKeypoint.new(1, BTN_BOTTOM) }) end
					tweenColor(selectedButton, "BackgroundColor3", BTN_TOP)
				end
				selectedButton = btn
				grad.Color = ColorSequence.new({
					ColorSequenceKeypoint.new(0, SEL_TOP),
					ColorSequenceKeypoint.new(1, SEL_BOTTOM) })
				tweenColor(btn, "BackgroundColor3", SEL_TOP)
			end)
		end
	end
end

refreshList()
Players.PlayerAdded:Connect(refreshList)
Players.PlayerRemoving:Connect(function(plr)
	if plr == target then
		target = nil
		if enabled then
			toggleLabel.Text = "◈  OFF"
			tweenColor(toggle, "BackgroundColor3", Color3.fromRGB(35, 55, 95))
			tweenColor(toggleAccent, "BackgroundColor3", Color3.fromRGB(60, 130, 255))
			stopHugs()
		end
	end
	task.defer(refreshList)
end)

local dragging, dragStart, startPos
titleBar.InputBegan:Connect(function(input)
	if input.UserInputType == Enum.UserInputType.MouseButton1
	or input.UserInputType == Enum.UserInputType.Touch then
		dragging = true; dragStart = input.Position; startPos = main.Position
	end
end)
UserInput.InputChanged:Connect(function(input)
	if input.UserInputType == Enum.UserInputType.MouseMovement
	or input.UserInputType == Enum.UserInputType.Touch then
		if draggingSlider then
			updateSlider(input.Position.X)
		elseif dragging then
			local delta = input.Position - dragStart
			main.Position = UDim2.new(startPos.X.Scale, startPos.X.Offset + delta.X,
				startPos.Y.Scale, startPos.Y.Offset + delta.Y)
		end
	end
end)
UserInput.InputEnded:Connect(function(input)
	if input.UserInputType == Enum.UserInputType.MouseButton1
	or input.UserInputType == Enum.UserInputType.Touch then
		dragging = false
		draggingSlider = false
	end
end)
