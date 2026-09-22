-- ============================================
-- ⚔ BEN HUB — Chams (кнопка на месте меню)
-- ============================================
local Players = game:GetService("Players")
local UserInputService = game:GetService("UserInputService")
local TweenService = game:GetService("TweenService")
local localPlayer = Players.LocalPlayer
local playerGui = localPlayer:WaitForChild("PlayerGui")

local COLORS = {
    Player     = Color3.fromRGB(255, 60, 60),
    Drone      = Color3.fromRGB(255, 130, 0),
    MyDrone    = Color3.fromRGB(60, 255, 60),
    Gazel      = Color3.fromRGB(180, 60, 255),
    Leaderboard = Color3.fromRGB(0, 150, 255),
}
local TRANSPARENCY = 0.5
local DRONE_MARKER_SHOW_DISTANCE = 1500
local DRONE_MARKER_MAX_DISTANCE = 10000
local LEADERBOARD_NAME = ""

local DRONE_KEYWORDS = {"shahed", "шахед", "дрон", "бпла", "uav", "drone"}
local GAZEL_KEYWORDS = {"gazel", "газель", "газел"}
local LEADERBOARD_KEYWORDS = {
    "leaderboard", "лидерборд", "лидербор", "таблица", "рейтинг",
    "top", "rank", "stats", "board", "scoreboard", "табло"
}
local MY_NAMES = {"benmaster505", "benmaster", "benloniks", "benlonik"}

local state = { Player = false, Drone = true, MyDrone = false, Gazel = true, Leaderboard = false }
local activeChams = {}
local droneMarkers = {}

-- ============ МАРКЕР ============
local function applyDroneMarker(model)
    if droneMarkers[model] then return end
    local basePart = model:FindFirstChildWhichIsA("BasePart") or model.PrimaryPart
    if not basePart then
        for _, d in ipairs(model:GetDescendants()) do
            if d:IsA("BasePart") then basePart = d; break end
        end
    end
    if not basePart then return end

    local bb = Instance.new("BillboardGui")
    bb.Name = "DroneMarkerGui"
    bb.Size = UDim2.new(0, 100, 0, 24)
    bb.StudsOffset = Vector3.new(0, 6, 0)
    bb.AlwaysOnTop = true
    bb.MaxDistance = DRONE_MARKER_MAX_DISTANCE
    bb.LightInfluence = 0
    bb.Adornee = basePart
    bb.Enabled = false
    bb.Parent = basePart

    local distLbl = Instance.new("TextLabel")
    distLbl.Size = UDim2.new(1, 0, 1, 0)
    distLbl.BackgroundTransparency = 1
    distLbl.Text = "0"
    distLbl.TextColor3 = COLORS.Drone
    distLbl.TextStrokeTransparency = 0
    distLbl.TextStrokeColor3 = Color3.fromRGB(0, 0, 0)
    distLbl.Font = Enum.Font.GothamBold
    distLbl.TextSize = 18
    distLbl.Parent = bb

    droneMarkers[model] = {gui = bb, label = distLbl}
end

local function removeDroneMarker(model)
    local m = droneMarkers[model]
    if m then
        if m.gui then m.gui:Destroy() end
        droneMarkers[model] = nil
    end
end

local function updateDroneMarkerDistances()
    local char = localPlayer.Character
    local root = char and char:FindFirstChild("HumanoidRootPart")
    if not root then return end
    local myPos = root.Position
    for model, m in pairs(droneMarkers) do
        if not model.Parent or not m.gui or not m.gui.Parent then
            droneMarkers[model] = nil
        else
            local pivot = model:GetPivot().Position
            local dist = (myPos - pivot).Magnitude
            m.label.Text = tostring(math.floor(dist))
            m.gui.Enabled = dist <= DRONE_MARKER_SHOW_DISTANCE
        end
    end
end

-- ============ CHAMS ============
local function removeCham(model)
    local d = activeChams[model]
    if d then
        if d.highlight then d.highlight:Destroy() end
        activeChams[model] = nil
    end
    removeDroneMarker(model)
end

local function removeChamsByType(t)
    for m, d in pairs(activeChams) do
        if d.type == t then
            if d.highlight then d.highlight:Destroy() end
            activeChams[m] = nil
            if t == "Drone" then removeDroneMarker(m) end
        end
    end
end

