--// SERVICES
local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local UserInputService = game:GetService("UserInputService")
local LocalPlayer = Players.LocalPlayer
local Camera = workspace.CurrentCamera

--// SIMPLE GUI
local ScreenGui = Instance.new("ScreenGui")
ScreenGui.ResetOnSpawn = false
ScreenGui.Parent = LocalPlayer:WaitForChild("PlayerGui")

-- Toggle Button (top-left)
local ToggleBtn = Instance.new("TextButton")
ToggleBtn.Size = UDim2.new(0, 50, 0, 30)
ToggleBtn.Position = UDim2.new(0, 10, 0, 10)
ToggleBtn.Text = "☰"
ToggleBtn.BackgroundColor3 = Color3.fromRGB(50, 50, 50)
ToggleBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
ToggleBtn.Parent = ScreenGui

-- Main Panel
local Panel = Instance.new("Frame")
Panel.Size = UDim2.new(0, 180, 0, 170) -- taller for new buttons
Panel.Position = UDim2.new(0, 10, 0, 50)
Panel.BackgroundColor3 = Color3.fromRGB(40, 40, 40)
Panel.Visible = false
Panel.Parent = ScreenGui

-- ESP Names Button
local ESPBtn = Instance.new("TextButton")
ESPBtn.Size = UDim2.new(1, 0, 0, 30)
ESPBtn.Position = UDim2.new(0, 0, 0, 0)
ESPBtn.Text = "ESP: OFF"
ESPBtn.BackgroundColor3 = Color3.fromRGB(60, 60, 60)
ESPBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
ESPBtn.Parent = Panel

-- Camlock Button
local CamlockBtn = Instance.new("TextButton")
CamlockBtn.Size = UDim2.new(1, 0, 0, 30)
CamlockBtn.Position = UDim2.new(0, 0, 0, 35)
CamlockBtn.Text = "Camlock: OFF"
CamlockBtn.BackgroundColor3 = Color3.fromRGB(60, 60, 60)
CamlockBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
CamlockBtn.Parent = Panel

-- Highlighted ESP Button
local HighlightESPBtn = Instance.new("TextButton")
HighlightESPBtn.Size = UDim2.new(1, 0, 0, 30)
HighlightESPBtn.Position = UDim2.new(0, 0, 0, 70)
HighlightESPBtn.Text = "H-ESP: OFF"
HighlightESPBtn.BackgroundColor3 = Color3.fromRGB(60, 60, 60)
HighlightESPBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
HighlightESPBtn.Parent = Panel

-- Click TP Button
local TPBtn = Instance.new("TextButton")
TPBtn.Size = UDim2.new(1, 0, 0, 30)
TPBtn.Position = UDim2.new(0, 0, 0, 105)
TPBtn.Text = "Click TP: OFF"
TPBtn.BackgroundColor3 = Color3.fromRGB(60, 60, 60)
TPBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
TPBtn.Parent = Panel

-- Tool ESP Button
local ToolESPBtn = Instance.new("TextButton")
ToolESPBtn.Size = UDim2.new(1, 0, 0, 30)
ToolESPBtn.Position = UDim2.new(0, 0, 0, 140)
ToolESPBtn.Text = "Tool ESP: OFF"
ToolESPBtn.BackgroundColor3 = Color3.fromRGB(60, 60, 60)
ToolESPBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
ToolESPBtn.Parent = Panel

-- Toggle visibility
ToggleBtn.MouseButton1Click:Connect(function()
    Panel.Visible = not Panel.Visible
end)

--// ESP NAMES
local espEnabled = false
local ESPBoxes = {}
local NameLabels = {}

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

ESPBtn.MouseButton1Click:Connect(function()
    espEnabled = not espEnabled
    ESPBtn.Text = "ESP: " .. (espEnabled and "ON" or "OFF")
    ESPBtn.BackgroundColor3 = espEnabled and Color3.fromRGB(100, 200, 100) or Color3.fromRGB(60, 60, 60)
end)

--// HIGHLIGHTED ESP
local highlightedESPEnabled = false
local highlightedESPs = {}

local function createHighlight(player)
    if player == LocalPlayer then return end
    local character = player.Character
    if not character then return end
    if character:FindFirstChild("ESPHighlight") then return end

    local highlight = Instance.new("Highlight")
    highlight.Name = "ESPHighlight"
    highlight.Adornee = character
    highlight.FillColor = Color3.fromRGB(128, 0, 128)
    highlight.OutlineColor = Color3.fromRGB(128, 0, 128)
    highlight.FillTransparency = 0.5
    highlight.OutlineTransparency = 0
    highlight.DepthMode = Enum.HighlightDepthMode.AlwaysOnTop
    highlight.Parent = character

    highlightedESPs[player] = highlight
end

