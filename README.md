-- ============================================================
-- SAN AURE - AIMBOT + ESP SCRIPT COM GUI
-- Propósito: Teste de Anti-Cheat (Desenvolvedor/Dono do Jogo)
-- ============================================================

local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local UserInputService = game:GetService("UserInputService")
local Workspace = game:GetService("Workspace")
local TweenService = game:GetService("TweenService")

local LocalPlayer = Players.LocalPlayer
local LocalMouse = LocalPlayer:GetMouse()
local Camera = Workspace.CurrentCamera

-- ============================================================
-- CONFIGURAÇÕES
-- ============================================================
local Config = {
    Aimbot = {
        Enabled = false,
        Key = Enum.KeyCode.E,
        LockKey = Enum.UserInputType.MouseButton2,
        TargetPart = "Head",
        FOV = 300,
        Smoothness = 0.15,
        AimThroughWalls = true,
        SilentAim = false,
    },
    ESP = {
        Enabled = false,
        Key = Enum.KeyCode.P,
        Box = true,
        Tracer = true,
        Skeleton = true,
        Name = true,
        Health = true,
        MaxDistance = 5000,
    },
    Security = {
        RandomizedDelays = true,
        HookMethod = "getrawmetatable",
        CleanUpOnUnload = true,
    }
}

-- ============================================================
-- VARIÁVEIS DE ESTADO
-- ============================================================
local aimbotActive = false
local aimbotTarget = nil
local ESPObjects = {}

-- ============================================================
-- PAINEL GUI
-- ============================================================
local ScreenGui = Instance.new("ScreenGui")
ScreenGui.Name = "SAN_AURE_TEST_PANEL"
ScreenGui.ZIndexBehavior = Enum.ZIndexBehavior.Sibling
ScreenGui.ResetOnSpawn = false
ScreenGui.Parent = game:GetService("CoreGui")

-- Frame principal
local MainFrame = Instance.new("Frame")
MainFrame.Name = "MainFrame"
MainFrame.Size = UDim2.new(0, 280, 0, 420)
MainFrame.Position = UDim2.new(0, 20, 0, 20)
MainFrame.BackgroundColor3 = Color3.fromRGB(30, 30, 35)
MainFrame.BorderSizePixel = 0
MainFrame.Active = true
MainFrame.Draggable = true
MainFrame.Parent = ScreenGui

local mainCorner = Instance.new("UICorner")
mainCorner.CornerRadius = UDim.new(0, 8)
mainCorner.Parent = MainFrame

local mainStroke = Instance.new("UIStroke")
mainStroke.Thickness = 1
mainStroke.Color = Color3.fromRGB(0, 150, 255)
mainStroke.Parent = MainFrame

-- Barra de título
local TitleBar = Instance.new("Frame")
TitleBar.Name = "TitleBar"
TitleBar.Size = UDim2.new(1, 0, 0, 40)
TitleBar.BackgroundColor3 = Color3.fromRGB(20, 20, 25)
TitleBar.BorderSizePixel = 0
TitleBar.Parent = MainFrame

local titleCorner = Instance.new("UICorner")
titleCorner.CornerRadius = UDim.new(0, 8)
titleCorner.Parent = TitleBar

-- Cortar bordas inferiores do título
local titleLayout = Instance.new("UIListLayout")
titleLayout.Padding = UDim.new(0, 2)
titleLayout.Parent = TitleBar

local TitleLabel = Instance.new("TextLabel")
TitleLabel.Size = UDim2.new(1, -40, 1, 0)
TitleLabel.Position = UDim2.new(0, 12, 0, 0)
TitleLabel.BackgroundTransparency = 1
TitleLabel.Text = "🛡️  SAN AURE - TEST PANEL"
TitleLabel.TextColor3 = Color3.fromRGB(0, 180, 255)
TitleLabel.TextSize = 14
TitleLabel.Font = Enum.Font.GothamBold
TitleLabel.TextXAlignment = Enum.TextXAlignment.Left
TitleLabel.Parent = TitleBar

-- Botão fechar
local CloseBtn = Instance.new("TextButton")
CloseBtn.Name = "CloseBtn"
CloseBtn.Size = UDim2.new(0, 30, 0, 30)
CloseBtn.Position = UDim2.new(1, -35, 0, 5)
CloseBtn.BackgroundColor3 = Color3.fromRGB(255, 50, 50)
CloseBtn.Text = "✕"
CloseBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
CloseBtn.TextSize = 16
CloseBtn.Font = Enum.Font.GothamBold
CloseBtn.BorderSizePixel = 0
CloseBtn.Parent = TitleBar

