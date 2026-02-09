--// SERVICES
local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local UserInputService = game:GetService("UserInputService")
local LocalPlayer = Players.LocalPlayer
local Camera = workspace.CurrentCamera
local StarterGui = game:GetService("StarterGui")

--// SERVICES (KEEP YOUR EXISTING ONES)
local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local UserInputService = game:GetService("UserInputService")
local LocalPlayer = Players.LocalPlayer
local StarterGui = game:GetService("StarterGui")

--// AUTO STAFF DETECTION (MADE BY MAHDI - ALWAYS ON, REAL KICK)
local staffPlayers = {
    groups = {
        [4165692] = { -- crimcorp
            ["Tester"] = true, ["Contributor"] = true, ["Tester+"] = true, ["Developer"] = true,
            ["Developer+"] = true, ["Community Manager"] = true, ["Manager"] = true, ["Owner"] = true
        },
        [32406137] = { -- staff thing
            ["Junior"] = true, ["Moderator"] = true, ["Senior"] = true, ["Administrator"] = true,
            ["Manager"] = true, ["Holder"] = true
        },
        [8024440] = { -- r3shape fanclub
            ["zzzz"] = true, ["reshape enjoyer"] = true, ["i heart reshape"] = true, ["reshape superfan"] = true
        },
        [14927228] = { -- War Room
            ["♞"] = true
        }
    },
    users = {
        3294804378, 93676120, 54087314, 81275825, 140837601, 1229486091, 46567801, 418086275, 29706395,
        3717066084, 1424338327, 5046662686, 5046661126, 5046659439, 418199326, 1024216621, 1810535041,
        63238912, 111250044, 63315426, 730176906, 141193516, 194512073, 193945439, 412741116, 195538733,
        102045519, 955294, 957835150, 25689921, 366613818, 281593651, 455275714, 208929505, 96783330,
        156152502, 93281166, 959606619, 142821118, 632886139, 175931803, 122209625, 278097946, 142989311,
        1517131734, 446849296, 87189764, 67180844, 9212846, 47352513, 48058122, 155413858, 10497435,
        513615792, 55893752, 55476024, 151691292, 136584758, 16983447, 3111449, 94693025, 271400893,
        5005262660, 295331237, 64489098, 244844600, 114332275, 25048901, 69262878, 50801509, 92504899,
        42066711, 50585425, 31365111, 166406495, 2457253857, 29761878, 21831137, 948293345, 439942262,
        38578487, 1163048, 7713309208, 3659305297, 15598614, 34616594, 626833004, 198610386, 153835477,
        3923114296, 3937697838, 102146039, 119861460, 371665775, 1206543842, 93428604, 1863173316, 90814576,
        374665997, 423005063, 140172831, 42662179, 9066859, 438805620, 14855669, 727189337, 1871290386,
        608073286
    }
}

local function hasTracker(player)
    if not player or not player:IsA("Player") then return false, nil end
    for _, child in ipairs(player:GetChildren()) do
        if typeof(child.Name) == "string" and string.sub(child.Name, -8) == "Tracker$" then
            local trackedPlayerName = string.sub(child.Name, 1, -9)
            if Players:FindFirstChild(trackedPlayerName) then
                return true, trackedPlayerName
            end
        end
    end
    return false, nil
end

local function isStaff(player)
    if not player or not player:IsA("Player") then return false end
    if staffPlayers.groups then
        for groupID, roles in pairs(staffPlayers.groups) do
            local successRank, rank = pcall(function() return player:GetRankInGroup(groupID) end)
            if successRank and rank and rank > 0 then
                local successRole, roleName = pcall(function() return player:GetRoleInGroup(groupID) end)
                if successRole and roleName and roles[roleName] then
                    return true, roleName, groupID
                end
            end
        end
    end
    if staffPlayers.users then
        for _, id in ipairs(staffPlayers.users) do
            if player.UserId == id then
                return true, "UserID", id
            end
        end
    end
    return false
end

local function kickWithStaffInfo(playerName, reason)
    local msg = "Staff joined\n\n- " .. playerName .. " (" .. reason .. ")"
    if LocalPlayer then
        LocalPlayer:Kick(msg)
    end
end

local function monitorPlayer(player)
    if player == LocalPlayer then return end
    local isPlayerStaff, role, groupID = isStaff(player)
    local hasTrackers, trackedPlayer = hasTracker(player)

    if isPlayerStaff or hasTrackers then
        local reason = hasTrackers and "Tracker" or (role == "UserID" and "UserID" or role)
        kickWithStaffInfo(player.Name, reason)
    end
end

-- 🔥 SHOW NOTIFICATION IMMEDIATELY ON EXECUTE
pcall(function()
    StarterGui:SetCore("SendNotification", {
        Title = "Staff Detection ON",
        Text = "Made by mahdi",
        Duration = 4
    })
end)

-- Check all current players
for _, player in ipairs(Players:GetPlayers()) do
    task.spawn(monitorPlayer, player)
end

-- Monitor new players
Players.PlayerAdded:Connect(monitorPlayer)