local function removeAllChams()
    for _, d in pairs(activeChams) do
        if d.highlight then d.highlight:Destroy() end
    end
    activeChams = {}
    for m in pairs(droneMarkers) do removeDroneMarker(m) end
end

local function applyCham(model, typeName)
    if not model or not model.Parent then return end
    local color = COLORS[typeName]
    if not color then return end
    local ex = activeChams[model]
    if ex and ex.type == typeName then return end
    if ex then removeCham(model) end
    local h = Instance.new("Highlight")
    h.Name = "ChamsHighlight"
    h.FillColor = color
    h.OutlineColor = color
    h.FillTransparency = TRANSPARENCY
    h.OutlineTransparency = 0.1
    h.DepthMode = Enum.HighlightDepthMode.AlwaysOnTop
    h.Adornee = model
    h.Parent = model
    activeChams[model] = {highlight = h, type = typeName}
    if typeName == "Drone" then applyDroneMarker(model) end
end

local function isMyDrone(model)
    if localPlayer.Character and model:IsDescendantOf(localPlayer.Character) then return true end
    for _, c in ipairs(model:GetChildren()) do
        if c:IsA("ObjectValue") and c.Value == localPlayer then return true end
        if c:IsA("StringValue") and c.Value then
            local v = tostring(c.Value):lower()
            for _, n in ipairs(MY_NAMES) do
                if string.find(v, n) then return true end
            end
        end
    end
    local nm = model.Name:lower()
    for _, n in ipairs(MY_NAMES) do
        if string.find(nm, n) then return true end
    end
    if model.Parent then
        local pn = model.Parent.Name:lower()
        for _, n in ipairs(MY_NAMES) do
            if string.find(pn, n) then return true end
        end
    end
    return false
end

local function isLeaderboardModel(model)
    local nm = model.Name:lower()
    if LEADERBOARD_NAME ~= "" then
        return string.find(nm, LEADERBOARD_NAME:lower()) ~= nil
    end
    for _, kw in ipairs(LEADERBOARD_KEYWORDS) do
        if string.find(nm, kw) then return true end
    end
    if model.Parent then
        local pn = model.Parent.Name:lower()
        for _, kw in ipairs(LEADERBOARD_KEYWORDS) do
            if string.find(pn, kw) then return true end
        end
    end
    return false
end

local function classify(model)
    if not model or not model.Parent or not model:IsA("Model") then return nil end
    for _, p in ipairs(Players:GetPlayers()) do
        if p.Character == model then
            if p == localPlayer then return nil end
            return "Player"
        end
    end
    if isMyDrone(model) then return "MyDrone" end
    local nm = model.Name:lower()
    for _, kw in ipairs(GAZEL_KEYWORDS) do
        if string.find(nm, kw) then return "Gazel" end
    end
    for _, kw in ipairs(DRONE_KEYWORDS) do
        if string.find(nm, kw) then return "Drone" end
    end
    if isLeaderboardModel(model) then return "Leaderboard" end
    return nil
end

local function updateModel(model)
    if not model or not model.Parent then removeCham(model); return end
    local t = classify(model)
    if not t then removeCham(model); return end
    if state[t] then applyCham(model, t) else removeCham(model) end
end

local function rescanAll()
    for _, obj in ipairs(workspace:GetDescendants()) do
        if obj:IsA("Model") then updateModel(obj) end
    end
end

-- ============ GUI ============
local screenGui = Instance.new("ScreenGui")
screenGui.Name = "BenHubGui"
screenGui.ResetOnSpawn = false
screenGui.ZIndexBehavior = Enum.ZIndexBehavior.Sibling
screenGui.IgnoreGuiInset = true
screenGui.Parent = playerGui

-- Меньше размер (было 420×380)
local MENU_WIDTH = 360
local MENU_HEIGHT = 340

-- Главное меню
local main = Instance.new("Frame")
main.Size = UDim2.new(0, MENU_WIDTH, 0, MENU_HEIGHT)
main.Position = UDim2.new(0, 20, 0.5, -MENU_HEIGHT/2)
main.BackgroundColor3 = Color3.fromRGB(18, 18, 24)
main.BackgroundTransparency = 0.05
main.BorderSizePixel = 0
main.Visible = true
main.Parent = screenGui
Instance.new("UICorner", main).CornerRadius = UDim.new(0, 12)