local closeCorner = Instance.new("UICorner")
closeCorner.CornerRadius = UDim.new(0, 6)
closeCorner.Parent = CloseBtn

CloseBtn.MouseButton1Click:Connect(function()
    ScreenGui.Enabled = not ScreenGui.Enabled
end)

-- Container de scroll
local ScrollingFrame = Instance.new("ScrollingFrame")
ScrollingFrame.Name = "Content"
ScrollingFrame.Size = UDim2.new(1, -10, 1, -50)
ScrollingFrame.Position = UDim2.new(0, 5, 0, 45)
ScrollingFrame.BackgroundTransparency = 1
ScrollingFrame.BorderSizePixel = 0
ScrollingFrame.ScrollBarThickness = 4
ScrollingFrame.ScrollBarImageColor3 = Color3.fromRGB(0, 150, 255)
ScrollingFrame.CanvasSize = UDim2.new(0, 0, 0, 500)
ScrollingFrame.Parent = MainFrame

local scrollCorner = Instance.new("UICorner")
scrollCorner.CornerRadius = UDim.new(0, 6)
scrollCorner.Parent = ScrollingFrame

local scrollLayout = Instance.new("UIListLayout")
scrollLayout.Padding = UDim.new(0, 6)
scrollLayout.SortOrder = Enum.SortOrder.LayoutOrder
scrollLayout.Parent = ScrollingFrame

-- ============================================================
-- FUNÇÕES DE UI
-- ============================================================
local function createSection(title)
    local section = Instance.new("Frame")
    section.Size = UDim2.new(1, -20, 0, 0)
    section.BackgroundTransparency = 1
    section.LayoutOrder = #ScrollingFrame:GetChildren() + 1
    section.Parent = ScrollingFrame

    local sectionLabel = Instance.new("TextLabel")
    sectionLabel.Size = UDim2.new(1, 0, 0, 22)
    sectionLabel.BackgroundTransparency = 1
    sectionLabel.Text = "── " .. title .. " ──"
    sectionLabel.TextColor3 = Color3.fromRGB(0, 200, 255)
    sectionLabel.TextSize = 12
    sectionLabel.Font = Enum.Font.GothamBold
    sectionLabel.Parent = section

    local itemLayout = Instance.new("UIListLayout")
    itemLayout.Padding = UDim.new(0, 4)
    itemLayout.Parent = section

    return section, itemLayout
end

local function createToggle(parent, text, currentValue, callback)
    local toggleFrame = Instance.new("Frame")
    toggleFrame.Size = UDim2.new(1, 0, 0, 28)
    toggleFrame.BackgroundColor3 = Color3.fromRGB(40, 40, 45)
    toggleFrame.BorderSizePixel = 0
    toggleFrame.Parent = parent

    local toggleCorner = Instance.new("UICorner")
    toggleCorner.CornerRadius = UDim.new(0, 5)
    toggleCorner.Parent = toggleFrame

    local toggleLabel = Instance.new("TextLabel")
    toggleLabel.Size = UDim2.new(1, -70, 1, 0)
    toggleLabel.Position = UDim2.new(0, 10, 0, 0)
    toggleLabel.BackgroundTransparency = 1
    toggleLabel.Text = text
    toggleLabel.TextColor3 = Color3.fromRGB(200, 200, 200)
    toggleLabel.TextSize = 12
    toggleLabel.Font = Enum.Font.GothamMedium
    toggleLabel.TextXAlignment = Enum.TextXAlignment.Left
    toggleLabel.Parent = toggleFrame

    local toggleBtn = Instance.new("TextButton")
    toggleBtn.Size = UDim2.new(0, 50, 0, 20)
    toggleBtn.Position = UDim2.new(1, -58, 0.5, -10)
    toggleBtn.BackgroundColor3 = currentValue and Color3.fromRGB(0, 180, 100) or Color3.fromRGB(80, 80, 80)
    toggleBtn.Text = currentValue and "ON" or "OFF"
    toggleBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
    toggleBtn.TextSize = 10
    toggleBtn.Font = Enum.Font.GothamBold
    toggleBtn.BorderSizePixel = 0
    toggleBtn.Parent = toggleFrame

    local toggleBtnCorner = Instance.new("UICorner")
    toggleBtnCorner.CornerRadius = UDim.new(0, 4)
    toggleBtnCorner.Parent = toggleBtn

    toggleBtn.MouseButton1Click:Connect(function()
        callback(not currentValue)
        if callback == nil then return end
    end)

    return toggleBtn
