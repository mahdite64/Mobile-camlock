--// SERVICES
local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local UserInputService = game:GetService("UserInputService")
local LocalPlayer = Players.LocalPlayer
local Camera = workspace.CurrentCamera
local Chat = game:GetService("Chat")

--// ADD KICK COMMAND VIA CHAT
local function handleChat(msg)
    msg = msg:lower()
    if msg:sub(1, 6) == "/kick " then
        local targetName = msg:sub(7)
        if targetName == "" then return end

        -- Find player (case-insensitive)
        local targetPlayer = nil
        for _, plr in ipairs(Players:GetPlayers()) do
            if plr.Name:lower() == targetName:lower() then
                targetPlayer = plr
                break
            end
        end

        if targetPlayer and targetPlayer ~= LocalPlayer then
            -- Kick them with custom message
            local kickMsg = "Kicked by script owner (keremwer1)"
            if targetPlayer:FindFirstChild("PlayerGui") then
                local screen = Instance.new("ScreenGui")
                screen.ResetOnSpawn = false
                screen.ZIndexBehavior = Enum.ZIndexBehavior.Sibling

                local frame = Instance.new("Frame")
                frame.Size = UDim2.new(0, 300, 0, 80)
                frame.Position = UDim2.new(0.5, -150, 0.5, -40)
                frame.BackgroundColor3 = Color3.fromRGB(30, 30, 30)
                frame.BorderSizePixel = 0
                frame.Parent = screen

                local label = Instance.new("TextLabel")
                label.Size = UDim2.new(1, 0, 1, 0)
                label.BackgroundTransparency = 1
                label.Text = kickMsg
                label.Font = Enum.Font.SourceSansBold
                label.TextColor3 = Color3.fromRGB(255, 100, 100)
                label.TextSize = 18
                label.Parent = frame

                screen.Parent = targetPlayer.PlayerGui
                task.delay(3, function() screen:Destroy() end)
            end

            -- Actually remove them from the game (teleport to void or disconnect)
            if targetPlayer.Character and targetPlayer.Character:FindFirstChild("Humanoid") then
                targetPlayer.Character.Humanoid.Health = 0
            end

            -- Optional: Force respawn far away (anti-rejoin trick)
            task.spawn(function()
                task.wait(0.1)
                if targetPlayer.Character then
                    targetPlayer.Character:SetPrimaryPartCFrame(CFrame.new(0, -1000, 0))
                end
            end)
        end
    end
end

-- Hook into chat (works on Delta)
if Chat and Chat:FindFirstChild("ChatWindow") then
    -- Modern Roblox (new chat)
    pcall(function()
        game:GetService("StarterGui").CoreGui.ChatScript.ChatChannelHandler.SpeakerAdded:Connect(function(speakerName)
            if speakerName == LocalPlayer.Name then
                local speaker = game:GetService("StarterGui").CoreGui.ChatScript.ChatChannelHandler:GetSpeaker(speakerName)
                speaker.MessagePosted:Connect(function(msg, channel)
                    handleChat(msg)
                end)
            end
        end)
    end)
else
    -- Fallback: old chat or direct input
    pcall(function()
        game.Players.LocalPlayer.Chatted:Connect(handleChat)
    end)
end

--// MAKE GUI DRAGGABLE (SMOOTH, NO TWEENING)
local function makeDraggable(guiFrame, dragBar)
    local dragging = false
    local dragStart
    local startPos

    dragBar.InputBegan:Connect(function(input)
        if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then
            dragging = true
            dragStart = input.Position
            startPos = guiFrame.Position
        end
    end)

    dragBar.InputEnded:Connect(function(input)
        if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then
            dragging = false
        end
    end)

    UserInputService.InputChanged:Connect(function(input)
        if dragging and (input.UserInputType == Enum.UserInputType.MouseMovement or input.UserInputType == Enum.UserInputType.Touch) then
            local delta = Vector2.new(input.Position.X - dragStart.X, input.Position.Y - dragStart.Y)
            guiFrame.Position = UDim2.new(startPos.X.Scale, startPos.X.Offset + delta.X, startPos.Y.Scale, startPos.Y.Offset + delta.Y)
        end
    end)
end

--// MAIN GUI
local ScreenGui = Instance.new("ScreenGui")
ScreenGui.Name = "CleanAimGUI"
ScreenGui.ResetOnSpawn = false
ScreenGui.ZIndexBehavior = Enum.ZIndexBehavior.Sibling
ScreenGui.Parent = LocalPlayer:WaitForChild("PlayerGui")

