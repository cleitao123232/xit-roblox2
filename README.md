local config = {
    teamCheck = true,
    fov = 35,
    smoothing = 1,
    highlightEnabled = true,
    lockPart = "Head",
    ToggleKey = Enum.KeyCode.RightAlt,
    espColor = Color3.fromRGB(255, 0, 0)
}

local whitelist = {
    "Itachi_cnp7",
    "Player2",
    "Player3"
}

local services = {
    RunService = game:GetService("RunService"),
    UserInputService = game:GetService("UserInputService"),
    StarterGui = game:GetService("StarterGui"),
    Players = game:GetService("Players"),
    Camera = workspace.CurrentCamera
}

local aimbotEnabled = false
local espEnabled = false
local ScreenGui
local currentTargetHighlight
local FOVring

local function isWhitelisted(player)
    for _, username in ipairs(whitelist) do
        if player.Name == username then
            return true
        end
    end
    return false
end

local function createGUI()
    if ScreenGui then
        ScreenGui:Destroy()
    end

    ScreenGui = Instance.new("ScreenGui", game.Players.LocalPlayer:WaitForChild("PlayerGui"))
    ScreenGui.Name = "AimbotESP_GUI"

    local Frame = Instance.new("Frame", ScreenGui)
    Frame.Size = UDim2.new(0, 600, 0, 500)
    Frame.Position = UDim2.new(0.5, -300, 0.5, -250)
    Frame.Visible = true
    Frame.BackgroundColor3 = Color3.fromRGB(50, 50, 50)

    local titleLabel = Instance.new("TextLabel", Frame)
    titleLabel.Size = UDim2.new(0, 580, 0, 50)
    titleLabel.Position = UDim2.new(0, 10, 0, 10)
    titleLabel.Text = "LT7 SYSTEM"
    titleLabel.TextScaled = true
    titleLabel.Font = Enum.Font.SourceSansBold
    titleLabel.TextColor3 = Color3.fromRGB(255, 255, 255)
    titleLabel.BackgroundTransparency = 1

    local aimbotButton = Instance.new("TextButton", Frame)
    aimbotButton.Size = UDim2.new(0, 580, 0, 30)
    aimbotButton.Position = UDim2.new(0, 10, 0, 70)
    aimbotButton.Text = "Toggle Aimbot"
    aimbotButton.BackgroundColor3 = Color3.fromRGB(255, 0, 0)

    local espButton = Instance.new("TextButton", Frame)
    espButton.Size = UDim2.new(0, 580, 0, 30)
    espButton.Position = UDim2.new(0, 10, 0, 110)
    espButton.Text = "Toggle ESP"
    espButton.BackgroundColor3 = Color3.fromRGB(255, 0, 0)

    local teamCheckButton = Instance.new("TextButton", Frame)
    teamCheckButton.Size = UDim2.new(0, 580, 0, 30)
    teamCheckButton.Position = UDim2.new(0, 10, 0, 150)
    teamCheckButton.Text = "Toggle Team Check"
    teamCheckButton.BackgroundColor3 = Color3.fromRGB(255, 0, 0)

    local espColorButton = Instance.new("TextButton", Frame)
    espColorButton.Size = UDim2.new(0, 580, 0, 30)
    espColorButton.Position = UDim2.new(0, 10, 0, 190)
    espColorButton.Text = "Change ESP Color"

    local fovSliderLabel = Instance.new("TextLabel", Frame)
    fovSliderLabel.Size = UDim2.new(0, 580, 0, 30)
    fovSliderLabel.Position = UDim2.new(0, 10, 0, 230)
    fovSliderLabel.Text = "FOV: " .. config.fov
    fovSliderLabel.BackgroundColor3 = Color3.fromRGB(200, 200, 200)

    local fovSlider = Instance.new("TextButton", Frame)
    fovSlider.Size = UDim2.new(0, 580, 0, 30)
    fovSlider.Position = UDim2.new(0, 10, 0, 270)
    fovSlider.Text = "Adjust FOV"
    fovSlider.BackgroundColor3 = Color3.fromRGB(255, 255, 255)

    local smoothingLabel = Instance.new("TextLabel", Frame)
    smoothingLabel.Size = UDim2.new(0, 580, 0, 30)
    smoothingLabel.Position = UDim2.new(0, 10, 0, 310)
    smoothingLabel.Text = "Smoothing: " .. config.smoothing
    smoothingLabel.BackgroundColor3 = Color3.fromRGB(200, 200, 200)

    local smoothingSlider = Instance.new("TextButton", Frame)
    smoothingSlider.Size = UDim2.new(0, 580, 0, 30)
    smoothingSlider.Position = UDim2.new(0, 10, 0, 350)
    smoothingSlider.Text = "Adjust Smoothing"
    smoothingSlider.BackgroundColor3 = Color3.fromRGB(255, 255, 255)

    services.UserInputService.InputBegan:Connect(function(input, gameProcessed)
        if not gameProcessed and input.KeyCode == config.ToggleKey then
            Frame.Visible = not Frame.Visible
        end
    end)

    aimbotButton.MouseButton1Click:Connect(function()
        aimbotEnabled = not aimbotEnabled
        if FOVring then
            FOVring.Visible = aimbotEnabled
        end
        if aimbotEnabled then
            aimbotButton.BackgroundColor3 = Color3.fromRGB(0, 255, 0)
        else
            aimbotButton.BackgroundColor3 = Color3.fromRGB(255, 0, 0)
        end
    end)

    espButton.MouseButton1Click:Connect(function()
        espEnabled = not espEnabled
        if espEnabled then
            espButton.BackgroundColor3 = Color3.fromRGB(0, 255, 0)
            for _, player in pairs(services.Players:GetPlayers()) do
                if player.Character then
                    applyESP(player)
                end
            end
        else
            espButton.BackgroundColor3 = Color3.fromRGB(255, 0, 0)
            for _, player in pairs(services.Players:GetPlayers()) do
                if player.Character then
                    for _, highlight in pairs(player.Character:GetChildren()) do
                        if highlight:IsA("Highlight") then
                            highlight:Destroy()
                        end
                    end
                end
            end
        end
    end)

    teamCheckButton.MouseButton1Click:Connect(function()
        config.teamCheck = not config.teamCheck
        if config.teamCheck then
            teamCheckButton.BackgroundColor3 = Color3.fromRGB(0, 255, 0)
        else
            teamCheckButton.BackgroundColor3 = Color3.fromRGB(255, 0, 0)
        end
    end)

    espColorButton.MouseButton1Click:Connect(function()
        config.espColor = Color3.fromRGB(math.random(0, 255), math.random(0, 255), math.random(0, 255))
        if espEnabled then
            for _, player in pairs(services.Players:GetPlayers()) do
                if player.Character then
                    applyESP(player)
                end
            end
        end
    end)

    fovSlider.MouseButton1Click:Connect(function()
        config.fov = math.random(10, 100)
        fovSliderLabel.Text = "FOV: " .. config.fov
        if FOVring then
            FOVring.Radius = config.fov
        end
    end)

    smoothingSlider.MouseButton1Click:Connect(function()
        config.smoothing = math.random(1, 10)
        smoothingLabel.Text = "Smoothing: " .. config.smoothing
    end)