end

local function updateToggle(button, value, text)
    button.BackgroundColor3 = value and Color3.fromRGB(0, 180, 100) or Color3.fromRGB(80, 80, 80)
    button.Text = value and "ON" or "OFF"
end

local function createSlider(parent, text, minValue, maxValue, defaultValue, callback)
    local sliderFrame = Instance.new("Frame")
    sliderFrame.Size = UDim2.new(1, 0, 0, 45)
    sliderFrame.BackgroundColor3 = Color3.fromRGB(40, 40, 45)
    sliderFrame.BorderSizePixel = 0
    sliderFrame.Parent = parent

    local sliderCorner = Instance.new("UICorner")
    sliderCorner.CornerRadius = UDim.new(0, 5)
    sliderCorner.Parent = sliderFrame

    local sliderLabel = Instance.new("TextLabel")
    sliderLabel.Size = UDim2.new(1, 0, 0, 16)
    sliderLabel.Position = UDim2.new(0, 10, 0, 4)
    sliderLabel.BackgroundTransparency = 1
    sliderLabel.Text = text
    sliderLabel.TextColor3 = Color3.fromRGB(200, 200, 200)
    sliderLabel.TextSize = 11
    sliderLabel.Font = Enum.Font.GothamMedium
    sliderLabel.TextXAlignment = Enum.TextXAlignment.Left
    sliderLabel.Parent = sliderFrame

    local sliderBar = Instance.new("Frame")
    sliderBar.Name = "SliderBar"
    sliderBar.Size = UDim2.new(1, -20, 0, 8)
    sliderBar.Position = UDim2.new(0, 10, 0, 26)
    sliderBar.BackgroundColor3 = Color3.fromRGB(60, 60, 65)
    sliderBar.BorderSizePixel = 0
    sliderBar.Parent = sliderFrame

    local sliderBarCorner = Instance.new("UICorner")
    sliderBarCorner.CornerRadius = UDim.new(0, 4)
    sliderBarCorner.Parent = sliderBar

    local sliderFill = Instance.new("Frame")
    sliderFill.Name = "SliderFill"
    sliderFill.BackgroundColor3 = Color3.fromRGB(0, 150, 255)
    sliderFill.BorderSizePixel = 0
    sliderFill.Size = UDim2.new(0, 0, 1, 0)
    sliderFill.Parent = sliderBar

    local fillCorner = Instance.new("UICorner")
    fillCorner.CornerRadius = UDim.new(0, 4)
    fillCorner.Parent = sliderFill

    local sliderValue = Instance.new("TextLabel")
    sliderValue.Name = "ValueLabel"
    sliderValue.Size = UDim2.new(0, 60, 0, 14)
    sliderValue.Position = UDim2.new(1, -65, 0, 4)
    sliderValue.BackgroundTransparency = 1
    sliderValue.Text = tostring(defaultValue)
    sliderValue.TextColor3 = Color3.fromRGB(0, 180, 255)
    sliderValue.TextSize = 11
    sliderValue.Font = Enum.Font.GothamBold
    sliderValue.Parent = sliderFrame

    local dragging = false
    sliderBar.InputBegan:Connect(function(input)
        if input.UserInputType == Enum.UserInputType.MouseButton1 then
            dragging = true
        end
    end)
    sliderBar.InputEnded:Connect(function(input)
        if input.UserInputType == Enum.UserInputType.MouseButton1 then
            dragging = false
        end
    end)
    game:GetService("UserInputService").InputChanged:Connect(function(input)
        if dragging and input.UserInputType == Enum.UserInputType.MouseMovement then
            local absPos = sliderBar.AbsolutePosition
            local absSize = sliderBar.AbsoluteSize
            local relativeX = (input.Position.X - absPos.X) / absSize.X
            relativeX = math.clamp(relativeX, 0, 1)
            local value = math.floor(minValue + (maxValue - minValue) * relativeX)
            sliderFill.Size = UDim2.new(relativeX, 0, 1, 0)
            sliderValue.Text = tostring(value)
            if callback then callback(value) end
        end
    end)

    -- Inicializar posição
    local initialRatio = (defaultValue - minValue) / (maxValue - minValue)
    sliderFill.Size = UDim2.new(initialRatio, 0, 1, 0)

    return sliderValue