local mainStroke = Instance.new("UIStroke")
mainStroke.Color = Color3.fromRGB(120, 80, 255)
mainStroke.Thickness = 1.5
mainStroke.Transparency = 0.3
mainStroke.Parent = main

local titleBar = Instance.new("Frame")
titleBar.Size = UDim2.new(1, 0, 0, 38)
titleBar.BackgroundColor3 = Color3.fromRGB(35, 25, 60)
titleBar.BorderSizePixel = 0
titleBar.Parent = main
Instance.new("UICorner", titleBar).CornerRadius = UDim.new(0, 12)

local titleFix = Instance.new("Frame")
titleFix.Size = UDim2.new(1, 0, 0, 10)
titleFix.Position = UDim2.new(0, 0, 1, -10)
titleFix.BackgroundColor3 = Color3.fromRGB(35, 25, 60)
titleFix.BorderSizePixel = 0
titleFix.Parent = titleBar

local titleLabel = Instance.new("TextLabel")
titleLabel.Size = UDim2.new(1, -80, 1, 0)
titleLabel.Position = UDim2.new(0, 12, 0, 0)
titleLabel.BackgroundTransparency = 1
titleLabel.Text = "⚔ BEN HUB"
titleLabel.TextColor3 = Color3.fromRGB(220, 180, 255)
titleLabel.TextXAlignment = Enum.TextXAlignment.Left
titleLabel.Font = Enum.Font.GothamBold
titleLabel.TextSize = 14
titleLabel.Parent = titleBar

local minBtn = Instance.new("TextButton")
minBtn.Size = UDim2.new(0, 26, 0, 26)
minBtn.Position = UDim2.new(1, -62, 0, 6)
minBtn.BackgroundColor3 = Color3.fromRGB(255, 180, 0)
minBtn.Text = "—"
minBtn.TextColor3 = Color3.fromRGB(20, 20, 25)
minBtn.Font = Enum.Font.GothamBold
minBtn.TextSize = 17
minBtn.BorderSizePixel = 0
minBtn.Parent = titleBar
Instance.new("UICorner", minBtn).CornerRadius = UDim.new(0, 6)

local closeBtn = Instance.new("TextButton")
closeBtn.Size = UDim2.new(0, 26, 0, 26)
closeBtn.Position = UDim2.new(1, -31, 0, 6)
closeBtn.BackgroundColor3 = Color3.fromRGB(255, 60, 60)
closeBtn.Text = "✕"
closeBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
closeBtn.Font = Enum.Font.GothamBold
closeBtn.TextSize = 13
closeBtn.BorderSizePixel = 0
closeBtn.Parent = titleBar
Instance.new("UICorner", closeBtn).CornerRadius = UDim.new(0, 6)

local sectionLabel = Instance.new("TextLabel")
sectionLabel.Size = UDim2.new(1, -20, 0, 26)
sectionLabel.Position = UDim2.new(0, 10, 0, 44)
sectionLabel.BackgroundColor3 = Color3.fromRGB(24, 24, 32)
sectionLabel.BackgroundTransparency = 0.3
sectionLabel.Text = "VISUALS"
sectionLabel.TextColor3 = Color3.fromRGB(200, 180, 255)
sectionLabel.Font = Enum.Font.GothamBold
sectionLabel.TextSize = 12
sectionLabel.BorderSizePixel = 0
sectionLabel.Parent = main
Instance.new("UICorner", sectionLabel).CornerRadius = UDim.new(0, 8)

local contentHolder = Instance.new("Frame")
contentHolder.Size = UDim2.new(1, -20, 1, -96)
contentHolder.Position = UDim2.new(0, 10, 0, 76)
contentHolder.BackgroundTransparency = 1
contentHolder.Parent = main

local pageLayout = Instance.new("UIListLayout")
pageLayout.Padding = UDim.new(0, 5)
pageLayout.SortOrder = Enum.SortOrder.LayoutOrder
pageLayout.Parent = contentHolder