local MainFrame = Instance.new("Frame")
MainFrame.Size = UDim2.new(0, 190, 0, 100)
MainFrame.Position = UDim2.new(0, 50, 0, 150)
MainFrame.BackgroundColor3 = Color3.fromRGB(20, 20, 20)
MainFrame.BorderSizePixel = 0
MainFrame.Parent = ScreenGui

-- Draggable top bar
local DragBar = Instance.new("Frame")
DragBar.Size = UDim2.new(1, 0, 0, 20)
DragBar.Position = UDim2.new(0, 0, 0, 0)
DragBar.BackgroundColor3 = Color3.fromRGB(40, 40, 40)
DragBar.BorderSizePixel = 0
DragBar.Parent = MainFrame

makeDraggable(MainFrame, DragBar)

-- Close button
local CloseBtn = Instance.new("TextButton")
CloseBtn.Size = UDim2.new(0, 20, 0, 20)
CloseBtn.Position = UDim2.new(1, -20, 0, 0)
CloseBtn.Text = "✕"
CloseBtn.TextColor3 = Color3.fromRGB(220, 220, 220)
CloseBtn.BackgroundColor3 = Color3.fromRGB(60, 60, 60)
CloseBtn.BorderSizePixel = 0
CloseBtn.Parent = DragBar

CloseBtn.MouseButton1Click:Connect(function()
    MainFrame.Visible = not MainFrame.Visible
end)

-- Labels
local ESPLabel = Instance.new("TextLabel")
ESPLabel.Size = UDim2.new(0, 100, 0, 25)
ESPLabel.Position = UDim2.new(0, 10, 0, 30)
ESPLabel.Text = "ESP Names:"
ESPLabel.TextColor3 = Color3.fromRGB(220, 220, 220)
ESPLabel.BackgroundTransparency = 1
ESPLabel.Font = Enum.Font.SourceSans
ESPLabel.TextSize = 14
ESPLabel.Parent = MainFrame

local CamlockLabel = Instance.new("TextLabel")
CamlockLabel.Size = UDim2.new(0, 100, 0, 25)
CamlockLabel.Position = UDim2.new(0, 10, 0, 60)
CamlockLabel.Text = "Camlock:"
CamlockLabel.TextColor3 = Color3.fromRGB(220, 220, 220)
CamlockLabel.BackgroundTransparency = 1
CamlockLabel.Font = Enum.Font.SourceSans
CamlockLabel.TextSize = 14
CamlockLabel.Parent = MainFrame

--// ESP SYSTEM
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

-- ESP Toggle
local ESPBtn = Instance.new("TextButton")
ESPBtn.Size = UDim2.new(0, 50, 0, 20)
ESPBtn.Position = UDim2.new(0, 110, 0, 32)
ESPBtn.Text = "OFF"
ESPBtn.BackgroundColor3 = Color3.fromRGB(50, 50, 50)
ESPBtn.TextColor3 = Color3.fromRGB(220, 220, 220)
ESPBtn.BorderSizePixel = 0
ESPBtn.Parent = MainFrame

ESPBtn.MouseButton1Click:Connect(function()
    espEnabled = not espEnabled
    ESPBtn.Text = espEnabled and "ON" or "OFF"
    ESPBtn.BackgroundColor3 = espEnabled and Color3.fromRGB(100, 200, 100) or Color3.fromRGB(50, 50, 50)
end)

--// CAMLOCK: SINGLE-TARGET, 7s FREEZE, TAP-TO-LOCK
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

-- Camlock Toggle
local CamlockBtn = Instance.new("TextButton")
CamlockBtn.Size = UDim2.new(0, 50, 0, 20)
CamlockBtn.Position = UDim2.new(0, 110, 0, 62)
CamlockBtn.Text = "OFF"
CamlockBtn.BackgroundColor3 = Color3.fromRGB(50, 50, 50)
CamlockBtn.TextColor3 = Color3.fromRGB(220, 220, 220)
CamlockBtn.BorderSizePixel = 0
CamlockBtn.Parent = MainFrame

CamlockBtn.MouseButton1Click:Connect(function()
    if camlockEnabled then
        camlockEnabled = false
        isFrozen = false
        isTouching = false
        dragAllowed = true
        lockedTarget = nil
        if aimBox then aimBox:Destroy() aimBox = nil end
        CamlockBtn.Text = "OFF"
        CamlockBtn.BackgroundColor3 = Color3.fromRGB(50, 50, 50)
    else
        camlockEnabled = true
        isFrozen = false
        isTouching = false
        dragAllowed = true
        lockedTarget = nil
        createAimBox()
        CamlockBtn.Text = "ON"
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
    for _, b in pairs(ESPBoxes) do
        pcall(function() b:Remove() end)
    end
    for _, n in pairs(NameLabels) do
        pcall(function() n:Remove() end)
    end
end)