end

if isWhitelisted(services.Players.LocalPlayer) then
    createGUI()

    local function onCharacterAdded(character)
        createGUI()
        createFOVRing()
    end

    services.Players.LocalPlayer.CharacterAdded:Connect(onCharacterAdded)

    services.StarterGui:SetCore("SendNotification", {
        Title = "Vip aimbot",
        Text = "Aimbot e ESP!",
        Duration = 5,
    })

    local function createFOVRing()
        if FOVring then
            FOVring:Remove()
        end
        FOVring = Drawing.new("Circle")
        FOVring.Visible = aimbotEnabled
        FOVring.Thickness = 1
        FOVring.Radius = config.fov
        FOVring.Transparency = 0.8
        FOVring.Color = Color3.fromRGB(255, 0, 0)
        FOVring.Position = services.Camera.ViewportSize / 2
    end

    createFOVRing()

    local function applyESP(player)
        if player.Character and (not config.teamCheck or player.Team ~= services.Players.LocalPlayer.Team) then
            local highlight = Instance.new("Highlight")
            highlight.Adornee = player.Character
            highlight.FillColor = config.espColor
            highlight.OutlineColor = Color3.fromRGB(255, 255, 255)
            highlight.Parent = player.Character
        end
    end

    local lastESPUpdate = tick()
    local function refreshESP()
        if tick() - lastESPUpdate >= 0.5 then
            if espEnabled then
                for _, player in ipairs(services.Players:GetPlayers()) do
                    if player ~= services.Players.LocalPlayer then
                        applyESP(player)
                    end
                end
            end
            lastESPUpdate = tick()
        end
    end

    services.Players.PlayerAdded:Connect(function(player)
        player.CharacterAdded:Connect(function()
            if espEnabled then
                applyESP(player)
            end
        end)
    end)

    services.RunService.Heartbeat:Connect(refreshESP)

    local currentTarget = nil

    local function getClosest(cframe, maxDistance)
        local target = nil
        local mag = math.huge
        local screenCenter = services.Camera.ViewportSize / 2

        for _, v in pairs(services.Players:GetPlayers()) do
            if v.Character and v.Character:FindFirstChild(config.lockPart) and v.Character:FindFirstChild("Humanoid") and v ~= services.Players.LocalPlayer then
                if not config.teamCheck or v.Team ~= services.Players.LocalPlayer.Team then
                    local screenPoint, onScreen = services.Camera:WorldToViewportPoint(v.Character[config.lockPart].Position)
                    local distanceFromCenter = (Vector2.new(screenPoint.X, screenPoint.Y) - screenCenter).Magnitude

                    if onScreen and distanceFromCenter <= config.fov then
                        local magBuf = (v.Character[config.lockPart].Position - cframe.Position).Magnitude
                        if magBuf < mag and magBuf <= maxDistance then
                            mag = magBuf
                            target = v
                        end
                    end
                end
            end
        end
        return target
    end

    services.RunService.RenderStepped:Connect(function()
        if aimbotEnabled then
            local cam = services.Camera
            if services.UserInputService:IsMouseButtonPressed(Enum.UserInputType.MouseButton2) then
                if not currentTarget then
                    currentTarget = getClosest(cam.CFrame, 500)
                    if currentTarget and currentTarget.Character then
                        if not currentTargetHighlight then
                            currentTargetHighlight = Instance.new("Highlight")
                            currentTargetHighlight.OutlineColor = Color3.fromRGB(0, 255, 0)
                            currentTargetHighlight.FillTransparency = 1
                            currentTargetHighlight.Parent = currentTarget.Character
                        end
                        currentTargetHighlight.Adornee = currentTarget.Character
                    end
                end
                if currentTarget and currentTarget.Character and currentTarget.Character:FindFirstChild(config.lockPart) then
                    services.Camera.CFrame = CFrame.new(cam.CFrame.Position, currentTarget.Character[config.lockPart].Position)
                end
            else
                currentTarget = nil
                if currentTargetHighlight then
                    currentTargetHighlight:Destroy()
                    currentTargetHighlight = nil
                end
            end
        end
    end)
else
    services.StarterGui:SetCore("SendNotification", {
        Title = "Access Denied",
        Text = "You are not whitelisted to use this script.",
        Duration = 5,
    })
end
