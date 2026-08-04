local Players = game:GetService("Players")
local UserInputService = game:GetService("UserInputService")
local LocalPlayer = Players.LocalPlayer

-- ====== MENU GUI ======
local screenGui = Instance.new("ScreenGui")
screenGui.Name = "CustomMenuGUI"
screenGui.ResetOnSpawn = false
screenGui.ZIndexBehavior = Enum.ZIndexBehavior.Global  -- ✅ Cho phép đè lên GUI khác
screenGui.DisplayOrder = 9999  -- ✅ Ưu tiên cao nhất
screenGui.IgnoreGuiInset = true  -- ✅ Không bị lệch bởi topbar trên mobile
screenGui.Parent = LocalPlayer:WaitForChild("PlayerGui")

-- Nút nhỏ mở menu
local toggleButton = Instance.new("ImageButton")
toggleButton.Size = UDim2.new(0, 50, 0, 50)
toggleButton.Position = UDim2.new(0.5, -25, 0.5, -25)
toggleButton.BackgroundColor3 = Color3.fromRGB(40, 40, 40)
toggleButton.Image = "rbxassetid://98183807650041"
toggleButton.ImageRectOffset = Vector2.new(0, 0)
toggleButton.ImageRectSize = Vector2.new(0, 0)
toggleButton.BackgroundTransparency = 1
toggleButton.Parent = screenGui
-- Viền tím cho nút nhỏ
local buttonStroke = Instance.new("UIStroke")
buttonStroke.Thickness = 3
buttonStroke.Color = Color3.fromRGB(120, 0, 255)  -- tím cùng tone với menu
buttonStroke.Transparency = 0
buttonStroke.Parent = toggleButton

local btnCorner = Instance.new("UICorner")
btnCorner.CornerRadius = UDim.new(0, 8)
btnCorner.Parent = toggleButton

-- Menu chính
local menuFrame = Instance.new("Frame")
menuFrame.Size = UDim2.new(0, 220, 0, 240)
menuFrame.Position = UDim2.new(0.5, -110, 0.5, -90)
menuFrame.BackgroundColor3 = Color3.fromRGB(25, 25, 25)
menuFrame.Visible = false
menuFrame.Parent = screenGui

local menuCorner = Instance.new("UICorner")
menuCorner.CornerRadius = UDim.new(0, 12)
menuCorner.Parent = menuFrame

local UIStroke = Instance.new("UIStroke")
UIStroke.Thickness = 2
UIStroke.Color = Color3.fromRGB(120, 0, 255)
UIStroke.Parent = menuFrame

-- Tiêu đề (dùng kéo menu)
local title = Instance.new("TextLabel")
title.Size = UDim2.new(1, 0, 0, 35)
title.BackgroundColor3 = Color3.fromRGB(35, 35, 35)
title.Text = "Sì Trây"
title.TextColor3 = Color3.fromRGB(255, 255, 255)
title.TextSize = 16
title.Font = Enum.Font.SourceSansBold
title.Parent = menuFrame

local titleCorner = Instance.new("UICorner")
titleCorner.CornerRadius = UDim.new(0, 12)
titleCorner.Parent = title

-- Toggle menu
local open = false
toggleButton.MouseButton1Click:Connect(function()
    open = not open
    menuFrame.Visible = open
end)

-- Drag nút nhỏ
local dragging, dragInput, dragStart, startPos
local function update(input)
    local delta = input.Position - dragStart
    toggleButton.Position = UDim2.new(
        startPos.X.Scale, startPos.X.Offset + delta.X,
        startPos.Y.Scale, startPos.Y.Offset + delta.Y
    )
end
toggleButton.InputBegan:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then
        dragging = true
        dragStart = input.Position
        startPos = toggleButton.Position
        input.Changed:Connect(function()
            if input.UserInputState == Enum.UserInputState.End then dragging = false end
        end)
    end
end)
toggleButton.InputChanged:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseMovement or input.UserInputType == Enum.UserInputType.Touch then
        dragInput = input
    end
end)
UserInputService.InputChanged:Connect(function(input)
    if input == dragInput and dragging then update(input) end
end)