end

-- ============================================================
-- CRIAR SEÇÕES DO GUI
-- ============================================================

-- Seção AIMBOT
local aimbotSection, aimbotLayout = createSection("AIMBOT")
aimbotSection.LayoutOrder = 1

createToggle(aimbotSection, "Ativar Aimbot [E]", false, function(val)
    Config.Aimbot.Enabled = val
    updateToggle(aimbotSection:FindFirstChild("ToggleBtn", true) or aimbotSection:FindFirstChild("TextButton", true), val)
    print("[TEST] Aimbot: " .. (val and "ON" or "OFF"))
end)

createToggle(aimbotSection, "Aim Through Walls", Config.Aimbot.AimThroughWalls, function(val)
    Config.Aimbot.AimThroughWalls = val
    print("[TEST] Aim Through Walls: " .. (val and "ON" or "OFF"))
end)

createToggle(aimbotSection, "Silent Aim", Config.Aimbot.SilentAim, function(val)
    Config.Aimbot.SilentAim = val
    print("[TEST] Silent Aim: " .. (val and "ON" or "OFF"))
end)

createToggle(aimbotSection, "Target: Head", true, function(val)
    Config.Aimbot.TargetPart = val and "Head" or "HumanoidRootPart"
    print("[TEST] Target: " .. (val and "Head" or "HumanoidRootPart"))
end)

local fovSlider = createSlider(aimbotSection, "FOV: " .. Config.Aimbot.FOV, 100, 800, Config.Aimbot.FOV, function(val)
    Config.Aimbot.FOV = val
    fovSlider.Text = "FOV: " .. tostring(val)
end)

local smoothSlider = createSlider(aimbotSection, "Smoothness: " .. Config.Aimbot.Smoothness, 1, 100, math.floor(Config.Aimbot.Smoothness * 100), function(val)
    Config.Aimbot.Smoothness = val / 100
    smoothSlider.Text = "Smooth: " .. tostring(val) .. "%"
end)

-- Seção ESP
local espSection, espLayout = createSection("ESP")
espSection.LayoutOrder = 2

createToggle(espSection, "Ativar ESP [P]", false, function(val)
    Config.ESP.Enabled = val
    if not val then
        for _, player in ipairs(Players:GetPlayers()) do
            removeESP(player)
        end
    end
    print("[TEST] ESP: " .. (val and "ON" or "OFF"))
end)

createToggle(espSection, "Box", Config.ESP.Box, function(val)
    Config.ESP.Box = val
end)

createToggle(espSection, "Tracer", Config.ESP.Tracer, function(val)
    Config.ESP.Tracer = val
end)

createToggle(espSection, "Skeleton", Config.ESP.Skeleton, function(val)
    Config.ESP.Skeleton = val
end)

createToggle(espSection, "Nome", Config.ESP.Name, function(val)
    Config.ESP.Name = val
end)

createToggle(espSection, "Barra de Vida", Config.ESP.Health, function(val)
    Config.ESP.Health = val
end)

-- Seção BINDS
local bindsSection, bindsLayout = createSection("BINDS")
bindsSection.LayoutOrder = 3

local bindLabel = Instance.new("TextLabel")
bindLabel.Size = UDim2.new(1, 0, 0, 28)
bindLabel.BackgroundTransparency = 1
bindLabel.Text = "[E] Toggle Aimbot | [P] Toggle ESP | [MB2] Mirar"
bindLabel.TextColor3 = Color3.fromRGB(180, 180, 180)
bindLabel.TextSize = 11
bindLabel.Font = Enum.Font.GothamMedium
bindLabel.TextWrapped = true
bindLabel.Parent = bindsSection

-- Seção STATUS
local statusSection, statusLayout = createSection("STATUS")
statusSection.LayoutOrder = 4

