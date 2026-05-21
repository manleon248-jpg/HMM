--// ARIS HUB ULTIMATE - CLEAN VERSION + AIMBOT INTEGRATED

local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local UserInputService = game:GetService("UserInputService")
local TweenService = game:GetService("TweenService")

local LocalPlayer = Players.LocalPlayer
local Camera = workspace.CurrentCamera
repeat task.wait() until LocalPlayer and LocalPlayer:FindFirstChild("PlayerGui")

--// CONFIG
local config = {
	walkSpeedValue = 50,
	menuKey = Enum.KeyCode.RightControl,
	speedKey = Enum.KeyCode.V,
	espKey = Enum.KeyCode.E,
	jumpKey = Enum.KeyCode.Space,
	noclipKey = Enum.KeyCode.N,
	flyKey = Enum.KeyCode.F,
	aimbotKey = Enum.KeyCode.MouseButton2,
	
	aimbotEnabled = false,
	aimbotFOV = 150,
	aimbotSmoothness = 0.25,
	
	accent = Color3.fromRGB(255, 0, 0)
}

--// STATES
local menuOpen = true
local speedEnabled = false
local espEnabled = false
local infJumpEnabled = false
local noclipEnabled = false
local flyEnabled = false
local espData = {}
local currentPage = nil

--// SOUNDS
local function playSound(id, vol)
	local s = Instance.new("Sound", game:GetService("SoundService"))
	s.SoundId = "rbxassetid://" .. id
	s.Volume = vol or 0.5
	s:Play()
	game:GetService("Debris"):AddItem(s, 2)
end

--// COLORS
local COLORS = {
	BG = Color3.fromRGB(12, 12, 12),
	SIDEBAR = Color3.fromRGB(8, 8, 8),
	ACCENT = Color3.fromRGB(255, 0, 0),
	TEXT = Color3.fromRGB(240, 240, 240),
	SUBTEXT = Color3.fromRGB(150, 150, 150),
	ITEM_BG = Color3.fromRGB(20, 20, 20)
}

--// FOV CIRCLE
local fovCircle = Drawing.new("Circle")
fovCircle.Visible = false
fovCircle.Radius = config.aimbotFOV
fovCircle.Color = COLORS.ACCENT
fovCircle.Thickness = 1
fovCircle.Filled = false
fovCircle.Transparency = 1

--// GUI
local ScreenGui = Instance.new("ScreenGui")
ScreenGui.Name = "ArisHubUltimate"
ScreenGui.ResetOnSpawn = false
ScreenGui.Parent = LocalPlayer.PlayerGui

local MainFrame = Instance.new("Frame")
MainFrame.Size = UDim2.new(0, 500, 0, 350)
MainFrame.Position = UDim2.new(0.5, -250, 0.5, -175)
MainFrame.BackgroundColor3 = COLORS.BG
MainFrame.BorderSizePixel = 0
MainFrame.ClipsDescendants = true
MainFrame.Parent = ScreenGui

local UICorner = Instance.new("UICorner", MainFrame)
UICorner.CornerRadius = UDim.new(0, 10)

local Stroke = Instance.new("UIStroke", MainFrame)
Stroke.Color = COLORS.ACCENT
Stroke.Thickness = 1.5
Stroke.Transparency = 0.2

--// SIDEBAR
local Sidebar = Instance.new("Frame")
Sidebar.Size = UDim2.new(0, 130, 1, 0)
Sidebar.BackgroundColor3 = COLORS.SIDEBAR
Sidebar.BorderSizePixel = 0
Sidebar.Parent = MainFrame

local SidebarCorner = Instance.new("UICorner", Sidebar)
SidebarCorner.CornerRadius = UDim.new(0, 10)

local SidebarTitle = Instance.new("TextLabel")
SidebarTitle.Size = UDim2.new(1, 0, 0, 50)
SidebarTitle.BackgroundTransparency = 1
SidebarTitle.Text = "ARIS HUB"
SidebarTitle.Font = Enum.Font.GothamBold
SidebarTitle.TextSize = 16
SidebarTitle.TextColor3 = COLORS.ACCENT
SidebarTitle.Parent = Sidebar

