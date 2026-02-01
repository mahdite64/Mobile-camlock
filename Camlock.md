--// SERVICES
local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local UserInputService = game:GetService("UserInputService")
local LocalPlayer = Players.LocalPlayer
local Camera = workspace.CurrentCamera

--// GUI (Minimal Mobile)
local ScreenGui = Instance.new("ScreenGui")
ScreenGui.Name = "MobileAimESP"
ScreenGui.ResetOnSpawn = false
ScreenGui.Parent = LocalPlayer:WaitForChild("PlayerGui")

local ToggleBtn = Instance.new("TextButton")
ToggleBtn.Size = UDim2.new(0, 80, 0, 30)
ToggleBtn.Position = UDim2.new(0, 10, 0, 10)
ToggleBtn.Text = "☰"
ToggleBtn.BackgroundColor3 = Color3.fromRGB(30, 30, 30)
ToggleBtn.TextColor3 = Color3.fromRGB(220, 220, 220)
ToggleBtn.BorderSizePixel = 0
ToggleBtn.Parent = ScreenGui

local Panel = Instance.new("Frame")
Panel.Size = UDim2.new(0, 180, 0, 80)
Panel.Position = UDim2.new(0, 10, 0, 50)
Panel.BackgroundColor3 = Color3.fromRGB(20, 20, 20)
Panel.Visible = false
Panel.Parent = ScreenGui

ToggleBtn.MouseButton1Click:Connect(function()
    Panel.Visible = not Panel.Visible
end)

--// ESP (FROM YOUR ORIGINAL — NO CHANGES)
local ESPBoxes = {}
local NameLabels = {}
local espEnabled = false

local function addESP(player)
    if player == LocalPlayer or not player.Character then return end
    local box = ESPBoxes[player] or Drawing.new("Square")
    box.Color = Color3.fromRGB(255, 255, 255)
    box.Thickness = 2
    box.Transparency = 1
    box.Filled = false
    ESPBoxes[player] = box

    local name = NameLabels[player] or Drawing.new("Text")
    name.Text = player.Name
    name.Size = 15
    name.Color = Color3.fromRGB(255, 255, 255)
    name.Center = true
    name.Outline = true
    NameLabels[player] = name
end

local function updateESP()
    if not espEnabled then return end
    for player, box in pairs(ESPBoxes) do
        if player.Character and player.Character:FindFirstChild("HumanoidRootPart") then
            local root = player.Character.HumanoidRootPart
            local screenPos, onScreen = Camera:WorldToViewportPoint(root.Position)
            if onScreen then
                box.Size = Vector2.new(2000 / screenPos.Z, 3000 / screenPos.Z)
                box.Position = Vector2.new(screenPos.X - box.Size.X/2, screenPos.Y - box.Size.Y/2)
                box.Visible = true
                NameLabels[player].Position = Vector2.new(screenPos.X, screenPos.Y - box.Size.Y/2 - 10)
                NameLabels[player].Visible = true
            else
                box.Visible = false
                NameLabels[player].Visible = false
            end
        else
            box.Visible = false
            NameLabels[player].Visible = false
        end
    end
end

-- ESP TOGGLE
local ESPBtn = Instance.new("TextButton")
ESPBtn.Size = UDim2.new(1, -10, 0, 30)
ESPBtn.Position = UDim2.new(0, 5, 0, 5)
ESPBtn.Text = "ESP: OFF"
ESPBtn.BackgroundColor3 = Color3.fromRGB(50, 50, 50)
ESPBtn.TextColor3 = Color3.fromRGB(220, 220, 220)
ESPBtn.BorderSizePixel = 0
ESPBtn.Parent = Panel

ESPBtn.MouseButton1Click:Connect(function()
    espEnabled = not espEnabled
    ESPBtn.Text = "ESP: " .. (espEnabled and "ON" or "OFF")
    ESPBtn.BackgroundColor3 = espEnabled and Color3.fromRGB(100, 200, 100) or Color3.fromRGB(50, 50, 50)

    if espEnabled then
        for _, plr in ipairs(Players:GetPlayers()) do
            addESP(plr)
        end
        Players.PlayerAdded:Connect(function(plr)
            plr.CharacterAdded:Connect(function()
                task.wait(0.5)
                addESP(plr)
            end)
        end)
    else
        for _, box in pairs(ESPBoxes) do
            box.Visible = false
        end
        for _, name in pairs(NameLabels) do
            name.Visible = false
        end
    end
end)

--// CAMLOCK: BOX THAT FREEZES AFTER 7s → THEN TAP-TO-LOCK
local camlockEnabled = false
local aimBox = nil
local isFrozen = false
local isTouching = false
local dragAllowed = true
local predictionTime = 0.1