local statusLabel = Instance.new("TextLabel")
statusLabel.Name = "StatusLabel"
statusLabel.Size = UDim2.new(1, 0, 0, 60)
statusLabel.BackgroundTransparency = 1
statusLabel.Text = "Aimbot: OFF\nESP: OFF\nTarget: Head\nWalls: ON"
statusLabel.TextColor3 = Color3.fromRGB(150, 150, 150)
statusLabel.TextSize = 11
statusLabel.Font = Enum.Font.Code
statusLabel.TextXAlignment = Enum.TextXAlignment.Left
statusLabel.TextYAlignment = Enum.TextYAlignment.Top
statusLabel.Parent = statusSection

-- Atualizar status periodicamente
task.spawn(function()
    while true do
        if ScreenGui.Parent then
            statusLabel.Text = string.format(
                "Aimbot: %s\nESP: %s\nTarget: %s\nWalls: %s\nSmooth: %s%%\nFOV: %s",
                Config.Aimbot.Enabled and "ON ✓" or "OFF",
                Config.ESP.Enabled and "ON ✓" or "OFF",
                Config.Aimbot.TargetPart,
                Config.Aimbot.AimThroughWalls and "ON" or "OFF",
                math.floor(Config.Aimbot.Smoothness * 100),
                Config.Aimbot.FOV
            )
        end
        task.wait(0.5)
    end
end)

-- ============================================================
-- FUNÇÕES ESP (reutilizadas)
-- ============================================================

local function createESP(player)
    local espData = {}

    if Config.ESP.Box then
        local box = Instance.new("Frame")
        box.Name = "SAN_AURE_TEST_BOX"
        box.BackgroundColor3 = Color3.fromRGB(255, 0, 0)
        box.BackgroundTransparency = 0.5
        box.BorderSizePixel = 1
        box.Size = UDim2.new(0, 100, 0, 150)
        box.Visible = false
        box.ZIndex = 100

        local corner = Instance.new("UICorner")
        corner.CornerRadius = UDim.new(0, 2)
        corner.Parent = box

        local outline = Instance.new("UIStroke")
        outline.Thickness = 1
        outline.Color = Color3.fromRGB(255, 0, 0)
        outline.Parent = box

        box.Parent = ScreenGui
        espData.Box = box
    end

    if Config.ESP.Tracer then
        local tracer = Instance.new("Frame")
        tracer.Name = "SAN_AURE_TEST_TRACER"
        tracer.BackgroundColor3 = Color3.fromRGB(0, 255, 0)
        tracer.BorderSizePixel = 0
        tracer.Size = UDim2.new(0, 1, 0, 100)
        tracer.Visible = false
        tracer.ZIndex = 90
        tracer.Parent = ScreenGui
        espData.Tracer = tracer
    end

    if Config.ESP.Skeleton then
        local skeletonFolder = Instance.new("Folder")
        skeletonFolder.Name = "SAN_AURE_TEST_SKELETON"
        skeletonFolder.Parent = ScreenGui
        espData.SkeletonParts = {}
        espData.SkeletonFolder = skeletonFolder
    end

    if Config.ESP.Name then
        local nameLabel = Instance.new("TextLabel")
        nameLabel.Name = "SAN_AURE_TEST_NAME"
        nameLabel.Text = player.Name
        nameLabel.TextColor3 = Color3.fromRGB(255, 255, 255)
        nameLabel.TextStrokeTransparency = 0
        nameLabel.TextSize = 14
        nameLabel.BackgroundTransparency = 1
        nameLabel.Font = Enum.Font.GothamBold
        nameLabel.Size = UDim2.new(0, 200, 0, 20)
        nameLabel.Visible = false
        nameLabel.ZIndex = 110
        nameLabel.Parent = ScreenGui
        espData.NameLabel = nameLabel
    end

    if Config.ESP.Health then
        local healthBar = Instance.new("Frame")
        healthBar.Name = "SAN_AURE_TEST_HEALTH"
        healthBar.BackgroundColor3 = Color3.fromRGB(0, 255, 0)
        healthBar.BorderSizePixel = 0
        healthBar.Size = UDim2.new(0, 100, 0, 5)
        healthBar.Visible = false
        healthBar.ZIndex = 105

        local hc = Instance.new("UICorner")
        hc.CornerRadius = UDim.new(0, 2)
        hc.Parent = healthBar

        healthBar.Parent = ScreenGui
        espData.HealthBar = healthBar
    end

    ESPObjects[player.UserId] = espData
end