HighlightESPBtn.MouseButton1Click:Connect(function()
    highlightedESPEnabled = not highlightedESPEnabled
    HighlightESPBtn.Text = "H-ESP: " .. (highlightedESPEnabled and "ON" or "OFF")
    HighlightESPBtn.BackgroundColor3 = highlightedESPEnabled and Color3.fromRGB(100, 200, 100) or Color3.fromRGB(60, 60, 60)

    if not highlightedESPEnabled then
        for player, highlight in pairs(highlightedESPs) do
            if highlight and highlight.Parent then
                highlight:Destroy()
            end
            highlightedESPs[player] = nil
        end
    else
        for _, player in pairs(Players:GetPlayers()) do
            if player ~= LocalPlayer then
                createHighlight(player)
            end
        end
    end
end)

task.spawn(function()
    while true do
        if highlightedESPEnabled then
            for _, player in pairs(Players:GetPlayers()) do
                if player ~= LocalPlayer then
                    local character = player.Character
                    if character and not character:FindFirstChild("ESPHighlight") then
                        createHighlight(player)
                    end
                end
            end
        end
        task.wait(0.01)
    end
end)

Players.PlayerAdded:Connect(function(player)
    player.CharacterAdded:Connect(function()
        if highlightedESPEnabled then
            task.wait(0.5)
            createHighlight(player)
        end
    end)
end)

--// CLICK TP TOOL
local mouse = LocalPlayer:GetMouse()
local tool
local renderSteppedConnection
local tpEnabled = false

local function createClickTPTool()
    if LocalPlayer.Backpack:FindFirstChild("Click TP Tool") then
        LocalPlayer.Backpack:FindFirstChild("Click TP Tool"):Destroy()
    end

    tool = Instance.new("Tool")
    tool.Name = "Click TP Tool"
    tool.RequiresHandle = false
    tool.Parent = LocalPlayer.Backpack

    local crosshair
    tool.Equipped:Connect(function()
        if crosshair then
            crosshair:Remove()
        end
        if renderSteppedConnection then
            renderSteppedConnection:Disconnect()
            renderSteppedConnection = nil
        end

        crosshair = Drawing.new("Text")
        crosshair.Size = 20
        crosshair.Center = true
        crosshair.Outline = true
        crosshair.Text = ""
        crosshair.Color = Color3.new(1, 1, 1)
        crosshair.Visible = true

        renderSteppedConnection = RunService.RenderStepped:Connect(function()
            if crosshair and crosshair.Visible then
                local success, mouseLocation = pcall(function()
                    return UserInputService:GetMouseLocation()
                end)
                if success and mouseLocation then
                    crosshair.Position = Vector2.new(mouseLocation.X, mouseLocation.Y)
                end
            end
        end)
    end)

    tool.Unequipped:Connect(function()
        if crosshair then
            crosshair:Remove()
            crosshair = nil
        end
        if renderSteppedConnection then
            renderSteppedConnection:Disconnect()
            renderSteppedConnection = nil
        end
    end)

    tool.Activated:Connect(function()
        local targetPosition = mouse.Hit.p
        local character = LocalPlayer.Character
        if character and character:FindFirstChild("HumanoidRootPart") then
            character.HumanoidRootPart.CFrame = CFrame.new(targetPosition)
        end
    end)
end

TPBtn.MouseButton1Click:Connect(function()
    tpEnabled = not tpEnabled
    TPBtn.Text = "Click TP: " .. (tpEnabled and "ON" or "OFF")
    TPBtn.BackgroundColor3 = tpEnabled and Color3.fromRGB(100, 200, 100) or Color3.fromRGB(60, 60, 60)

    if tpEnabled then
        createClickTPTool()
    else
        if LocalPlayer.Backpack:FindFirstChild("Click TP Tool") then
            LocalPlayer.Backpack:FindFirstChild("Click TP Tool"):Destroy()
        end
        if LocalPlayer.Character and LocalPlayer.Character:FindFirstChild("Click TP Tool") then
            LocalPlayer.Character:FindFirstChild("Click TP Tool"):Destroy()
        end
    end
end)

--// TOOL ESP
local ToolLabels = {}
local toolESPEnabled = false

local function addToolESP(player)
    if player == LocalPlayer or not player.Character then return end
    local label = ToolLabels[player] or Drawing.new("Text")
    label.Text = "Tool: loading..."
    label.Size = 14
    label.Color = Color3.fromRGB(255, 255, 0) -- Yellow
    label.Center = true
    label.Outline = true
    label.Visible = false
    ToolLabels[player] = label
end