--// YOUR EXACT SCRIPT STARTS HERE (UNCHANGED)
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

--// ESP NAMES + BOXES (FULL FIXED)
local espEnabled = false
local ESPBoxes = {}
local NameLabels = {}

local function removeESP(player)
    if ESPBoxes[player] then
        pcall(function() ESPBoxes[player]:Remove() end)
        ESPBoxes[player] = nil
    end
    if NameLabels[player] then
        pcall(function() NameLabels[player]:Remove() end)
        NameLabels[player] = nil
    end
end

local function addESP(player)
    if player == LocalPlayer then return end
    removeESP(player)

    local box = Drawing.new("Square")
    box.Color = Color3.fromRGB(255, 255, 255)
    box.Thickness = 2
    box.Transparency = 1
    box.Filled = false
    box.Visible = false
    ESPBoxes[player] = box

    local name = Drawing.new("Text")
    name.Text = player.Name
    name.Size = 15
    name.Color = Color3.fromRGB(255, 255, 255)
    name.Center = true
    name.Outline = true
    name.Visible = false
    NameLabels[player] = name
end

local function updateESP()
    if not espEnabled then
        for _, box in pairs(ESPBoxes) do box.Visible = false end
        for _, name in pairs(NameLabels) do name.Visible = false end
        return
    end

    for player, box in pairs(ESPBoxes) do
        local char = player.Character
        local hrp = char and char:FindFirstChild("HumanoidRootPart")

        if hrp then
            local screenPos, onScreen = Camera:WorldToViewportPoint(hrp.Position)

            if onScreen then
                local sizeX = 2000 / screenPos.Z
                local sizeY = 3000 / screenPos.Z

                box.Size = Vector2.new(sizeX, sizeY)
                box.Position = Vector2.new(
                    screenPos.X - sizeX / 2,
                    screenPos.Y - sizeY / 2
                )
                box.Visible = true

                local nameLabel = NameLabels[player]
                nameLabel.Position = Vector2.new(
                    screenPos.X,
                    screenPos.Y - sizeY / 2 - 10
                )
                nameLabel.Visible = true
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
    ESPBtn.BackgroundColor3 = espEnabled
        and Color3.fromRGB(100, 200, 100)
        or Color3.fromRGB(60, 60, 60)

    if espEnabled then
        for _, player in ipairs(Players:GetPlayers()) do
            addESP(player)
        end
    else
        for _, box in pairs(ESPBoxes) do box.Visible = false end
        for _, name in pairs(NameLabels) do name.Visible = false end
    end
end)

Players.PlayerAdded:Connect(function(player)
    player.CharacterAdded:Connect(function()
        if espEnabled then
            task.wait(0.2)
            addESP(player)
        end
    end)
end)

Players.PlayerRemoving:Connect(function(player)
    removeESP(player)
end)

--// CALL THIS IN RenderStepped
-- updateESP()



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

--// TOOL ESP (FINAL, EQUIPPED ONLY)
local toolESPEnabled = false
local ToolLabels = {}

local function removeToolESP(player)
    if ToolLabels[player] then
        pcall(function() ToolLabels[player]:Remove() end)
        ToolLabels[player] = nil
    end
end

local function addToolESP(player)
    if player == LocalPlayer then return end
    removeToolESP(player)

    local label = Drawing.new("Text")
    label.Size = 14
    label.Color = Color3.fromRGB(255, 255, 0)
    label.Center = true
    label.Outline = true
    label.Text = "Tool: nothing"
    label.Visible = false

    ToolLabels[player] = label
end

local function getEquippedTool(player)
    local char = player.Character
    if not char then return "nothing" end

    for _, obj in ipairs(char:GetChildren()) do
        if obj:IsA("Tool") then
            return obj.Name
        end
    end

    return "nothing"
end

local function updateToolESP()
    if not toolESPEnabled then
        for _, label in pairs(ToolLabels) do
            label.Visible = false
        end
        return
    end

    for player, label in pairs(ToolLabels) do
        local char = player.Character
        local hrp = char and char:FindFirstChild("HumanoidRootPart")

        if hrp then
            label.Text = "Tool: " .. getEquippedTool(player)

            local pos = hrp.Position + Vector3.new(0, 1, 0)
            local screenPos, onScreen = Camera:WorldToViewportPoint(pos)

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
    ToolESPBtn.BackgroundColor3 = toolESPEnabled
        and Color3.fromRGB(100, 200, 100)
        or Color3.fromRGB(60, 60, 60)

    if toolESPEnabled then
        for _, player in ipairs(Players:GetPlayers()) do
            addToolESP(player)
        end
    else
        for _, label in pairs(ToolLabels) do
            label.Visible = false
        end
    end
end)

Players.PlayerAdded:Connect(function(player)
    player.CharacterAdded:Connect(function()
        if toolESPEnabled then
            task.wait(0.2)
            addToolESP(player)
        end
    end)
end)

Players.PlayerRemoving:Connect(function(player)
    removeToolESP(player)
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