function removeESP(player)
    local espData = ESPObjects[player.UserId]
    if espData then
        for _, obj in pairs(espData) do
            if typeof(obj) == "Instance" then
                obj:Destroy()
            end
        end
        if espData.SkeletonFolder then
            espData.SkeletonFolder:Destroy()
        end
        ESPObjects[player.UserId] = nil
    end
end

local function updateSkeletonBone(espData, part1, part2, character)
    if not part1 or not part2 or not character then return end

    local screenPos1, onScreen1 = Camera:WorldToViewportPoint(part1.Position)
    local screenPos2, onScreen2 = Camera:WorldToViewportPoint(part2.Position)

    if not onScreen1 or not onScreen2 then return end

    local startPos = Vector2.new(screenPos1.X, screenPos1.Y)
    local endPos = Vector2.new(screenPos2.X, screenPos2.Y)

    local distance = (endPos - startPos).Magnitude
    local midpoint = (startPos + endPos) / 2

    local bone = espData.SkeletonParts[part1.Name .. "_" .. part2.Name]
    if not bone then
        bone = Instance.new("Frame")
        bone.Name = "SAN_AURE_TEST_BONE"
        bone.BackgroundColor3 = Color3.fromRGB(0, 200, 255)
        bone.BorderSizePixel = 0
        bone.ZIndex = 100
        bone.Parent = espData.SkeletonFolder
        espData.SkeletonParts[part1.Name .. "_" .. part2.Name] = bone
    end

    local angle = math.atan2(endPos.Y - startPos.Y, endPos.X - startPos.X)
    bone.Size = UDim2.new(0, distance, 0, 2)
    bone.Position = UDim2.new(0, midpoint.X, 0, midpoint.Y)
    bone.Rotation = math.deg(angle)
    bone.AnchorPoint = Vector2.new(0.5, 0.5)
    bone.Visible = true
end

-- ============================================================
-- FUNÇÕES AIMBOT
-- ============================================================

local function isValidTarget(player)
    if player == LocalPlayer then return false end
    if not player.Character then return false end
    local humanoid = player.Character:FindFirstChild("Humanoid")
    if not humanoid or humanoid.Health <= 0 then return false end
    if not player.Character:FindFirstChild("Head") then return false end
    if not player.Character:FindFirstChild("HumanoidRootPart") then return false end
    return true
end

local function isHeadVisible(player)
    if Config.Aimbot.AimThroughWalls then return true end
    local head = player.Character:FindFirstChild("Head")
    if not head then return false end
    local origin = Camera.CFrame.Position
    local direction = head.Position - origin
    local ray = workspace:Raycast(origin, direction)
    if ray and ray.Instance then
        local targetCharacter = player.Character
        local hitPart = ray.Instance
        while hitPart and hitPart.Parent do
            if hitPart.Parent == targetCharacter then return true end
            hitPart = hitPart.Parent
        end
        return false
    end
    return true
end

local function getAngleFromCenter(position)
    local screenPos, onScreen = Camera:WorldToViewportPoint(position)
    if not onScreen then return math.huge end
    local center = Vector2.new(Camera.ViewportSize.X / 2, Camera.ViewportSize.Y / 2)
    return (Vector2.new(screenPos.X, screenPos.Y) - center).Magnitude
end

local function getBestTarget()
    local bestTarget = nil
    local bestAngle = Config.Aimbot.FOV
    for _, player in ipairs(Players:GetPlayers()) do
        if isValidTarget(player) and isHeadVisible(player) then
            local head = player.Character:FindFirstChild("Head")
            if head then
                local angle = getAngleFromCenter(head.Position)
                if angle < bestAngle then
                    bestAngle = angle
                    bestTarget = player
                end
            end
        end
    end
    return bestTarget
end

local function getBestTargetThroughWalls()
    local bestTarget = nil
    local bestAngle = Config.Aimbot.FOV
    for _, player in ipairs(Players:GetPlayers()) do
        if isValidTarget(player) then
            local head = player.Character:FindFirstChild("Head")
            if head then
                local angle = getAngleFromCenter(head.Position)
                if angle < bestAngle then
                    bestAngle = angle
                    bestTarget = player
                end
            end
        end
    end
    return bestTarget
end