local SidebarContainer = Instance.new("Frame")
SidebarContainer.Size = UDim2.new(1, 0, 1, -60)
SidebarContainer.Position = UDim2.new(0, 0, 0, 60)
SidebarContainer.BackgroundTransparency = 1
SidebarContainer.Parent = Sidebar

local SidebarLayout = Instance.new("UIListLayout", SidebarContainer)
SidebarLayout.Padding = UDim.new(0, 3)
SidebarLayout.HorizontalAlignment = Enum.HorizontalAlignment.Center

--// CONTENT AREA
local Content = Instance.new("Frame")
Content.Size = UDim2.new(1, -145, 1, -20)
Content.Position = UDim2.new(0, 140, 0, 10)
Content.BackgroundTransparency = 1
Content.Parent = MainFrame

--// DRAGGING
local dragging, dragStart, startPos
MainFrame.InputBegan:Connect(function(input)
	if input.UserInputType == Enum.UserInputType.MouseButton1 then
		dragging = true
		dragStart = input.Position
		startPos = MainFrame.Position
	end
end)
UserInputService.InputChanged:Connect(function(input)
	if dragging and input.UserInputType == Enum.UserInputType.MouseMovement then
		local delta = input.Position - dragStart
		MainFrame.Position = UDim2.new(startPos.X.Scale, startPos.X.Offset + delta.X, startPos.Y.Scale, startPos.Y.Offset + delta.Y)
	end
end)
UserInputService.InputEnded:Connect(function(input)
	if input.UserInputType == Enum.UserInputType.MouseButton1 then dragging = false end
end)

--// PAGES
local pages = {}
local function createPage(name)
	local page = Instance.new("ScrollingFrame")
	page.Size = UDim2.new(1, 0, 1, 0)
	page.BackgroundTransparency = 1
	page.Visible = false
	page.BorderSizePixel = 0
	page.ScrollBarThickness = 2
	page.ScrollBarImageColor3 = COLORS.ACCENT
	page.Parent = Content
	
	local layout = Instance.new("UIListLayout", page)
	layout.Padding = UDim.new(0, 6)
	layout.HorizontalAlignment = Enum.HorizontalAlignment.Center
	
	pages[name] = page
	return page
end

local mainPage = createPage("MAIN")
local visualsPage = createPage("VISUALS")
local aimbotPage = createPage("AIMBOT")
local configPage = createPage("CONFIG")
local keysPage = createPage("KEYS")

local function showPage(name)
	if currentPage then pages[currentPage].Visible = false end
	pages[name].Visible = true
	currentPage = name
	playSound(12222242, 0.25)
end

--// SIDEBAR BUTTONS
local function createTab(name)
	local btn = Instance.new("TextButton")
	btn.Size = UDim2.new(0.85, 0, 0, 32)
	btn.BackgroundColor3 = Color3.fromRGB(15, 15, 15)
	btn.Text = name
	btn.TextColor3 = COLORS.SUBTEXT
	btn.Font = Enum.Font.GothamMedium
	btn.TextSize = 12
	btn.Parent = SidebarContainer
	Instance.new("UICorner", btn).CornerRadius = UDim.new(0, 5)
	
	btn.MouseEnter:Connect(function()
		TweenService:Create(btn, TweenInfo.new(0.2), {BackgroundColor3 = Color3.fromRGB(25, 25, 25), TextColor3 = COLORS.TEXT}):Play()
	end)
	btn.MouseLeave:Connect(function()
		if currentPage ~= name then
			TweenService:Create(btn, TweenInfo.new(0.2), {BackgroundColor3 = Color3.fromRGB(15, 15, 15), TextColor3 = COLORS.SUBTEXT}):Play()
		end
	end)
	
	btn.MouseButton1Click:Connect(function()
		showPage(name)
		for _, b in pairs(SidebarContainer:GetChildren()) do
			if b:IsA("TextButton") and b.Text ~= name then
				TweenService:Create(b, TweenInfo.new(0.2), {BackgroundColor3 = Color3.fromRGB(15, 15, 15), TextColor3 = COLORS.SUBTEXT}):Play()
			end
		end
		TweenService:Create(btn, TweenInfo.new(0.2), {BackgroundColor3 = Color3.fromRGB(35, 0, 0), TextColor3 = COLORS.ACCENT}):Play()
	end)