local function createToggle(text, key, color, order)
    local row = Instance.new("TextButton")
    row.Size = UDim2.new(1, 0, 0, 40)
    row.BackgroundColor3 = Color3.fromRGB(32, 32, 42)
    row.BorderSizePixel = 0
    row.Text = ""
    row.AutoButtonColor = false
    row.LayoutOrder = order
    row.Parent = contentHolder
    Instance.new("UICorner", row).CornerRadius = UDim.new(0, 8)

    local stroke = Instance.new("UIStroke")
    stroke.Color = color
    stroke.Thickness = 1
    stroke.Transparency = 0.7
    stroke.Parent = row

    local dot = Instance.new("Frame")
    dot.Size = UDim2.new(0, 9, 0, 9)
    dot.Position = UDim2.new(0, 10, 0.5, -4.5)
    dot.BackgroundColor3 = color
    dot.BorderSizePixel = 0
    dot.Parent = row
    Instance.new("UICorner", dot).CornerRadius = UDim.new(1, 0)

    local lbl = Instance.new("TextLabel")
    lbl.Size = UDim2.new(1, -90, 1, 0)
    lbl.Position = UDim2.new(0, 25, 0, 0)
    lbl.BackgroundTransparency = 1
    lbl.Text = text
    lbl.TextColor3 = Color3.fromRGB(230, 230, 230)
    lbl.TextXAlignment = Enum.TextXAlignment.Left
    lbl.Font = Enum.Font.GothamMedium
    lbl.TextSize = 12
    lbl.Parent = row

    local status = Instance.new("TextLabel")
    status.Size = UDim2.new(0, 50, 0, 24)
    status.Position = UDim2.new(1, -58, 0.5, -12)
    status.Text = "ВЫКЛ"
    status.Font = Enum.Font.GothamBold
    status.TextSize = 11
    status.BorderSizePixel = 0
    status.Parent = row
    Instance.new("UICorner", status).CornerRadius = UDim.new(0, 6)

    local function refresh()
        if state[key] then
            status.Text = "ВКЛ"
            status.BackgroundColor3 = Color3.fromRGB(30, 70, 40)
            status.TextColor3 = Color3.fromRGB(80, 255, 120)
            stroke.Transparency = 0.3
            row.BackgroundColor3 = Color3.fromRGB(40, 44, 54)
        else
            status.Text = "ВЫКЛ"
            status.BackgroundColor3 = Color3.fromRGB(70, 30, 30)
            status.TextColor3 = Color3.fromRGB(255, 120, 120)
            stroke.Transparency = 0.7
            row.BackgroundColor3 = Color3.fromRGB(32, 32, 42)
        end
    end

    row.MouseButton1Click:Connect(function()
        state[key] = not state[key]
        refresh()
        if state[key] then rescanAll() else removeChamsByType(key) end
    end)

    refresh()
end

createToggle("Игроки",     "Player",      COLORS.Player,      1)
createToggle("Шахеды",     "Drone",       COLORS.Drone,       2)
createToggle("Мой Шахед",  "MyDrone",     COLORS.MyDrone,     3)
createToggle("Газели",     "Gazel",       COLORS.Gazel,       4)
createToggle("Лидерборды", "Leaderboard", COLORS.Leaderboard, 5)

-- ============ КНОПКА-КВАДРАТ BEN HUB ============
local toggleBtn = Instance.new("TextButton")
toggleBtn.Name = "BenHubToggle"
toggleBtn.Size = UDim2.new(0, 60, 0, 60)
toggleBtn.Position = UDim2.new(0, 20, 0.5, -30)
toggleBtn.BackgroundColor3 = Color3.fromRGB(35, 25, 60)
toggleBtn.Text = "BEN\nHUB"
toggleBtn.TextColor3 = Color3.fromRGB(220, 180, 255)
toggleBtn.Font = Enum.Font.GothamBold
toggleBtn.TextSize = 13
toggleBtn.TextWrapped = true
toggleBtn.BorderSizePixel = 0
toggleBtn.Visible = false
toggleBtn.Parent = screenGui
Instance.new("UICorner", toggleBtn).CornerRadius = UDim.new(0, 10)

local toggleStroke = Instance.new("UIStroke")
toggleStroke.Color = Color3.fromRGB(120, 80, 255)
toggleStroke.Thickness = 2
toggleStroke.Transparency = 0.2
toggleStroke.Parent = toggleBtn

-- ============ DRAG меню ============
local dragging, dragStart, startPos = false, nil, nil
titleBar.InputBegan:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseButton1
    or input.UserInputType == Enum.UserInputType.Touch then
        local p = input.Position
        local function inRect(pp, tl, sz)
            return pp.X >= tl.X and pp.X <= tl.X + sz.X
               and pp.Y >= tl.Y and pp.Y <= tl.Y + sz.Y
        end
        if inRect(p, minBtn.AbsolutePosition, minBtn.AbsoluteSize) then return end
        if inRect(p, closeBtn.AbsolutePosition, closeBtn.AbsoluteSize) then return end
        dragging = true
        dragStart = input.Position
        startPos = main.Position
    end