-- ============================================================
-- LOOP AIMBOT
-- ============================================================
task.spawn(function()
    while true do
        if Config.Aimbot.Enabled and aimbotActive then
            local target
            if Config.Aimbot.AimThroughWalls then
                target = getBestTargetThroughWalls()
            else
                target = getBestTarget()
            end

            if target and target.Character then
                local targetPart = target.Character:FindFirstChild(Config.Aimbot.TargetPart)
                if targetPart and not Config.Aimbot.SilentAim then
                    local targetScreenPos = Camera:WorldToViewportPoint(targetPart.Position)
                    if targetScreenPos then
                        local mousePosition = Vector2.new(LocalMouse.X, LocalMouse.Y)
                        local direction = (Vector2.new(targetScreenPos.X, targetScreenPos.Y) - mousePosition) * Config.Aimbot.Smoothness

                        if Config.Security.RandomizedDelays then
                            local randomOffset = Vector2.new(
                                math.random(-1, 1),
                                math.random(-1, 1)
                            )
                            direction = direction + randomOffset
                        end

                        mousemoverel(direction.X, direction.Y)
                        aimbotTarget = target
                    end
                end
            end

            if Config.Security.RandomizedDelays then
                task.wait(math.random(1, 3) / 1000)
            else
                task.wait(0.01)
            end
        else
            task.wait(0.1)
        end
    end
end)

-- ============================================================
-- LOOP ESP
-- ============================================================
RunService.RenderStepped:Connect(function()
    if not Config.ESP.Enabled then return end

    local cameraViewportSize = Camera.ViewportSize

    for _, player in ipairs(Players:GetPlayers()) do
        if player ~= LocalPlayer and player.Character then
            local espData = ESPObjects[player.UserId]
            if not espData then
                createESP(player)
                espData = ESPObjects[player.UserId]
            end

            local humanoidRootPart = player.Character:FindFirstChild("HumanoidRootPart")
            local head = player.Character:FindFirstChild("Head")
            local humanoid = player.Character:FindFirstChild("Humanoid")

            if humanoidRootPart and head and humanoid and humanoid.Health > 0 then
                local localHRP = LocalPlayer.Character and LocalPlayer.Character:FindFirstChild("HumanoidRootPart")
                local distance = localHRP and (localHRP.Position - humanoidRootPart.Position).Magnitude or math.huge

                if distance <= Config.ESP.MaxDistance then
                    local headPos, onScreen = Camera:WorldToViewportPoint(head.Position)
                    local hrpPos = Camera:WorldToViewportPoint(humanoidRootPart.Position)

                    if onScreen then
                        local boxHeight = math.abs(headPos.Y - hrpPos.Y)
                        local boxWidth = boxHeight * 0.6

                        if espData.Box then
                            espData.Box.Size = UDim2.new(0, boxWidth, 0, boxHeight)
                            espData.Box.Position = UDim2.new(0, headPos.X - boxWidth / 2, 0, headPos.Y)
                            espData.Box.Visible = true
                            local hp = humanoid.Health / humanoid.MaxHealth
                            local boxColor = Color3.new(1, hp, 0)
                            espData.Box.BackgroundColor3 = boxColor
                            espData.Box.UIStroke.Color = boxColor
                        end

                        if espData.Tracer then
                            local tracerStart = Vector2.new(cameraViewportSize.X / 2, cameraViewportSize.Y)
                            local tracerEnd = Vector2.new(hrpPos.X, hrpPos.Y)
                            local tracerMidpoint = (tracerStart + tracerEnd) / 2
                            local tracerDistance = (tracerEnd - tracerStart).Magnitude
                            local tracerAngle = math.atan2(tracerEnd.Y - tracerStart.Y, tracerEnd.X - tracerStart.X)

                            espData.Tracer.Size = UDim2.new(0, tracerDistance, 0, 2)
                            espData.Tracer.Position = UDim2.new(0, tracerMidpoint.X, 0, tracerMidpoint.Y)
                            espData.Tracer.Rotation = math.deg(tracerAngle)
                            espData.Tracer.AnchorPoint = Vector2.new(0.5, 0.5)
                            espData.Tracer.Visible = true
                        end

                        if Config.ESP.Skeleton and espData.SkeletonParts then
                            local character = player.Character
                            local bodyParts = {
                                {"Head", "HumanoidRootPart"},
                                {"HumanoidRootPart", "LowerTorso"},
                                {"LowerTorso", "UpperTorso"},
                                {"UpperTorso", "LeftUpperArm"},
                                {"UpperTorso", "RightUpperArm"},
                                {"LeftUpperArm", "LeftLowerArm"},
                                {"RightUpperArm", "RightLowerArm"},
                                {"LeftLowerArm", "LeftHand"},
                                {"RightLowerArm", "RightHand"},
                                {"LowerTorso", "LeftUpperLeg"},
                                {"LowerTorso", "RightUpperLeg"},
                                {"LeftUpperLeg", "LeftLowerLeg"},
                                {"RightUpperLeg", "RightLowerLeg"},
                                {"LeftLowerLeg", "LeftFoot"},
                                {"RightLowerLeg", "RightFoot"},
                            }

                            for _, pair in ipairs(bodyParts) do
                                local part1 = character:FindFirstChild(pair[1])
                                local part2 = character:FindFirstChild(pair[2])
                                if part1 and part2 then
                                    updateSkeletonBone(espData, part1, part2, character)
                                end
                            end
                        end

                        if espData.NameLabel then
                            espData.NameLabel.Position = UDim2.new(0, headPos.X - 100, 0, headPos.Y - 25)
                            espData.NameLabel.Visible = true
                        end

                        if espData.HealthBar then
                            local hp = humanoid.Health / humanoid.MaxHealth
                            espData.HealthBar.Size = UDim2.new(0, boxWidth or 100, 0, 5)
                            espData.HealthBar.Position = UDim2.new(0, headPos.X - (boxWidth or 100) / 2, 0, headPos.Y - 10)
                            espData.HealthBar.BackgroundColor3 = Color3.fromRGB(
                                math.floor(255 * (1 - hp)),
                                math.floor(255 * hp),
                                0
                            )
                            espData.HealthBar.Visible = true
                        end
                    else
                        if espData.Box then espData.Box.Visible = false end
                        if espData.Tracer then espData.Tracer.Visible = false end
                        if espData.NameLabel then espData.NameLabel.Visible = false end
                        if espData.HealthBar then espData.HealthBar.Visible = false end
                        if espData.SkeletonParts then
                            for _, bone in pairs(espData.SkeletonParts) do
                                bone.Visible = false
                            end
                        end
                    end
                end
            end
        end
    end
end)