end

createTab("MAIN")
createTab("VISUALS")
createTab("AIMBOT")
createTab("CONFIG")
createTab("KEYS")

--// UI ELEMENTS
local function createToggle(name, default, callback, parent)
	local btn = Instance.new("TextButton")
	btn.Size = UDim2.new(0.95, 0, 0, 38)
	btn.BackgroundColor3 = COLORS.ITEM_BG
	btn.Text = "   " .. name
	btn.TextColor3 = default and COLORS.TEXT or COLORS.SUBTEXT
	btn.TextXAlignment = Enum.TextXAlignment.Left
	btn.Font = Enum.Font.GothamMedium
	btn.TextSize = 13
	btn.Parent = parent
	Instance.new("UICorner", btn).CornerRadius = UDim.new(0, 6)
	
	local indicator = Instance.new("Frame")
	indicator.Size = UDim2.new(0, 10, 0, 10)
	indicator.Position = UDim2.new(1, -22, 0.5, -5)
	indicator.BackgroundColor3 = default and COLORS.ACCENT or Color3.fromRGB(40, 40, 40)
	indicator.Parent = btn
	Instance.new("UICorner", indicator).CornerRadius = UDim.new(1, 0)
	
	local state = default
	btn.MouseButton1Click:Connect(function()
		state = not state
		playSound(12222242, 0.3)
		TweenService:Create(indicator, TweenInfo.new(0.2), {BackgroundColor3 = state and COLORS.ACCENT or Color3.fromRGB(40, 40, 40)}):Play()
		TweenService:Create(btn, TweenInfo.new(0.2), {TextColor3 = state and COLORS.TEXT or COLORS.SUBTEXT}):Play()
		callback(state)
	end)
	
	return {
		SetState = function(ns)
			state = ns
			indicator.BackgroundColor3 = state and COLORS.ACCENT or Color3.fromRGB(40, 40, 40)
			btn.TextColor3 = state and COLORS.TEXT or COLORS.SUBTEXT
		end
	}
end

local function createSlider(name, min, max, default, callback, parent)
	local frame = Instance.new("Frame")
	frame.Size = UDim2.new(0.95, 0, 0, 48)
	frame.BackgroundColor3 = Color3.fromRGB(18, 18, 18)
	frame.Parent = parent
	Instance.new("UICorner", frame).CornerRadius = UDim.new(0, 6)
	
	local label = Instance.new("TextLabel")
	label.Size = UDim2.new(1, -20, 0, 20)
	label.Position = UDim2.new(0, 10, 0, 5)
	label.BackgroundTransparency = 1
	label.Text = name .. ": " .. default
	label.TextColor3 = COLORS.TEXT
	label.Font = Enum.Font.Gotham
	label.TextSize = 11
	label.TextXAlignment = Enum.TextXAlignment.Left
	label.Parent = frame
	
	local bar = Instance.new("Frame")
	bar.Size = UDim2.new(1, -20, 0, 3)
	bar.Position = UDim2.new(0, 10, 0, 35)
	bar.BackgroundColor3 = Color3.fromRGB(35, 35, 35)
	bar.Parent = frame
	
	local fill = Instance.new("Frame")
	fill.Size = UDim2.new((default - min) / (max - min), 0, 1, 0)
	fill.BackgroundColor3 = COLORS.ACCENT
	fill.Parent = bar
	
	local drag = false
	local function update(input)
		local perc = math.clamp((input.Position.X - bar.AbsolutePosition.X) / bar.AbsoluteSize.X, 0, 1)
		local val = math.floor(min + (max - min) * perc)
		fill.Size = UDim2.new(perc, 0, 1, 0)
		label.Text = name .. ": " .. val
		callback(val)
	end
	
	bar.InputBegan:Connect(function(input) if input.UserInputType == Enum.UserInputType.MouseButton1 then drag = true update(input) end end)
	UserInputService.InputChanged:Connect(function(input) if drag and input.UserInputType == Enum.UserInputType.MouseMovement then update(input) end end)
	UserInputService.InputEnded:Connect(function(input) if input.UserInputType == Enum.UserInputType.MouseButton1 then drag = false end end)
