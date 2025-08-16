-- Tentativa segura de pegar o player
local Players = game:GetService("Players")
local UserInputService = game:GetService("UserInputService")

local player
pcall(function() player = Players.LocalPlayer end)
if not player then return end -- aborta se não encontrar LocalPlayer

-- Cria ScreenGui seguro
local gui = Instance.new("ScreenGui")
gui.Name = "SpeedGui"
gui.ResetOnSpawn = false
gui.Parent = player:FindFirstChild("PlayerGui") or nil
if not gui.Parent then return end

-- Frame principal
local frame = Instance.new("Frame")
frame.Size = UDim2.new(0, 260, 0, 170)
frame.Position = UDim2.new(0.05, 0, 0.65, 0)
frame.BackgroundColor3 = Color3.fromRGB(25, 25, 25)
frame.BorderSizePixel = 0
frame.Active = true
frame.Parent = gui

local corner = Instance.new("UICorner", frame)
corner.CornerRadius = UDim.new(0, 16)

-- Título
local title = Instance.new("TextLabel")
title.Size = UDim2.new(1, -16, 0, 36)
title.Position = UDim2.new(0, 8, 0, 6)
title.BackgroundTransparency = 1
title.Text = "⚡ Velocidade do Player"
title.TextColor3 = Color3.fromRGB(255, 255, 255)
title.Font = Enum.Font.GothamBold
title.TextScaled = true
title.Parent = frame

-- Slider
local slider = Instance.new("Frame")
slider.Size = UDim2.new(1, -40, 0, 12)
slider.Position = UDim2.new(0, 20, 0, 78)
slider.BackgroundColor3 = Color3.fromRGB(70, 70, 70)
slider.BorderSizePixel = 0
slider.Parent = frame

local sliderCorner = Instance.new("UICorner", slider)
sliderCorner.CornerRadius = UDim.new(1, 0)

-- Knob
local knob = Instance.new("Frame")
knob.Size = UDim2.new(0, 24, 0, 24)
knob.AnchorPoint = Vector2.new(0.5, 0.5)
knob.Position = UDim2.new(0, 12, 0.5, 0)
knob.BackgroundColor3 = Color3.fromRGB(50, 150, 250)
knob.BorderSizePixel = 0
knob.Parent = slider

local knobCorner = Instance.new("UICorner", knob)
knobCorner.CornerRadius = UDim.new(1, 0)

-- Texto da velocidade
local speedLabel = Instance.new("TextLabel")
speedLabel.Size = UDim2.new(1, -16, 0, 32)
speedLabel.Position = UDim2.new(0, 8, 0, 118)
speedLabel.BackgroundTransparency = 1
speedLabel.TextColor3 = Color3.fromRGB(255, 255, 255)
speedLabel.Font = Enum.Font.GothamBold
speedLabel.TextScaled = true
speedLabel.Parent = frame

-- Função segura para pegar humanoid
local minSpeed, maxSpeed = 16, 1000
local currentSpeed = minSpeed

local function getHumanoid()
	if not player then return nil end
	local char = player.Character or player.CharacterAdded:Wait()
	return char:FindFirstChildWhichIsA("Humanoid")
end

local humanoid = getHumanoid()
if humanoid then humanoid.WalkSpeed = currentSpeed end

player.CharacterAdded:Connect(function(char)
	humanoid = char:WaitForChild("Humanoid")
	humanoid.WalkSpeed = currentSpeed
end)

-- Ajuste da velocidade pelo slider
local sliding = false
local function setSpeedFromPercent(p)
	p = math.clamp(p, 0, 1)
	currentSpeed = math.floor(minSpeed + (maxSpeed - minSpeed) * p)
	if humanoid and humanoid.Parent then
		humanoid.WalkSpeed = currentSpeed
	end
	speedLabel.Text = ("Velocidade: %d"):format(currentSpeed)
end

local function updateFromX(x)
	local left = slider.AbsolutePosition.X
	local width = slider.AbsoluteSize.X
	local p = (x - left) / width
	pcall(setSpeedFromPercent, p)
end

-- Eventos de mouse/toque
local UserInput = UserInputService
local function beginSlide(input)
	sliding = true
	updateFromX(input.Position.X)
	input.Changed:Connect(function()
		if input.UserInputState == Enum.UserInputState.End then sliding = false end
	end)
end

for _, part in ipairs({slider, knob}) do
	part.InputBegan:Connect(function(input)
		if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then
			beginSlide(input)
		end
	end)
end

UserInput.InputChanged:Connect(function(input)
	if sliding and (input.UserInputType == Enum.UserInputType.MouseMovement or input.UserInputType == Enum.UserInputType.Touch) then
		updateFromX(input.Position.X)
	end
end)

-- Arrastar janela
local dragging = false
local dragStart, startPos

local function beginDrag(input)
	dragging = true
	dragStart = input.Position
	startPos = frame.Position
	input.Changed:Connect(function()
		if input.UserInputState == Enum.UserInputState.End then dragging = false end
	end)
end

local function updateDrag(input)
	if not dragging then return end
	local delta = input.Position - dragStart
	frame.Position = UDim2.new(startPos.X.Scale, startPos.X.Offset + delta.X, startPos.Y.Scale, startPos.Y.Offset + delta.Y)
end

for _, area in ipairs({frame, title}) do
	area.InputBegan:Connect(function(input)
		if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then
			beginDrag(input)
		end
	end)
	area.InputChanged:Connect(function(input)
		if input.UserInputType == Enum.UserInputType.MouseMovement or input.UserInputType == Enum.UserInputType.Touch then
			updateDrag(input)
		end
	end)
end

UserInput.InputChanged:Connect(function(input)
	updateDrag(input)
end)