end)

UserInputService.InputChanged:Connect(function(input)
    if dragging and (input.UserInputType == Enum.UserInputType.MouseMovement
    or input.UserInputType == Enum.UserInputType.Touch) then
        local d = input.Position - dragStart
        main.Position = UDim2.new(startPos.X.Scale, startPos.X.Offset + d.X, startPos.Y.Scale, startPos.Y.Offset + d.Y)
    end
end)

UserInputService.InputEnded:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseButton1
    or input.UserInputType == Enum.UserInputType.Touch then
        dragging = false
    end
end)

-- ============ DRAG кнопки-квадрата ============
local btnDragging, btnDragStart, btnStartPos = false, nil, nil
local btnMoved = false

toggleBtn.InputBegan:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseButton1
    or input.UserInputType == Enum.UserInputType.Touch then
        btnDragging = true
        btnMoved = false
        btnDragStart = input.Position
        btnStartPos = toggleBtn.Position
    end
end)

UserInputService.InputChanged:Connect(function(input)
    if btnDragging and (input.UserInputType == Enum.UserInputType.MouseMovement
    or input.UserInputType == Enum.UserInputType.Touch) then
        local d = input.Position - btnDragStart
        if math.abs(d.X) > 5 or math.abs(d.Y) > 5 then
            btnMoved = true
        end
        toggleBtn.Position = UDim2.new(btnStartPos.X.Scale, btnStartPos.X.Offset + d.X, btnStartPos.Y.Scale, btnStartPos.Y.Offset + d.Y)
    end
end)

UserInputService.InputEnded:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseButton1
    or input.UserInputType == Enum.UserInputType.Touch then
        btnDragging = false
    end
end)

-- ============ MIN / OPEN / CLOSE ============

-- Свернуть: квадрат появляется ТАМ ЖЕ, где было меню
minBtn.MouseButton1Click:Connect(function()
    local mPos = main.Position
    main.Visible = false
    -- Ставим кнопку точно на место меню
    toggleBtn.Position = UDim2.new(mPos.X.Scale, mPos.X.Offset, mPos.Y.Scale, mPos.Y.Offset)
    toggleBtn.Visible = true
end)

-- Клик по кнопке: меню появляется ТАМ ЖЕ, где был квадрат
toggleBtn.MouseButton1Click:Connect(function()
    if btnMoved then return end
    local bPos = toggleBtn.Position
    toggleBtn.Visible = false
    main.Position = UDim2.new(bPos.X.Scale, bPos.X.Offset, bPos.Y.Scale, bPos.Y.Offset)
    main.Visible = true
end)

-- Полное закрытие: удаляет всё
closeBtn.MouseButton1Click:Connect(function()
    removeAllChams()
    screenGui:Destroy()
end)

-- ============ EVENTS ============
workspace.DescendantAdded:Connect(function(obj)
    if obj:IsA("Model") then task.defer(updateModel, obj) end
end)

task.spawn(function()
    while true do
        task.wait(3)
        for m, d in pairs(activeChams) do
            if not m.Parent then
                activeChams[m] = nil
                droneMarkers[m] = nil
            elseif not d.highlight or not d.highlight.Parent then
                local color = COLORS[d.type]
                if color then
                    local h = Instance.new("Highlight")
                    h.Name = "ChamsHighlight"
                    h.FillColor = color
                    h.OutlineColor = color
                    h.FillTransparency = TRANSPARENCY
                    h.OutlineTransparency = 0.1
                    h.DepthMode = Enum.HighlightDepthMode.AlwaysOnTop
                    h.Adornee = m
                    h.Parent = m
                    d.highlight = h
                end
            end
        end
        if state.Drone then
            for m, d in pairs(activeChams) do
                if d.type == "Drone" and m.Parent then
                    if not droneMarkers[m] then applyDroneMarker(m) end
                end
            end
        end
        if tick() % 10 < 3 then rescanAll() end
    end
end)

task.spawn(function()
    while true do
        task.wait(0.3)
        updateDroneMarkerDistances()
    end
end)

task.wait(1)
rescanAll()