end

local function createKeybind(name, default, callback, parent)
	local btn = Instance.new("TextButton")
	btn.Size = UDim2.new(0.95, 0, 0, 38)
	btn.BackgroundColor3 = Color3.fromRGB(20, 20, 20)
	btn.Text = "   " .. name
	btn.TextColor3 = COLORS.SUBTEXT
	btn.TextXAlignment = Enum.TextXAlignment.Left
	btn.Font = Enum.Font.Gotham
	btn.TextSize = 12
	btn.Parent = parent
	Instance.new("UICorner", btn).CornerRadius = UDim.new(0, 6)
	
	local keyLabel = Instance.new("TextLabel")
	keyLabel.Size = UDim2.new(0, 70, 0, 18)
	keyLabel.Position = UDim2.new(1, -80, 0.5, -9)
	keyLabel.BackgroundColor3 = Color3.fromRGB(25, 0, 0)
	keyLabel.Text = default.Name
	keyLabel.TextColor3 = COLORS.ACCENT
	keyLabel.Font = Enum.Font.GothamBold
	keyLabel.TextSize = 10
	keyLabel.Parent = btn
	Instance.new("UICorner", keyLabel).CornerRadius = UDim.new(0, 3)
	
	local listening = false
	btn.MouseButton1Click:Connect(function()
		listening = true
		keyLabel.Text = "..."
		playSound(12222242, 0.2)
	end)
	
	UserInputService.InputBegan:Connect(function(input)
		if listening then
			if input.KeyCode ~= Enum.KeyCode.Unknown then
				listening = false
				keyLabel.Text = (input.UserInputType == Enum.UserInputType.MouseButton2 and "M2" or input.KeyCode.Name)
				callback(input.KeyCode)
			elseif input.UserInputType == Enum.UserInputType.MouseButton2 then
				listening = false
				keyLabel.Text = "M2"
				callback(Enum.KeyCode.MouseButton2)
			end
		end
	end)
end

--// AIMBOT LOGIC
local function getClosestPlayer()
	local target = nil
	local dist = config.aimbotFOV
	
	for _, player in pairs(Players:GetPlayers()) do
		if player ~= LocalPlayer and player.Character and player.Character:FindFirstChild("Head") then
			local pos, onScreen = Camera:WorldToViewportPoint(player.Character.Head.Position)
			if onScreen then
				local magnitude = (Vector2.new(pos.X, pos.Y) - Vector2.new(Camera.ViewportSize.X/2, Camera.ViewportSize.Y/2)).Magnitude
				if magnitude < dist then
					dist = magnitude
					target = player
				end
			end
		end
	end
	return target
end

--// FEATURE LOGIC
local function removeESP()
    for _, player in pairs(Players:GetPlayers()) do
        if player.Character then
            local char = player.Character
            if char:FindFirstChild("PlayerESP") then char.PlayerESP:Destroy() end
            local head = char:FindFirstChild("Head")
            if head and head:FindFirstChild("NameESP") then head.NameESP:Destroy() end
        end
    end
    espData = {}
end

local function createESP(player, character)
    if player == LocalPlayer or not character then return end
    local head = character:WaitForChild("Head", 5)
    if not head then return end
    local highlight = Instance.new("Highlight", character)
    highlight.Name = "PlayerESP"
    highlight.FillColor = COLORS.ACCENT
    highlight.FillTransparency = 0.5
    highlight.DepthMode = Enum.HighlightDepthMode.AlwaysOnTop
    local bb = Instance.new("BillboardGui", head)
    bb.Name = "NameESP"
    bb.Size = UDim2.new(0, 200, 0, 50)
    bb.StudsOffset = Vector3.new(0, 2.5, 0)
    bb.AlwaysOnTop = true
    local text = Instance.new("TextLabel", bb)
    text.Size = UDim2.new(1, 0, 1, 0)
    text.BackgroundTransparency = 1
    text.TextColor3 = COLORS.ACCENT
    text.Font = Enum.Font.SourceSansBold
    text.TextSize = 18
    espData[player] = { text = text, character = character }