-- Drag menu qua tiêu đề
local menuDragging, menuDragInput, menuDragStart, menuStartPos
local function updateMenu(input)
    local delta = input.Position - menuDragStart
    menuFrame.Position = UDim2.new(
        menuStartPos.X.Scale, menuStartPos.X.Offset + delta.X,
        menuStartPos.Y.Scale, menuStartPos.Y.Offset + delta.Y
    )
end
title.InputBegan:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then
        menuDragging = true
        menuDragStart = input.Position
        menuStartPos = menuFrame.Position
        input.Changed:Connect(function()
            if input.UserInputState == Enum.UserInputState.End then menuDragging = false end
        end)
    end
end)
title.InputChanged:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseMovement or input.UserInputType == Enum.UserInputType.Touch then
        menuDragInput = input
    end
end)
UserInputService.InputChanged:Connect(function(input)
    if input == menuDragInput and menuDragging then updateMenu(input) end
end)

-- ====== FARM (AutoFarm) ======
-- ====== FARM (Teleport Version) ======
local farming = false

local function getCoins()
    for _, map in ipairs(workspace:GetChildren()) do
        if map:FindFirstChild("CoinContainer") then
            return map.CoinContainer
        end
    end
end

local function getRandomCoin()
    local container = getCoins()
    if not container then
        return nil
    end

    local list = {}

    for _, coin in ipairs(container:GetChildren()) do
        if coin:IsA("BasePart") then
            table.insert(list, coin)
        end
    end

    if #list == 0 then
        return nil
    end

    return list[math.random(1, #list)]
end

local function autoFarm()

    while farming do

        local character = LocalPlayer.Character
        local hrp = character and character:FindFirstChild("HumanoidRootPart")

        if not hrp then
            task.wait(0.5)
            continue
        end

        local coin = getRandomCoin()

        if coin and coin.Parent then

            local distance = (hrp.Position - coin.Position).Magnitude

            -- Teleport lên trên coin
            hrp.CFrame = coin.CFrame + Vector3.new(0, 2.5, 0)

            -- Coin gần: 2 giây
            -- Coin xa: 3 giây
            local waitTime = (distance <= 35) and 2 or 3

            local start = tick()

            repeat
                task.wait(0.1)
            until
                not farming
                or not coin.Parent
                or tick() - start >= waitTime

        else
            task.wait(0.1)
        end

    end

end

-- ====== Toggle Farm (Switch đẹp) ======
local farmLabel = Instance.new("TextLabel")
farmLabel.Size = UDim2.new(0.6, 0, 0, 30)
farmLabel.Position = UDim2.new(0, 10, 0, 60)
farmLabel.BackgroundTransparency = 1
farmLabel.Text = "Coin Farm"
farmLabel.TextColor3 = Color3.fromRGB(220, 220, 220)
farmLabel.Font = Enum.Font.Gotham
farmLabel.TextSize = 16
farmLabel.TextXAlignment = Enum.TextXAlignment.Left
farmLabel.Parent = menuFrame

local toggleFrame = Instance.new("Frame")
toggleFrame.Size = UDim2.new(0, 50, 0, 25)
toggleFrame.Position = UDim2.new(1, -60, 0, 60)
toggleFrame.BackgroundColor3 = Color3.fromRGB(60, 60, 80)
toggleFrame.Parent = menuFrame
toggleFrame.BorderSizePixel = 0

local corner = Instance.new("UICorner")
corner.CornerRadius = UDim.new(1, 0)
corner.Parent = toggleFrame

local knob = Instance.new("Frame")
knob.Size = UDim2.new(0, 21, 0, 21)
knob.Position = UDim2.new(0, 2, 0, 2)
knob.BackgroundColor3 = Color3.fromRGB(200, 200, 200)
knob.Parent = toggleFrame
knob.BorderSizePixel = 0

local knobCorner = Instance.new("UICorner")
knobCorner.CornerRadius = UDim.new(1, 0)
knobCorner.Parent = knob

local toggleStroke = Instance.new("UIStroke")
toggleStroke.Thickness = 1.5
toggleStroke.Color = Color3.fromRGB(150, 80, 255)
toggleStroke.Transparency = 0.3
toggleStroke.Parent = toggleFrame

-- Function đổi trạng thái toggle
local function setToggle(state)
    if state then
        TweenService:Create(toggleFrame, TweenInfo.new(0.25, Enum.EasingStyle.Quad), {BackgroundColor3 = Color3.fromRGB(150, 80, 255)}):Play()
        TweenService:Create(knob, TweenInfo.new(0.25, Enum.EasingStyle.Quad), {Position = UDim2.new(1, -23, 0, 2), BackgroundColor3 = Color3.fromRGB(255,255,255)}):Play()
    else
        TweenService:Create(toggleFrame, TweenInfo.new(0.25, Enum.EasingStyle.Quad), {BackgroundColor3 = Color3.fromRGB(60, 60, 80)}):Play()
        TweenService:Create(knob, TweenInfo.new(0.25, Enum.EasingStyle.Quad), {Position = UDim2.new(0, 2, 0, 2), BackgroundColor3 = Color3.fromRGB(200,200,200)}):Play()
    end
end

-- Click toggle
toggleFrame.InputBegan:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then
        farming = not farming
        setToggle(farming)
        if farming then
            task.spawn(autoFarm)
        end
    end
end)

setToggle(true) --(On)
farming = true
task.spawn(autoFarm)


-- ====== Coin Counter GUI (gắn vào menuFrame) ======
_G.TotalCoinsFarmed = _G.TotalCoinsFarmed or 0

local function setupCoinGUI()
    local label = menuFrame:FindFirstChild("CoinLabel")
    if not label then
        label = Instance.new("TextLabel")
        label.Name = "CoinLabel"
        label.Size = UDim2.new(1, -20, 0, 30)
        label.Position = UDim2.new(0, 10, 0, 170)
        label.BackgroundColor3 = Color3.fromRGB(30,30,30)
        label.BackgroundTransparency = 0.2
        label.TextColor3 = Color3.fromRGB(255,255,0)
        label.TextSize = 16
        label.Font = Enum.Font.SourceSansBold
        label.Text = "Coins: 0/40 | Total: 0"
        label.Parent = menuFrame
    end
    return label
end

local function hookGameCoinLabel(label)
    local coinLabel = LocalPlayer:WaitForChild("PlayerGui")
        :WaitForChild("MainGUI")
        :WaitForChild("Lobby")
        :WaitForChild("Dock")
        :WaitForChild("CoinBags")
        :WaitForChild("Container")
        :WaitForChild("Coin")
        :WaitForChild("CurrencyFrame")
        :WaitForChild("Icon")
        :WaitForChild("Coins")

    local lastAmount = tonumber(coinLabel.Text) or 0

    local function updateCoins()
        local amount = tonumber(coinLabel.Text) or 0
        if amount > lastAmount then
            _G.TotalCoinsFarmed += (amount - lastAmount)
        end
        lastAmount = amount

        label.Text = "Coins: " .. amount .. "/40 | Total: " .. _G.TotalCoinsFarmed

        if amount >= 40 then
            label.TextColor3 = Color3.fromRGB(255,0,0)
            label.Text = "FULL! ("..amount.."/40) | Total: " .. _G.TotalCoinsFarmed .. " - Resetting..."
            local humanoid = LocalPlayer.Character and LocalPlayer.Character:FindFirstChildOfClass("Humanoid")
            if humanoid then humanoid.Health = 0 end
        else
            label.TextColor3 = Color3.fromRGB(0,255,0)
        end
    end

    updateCoins()
    coinLabel:GetPropertyChangedSignal("Text"):Connect(updateCoins)
end

local coinLabelUI = setupCoinGUI()
hookGameCoinLabel(coinLabelUI)

-- 🌟 Thêm sau phần hookGameCoinLabel(coinLabelUI)
-- (Dán ngay bên dưới dòng “hookGameCoinLabel(coinLabelUI)”)
------------------------------------------------------

-- 🌟 Thêm sau phần hookGameCoinLabel(coinLabelUI)
------------------------------------------------------

-- ====== TIMER GUI ======
local startTime = tick()  -- Ghi thời điểm bật script
local timerLabel = Instance.new("TextLabel")
timerLabel.Name = "TimerLabel"
timerLabel.Size = UDim2.new(1, -20, 0, 30)
timerLabel.Position = UDim2.new(0, 10, 0, 135)
timerLabel.BackgroundColor3 = Color3.fromRGB(30, 30, 30)
timerLabel.BackgroundTransparency = 0.2
timerLabel.TextColor3 = Color3.fromRGB(0, 200, 255)
timerLabel.TextSize = 16
timerLabel.Font = Enum.Font.SourceSansBold
timerLabel.Text = "Time: 00:00:00"
timerLabel.Parent = menuFrame
-- ====== COIN PER HOUR (Tốc độ farm trung bình) ======
local rateLabel = Instance.new("TextLabel")
rateLabel.Name = "RateLabel"
rateLabel.Size = UDim2.new(1, -20, 0, 30)
rateLabel.Position = UDim2.new(0, 10, 0, 100)
rateLabel.BackgroundColor3 = Color3.fromRGB(30, 30, 30)
rateLabel.BackgroundTransparency = 0.2
rateLabel.TextColor3 = Color3.fromRGB(255, 255, 255)
rateLabel.TextSize = 16
rateLabel.Font = Enum.Font.SourceSansBold
rateLabel.Text = "Rate: 0 / hour"
rateLabel.Parent = menuFrame

-- Cập nhật tốc độ trung bình mỗi giây
task.spawn(function()
    while task.wait(1) do
        local elapsed = tick() - startTime
        local hours = elapsed / 3600
        local total = _G.TotalCoinsFarmed or 0
        local rate = 0
        if hours > 0 then
            rate = total / hours
        end
        rateLabel.Text = string.format("Rate: %.1f / hour", rate)
        if rate < 100 then
            rateLabel.TextColor3 = Color3.fromRGB(255, 200, 0)
        elseif rate < 500 then
            rateLabel.TextColor3 = Color3.fromRGB(0, 255, 0)
        else
            rateLabel.TextColor3 = Color3.fromRGB(255, 0, 0)
        end
    end
end)

-- Hàm cập nhật thời gian dạng HH.MM.SS
task.spawn(function()
    while task.wait(1) do
        local elapsed = tick() - startTime
        local hours = math.floor(elapsed / 3600)
        local minutes = math.floor((elapsed % 3600) / 60)
        local seconds = math.floor(elapsed % 60)
        timerLabel.Text = string.format("Time: %02d:%02d:%02d", hours, minutes, seconds)
    end
end)

LocalPlayer.CharacterAdded:Connect(function()
    task.wait(1)
    local label = setupCoinGUI()
    hookGameCoinLabel(label)
end)

-- ====== HIỂN THỊ TỔNG COIN BAG ======
local totalCoinLabel = Instance.new("TextLabel")
totalCoinLabel.Name = "TotalCoinLabel"
totalCoinLabel.Size = UDim2.new(1, -20, 0, 30)
totalCoinLabel.Position = UDim2.new(0, 10, 0, 205)
totalCoinLabel.BackgroundColor3 = Color3.fromRGB(30, 30, 30)
totalCoinLabel.BackgroundTransparency = 0.2
totalCoinLabel.TextColor3 = Color3.fromRGB(255, 255, 0)
totalCoinLabel.TextSize = 16
totalCoinLabel.Font = Enum.Font.SourceSansBold
totalCoinLabel.Text = "Coin Bag: Load..."
totalCoinLabel.Parent = menuFrame

-- Cập nhật số coin hiển thị theo GUI
task.spawn(function()
	while task.wait(10) do
		local success, label = pcall(function()
			return LocalPlayer:WaitForChild("PlayerGui")
				:WaitForChild("CrossPlatform")
				:WaitForChild("Shop")
				:WaitForChild("Small")
				:WaitForChild("Container")
				:WaitForChild("Title")
				:WaitForChild("Container")
				:WaitForChild("Coins")
				:WaitForChild("Container")
				:WaitForChild("Amount")
		end)

		if success and label and label.Text then
			totalCoinLabel.Text = "Coin Bag: " .. label.Text
		else
			totalCoinLabel.Text = "Coin Bag: ???"
		end
	end
end)