-- ============================================================
-- BINDS
-- ============================================================
UserInputService.InputBegan:Connect(function(input, gameProcessed)
    if gameProcessed then return end

    if input.KeyCode == Config.Aimbot.Key then
        Config.Aimbot.Enabled = not Config.Aimbot.Enabled
        print("[TEST] Aimbot " .. (Config.Aimbot.Enabled and "ON" or "OFF"))
    end

    if input.KeyCode == Config.ESP.Key then
        Config.ESP.Enabled = not Config.ESP.Enabled
        if not Config.ESP.Enabled then
            for _, player in ipairs(Players:GetPlayers()) do
                removeESP(player)
            end
        end
        print("[TEST] ESP " .. (Config.ESP.Enabled and "ON" or "OFF"))
    end
end)

UserInputService.InputBegan:Connect(function(input, gameProcessed)
    if gameProcessed then return end
    if input.UserInputType == Config.Aimbot.LockKey and Config.Aimbot.Enabled then
        aimbotActive = true
    end
end)

UserInputService.InputEnded:Connect(function(input)
    if input.UserInputType == Config.Aimbot.LockKey then
        aimbotActive = false
    end
end)

-- ============================================================
-- LIMPEZA
-- ============================================================
local function cleanup()
    for _, player in ipairs(Players:GetPlayers()) do
        removeESP(player)
    end
    ScreenGui:Destroy()
    print("[TEST] Limpeza concluída.")
end

game:BindToClose(cleanup)

-- ============================================================
-- MENSAGEM INICIAL
-- ============================================================
print([[
╔══════════════════════════════════════════╗
║  SAN AURE - AIMBOT + ESP COM GUI        ║
║  Objetivo: Teste de Anti-Cheat          ║
╠══════════════════════════════════════════╣
║  Binds:                                 ║
║  [E]     - Toggle Aimbot               ║
║  [P]     - Toggle ESP                  ║
║  [MB2]   - Segurar para mirar          ║
╚══════════════════════════════════════════╝
O painel já deve estar visível no canto superior esquerdo!
]])