local function updateToolESP()
    if not toolESPEnabled then return end
    for player, label in pairs(ToolLabels) do
        if player.Character and player.Character:FindFirstChild("HumanoidRootPart") then
            local character = player.Character
            local toolName = "nothing"
            
            -- Check character for equipped tool
            for _, child in ipairs(character:GetChildren()) do
                if child:IsA("Tool") then
                    toolName = child.Name
                    break
                end
            end
            -- If none, check backpack
            if toolName == "nothing" then
                local backpack = player:FindFirstChild("Backpack")
                if backpack then
                    for _, item in ipairs(backpack:GetChildren()) do
                        if item:IsA("Tool") then
                            toolName = item.Name
                            break
                        end
                    end
                end
            end

            label.Text = "Tool: " .. toolName
            local pos = character.HumanoidRootPart.Position + Vector3.new(0, 4.5, 0)
            local onScreen, screenPos = Camera:WorldToViewportPoint(pos)
            if onScreen then
                label.Position = Vector2.new(screenPos.X, screenPos.Y)
                label.Visible = true
            else
                label.Visible = false
            end
        else
            label.Visible = false
        end
    end
end

ToolESPBtn.MouseButton1Click:Connect(function()
    toolESPEnabled = not toolESPEnabled
    ToolESPBtn.Text = "Tool ESP: " .. (toolESPEnabled and "ON" or "OFF")
    ToolESPBtn.BackgroundColor3 = toolESPEnabled and Color3.fromRGB(100, 200, 100) or Color3.fromRGB(60, 60, 60)

    if toolESPEnabled then
        for _, plr in ipairs(Players:GetPlayers()) do
            if not ToolLabels[plr] then
                addToolESP(plr)
            end
        end
    else
        for _, label in pairs(ToolLabels) do
            label.Visible = false
        end
    end
end)

--// CAMLOCK LOGIC
local camlockEnabled = false
local aimBox = nil
local isFrozen = false
local isTouching = false
local dragAllowed = true
local predictionTime = 0.1
local lockedTarget = nil

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

    local dragging = false
    local dragStart, startPos

    aimBox.InputBegan:Connect(function(input)
        if input.UserInputType == Enum.UserInputType.Touch or input.UserInputType == Enum.UserInputType.MouseButton1 then
            if isFrozen then
                isTouching = true
                lockedTarget = getNearestTarget()
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
            lockedTarget = nil
        end
    end)

    UserInputService.InputChanged:Connect(function(input)
        if dragging and dragAllowed and (input.UserInputType == Enum.UserInputType.MouseMovement or input.UserInputType == Enum.UserInputType.Touch) then
            local delta = Vector2.new(input.Position.X - dragStart.X, input.Position.Y - dragStart.Y)
            aimBox.Position = UDim2.new(
                startPos.X.Scale, startPos.X.Offset + delta.X,
                startPos.Y.Scale, startPos.Y.Offset + delta.Y
            )
        end
    end)

    task.delay(7, function()
        if camlockEnabled and aimBox then
            isFrozen = true
            dragAllowed = false
            aimBox.BackgroundColor3 = Color3.fromRGB(180, 70, 70)
            aimBox.Text = "🔒"
        end
    end)
end

CamlockBtn.MouseButton1Click:Connect(function()
    if camlockEnabled then
        camlockEnabled = false
        isFrozen = false
        isTouching = false
        dragAllowed = true
        lockedTarget = nil
        if aimBox then aimBox:Destroy() aimBox = nil end
        CamlockBtn.Text = "Camlock: OFF"
        CamlockBtn.BackgroundColor3 = Color3.fromRGB(60, 60, 60)
    else
        camlockEnabled = true
        isFrozen = false
        isTouching = false
        dragAllowed = true
        lockedTarget = nil
        createAimBox()
        CamlockBtn.Text = "Camlock: ON"
        CamlockBtn.BackgroundColor3 = Color3.fromRGB(70, 130, 180)
    end
end)

--// MAIN LOOP
RunService.RenderStepped:Connect(function()
    if espEnabled then
        for _, plr in ipairs(Players:GetPlayers()) do
            if not ESPBoxes[plr] then addESP(plr) end
        end
        updateESP()
    end

    if toolESPEnabled then
        for _, plr in ipairs(Players:GetPlayers()) do
            if not ToolLabels[plr] then addToolESP(plr) end
        end
        updateToolESP()
    end

    if camlockEnabled and isFrozen and isTouching and lockedTarget then
        if lockedTarget.Parent and lockedTarget.Parent:FindFirstChild("HumanoidRootPart") then
            aimAt(lockedTarget)
        else
            isTouching = false
            lockedTarget = nil
        end
    end
end)

--// CLEANUP
game:BindToClose(function()
    if aimBox then aimBox:Destroy() end
    for _, b in pairs(ESPBoxes) do pcall(function() b:Remove() end) end
    for _, n in pairs(NameLabels) do pcall(function() n:Remove() end) end
    for _, h in pairs(highlightedESPs) do
        if h and h.Parent then h:Destroy() end
    end
    for _, t in pairs(ToolLabels) do
        pcall(function() t:Remove() end)
    end
    if tool then tool:Destroy() end
end)