-- Create box
local function createAimBox()
    if aimBox then aimBox:Destroy() end
    aimBox = Instance.new("TextButton")
    aimBox.Size = UDim2.new(0, 60, 0, 60)
    aimBox.Position = UDim2.new(0.5, -30, 0.5, -30)
    aimBox.AnchorPoint = Vector2.new(0.5, 0.5)
    aimBox.BackgroundColor3 = Color3.fromRGB(70, 130, 180)
    aimBox.Text = "🎯"
    aimBox.TextColor3 = Color3.fromRGB(255, 255, 255)
    aimBox.BorderSizePixel = 0
    aimBox.Parent = ScreenGui

    -- DRAG (only if not frozen)
    local dragging = false
    local dragStart, startPos

    aimBox.InputBegan:Connect(function(input)
        if input.UserInputType == Enum.UserInputType.Touch or input.UserInputType == Enum.UserInputType.MouseButton1 then
            if isFrozen then
                isTouching = true
            else
                if dragAllowed then
                    dragging = true
                    dragStart = input.Position
                    startPos = aimBox.Position
                end
            end
        end
    end)

    aimBox.InputEnded:Connect(function(input)
        if input.UserInputType == Enum.UserInputType.Touch or input.UserInputType == Enum.UserInputType.MouseButton1 then
            dragging = false
            isTouching = false
        end
    end)

    UserInputService.InputChanged:Connect(function(input)
        if dragging and dragAllowed and (input.UserInputType == Enum.UserInputType.MouseMovement or input.UserInputType == Enum.UserInputType.Touch) then
            local delta = input.Position - dragStart
            aimBox.Position = UDim2.new(
                startPos.X.Scale, startPos.X.Offset + delta.X,
                startPos.Y.Scale, startPos.Y.Offset + delta.Y
            )
            dragStart = input.Position
        end
    end)

    -- Start 7-second freeze timer
    task.delay(7, function()
        if camlockEnabled and aimBox then
            isFrozen = true
            dragAllowed = false
            aimBox.BackgroundColor3 = Color3.fromRGB(180, 70, 70)
            aimBox.Text = "🔒"
        end
    end)
end

-- Camlock Toggle
local CamlockBtn = Instance.new("TextButton")
CamlockBtn.Size = UDim2.new(1, -10, 0, 30)
CamlockBtn.Position = UDim2.new(0, 5, 0, 40)
CamlockBtn.Text = "Camlock: OFF"
CamlockBtn.BackgroundColor3 = Color3.fromRGB(50, 50, 50)
CamlockBtn.TextColor3 = Color3.fromRGB(220, 220, 220)
CamlockBtn.BorderSizePixel = 0
CamlockBtn.Parent = Panel

CamlockBtn.MouseButton1Click:Connect(function()
    if camlockEnabled then
        -- OFF
        camlockEnabled = false
        isFrozen = false
        isTouching = false
        dragAllowed = true
        if aimBox then aimBox:Destroy() aimBox = nil end
        CamlockBtn.Text = "Camlock: OFF"
        CamlockBtn.BackgroundColor3 = Color3.fromRGB(50, 50, 50)
    else
        -- ON
        camlockEnabled = true
        isFrozen = false
        isTouching = false
        dragAllowed = true
        createAimBox()
        CamlockBtn.Text = "Camlock: ON"
        CamlockBtn.BackgroundColor3 = Color3.fromRGB(70, 130, 180)
    end
end)

-- Aim logic (same as your original)
local function getNearestTarget()
    local nearest = nil
    local bestAngle = math.rad(15)
    for _, plr in ipairs(Players:GetPlayers()) do
        if plr ~= LocalPlayer and plr.Character and plr.Character:FindFirstChild("Head") then
            local head = plr.Character.Head
            local dir = (head.Position - Camera.CFrame.Position).Unit
            local angle = math.acos(Camera.CFrame.LookVector:Dot(dir))
            if angle < bestAngle then
                bestAngle = angle
                nearest = head
            end
        end
    end
    return nearest
end

local function aimAt(head)
    if not head or not head.Parent then return end
    local root = head.Parent:FindFirstChild("HumanoidRootPart") or head.Parent:FindFirstChild("Torso")
    if not root then return end
    local predicted = head.Position + (root.Velocity * predictionTime)
    Camera.CFrame = CFrame.lookAt(Camera.CFrame.Position, predicted)
end

-- Main loop
RunService.RenderStepped:Connect(function()
    -- ESP
    if espEnabled then
        for _, plr in ipairs(Players:GetPlayers()) do
            if not ESPBoxes[plr] then addESP(plr) end
        end
        updateESP()
    end

    -- Camlock: ONLY active when box is frozen AND you're touching it
    if camlockEnabled and isFrozen and isTouching then
        local target = getNearestTarget()
        if target then
            aimAt(target)
        end
    end
end)

-- Cleanup
game:BindToClose(function()
    if aimBox then aimBox:Destroy() end
    for _, b in pairs(ESPBoxes) do b:Remove() end
    for _, n in pairs(NameLabels) do n:Remove() end
end)