end

--// FLY LOGIC
local flyPart
local flySpeed = 50
local function toggleFly(v)
	flyEnabled = v
	local char = LocalPlayer.Character
	if not char then return end
	local hrp = char:FindFirstChild("HumanoidRootPart")
	if not hrp then return end

	if flyEnabled then
		flyPart = Instance.new("Part")
		flyPart.Size = Vector3.new(4, 1, 4)
		flyPart.Transparency = 1
		flyPart.Anchored = true
		flyPart.Parent = workspace
		
		task.spawn(function()
			while flyEnabled and char.Parent do
				local cam = workspace.CurrentCamera
				local moveDir = Vector3.new(0, 0, 0)
				
				if UserInputService:IsKeyDown(Enum.KeyCode.W) then moveDir += cam.CFrame.LookVector end
				if UserInputService:IsKeyDown(Enum.KeyCode.S) then moveDir -= cam.CFrame.LookVector end
				if UserInputService:IsKeyDown(Enum.KeyCode.D) then moveDir += cam.CFrame.RightVector end
				if UserInputService:IsKeyDown(Enum.KeyCode.A) then moveDir -= cam.CFrame.RightVector end
				if UserInputService:IsKeyDown(Enum.KeyCode.Space) then moveDir += Vector3.new(0, 1, 0) end
				if UserInputService:IsKeyDown(Enum.KeyCode.LeftShift) then moveDir -= Vector3.new(0, 1, 0) end
				
				hrp.Velocity = (moveDir.Magnitude > 0 and moveDir.Unit * flySpeed or Vector3.new(0, 0, 0))
				hrp.CFrame = CFrame.new(hrp.Position, hrp.Position + cam.CFrame.LookVector)
				flyPart.CFrame = hrp.CFrame * CFrame.new(0, -3.5, 0)
				task.wait()
			end
			if flyPart then flyPart:Destroy() end
		end)
	else
		if flyPart then flyPart:Destroy() end
		hrp.Velocity = Vector3.new(0,0,0)
	end
end

--// INITIALIZE UI
createToggle("Fly", false, toggleFly, mainPage)
local speedToggle = createToggle("Walkspeed", false, function(v) speedEnabled = v end, mainPage)
createToggle("Infinite Jump", false, function(v) infJumpEnabled = v end, mainPage)
createToggle("Noclip", false, function(v) noclipEnabled = v end, mainPage)

local espToggle = createToggle("Player ESP", false, function(v) 
	espEnabled = v
	if v then
		for _, p in pairs(Players:GetPlayers()) do if p ~= LocalPlayer and p.Character then createESP(p, p.Character) end end
	else removeESP() end
end, visualsPage)

createToggle("Aimbot Enabled", false, function(v) config.aimbotEnabled = v end, aimbotPage)
createToggle("Show FOV Circle", false, function(v) fovCircle.Visible = v end, aimbotPage)

createSlider("Aimbot FOV", 10, 800, 150, function(v) config.aimbotFOV = v fovCircle.Radius = v end, configPage)
createSlider("Aimbot Smooth", 1, 50, 25, function(v) config.aimbotSmoothness = v/100 end, configPage)
createSlider("Walkspeed Value", 16, 300, 50, function(v) config.walkSpeedValue = v end, configPage)
createSlider("Fly Speed", 10, 500, 50, function(v) flySpeed = v end, configPage)

