-- GUI principal
local screenGui = Instance.new("ScreenGui")
screenGui.Name = "SpeedGui"
screenGui.Parent = game.Players.LocalPlayer:WaitForChild("PlayerGui")

-- Frame principal
local frame = Instance.new("Frame")
frame.Size = UDim2.new(0, 250, 0, 150)
frame.Position = UDim2.new(0.05, 0, 0.7, 0)
frame.BackgroundColor3 = Color3.fromRGB(25, 25, 25)
frame.BackgroundTransparency = 0.1
frame.Active = true
frame.Draggable = true -- arrastável
frame.Parent = screenGui

-- Arredondamento
local corner = Instance.new("UICorner")
corner.CornerRadius = UDim.new(0, 15)
corner.Parent = frame

-- Título
local title = Instance.new("TextLabel")
title.Text = "⚡ Velocidade do Player ⚡"
title.Size = UDim2.new(1, 0, 0, 30)
title.BackgroundTransparency = 1
title.TextColor3 = Color3.fromRGB(255, 255, 255)
title.Font = Enum.Font.GothamBold
title.TextScaled = true
title.Parent = frame

-- Slider (barra)
local sliderFrame = Instance.new("Frame")
sliderFrame.Size = UDim2.new(1, -40, 0, 10)
sliderFrame.Position = UDim2.new(0, 20, 0, 70)
sliderFrame.BackgroundColor3 = Color3.fromRGB(80, 80, 80)
sliderFrame.BorderSizePixel = 0
sliderFrame.Parent = frame

local sliderCorner = Instance.new("UICorner")
sliderCorner.CornerRadius = UDim.new(1, 0)
sliderCorner.Parent = sliderFrame

-- Botão do slider
local knob = Instance.new("Frame")
knob.Size = UDim2.new(0, 20, 0, 20)
knob.Position = UDim2.new(0, -10, 0, -5)
knob.BackgroundColor3 = Color3.fromRGB(50, 150, 250)
knob.BorderSizePixel = 0
knob.Parent = sliderFrame

local knobCorner = Instance.new("UICorner")
knobCorner.CornerRadius = UDim.new(1, 0)
knobCorner.Parent = knob

-- Texto da velocidade
local speedLabel = Instance.new("TextLabel")
speedLabel.Text = "Velocidade: 16"
speedLabel.Size = UDim2.new(1, 0, 0, 30)
speedLabel.Position = UDim2.new(0, 0, 0, 100)
speedLabel.BackgroundTransparency = 1
speedLabel.TextColor3 = Color3.fromRGB(255, 255, 255)
speedLabel.Font = Enum.Font.GothamBold
speedLabel.TextScaled = true
speedLabel.Parent = frame

-- Script funcional
local player = game.Players.LocalPlayer
local humanoid = player.Character or player.CharacterAdded:Wait():WaitForChild("Humanoid")

local uis = game:GetService("UserInputService")
local dragging = false

local function updateSpeed(x)
	local sliderAbsPos = sliderFrame.AbsolutePosition.X
	local sliderAbsSize = sliderFrame.AbsoluteSize.X
	local percent = math.clamp((x - sliderAbsPos) / sliderAbsSize, 0, 1)

	-- posição do botão
	knob.Position = UDim2.new(percent, -10, 0, -5)

	-- velocidade entre 16 e 1000
	local speed = math.floor(16 + (1000 - 16) * percent)
	humanoid.WalkSpeed = speed
	speedLabel.Text = "Velocidade: " .. speed
end

knob.InputBegan:Connect(function(input)
	if input.UserInputType == Enum.UserInputType.MouseButton1 then
		dragging = true
	end
end)

uis.InputEnded:Connect(function(input)
	if input.UserInputType == Enum.UserInputType.MouseButton1 then
		dragging = false
	end
end)

uis.InputChanged:Connect(function(input)
	if dragging and input.UserInputType == Enum.UserInputType.MouseMovement then
		updateSpeed(input.Position.X)
	end
end)