createKeybind("Menu Toggle", config.menuKey, function(k) config.menuKey = k end, keysPage)
createKeybind("Aimbot Key", config.aimbotKey, function(k) config.aimbotKey = k end, keysPage)
createKeybind("Speed Toggle", config.speedKey, function(k) config.speedKey = k end, keysPage)
createKeybind("ESP Toggle", config.espKey, function(k) config.espKey = k end, keysPage)
createKeybind("Noclip Toggle", config.noclipKey, function(k) config.noclipKey = k end, keysPage)
createKeybind("Fly Toggle", config.flyKey, function(k) config.flyKey = k end, keysPage)

showPage("MAIN")

--// MAIN LOOP
RunService.RenderStepped:Connect(function()
	if LocalPlayer.Character then
		-- Speed
		local hum = LocalPlayer.Character:FindFirstChild("Humanoid")
		if hum then hum.WalkSpeed = speedEnabled and config.walkSpeedValue or 16 end
		
		-- Noclip
		if noclipEnabled then
			for _, part in pairs(LocalPlayer.Character:GetDescendants()) do
				if part:IsA("BasePart") then part.CanCollide = false end
			end
		end
		
		-- ESP
		if espEnabled then
			for player, data in pairs(espData) do
				if data.character and data.character:FindFirstChild("Head") then
					local d = math.floor((LocalPlayer.Character.HumanoidRootPart.Position - data.character.HumanoidRootPart.Position).Magnitude)
					data.text.Text = player.Name .. " [" .. d .. "m]"
				end
			end
		end
		
		-- AIMBOT
		fovCircle.Position = Vector2.new(Camera.ViewportSize.X/2, Camera.ViewportSize.Y/2)
		if config.aimbotEnabled and (UserInputService:IsKeyDown(config.aimbotKey) or UserInputService:IsMouseButtonPressed(Enum.UserInputType.MouseButton2)) then
			local target = getClosestPlayer()
			if target and target.Character and target.Character:FindFirstChild("Head") then
				Camera.CFrame = Camera.CFrame:Lerp(CFrame.new(Camera.CFrame.Position, target.Character.Head.Position), config.aimbotSmoothness)
			end
		end
	end
end)

--// INPUTS
UserInputService.InputBegan:Connect(function(input, gpe)
	if gpe then return end
	if input.KeyCode == config.menuKey then
		menuOpen = not menuOpen
		playSound(12222242, 0.4)
		if menuOpen then
			MainFrame.Visible = true
			TweenService:Create(MainFrame, TweenInfo.new(0.3, Enum.EasingStyle.Quart, Enum.EasingDirection.Out), {Size = UDim2.new(0, 500, 0, 350), Position = UDim2.new(0.5, -250, 0.5, -175)}):Play()
		else
			local tween = TweenService:Create(MainFrame, TweenInfo.new(0.3, Enum.EasingStyle.Quart, Enum.EasingDirection.In), {Size = UDim2.new(0, 500, 0, 0), Position = UDim2.new(0.5, -250, 0.5, 0)})
			tween:Play()
			tween.Completed:Connect(function() if not menuOpen then MainFrame.Visible = false end end)
		end
	end
	if input.KeyCode == config.speedKey then
		speedEnabled = not speedEnabled
		speedToggle.SetState(speedEnabled)
	end
	if input.KeyCode == config.espKey then
		espEnabled = not espEnabled
		espToggle.SetState(espEnabled)
		if espEnabled then
			for _, p in pairs(Players:GetPlayers()) do if p ~= LocalPlayer and p.Character then createESP(p, p.Character) end end
		else removeESP() end
	end
	if input.KeyCode == config.noclipKey then
		noclipEnabled = not noclipEnabled
	end
	if input.KeyCode == config.flyKey then
		flyEnabled = not flyEnabled
		toggleFly(flyEnabled)
	end
	if input.KeyCode == Enum.KeyCode.Space and infJumpEnabled then
		if LocalPlayer.Character and LocalPlayer.Character:FindFirstChildOfClass("Humanoid") then
			LocalPlayer.Character:FindFirstChildOfClass("Humanoid"):ChangeState(Enum.HumanoidStateType.Jumping)
		end
	end
end)

MainFrame.Visible = true
playSound(12222242, 0.6)
print("ARIS HUB ULTIMATE - CLEAN & AIMBOT READY")
