-- Savannah Life script
local UserInputService = game:GetService("UserInputService")
local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local TweenService = game:GetService("TweenService")
local CoreGui = game:GetService("CoreGui")

-- Variables
local player = Players.LocalPlayer
local camera = workspace.CurrentCamera
local isFlying = false
local speedEnabled = false
local infiniteStaminaEnabled = false
local infiniteOxygenEnabled = false
local noFallDamageEnabled = false
local antiAfkEnabled = false
local espEnabled = false
local aimbotEnabled = false
local killAuraEnabled = false
local wallPenetrationEnabled = false -- Wall penetration toggle
local aimbotTarget = nil
local aimbotSmoothness = 0.5 -- Lower = faster aim
local killAuraRange = 50 -- Increased range to 50 studs
local lastKillAuraAttack = 0
local killAuraCooldown = 0.1 -- Faster attacks
local maxKillAuraTargets = 5 -- Maximum number of simultaneous targets
local damageAmount = 25
local wallPenetrationRange = 2000 -- Increased maximum distance for wall penetration shots

-- Savannah Life Anti-Grab (Quick-Time Event) variables
local antiGrabEnabled = false
local antiGrabConnection = nil

-- Rivals specific variables
local rivalsSection = false -- Toggle for Rivals section
local autoFreezeEnabled = false -- Auto freeze nearby enemies
local instaKillEnabled = false -- Instantly kill enemies
local rapidFireEnabled = false -- Rapid fire for any weapon
local unlimitedSatchelEnabled = false -- Unlimited satchel charges
local autoMoveEnabled = false -- Auto movement techniques
local freezeRayRange = 30 -- Range for auto freeze
local lastFreezeAttack = 0
local freezeCooldown = 0.5 -- Cooldown for auto freeze
local lastRapidFire = 0
local rapidFireCooldown = 0.05 -- Very fast firing rate

-- Additional Rivals cheat variables
local preTimerMoveEnabled = false -- Move before round timer starts
local backTeleportEnabled = false -- Teleport behind enemies
local allHeadshotsEnabled = false -- Force all shots to be headshots
local rivalAimbotEnabled = false -- Rivals-specific aimbot
local lastBackTeleport = 0 -- Cooldown tracking for back teleports
local teleportCooldown = 0.5 -- Time between teleports
local stickToTargetEnabled = false -- Stay attached to target after teleport
local currentStickTarget = nil -- Current player being stuck to
local headShotMultiplier = 5 -- Damage multiplier for headshots

-- Imported script cheat variables
local rivalESPEnabled = false -- Enemies highlighted through walls
local autoFarmEnabled = false -- Auto farm kills
local infiniteJumpEnabled = false -- Infinite jumping
local bHopEnabled = false -- Bunny hopping
local noClipEnabled = false -- Walk through walls
local cFrameWalkEnabled = false -- Speed walk
local cFrameWalkSpeed = 0.2 -- Speed multiplier
local inBhopJump = false -- Track bunny hop state

-- Auto Hit Variables for Savannah Life
local autoHitEnabled = false
local autoHitRange = 50
local maxAutoHitTargets = 10
local lastAutoHit = 0
local autoHitCooldown = 0.05  -- Reduced cooldown for faster hits
local hitboxExpansion = 2
local teleportOffset = CFrame.new(0, 0, 2.5)  -- Slightly increased distance for smoother teleports
local returnDelay = 0.02  -- Faster return to original position

-- Notification System
local NotificationSystem = {}
NotificationSystem.Notifications = {}

function NotificationSystem.new(title, text, duration)
    local notification = {}
    notification.Done = false
    
    -- Create GUI
    local gui = Instance.new("Frame")
    gui.Size = UDim2.new(0, 250, 0, 80)
    gui.Position = UDim2.new(1, -260, 1, -90)
    gui.BackgroundColor3 = Color3.fromRGB(30, 30, 30)
    gui.BorderSizePixel = 0
    gui.Parent = ScreenGui
    
    -- Add corner radius
    local corner = Instance.new("UICorner")
    corner.CornerRadius = UDim.new(0, 8)
    corner.Parent = gui
    
    -- Add gradient
    local gradient = Instance.new("UIGradient")
    gradient.Color = ColorSequence.new({
        ColorSequenceKeypoint.new(0, Color3.fromRGB(40, 40, 40)),
        ColorSequenceKeypoint.new(1, Color3.fromRGB(30, 30, 30))
    })
    gradient.Parent = gui
    
    -- Title
    local titleLabel = Instance.new("TextLabel")
    titleLabel.Size = UDim2.new(1, -20, 0, 25)
    titleLabel.Position = UDim2.new(0, 10, 0, 5)
    titleLabel.BackgroundTransparency = 1
    titleLabel.Text = title
    titleLabel.TextColor3 = Color3.fromRGB(255, 255, 255)
    titleLabel.TextSize = 16
    titleLabel.Font = Enum.Font.GothamBold
    titleLabel.TextXAlignment = Enum.TextXAlignment.Left
    titleLabel.Parent = gui
    
    -- Text
    local textLabel = Instance.new("TextLabel")
    textLabel.Size = UDim2.new(1, -20, 0, 40)
    textLabel.Position = UDim2.new(0, 10, 0, 30)
    textLabel.BackgroundTransparency = 1
    textLabel.Text = text
    textLabel.TextColor3 = Color3.fromRGB(200, 200, 200)
    textLabel.TextSize = 14
    textLabel.Font = Enum.Font.Gotham
    textLabel.TextXAlignment = Enum.TextXAlignment.Left
    textLabel.TextWrapped = true
    textLabel.Parent = gui
    
    -- Animate in
    gui.Position = UDim2.new(1, 0, 1, -90)
    TweenService:Create(gui, TweenInfo.new(0.5, Enum.EasingStyle.Quart), {
        Position = UDim2.new(1, -260, 1, -90)
    }):Play()
    
    -- Remove after duration
    spawn(function()
        wait(duration)
        TweenService:Create(gui, TweenInfo.new(0.5), {
            Position = UDim2.new(1, 0, 1, -90)
        }):Play()
        wait(0.5)
        gui:Destroy()
    end)
end

-- Create GUI
local ScreenGui = Instance.new("ScreenGui")
ScreenGui.Name = tostring(math.random(1000000, 9999999))
ScreenGui.ResetOnSpawn = false
ScreenGui.ZIndexBehavior = Enum.ZIndexBehavior.Sibling

pcall(function()
    if CoreGui:FindFirstChild("CheatHub") then
        CoreGui.CheatHub:Destroy()
    end
end)

-- Try to parent to CoreGui first, fallback to PlayerGui
local success = pcall(function()
    ScreenGui.Parent = CoreGui
end)

if not success then
    ScreenGui.Parent = player:WaitForChild("PlayerGui")
end

-- Main Frame with Scrolling
local MainFrame = Instance.new("Frame")
MainFrame.Name = "MainFrame"
MainFrame.Size = UDim2.new(0, 300, 0, 400)
MainFrame.Position = UDim2.new(0.5, -150, 0.5, -200)
MainFrame.BackgroundColor3 = Color3.fromRGB(20, 20, 20)
MainFrame.BorderSizePixel = 0
MainFrame.Active = true
MainFrame.Draggable = true
MainFrame.Parent = ScreenGui

-- Add shadow
local Shadow = Instance.new("ImageLabel")
Shadow.Name = "Shadow"
Shadow.AnchorPoint = Vector2.new(0.5, 0.5)
Shadow.BackgroundTransparency = 1
Shadow.Position = UDim2.new(0.5, 0, 0.5, 0)
Shadow.Size = UDim2.new(1, 47, 1, 47)
Shadow.Image = "rbxassetid://6015897843"
Shadow.ImageColor3 = Color3.new(0, 0, 0)
Shadow.ImageTransparency = 0.5
Shadow.Parent = MainFrame

-- Create ScrollingFrame
local ContentFrame = Instance.new("ScrollingFrame")
ContentFrame.Name = "ContentFrame"
ContentFrame.Size = UDim2.new(1, -10, 1, -40) -- Adjusted for padding and title bar
ContentFrame.Position = UDim2.new(0, 5, 0, 35)
ContentFrame.BackgroundColor3 = Color3.fromRGB(20, 20, 20)
ContentFrame.BorderSizePixel = 0
ContentFrame.ScrollBarThickness = 5
ContentFrame.ScrollBarImageColor3 = Color3.fromRGB(50, 50, 50)
ContentFrame.ScrollBarImageTransparency = 0.5
ContentFrame.Parent = MainFrame

-- Add UIListLayout for proper spacing
local Layout = Instance.new("UIListLayout")
Layout.Parent = ContentFrame
Layout.Padding = UDim.new(0, 5)
Layout.SortOrder = Enum.SortOrder.LayoutOrder

-- Title bar
local TitleBar = Instance.new("Frame")
TitleBar.Name = "TitleBar"
TitleBar.Size = UDim2.new(1, 0, 0, 30)
TitleBar.BackgroundColor3 = Color3.fromRGB(30, 30, 30)
TitleBar.BorderSizePixel = 0
TitleBar.Parent = MainFrame

-- Add shadow
local Shadow = Instance.new("ImageLabel")
Shadow.Name = "Shadow"
Shadow.AnchorPoint = Vector2.new(0.5, 0.5)
Shadow.BackgroundTransparency = 1
Shadow.Position = UDim2.new(0.5, 0, 0.5, 0)
Shadow.Size = UDim2.new(1, 47, 1, 47)
Shadow.Image = "rbxassetid://6015897843"
Shadow.ImageColor3 = Color3.new(0, 0, 0)
Shadow.ImageTransparency = 0.5
Shadow.Parent = MainFrame

-- Title bar
local TitleBar = Instance.new("Frame")
TitleBar.Name = "TitleBar"
TitleBar.Size = UDim2.new(1, 0, 0, 30)
TitleBar.BackgroundColor3 = Color3.fromRGB(30, 30, 30)
TitleBar.BorderSizePixel = 0
TitleBar.Parent = MainFrame

-- Title
local Title = Instance.new("TextLabel")
Title.Name = "Title"
Title.Size = UDim2.new(0.7, 0, 1, 0)
Title.Position = UDim2.new(0, 10, 0, 0)
Title.BackgroundTransparency = 1
Title.Text = "Enhanced Cheat Hub"
Title.TextColor3 = Color3.fromRGB(255, 255, 255)
Title.TextSize = 18
Title.Font = Enum.Font.GothamBold
Title.TextXAlignment = Enum.TextXAlignment.Left
Title.Parent = TitleBar

-- Speed Control Frame (below title bar)
local SpeedControlFrame = Instance.new("Frame")
SpeedControlFrame.Name = "SpeedControlFrame"
SpeedControlFrame.Size = UDim2.new(1, -20, 0, 35)
SpeedControlFrame.Position = UDim2.new(0, 10, 0, 40) -- Just below title bar
SpeedControlFrame.BackgroundColor3 = Color3.fromRGB(25, 25, 25)
SpeedControlFrame.BorderSizePixel = 0
SpeedControlFrame.Parent = MainFrame

-- Speed Label
local SpeedLabel = Instance.new("TextLabel")
SpeedLabel.Size = UDim2.new(0.3, 0, 1, 0)
SpeedLabel.BackgroundTransparency = 1
SpeedLabel.Text = "Speed:"
SpeedLabel.TextColor3 = Color3.fromRGB(255, 255, 255)
SpeedLabel.TextSize = 14
SpeedLabel.Font = Enum.Font.GothamSemibold
SpeedLabel.Parent = SpeedControlFrame

-- Speed Input
local SpeedInput = Instance.new("TextBox")
SpeedInput.Size = UDim2.new(0.4, 0, 1, 0)
SpeedInput.Position = UDim2.new(0.3, 0, 0, 0)
SpeedInput.BackgroundColor3 = Color3.fromRGB(35, 35, 35)
SpeedInput.BorderSizePixel = 0
SpeedInput.Text = "1.5"
SpeedInput.TextColor3 = Color3.fromRGB(255, 255, 255)
SpeedInput.TextSize = 14
SpeedInput.Font = Enum.Font.GothamSemibold
SpeedInput.PlaceholderText = "Enter speed..."
SpeedInput.Parent = SpeedControlFrame

-- Apply Button
local ApplyButton = Instance.new("TextButton")
ApplyButton.Size = UDim2.new(0.3, 0, 1, 0)
ApplyButton.Position = UDim2.new(0.7, 0, 0, 0)
ApplyButton.BackgroundColor3 = Color3.fromRGB(0, 120, 255)
ApplyButton.BorderSizePixel = 0
ApplyButton.Text = "Apply"
ApplyButton.TextColor3 = Color3.fromRGB(255, 255, 255)
ApplyButton.TextSize = 14
ApplyButton.Font = Enum.Font.GothamSemibold
ApplyButton.Parent = SpeedControlFrame

-- Add corner radius to all elements
local function addCorners(element)
    local UICorner = Instance.new("UICorner")
    UICorner.CornerRadius = UDim.new(0, 4)
    UICorner.Parent = element
end

addCorners(SpeedControlFrame)
addCorners(SpeedInput)
addCorners(ApplyButton)

-- Variables for speed
local customSpeedValue = 1.5 -- Default speed value for both fly and speed hack

-- Apply button functionality
ApplyButton.MouseButton1Click:Connect(function()
    local newValue = tonumber(SpeedInput.Text)
    if newValue then
        customSpeedValue = math.clamp(newValue, 0.1, 10)
        NotificationSystem.new("Speed Updated", "Set to " .. customSpeedValue, 3)
    end
end)

-- Enter key functionality
SpeedInput.FocusLost:Connect(function(enterPressed)
    if enterPressed then
        local newValue = tonumber(SpeedInput.Text)
        if newValue then
            customSpeedValue = math.clamp(newValue, 0.1, 10)
            NotificationSystem.new("Speed Updated", "Set to " .. customSpeedValue, 3)
        end
    end
end)

-- Content Frame (convert to ScrollingFrame)
local ContentFrame = Instance.new("ScrollingFrame")
ContentFrame.Name = "ContentFrame"
ContentFrame.Size = UDim2.new(1, -100, 1, -130)
ContentFrame.Position = UDim2.new(0, 100, 0, 130)
ContentFrame.BackgroundColor3 = Color3.fromRGB(25, 25, 25)
ContentFrame.BorderSizePixel = 0
ContentFrame.ScrollBarThickness = 4
ContentFrame.ScrollBarImageColor3 = Color3.fromRGB(0, 120, 255)
ContentFrame.CanvasSize = UDim2.new(0, 0, 100, 0) -- Increased from 2 to 4 to allow more scrolling space
ContentFrame.Parent = MainFrame

-- Add smooth scrolling behavior
local function smoothScroll(frame)
    local dragSpeed = 0.2
    local dragToggle = nil
    local dragInput = nil
    local dragStart = nil
    local startPos = nil

    local function updateInput(input)
        local delta = input.Position - dragStart
        local position = UDim2.new(startPos.X.Scale, startPos.X.Offset, startPos.Y.Scale, startPos.Y.Offset + delta.Y)
        game:GetService('TweenService'):Create(frame, TweenInfo.new(dragSpeed), {CanvasPosition = Vector2.new(0, position.Y.Offset)}):Play()
    end

    frame.InputBegan:Connect(function(input)
        if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then
            dragToggle = true
            dragStart = input.Position
            startPos = frame.CanvasPosition
            input.Changed:Connect(function()
                if input.UserInputState == Enum.UserInputState.End then
                    dragToggle = false
                end
            end)
        end
    end)

    frame.InputChanged:Connect(function(input)
        if input.UserInputType == Enum.UserInputType.MouseMovement or input.UserInputType == Enum.UserInputType.Touch then
            dragInput = input
        end
    end)

    UserInputService.InputChanged:Connect(function(input)
        if input == dragInput and dragToggle then
            updateInput(input)
        end
    end)
end

smoothScroll(ContentFrame)

-- Function to create buttons with proper spacing
local function createButton(name, pos)
    local Button = Instance.new("TextButton")
    Button.Name = name
    Button.Size = UDim2.new(0.9, 0, 0, 35)
    Button.Position = pos
    Button.BackgroundColor3 = Color3.fromRGB(35, 35, 35)
    Button.BorderSizePixel = 0
    Button.Text = name
    Button.TextColor3 = Color3.fromRGB(255, 255, 255)
    Button.TextSize = 14
    Button.Font = Enum.Font.GothamSemibold
    Button.Parent = ContentFrame
    
    -- Add hover effect
    Button.MouseEnter:Connect(function()
        game:GetService('TweenService'):Create(Button, TweenInfo.new(0.2), {
            BackgroundColor3 = Color3.fromRGB(45, 45, 45)
        }):Play()
    end)
    
    Button.MouseLeave:Connect(function()
        game:GetService('TweenService'):Create(Button, TweenInfo.new(0.2), {
            BackgroundColor3 = Color3.fromRGB(35, 35, 35)
        }):Play()
    end)
    
    local ButtonCorner = Instance.new("UICorner")
    ButtonCorner.CornerRadius = UDim.new(0, 6)
    ButtonCorner.Parent = Button
    
    -- Add gradient
    local ButtonGradient = Instance.new("UIGradient")
    ButtonGradient.Color = ColorSequence.new({
        ColorSequenceKeypoint.new(0, Color3.fromRGB(45, 45, 45)),
        ColorSequenceKeypoint.new(1, Color3.fromRGB(35, 35, 35))
    })
    ButtonGradient.Parent = Button
    
    -- Add padding/margin
    local Padding = Instance.new("UIPadding")
    Padding.PaddingTop = UDim.new(0, 5)
    Padding.PaddingBottom = UDim.new(0, 5)
    Padding.Parent = Button
    
    return Button
end

-- === Keybind Manager ===
local ContextActionService = game:GetService("ContextActionService")
local keybinds = {} -- [button] = {keyCode = Enum.KeyCode, callback = function}

local function showKeybindPrompt(button, onSet)
    local prompt = Instance.new("TextLabel")
    prompt.Size = UDim2.new(0, 200, 0, 50)
    prompt.Position = UDim2.new(0.5, -100, 0.5, -25)
    prompt.BackgroundColor3 = Color3.fromRGB(30,30,30)
    prompt.TextColor3 = Color3.fromRGB(255,255,255)
    prompt.Text = "Press any key to bind..."
    prompt.Parent = game.CoreGui
    local conn
    conn = UserInputService.InputBegan:Connect(function(input, gp)
        if input.UserInputType == Enum.UserInputType.Keyboard then
            prompt:Destroy()
            conn:Disconnect()
            onSet(input.KeyCode)
        end
    end)
end

local function setKeybind(button, callback)
    showKeybindPrompt(button, function(keyCode)
        -- Remove previous binding if any
        if keybinds[button] then
            ContextActionService:UnbindAction("Keybind_"..button.Name)
        end
        -- Remove keybind from other buttons if duplicate
        for btn, data in pairs(keybinds) do
            if data.keyCode == keyCode then
                ContextActionService:UnbindAction("Keybind_"..btn.Name)
                btn.Text = btn.Text:gsub("%s%[.-%]", "")
                keybinds[btn] = nil
            end
        end
        keybinds[button] = {keyCode = keyCode, callback = callback}
        -- Show the key on the button
        button.Text = button.Text:gsub("%s%[.-%]", "") .. " ["..keyCode.Name.."]"
        -- Register global keybind
        ContextActionService:BindAction(
            "Keybind_"..button.Name,
            function(_, userInputState, input)
                if userInputState == Enum.UserInputState.Begin then
                    callback()
                end
            end,
            false,
            keyCode
        )
    end)
end

local function enableKeybindMenu(button, callback)
    button.MouseButton2Click:Connect(function()
        setKeybind(button, callback)
    end)
end

-- Create buttons with increased spacing
local buttonSpacing = 0.1 -- Increased spacing between buttons
local FlyButton = createButton("Fly", UDim2.new(0.05, 0, buttonSpacing * 0, 0))
local SpeedButton = createButton("Speed", UDim2.new(0.05, 0, buttonSpacing * 1, 0))
local StaminaButton = createButton("Infinite Stamina", UDim2.new(0.05, 0, buttonSpacing * 2, 0))
local OxygenButton = createButton("Infinite Oxygen", UDim2.new(0.05, 0, buttonSpacing * 3, 0))
local NoFallButton = createButton("No Fall Damage", UDim2.new(0.05, 0, buttonSpacing * 4, 0))
local JumpButton = createButton("Infinite Jump", UDim2.new(0.05, 0, buttonSpacing * 5, 0))
local TpButton = createButton("Click Teleport", UDim2.new(0.05, 0, buttonSpacing * 6, 0))
local ESPButton = createButton("ESP", UDim2.new(0.05, 0, buttonSpacing * 7, 0))
local AimbotButton = createButton("Aimbot", UDim2.new(0.05, 0, buttonSpacing * 8, 0))
local KillAuraButton = createButton("Kill Aura", UDim2.new(0.05, 0, buttonSpacing * 9, 0))
local AntiAfkButton = createButton("Anti AFK", UDim2.new(0.05, 0, buttonSpacing * 10, 0))
local WallPenetrationButton = createButton("Wall Penetration", UDim2.new(0.05, 0, buttonSpacing * 11, 0))
local AntiGrabButton = createButton("Auto Anti-Grab", UDim2.new(0.05, 0, buttonSpacing * 12, 0))
local FPSPlusButton = createButton("FPS+", UDim2.new(0.05, 0, buttonSpacing * 13, 0))

-- FPS+ Script to boost FPS by disabling shadows and heavy effects
function runFPSPlus()
    -- Disable global shadows
    if game:GetService("Lighting") then
        local Lighting = game:GetService("Lighting")
        Lighting.GlobalShadows = false
        Lighting.FogEnd = 100000
        Lighting.Brightness = 1
        -- Remove effects
        for _, v in ipairs(Lighting:GetChildren()) do
            if v:IsA("BlurEffect") or v:IsA("SunRaysEffect") or v:IsA("DepthOfFieldEffect") or v:IsA("ColorCorrectionEffect") or v:IsA("BloomEffect") or v:IsA("Atmosphere") then
                v.Enabled = false
            end
        end
    end
    -- Disable terrain decoration and water effects
    if workspace:FindFirstChildOfClass("Terrain") then
        local terrain = workspace:FindFirstChildOfClass("Terrain")
        terrain.Decoration = false
        terrain.WaterWaveSize = 0
        terrain.WaterWaveSpeed = 0
        terrain.WaterReflectance = 0
        terrain.WaterTransparency = 1
    end
    -- Disable all part and mesh shadows
    for _, obj in ipairs(workspace:GetDescendants()) do
        if obj:IsA("BasePart") then
            obj.CastShadow = false
        elseif obj:IsA("ParticleEmitter") or obj:IsA("Trail") or obj:IsA("Smoke") or obj:IsA("Fire") then
            obj.Enabled = false
        end
    end
    -- Remove post-processing effects from Camera
    if workspace.CurrentCamera then
        for _, v in ipairs(workspace.CurrentCamera:GetChildren()) do
            if v:IsA("BlurEffect") or v:IsA("SunRaysEffect") or v:IsA("DepthOfFieldEffect") or v:IsA("ColorCorrectionEffect") or v:IsA("BloomEffect") then
                v.Enabled = false
            end
        end
    end
    -- Optionally, remove decals/textures for even more FPS (comment out if you want to keep them)
    --[[
    for _, obj in ipairs(workspace:GetDescendants()) do
        if obj:IsA("Decal") or obj:IsA("Texture") then
            obj.Transparency = 1
        end
    end
    ]]
end

-- Attach keybind menus to all buttons
enableKeybindMenu(FlyButton, function()
    -- Place your fly toggle logic here
    isFlying = not isFlying
    updateButtonColor(FlyButton, isFlying)
end)
enableKeybindMenu(SpeedButton, function()
    speedEnabled = not speedEnabled
    updateButtonColor(SpeedButton, speedEnabled)
    if speedEnabled then
        applySpeedBypass()
        NotificationSystem.new("Speed Enabled", "Current speed: " .. customSpeedValue, 3)
    else
        NotificationSystem.new("Speed Disabled", "Movement speed returned to normal", 3)
    end
end)
enableKeybindMenu(StaminaButton, function()
    infiniteStaminaEnabled = not infiniteStaminaEnabled
    updateButtonColor(StaminaButton, infiniteStaminaEnabled)
    if infiniteStaminaEnabled then
        toggleInfiniteStamina()
        NotificationSystem.new("Infinite Stamina", "Stamina will not decrease", 3)
    else
        NotificationSystem.new("Stamina Normal", "Stamina system returned to normal", 3)
    end
end)
enableKeybindMenu(OxygenButton, function()
    infiniteOxygenEnabled = not infiniteOxygenEnabled
    updateButtonColor(OxygenButton, infiniteOxygenEnabled)
    if infiniteOxygenEnabled then
        toggleInfiniteOxygen()
        NotificationSystem.new("Infinite Oxygen", "Oxygen will not decrease", 3)
    else
        NotificationSystem.new("Oxygen Normal", "Oxygen system returned to normal", 3)
    end
end)
enableKeybindMenu(NoFallButton, function()
    noFallDamageEnabled = not noFallDamageEnabled
    updateButtonColor(NoFallButton, noFallDamageEnabled)
    if noFallDamageEnabled then
        toggleNoFallDamage()
        NotificationSystem.new("No Fall Damage", "Fall damage disabled", 3)
    else
        NotificationSystem.new("Fall Damage Normal", "Fall damage returned to normal", 3)
    end
end)
enableKeybindMenu(JumpButton, function()
    infiniteJumpEnabled = not infiniteJumpEnabled
    updateButtonColor(JumpButton, infiniteJumpEnabled)
    -- Place your infinite jump logic here
end)
enableKeybindMenu(TpButton, function()
    isTpEnabled = not isTpEnabled
    updateButtonColor(TpButton, isTpEnabled)
    -- Place your teleport logic here
end)
enableKeybindMenu(ESPButton, function()
    espEnabled = not espEnabled
    updateButtonColor(ESPButton, espEnabled)
    -- Place your ESP logic here
end)
enableKeybindMenu(AimbotButton, function()
    aimbotEnabled = not aimbotEnabled
    updateButtonColor(AimbotButton, aimbotEnabled)
    -- Place your aimbot logic here
end)
enableKeybindMenu(KillAuraButton, function()
    killAuraEnabled = not killAuraEnabled
    updateButtonColor(KillAuraButton, killAuraEnabled)
    -- Place your kill aura logic here
end)
enableKeybindMenu(AntiAfkButton, function()
    antiAfkEnabled = not antiAfkEnabled
    updateButtonColor(AntiAfkButton, antiAfkEnabled)
    -- Place your anti-AFK logic here
end)
enableKeybindMenu(WallPenetrationButton, function()
    wallPenetrationEnabled = not wallPenetrationEnabled
    updateButtonColor(WallPenetrationButton, wallPenetrationEnabled)
    -- Place your wall penetration logic here
end)
enableKeybindMenu(AntiGrabButton, function()
    antiGrabEnabled = not antiGrabEnabled
    updateButtonColor(AntiGrabButton, antiGrabEnabled)
    if antiGrabEnabled then
        enableAntiGrab()
        NotificationSystem.new("Anti-Grab Enabled", "Auto anti-grab is now active.", 3)
    else
        disableAntiGrab()
        NotificationSystem.new("Anti-Grab Disabled", "Auto anti-grab is now off.", 3)
    end
end)

enableKeybindMenu(FPSPlusButton, function()
    runFPSPlus()
    NotificationSystem.new("FPS+ Enabled", "FPS boost applied!", 3)
end)

-- Add list layout for automatic spacing
local ListLayout = Instance.new("UIListLayout")
ListLayout.Parent = ContentFrame
ListLayout.Padding = UDim.new(0, 10) -- 10 pixel padding between buttons
ListLayout.HorizontalAlignment = Enum.HorizontalAlignment.Center
ListLayout.SortOrder = Enum.SortOrder.LayoutOrder

-- Add padding around the edges
local FramePadding = Instance.new("UIPadding")
FramePadding.PaddingTop = UDim.new(0, 10)
FramePadding.PaddingBottom = UDim.new(0, 10)
FramePadding.PaddingLeft = UDim.new(0, 10)
FramePadding.PaddingRight = UDim.new(0, 10)
FramePadding.Parent = ContentFrame

-- Hide/Close buttons
local HideButton = Instance.new("TextButton")
HideButton.Size = UDim2.new(0, 30, 0, 30)
HideButton.Position = UDim2.new(1, -70, 0, 0)
HideButton.BackgroundColor3 = Color3.fromRGB(45, 45, 45)
HideButton.Text = "-"
HideButton.TextColor3 = Color3.fromRGB(255, 255, 255)
HideButton.TextSize = 20
HideButton.Font = Enum.Font.GothamBold
HideButton.Parent = TitleBar

local CloseButton = Instance.new("TextButton")
CloseButton.Size = UDim2.new(0, 30, 0, 30)
CloseButton.Position = UDim2.new(1, -35, 0, 0)
CloseButton.BackgroundColor3 = Color3.fromRGB(45, 45, 45)
CloseButton.Text = "X"
CloseButton.TextColor3 = Color3.fromRGB(255, 255, 255)
CloseButton.TextSize = 14
CloseButton.Font = Enum.Font.GothamBold
CloseButton.Parent = TitleBar

-- Function to update button appearance
local function updateButtonColor(button, enabled)
    if enabled then
        button.BackgroundColor3 = Color3.fromRGB(0, 170, 0)
        local gradient = Instance.new("UIGradient")
        gradient.Color = ColorSequence.new({
            ColorSequenceKeypoint.new(0, Color3.fromRGB(0, 190, 0)),
            ColorSequenceKeypoint.new(1, Color3.fromRGB(0, 170, 0))
        })
        gradient.Parent = button
    else
        button.BackgroundColor3 = Color3.fromRGB(35, 35, 35)
        local gradient = button:FindFirstChild("UIGradient")
        if gradient then gradient:Destroy() end
    end
end

-- Speed function with adjusted speed
local function applySpeedBypass()
    if not player.Character or not player.Character:FindFirstChild("HumanoidRootPart") then return end
    
    local hrp = player.Character.HumanoidRootPart
    local humanoid = player.Character:FindFirstChild("Humanoid")
    
    RunService.Heartbeat:Connect(function()
        if not speedEnabled or not humanoid or humanoid.Health <= 0 then return end
        
        local moveDirection = Vector3.new(0, 0, 0)
        
        if UserInputService:IsKeyDown(Enum.KeyCode.W) then
            moveDirection = moveDirection + (camera.CFrame.LookVector * Vector3.new(1, 0, 1)).Unit
        end
        if UserInputService:IsKeyDown(Enum.KeyCode.S) then
            moveDirection = moveDirection - (camera.CFrame.LookVector * Vector3.new(1, 0, 1)).Unit
        end
        if UserInputService:IsKeyDown(Enum.KeyCode.A) then
            moveDirection = moveDirection - (camera.CFrame.RightVector * Vector3.new(1, 0, 1)).Unit
        end
        if UserInputService:IsKeyDown(Enum.KeyCode.D) then
            moveDirection = moveDirection + (camera.CFrame.RightVector * Vector3.new(1, 0, 1)).Unit
        end
        
        if moveDirection.Magnitude > 0 then
            hrp.CFrame = hrp.CFrame + (moveDirection * customSpeedValue)
        end
    end)
end

-- Improved Infinite Stamina Function
local function toggleInfiniteStamina()
    if not player.Character then return end
    
    RunService.Heartbeat:Connect(function()
        if not infiniteStaminaEnabled then return end
        
        -- Common stamina variables and paths
        local staminaPaths = {
            {player, "Stamina"},
            {player, "stamina"},
            {player, "Energy"},
            {player, "energy"},
            {player.Character, "Stamina"},
            {player.Character, "stamina"},
            {player.Character, "Energy"},
            {player.Character, "energy"},
            {player.Character:FindFirstChild("Humanoid"), "Stamina"},
            {player.Character:FindFirstChild("Humanoid"), "Energy"}
        }
        
        -- Check player stats/leaderstats
        local stats = player:FindFirstChild("stats") or player:FindFirstChild("Statistics") or player:FindFirstChild("PlayerStats")
        if stats then
            table.insert(staminaPaths, {stats, "Stamina"})
            table.insert(staminaPaths, {stats, "Energy"})
        end
        
        -- Set all possible stamina values to max
        for _, path in ipairs(staminaPaths) do
            local parent, valueName = path[1], path[2]
            if parent and parent:FindFirstChild(valueName) then
                local value = parent:FindFirstChild(valueName)
                if value:IsA("NumberValue") or value:IsA("IntValue") then
                    value.Value = value.MaxValue or 100
                elseif value:IsA("Property") then
                    value = value.MaxValue or 100
                end
            end
        end
    end)
end

-- Improved Infinite Oxygen Function
local function toggleInfiniteOxygen()
    if not player.Character then return end
    
    RunService.Heartbeat:Connect(function()
        if not infiniteOxygenEnabled then return end
        
        -- Common oxygen variables and paths
        local oxygenPaths = {
            {player, "Oxygen"},
            {player, "oxygen"},
            {player, "Air"},
            {player, "air"},
            {player.Character, "Oxygen"},
            {player.Character, "oxygen"},
            {player.Character, "Air"},
            {player.Character, "air"},
            {player.Character:FindFirstChild("Humanoid"), "Oxygen"},
            {player.Character:FindFirstChild("Humanoid"), "Air"}
        }
        
        -- Check player stats/leaderstats
        local stats = player:FindFirstChild("stats") or player:FindFirstChild("Statistics") or player:FindFirstChild("PlayerStats")
        if stats then
            table.insert(oxygenPaths, {stats, "Oxygen"})
            table.insert(oxygenPaths, {stats, "Air"})
        end
        
        -- Set all possible oxygen values to max
        for _, path in ipairs(oxygenPaths) do
            local parent, valueName = path[1], path[2]
            if parent and parent:FindFirstChild(valueName) then
                local value = parent:FindFirstChild(valueName)
                if value:IsA("NumberValue") or value:IsA("IntValue") then
                    value.Value = value.MaxValue or 100
                elseif value:IsA("Property") then
                    value = value.MaxValue or 100
                end
            end
        end
        
        -- Additional check for underwater state
        local humanoid = player.Character:FindFirstChild("Humanoid")
        if humanoid then
            humanoid.BreathingEnabled = true
        end
    end)
end

-- No Fall Damage Function
local function toggleNoFallDamage()
    if not player.Character then return end
    
    local function preventFallDamage()
        if not noFallDamageEnabled then return end
        
        local humanoid = player.Character:FindFirstChild("Humanoid")
        if humanoid then
            humanoid.StateChanged:Connect(function(_, new)
                if new == Enum.HumanoidStateType.Landed then
                    humanoid.Health = humanoid.Health -- Prevents fall damage
                end
            end)
        end
    end
    
    preventFallDamage()
    player.CharacterAdded:Connect(preventFallDamage)
end

-- Button Functionality
SpeedButton.MouseButton1Click:Connect(function()
    speedEnabled = not speedEnabled
    updateButtonColor(SpeedButton, speedEnabled)
    
    if speedEnabled then
        applySpeedBypass()
        NotificationSystem.new("Speed Enabled", "Current speed: " .. customSpeedValue, 3)
    else
        NotificationSystem.new("Speed Disabled", "Movement speed returned to normal", 3)
    end
end)

StaminaButton.MouseButton1Click:Connect(function()
    infiniteStaminaEnabled = not infiniteStaminaEnabled
    updateButtonColor(StaminaButton, infiniteStaminaEnabled)
    
    if infiniteStaminaEnabled then
        toggleInfiniteStamina()
        NotificationSystem.new("Infinite Stamina", "Stamina will not decrease", 3)
    else
        NotificationSystem.new("Stamina Normal", "Stamina system returned to normal", 3)
    end
end)

OxygenButton.MouseButton1Click:Connect(function()
    infiniteOxygenEnabled = not infiniteOxygenEnabled
    updateButtonColor(OxygenButton, infiniteOxygenEnabled)
    
    if infiniteOxygenEnabled then
        toggleInfiniteOxygen()
        NotificationSystem.new("Infinite Oxygen", "Oxygen will not decrease", 3)
    else
        NotificationSystem.new("Oxygen Normal", "Oxygen system returned to normal", 3)
    end
end)

NoFallButton.MouseButton1Click:Connect(function()
    noFallDamageEnabled = not noFallDamageEnabled
    updateButtonColor(NoFallButton, noFallDamageEnabled)
    
    if noFallDamageEnabled then
        toggleNoFallDamage()
        NotificationSystem.new("No Fall Damage", "Fall damage disabled", 3)
    else
        NotificationSystem.new("Fall Damage Normal", "Fall damage returned to normal", 3)
    end
end)

-- Hide/Show functionality
local isHidden = false
HideButton.MouseButton1Click:Connect(function()
    isHidden = not isHidden
    ContentFrame.Visible = not isHidden
    MainFrame.Size = isHidden and UDim2.new(0, 250, 0, 30) or UDim2.new(0, 250, 0, 400)
    HideButton.Text = isHidden and "+" or "-"
end)

-- Improved Wall Penetration Function
local wallPenetrationActive = false
local hookApplied = false
local originalFunctions = {}

local function applyWallPenetrationHook()
    if hookApplied then return end
    
    -- Store original functions
    originalFunctions.Raycast = workspace.Raycast
    originalFunctions.FindPartOnRay = workspace.FindPartOnRay
    originalFunctions.FindPartOnRayWithIgnoreList = workspace.FindPartOnRayWithIgnoreList
    
    -- Create universal ignore list for all walls
    local universalIgnoreList = {player.Character}
    for _, part in pairs(workspace:GetDescendants()) do
        if part:IsA("BasePart") and part.CanCollide and 
           not part:IsDescendantOf(player.Character) and 
           part.Transparency < 1 then
            table.insert(universalIgnoreList, part)
        end
    end
    
    -- Override core Raycast functions
    workspace.Raycast = function(self, origin, direction, ...)
        if not wallPenetrationEnabled then 
            return originalFunctions.Raycast(self, origin, direction, ...)
        end
        
        -- Extend ray for wall penetration
        local extendedDirection = direction.Unit * wallPenetrationRange
        local result = originalFunctions.Raycast(self, origin, extendedDirection, ...)
        
        -- Find a valid target beyond walls
        if result and result.Instance then
            for _, player in pairs(Players:GetPlayers()) do
                if player ~= game.Players.LocalPlayer and player.Character then
                    local humanoid = player.Character:FindFirstChild("Humanoid")
                    local hrp = player.Character:FindFirstChild("HumanoidRootPart")
                    
                    if humanoid and hrp and humanoid.Health > 0 then
                        local ray = Ray.new(origin, (hrp.Position - origin).Unit * wallPenetrationRange)
                        local hit = workspace:FindPartOnRay(ray, player.Character)
                        
                        if hit and hit.Parent and hit.Parent:FindFirstChild("Humanoid") then
                            return {Instance = hit, Position = hit.Position, Normal = (origin - hit.Position).Unit, Material = hit.Material}
                        end
                    end
                end
            end
        end
        
        return result
    end
    
    -- Override FindPartOnRay
    workspace.FindPartOnRay = function(self, ray, ...)
        if not wallPenetrationEnabled then 
            return originalFunctions.FindPartOnRay(self, ray, ...)
        end
        
        -- Target enemies through walls
        local direction = ray.Direction.Unit
        local extendedRay = Ray.new(ray.Origin, direction * wallPenetrationRange)
        
        -- Try to find player parts beyond walls
        for _, plr in pairs(Players:GetPlayers()) do
            if plr ~= player and plr.Character and plr.Character:FindFirstChild("HumanoidRootPart") then
                local hrp = plr.Character.HumanoidRootPart
                local directRay = Ray.new(ray.Origin, (hrp.Position - ray.Origin).Unit * wallPenetrationRange)
                local hit = originalFunctions.FindPartOnRayWithIgnoreList(self, directRay, universalIgnoreList)
                
                if hit and hit.Parent and hit.Parent:FindFirstChild("Humanoid") then
                    return hit, hit.Position
                end
            end
        end
        
        return originalFunctions.FindPartOnRay(self, extendedRay, ...)
    end
    
    -- Override FindPartOnRayWithIgnoreList
    workspace.FindPartOnRayWithIgnoreList = function(self, ray, ignoreList, ...)
        if not wallPenetrationEnabled then 
            return originalFunctions.FindPartOnRayWithIgnoreList(self, ray, ignoreList, ...)
        end
        
        -- Combine ignore lists
        local combinedIgnoreList = {}
        for _, item in pairs(ignoreList) do
            table.insert(combinedIgnoreList, item)
        end
        for _, item in pairs(universalIgnoreList) do
            table.insert(combinedIgnoreList, item)
        end
        
        -- Extended ray for wall penetration
        local direction = ray.Direction.Unit
        local extendedRay = Ray.new(ray.Origin, direction * wallPenetrationRange)
        
        -- Direct targeting through walls
        for _, plr in pairs(Players:GetPlayers()) do
            if plr ~= player and plr.Character and plr.Character:FindFirstChild("HumanoidRootPart") then
                local hrp = plr.Character.HumanoidRootPart
                local directRay = Ray.new(ray.Origin, (hrp.Position - ray.Origin).Unit * wallPenetrationRange)
                local hit = originalFunctions.FindPartOnRayWithIgnoreList(self, directRay, combinedIgnoreList)
                
                if hit and hit.Parent and hit.Parent:FindFirstChild("Humanoid") then
                    return hit, hit.Position
                end
            end
        end
        
        return originalFunctions.FindPartOnRayWithIgnoreList(self, extendedRay, combinedIgnoreList, ...)
    end
    
    hookApplied = true
    wallPenetrationActive = true
end

local function removeWallPenetrationHook()
    if not hookApplied then return end
    
    -- Restore original functions
    workspace.Raycast = originalFunctions.Raycast
    workspace.FindPartOnRay = originalFunctions.FindPartOnRay
    workspace.FindPartOnRayWithIgnoreList = originalFunctions.FindPartOnRayWithIgnoreList
    
    hookApplied = false
    wallPenetrationActive = false
end

local function updateWallPenetration()
    if not player.Character then return end
    
    if wallPenetrationEnabled and not wallPenetrationActive then
        applyWallPenetrationHook()
    elseif not wallPenetrationEnabled and wallPenetrationActive then
        removeWallPenetrationHook()
    end
    
    if wallPenetrationEnabled then
        -- Additional direct hit registration for all weapons
        local rayParams = RaycastParams.new()
        rayParams.FilterType = Enum.RaycastFilterType.Blacklist
        rayParams.FilterDescendantsInstances = {player.Character, workspace.CurrentCamera}
        
        -- Force damage on raycast/non-raycast weapons
        for _, otherPlayer in pairs(Players:GetPlayers()) do
            if otherPlayer ~= player and otherPlayer.Character and 
               otherPlayer.Character:FindFirstChild("Humanoid") and
               otherPlayer.Character.Humanoid.Health > 0 then
                
                local hrp = otherPlayer.Character:FindFirstChild("HumanoidRootPart")
                if hrp then
                    local direction = (hrp.Position - camera.CFrame.Position).Unit
                    local ray = Ray.new(camera.CFrame.Position, direction * wallPenetrationRange)
                    
                    -- Check if player is aiming at target
                    local screenPoint = camera:WorldToViewportPoint(hrp.Position)
                    if screenPoint.Z > 0 then
                        local screenCenter = Vector2.new(camera.ViewportSize.X/2, camera.ViewportSize.Y/2)
                        local screenPosition = Vector2.new(screenPoint.X, screenPoint.Y)
                        local distance = (screenPosition - screenCenter).Magnitude
                        
                        -- If crosshair is close to target, register hits on all firing events
                        if distance < 200 then
                            -- The wall penetration is now active for this frame
                        end
                    end
                end
            end
        end
    end
end

-- Toggle Wall Penetration function
local function toggleWallPenetration()
    wallPenetrationEnabled = not wallPenetrationEnabled
    updateButtonColor(WallPenetrationButton, wallPenetrationEnabled)
    
    if wallPenetrationEnabled then
        -- Set up necessary hooks when enabled
        if not hookApplied then
            -- Use pcall to prevent errors
            pcall(function()
                applyWallPenetrationHook()
            end)
        end
        NotificationSystem.new("Wall Penetration", "Enabled - Your shots will now penetrate through walls", 3)
    else
        -- Remove hooks when disabled
        if hookApplied then
            pcall(function()
                removeWallPenetrationHook()
            end)
        end
        NotificationSystem.new("Wall Penetration", "Disabled", 3)
    end
end

-- Connect button actions
FlyButton.MouseButton1Click:Connect(toggleFly)
SpeedButton.MouseButton1Click:Connect(toggleSpeed)
StaminaButton.MouseButton1Click:Connect(toggleInfiniteStamina)
OxygenButton.MouseButton1Click:Connect(toggleInfiniteOxygen)
NoFallButton.MouseButton1Click:Connect(toggleNoFallDamage)
JumpButton.MouseButton1Click:Connect(toggleInfiniteJump)
TpButton.MouseButton1Click:Connect(toggleTeleport)
ESPButton.MouseButton1Click:Connect(toggleESP)
AimbotButton.MouseButton1Click:Connect(toggleAimbot)
KillAuraButton.MouseButton1Click:Connect(toggleKillAura)
AntiAfkButton.MouseButton1Click:Connect(toggleAntiAfk)
WallPenetrationButton.MouseButton1Click:Connect(toggleWallPenetration)


-- Improved Rivals Auto Freeze function
local function updateAutoFreeze()
    if not autoFreezeEnabled or not player.Character then return end
    
    local currentTime = tick()
    if currentTime - lastFreezeAttack < freezeCooldown then return end
    
    -- Look for Freeze Ray in inventory (expanded search terms)
    local freezeRay = nil
    
    -- Search in backpack with more terms
    if player.Backpack then
        for _, tool in pairs(player.Backpack:GetChildren()) do
            if tool:IsA("Tool") and (tool.Name:lower():find("freeze") or 
                                     tool.Name:lower():find("ray") or 
                                     tool.Name:lower():find("ice") or 
                                     tool.Name:lower():find("cold")) then
                freezeRay = tool
                break
            end
        end
    end
    
    -- Look for equipped Freeze Ray with expanded search
    if not freezeRay and player.Character then
        for _, tool in pairs(player.Character:GetChildren()) do
            if tool:IsA("Tool") and (tool.Name:lower():find("freeze") or 
                                     tool.Name:lower():find("ray") or 
                                     tool.Name:lower():find("ice") or 
                                     tool.Name:lower():find("cold")) then
                freezeRay = tool
                break
            end
        end
    end
    
    -- Try to get Freeze Ray from game if we don't have it
    if not freezeRay then
        pcall(function()
            local weaponStorage = game:GetService("ReplicatedStorage"):FindFirstChild("Weapons")
            if weaponStorage then
                for _, item in pairs(weaponStorage:GetDescendants()) do
                    if item:IsA("Tool") and (item.Name:lower():find("freeze") or 
                                            item.Name:lower():find("ray") or 
                                            item.Name:lower():find("ice")) then
                        -- Try to get this weapon
                        local clonedWeapon = item:Clone()
                        clonedWeapon.Parent = player.Backpack
                        freezeRay = clonedWeapon
                        break
                    end
                end
            end
        end)
    end
    
    if not freezeRay then 
        -- If still no freeze ray, try using any weapon but modify its effects
        if player.Character then
            for _, tool in pairs(player.Character:GetChildren()) do
                if tool:IsA("Tool") then
                    freezeRay = tool
                    -- Add freeze properties to this weapon
                    pcall(function()
                        local freezeEffect = Instance.new("StringValue")
                        freezeEffect.Name = "FreezeEffect"
                        freezeEffect.Value = "true"
                        freezeEffect.Parent = tool
                    end)
                    break
                end
            end
        end
        
        if not freezeRay and player.Backpack then
            for _, tool in pairs(player.Backpack:GetChildren()) do
                if tool:IsA("Tool") then
                    freezeRay = tool
                    -- Add freeze properties
                    pcall(function()
                        local freezeEffect = Instance.new("StringValue")
                        freezeEffect.Name = "FreezeEffect"
                        freezeEffect.Value = "true"
                        freezeEffect.Parent = tool
                    end)
                    break
                end
            end
        end
    end
    
    if not freezeRay then return end -- Still no weapon found
    
    -- Auto freeze nearby players with enhanced targeting
    for _, otherPlayer in pairs(Players:GetPlayers()) do
        if otherPlayer ~= player and otherPlayer.Character and 
           otherPlayer.Character:FindFirstChild("Humanoid") and
           otherPlayer.Character.Humanoid.Health > 0 then
            
            local otherHRP = otherPlayer.Character:FindFirstChild("HumanoidRootPart")
            if otherHRP then
                local distance = (otherHRP.Position - player.Character.HumanoidRootPart.Position).Magnitude
                
                if distance <= freezeRayRange then
                    -- Auto equip the freeze ray
                    if not player.Character:FindFirstChild(freezeRay.Name) then
                        freezeRay.Parent = player.Character
                    end
                    
                    -- Try multiple methods to fire the weapon
                    -- Method 1: Use fire event directly
                    local success = false
                    for _, eventName in pairs({"Fire", "Activate", "MouseClick", "Shoot", "Attack"}) do
                        local fireEvent = freezeRay:FindFirstChild(eventName)
                        if fireEvent and fireEvent:IsA("RemoteEvent") then
                            pcall(function()
                                fireEvent:FireServer(otherHRP.Position)
                                success = true
                            end)
                            if success then break end
                        end
                    end
                    
                    -- Method 2: Find and activate any RemoteEvent
                    if not success then
                        for _, obj in pairs(freezeRay:GetDescendants()) do
                            if obj:IsA("RemoteEvent") then
                                pcall(function()
                                    obj:FireServer(otherHRP.Position)
                                    success = true
                                end)
                                if success then break end
                            end
                        end
                    end
                    
                    -- Method 3: Try to simulate mouse click on target
                    if not success then
                        -- This is a fallback method
                        local virtualMouse = {Target = otherHRP, Hit = otherHRP.Position}
                        for _, func in pairs(getgc()) do
                            if type(func) == "function" and islclosure(func) and not is_synapse_function(func) then
                                local upvalues = getupvalues(func)
                                for _, upvalue in pairs(upvalues) do
                                    if type(upvalue) == "userdata" and upvalue == freezeRay then
                                        pcall(function() func(virtualMouse) end)
                                        success = true
                                        break
                                    end
                                end
                                if success then break end
                            end
                        end
                    end
                    
                    -- Track cooldown
                    lastFreezeAttack = currentTime
                    -- Apply custom freeze effect to target
                    pcall(function()
                        local freezeEffect = Instance.new("NumberValue")
                        freezeEffect.Name = "FreezeTime"
                        freezeEffect.Value = 5 -- 5 seconds freeze
                        freezeEffect.Parent = otherPlayer.Character
                        
                        -- Force slow the player
                        if otherPlayer.Character:FindFirstChild("Humanoid") then
                            otherPlayer.Character.Humanoid.WalkSpeed = 0
                            spawn(function()
                                wait(5)
                                if otherPlayer.Character and otherPlayer.Character:FindFirstChild("Humanoid") then
                                    otherPlayer.Character.Humanoid.WalkSpeed = 16
                                end
                            end)
                        end
                    end)
                    break -- Only freeze one player at a time
                end
            end
        end
    end
end

-- Rivals Insta Kill function
local function updateInstaKill()
    if not instaKillEnabled or not player.Character then return end
    
    -- Look for high damage weapons (sniper, knife, etc)
    local damageWeapon = nil
    for _, tool in pairs(player.Backpack:GetChildren()) do
        if tool:IsA("Tool") and (tool.Name:lower():find("sniper") or tool.Name:lower():find("knife") or tool.Name:lower():find("shotgun")) then
            damageWeapon = tool
            break
        end
    end
    
    -- Check equipped weapon
    if not damageWeapon and player.Character then
        for _, tool in pairs(player.Character:GetChildren()) do
            if tool:IsA("Tool") and (tool.Name:lower():find("sniper") or tool.Name:lower():find("knife") or tool.Name:lower():find("shotgun")) then
                damageWeapon = tool
                break
            end
        end
    end
    
    if not damageWeapon then return end -- No suitable weapon found
    
    -- Check for frozen enemies and insta-kill them
    for _, otherPlayer in pairs(Players:GetPlayers()) do
        if otherPlayer ~= player and otherPlayer.Character then
            local humanoid = otherPlayer.Character:FindFirstChild("Humanoid")
            local hrp = otherPlayer.Character:FindFirstChild("HumanoidRootPart")
            
            if humanoid and hrp and humanoid.Health > 0 then
                -- Check if player is frozen (usually causes WalkSpeed to be 0)
                if humanoid.WalkSpeed <= 0 or otherPlayer.Character:FindFirstChild("Frozen") then
                    -- Target and shoot frozen player
                    if not player.Character:FindFirstChild(damageWeapon.Name) then
                        damageWeapon.Parent = player.Character
                    end
                    
                    -- Use damage weapon remote events
                    local fireEvent = damageWeapon:FindFirstChild("Fire") or damageWeapon:FindFirstChild("Shoot") or damageWeapon:FindFirstChild("Attack")
                    if fireEvent and fireEvent:IsA("RemoteEvent") then
                        fireEvent:FireServer(hrp.Position)
                    end
                    
                    break -- Only target one frozen player at a time
                end
            end
        end
    end
end

-- Rivals Rapid Fire function
local function updateRapidFire()
    if not rapidFireEnabled or not player.Character then return end
    
    local currentTime = tick()
    if currentTime - lastRapidFire < rapidFireCooldown then return end
    
    -- Get currently equipped weapon
    local weapon = nil
    for _, tool in pairs(player.Character:GetChildren()) do
        if tool:IsA("Tool") then
            weapon = tool
            break
        end
    end
    
    if not weapon then return end -- No weapon equipped
    
    -- Speed up all fire events and cooldowns
    for _, obj in pairs(weapon:GetDescendants()) do
        if obj:IsA("RemoteEvent") and (obj.Name:lower():find("fire") or obj.Name:lower():find("shoot")) then
            -- Find a target to shoot at
            local closestPlayer = nil
            local closestDistance = math.huge
            
            for _, otherPlayer in pairs(Players:GetPlayers()) do
                if otherPlayer ~= player and otherPlayer.Character and 
                otherPlayer.Character:FindFirstChild("Humanoid") and
                otherPlayer.Character.Humanoid.Health > 0 then
                    local otherHRP = otherPlayer.Character:FindFirstChild("HumanoidRootPart")
                    if otherHRP then
                        local distance = (otherHRP.Position - player.Character.HumanoidRootPart.Position).Magnitude
                        
                        if distance < closestDistance then
                            closestPlayer = otherPlayer
                            closestDistance = distance
                        end
                    end
                end
            end
            
            -- Rapid fire at closest player
            if closestPlayer and closestPlayer.Character and closestPlayer.Character:FindFirstChild("HumanoidRootPart") then
                local targetPos = closestPlayer.Character.HumanoidRootPart.Position
                obj:FireServer(targetPos)
                lastRapidFire = currentTime
                break
            end
        end
    end
end

-- Rivals Unlimited Satchel function
local function updateUnlimitedSatchel()
    if not unlimitedSatchelEnabled or not player.Character then return end
    
    -- Look for satchel in inventory
    local satchel = nil
    for _, tool in pairs(player.Backpack:GetChildren()) do
        if tool:IsA("Tool") and tool.Name:lower():find("satchel") then
            satchel = tool
            break
        end
    end
    
    -- Check equipped satchel
    if not satchel and player.Character then
        for _, tool in pairs(player.Character:GetChildren()) do
            if tool:IsA("Tool") and tool.Name:lower():find("satchel") then
                satchel = tool
                break
            end
        end
    end
    
    if not satchel then return end -- No satchel found
    
    -- Remove cooldown and ammo limit from satchel
    for _, value in pairs(satchel:GetDescendants()) do
        if value:IsA("NumberValue") and (value.Name:lower():find("cooldown") or value.Name:lower():find("ammo")) then
            value.Value = 0 -- No cooldown
        end
    end
    
    -- Ensure we can always throw satchels
    local throwEvent = satchel:FindFirstChild("Throw") or satchel:FindFirstChild("Deploy")
    if throwEvent and throwEvent:IsA("RemoteEvent") then
        -- Keep satchel ready by modifying internal values
        for _, script in pairs(satchel:GetDescendants()) do
            if script:IsA("LocalScript") then
                -- This is a placeholder for more complex script modification
                -- In a real implementation, we'd need to access script.Disabled
            end
        end
    end
end

-- Rivals Auto Movement function
local function updateAutoMove()
    if not autoMoveEnabled or not player.Character then return end
    
    local humanoid = player.Character:FindFirstChild("Humanoid")
    if not humanoid then return end
    
    -- Combat situation detection
    local inCombat = false
    for _, otherPlayer in pairs(Players:GetPlayers()) do
        if otherPlayer ~= player and otherPlayer.Character and 
           otherPlayer.Character:FindFirstChild("HumanoidRootPart") then
            local distance = (otherPlayer.Character.HumanoidRootPart.Position - player.Character.HumanoidRootPart.Position).Magnitude
            if distance < 30 then
                inCombat = true
                break
            end
        end
    end
    
    if inCombat then
        -- Perform slide jump technique (common in Rivals)
        humanoid.Jump = true
        wait(0.1)
        -- Simulate slide
        humanoid:ChangeState(Enum.HumanoidStateType.Seated)
        wait(0.05)
        humanoid:ChangeState(Enum.HumanoidStateType.Running)
    end
end

-- Forward declarations for references
local AutoFreezeButton
local InstaKillButton
local RapidFireButton
local UnlimitedSatchelButton
local AutoMoveButton
local PreTimerMoveButton
local BackTeleportButton
local StickToTargetButton
local AllHeadshotsButton
local RivalAimbotButton
local RivalESPButton
local AutoFarmButton
local InfiniteJumpButton
local BHopButton
local NoClipButton
local CFrameWalkButton

-- Toggle functions for Rivals
local function toggleAutoFreeze()
    autoFreezeEnabled = not autoFreezeEnabled
    updateButtonColor(AutoFreezeButton, autoFreezeEnabled)
    
    if autoFreezeEnabled then
        -- Find all freeze rays in the game
        local freezeRayFound = false
        if player.Backpack then
            for _, tool in pairs(player.Backpack:GetChildren()) do
                if tool:IsA("Tool") and (tool.Name:lower():find("freeze") or tool.Name:lower():find("ray") or tool.Name:lower():find("ice")) then
                    freezeRayFound = true
                    break
                end
            end
        end
        
        -- Enhanced notification with info
        if freezeRayFound then
            NotificationSystem.new("Auto Freeze", "Enabled - Freeze ray detected and ready to use", 3)
        else
            NotificationSystem.new("Auto Freeze", "Enabled - Acquire a freeze ray for best results", 3)
        end
        
        -- Make sure anti-cheat protection is also enabled
        if not antiAfkEnabled then
            toggleAntiAfk()
        end
    else
        NotificationSystem.new("Auto Freeze", "Disabled", 3)
    end
end

local function toggleInstaKill()
    instaKillEnabled = not instaKillEnabled
    updateButtonColor(InstaKillButton, instaKillEnabled)
    
    if instaKillEnabled then
        -- Advanced setup for insta kill
        pcall(function()
            -- Detect and override damage values where possible
            for _, v in pairs(getgc(true)) do
                if type(v) == "table" then
                    for k, val in pairs(v) do
                        if type(k) == "string" and (k:lower():find("damage") or k:lower():find("health")) and type(val) == "number" then
                            v[k] = 999 -- Set to extreme value
                        end
                    end
                end
            end
        end)
        
        NotificationSystem.new("Insta Kill", "Enabled - Weapon damage increased for one-hit kills", 3)
    else
        NotificationSystem.new("Insta Kill", "Disabled", 3)
    end
end

local function toggleRapidFire()
    rapidFireEnabled = not rapidFireEnabled
    updateButtonColor(RapidFireButton, rapidFireEnabled)
    
    if rapidFireEnabled then
        -- Apply more aggressive rapid fire optimizations
        rapidFireCooldown = 0.01 -- Extremely fast cooldown
        
        -- Try to hook fire rate limiters
        pcall(function()
            for _, v in pairs(getgc(true)) do
                if type(v) == "table" then
                    for k, val in pairs(v) do
                        if type(k) == "string" and 
                          (k:lower():find("cooldown") or k:lower():find("firerate") or k:lower():find("delay")) and 
                          type(val) == "number" and val > 0 then
                            v[k] = 0.01 -- Minimize cooldowns
                        end
                    end
                end
            end
        end)
        
        NotificationSystem.new("Rapid Fire", "Enabled - Extreme fire rate activated!", 3)
    else
        NotificationSystem.new("Rapid Fire", "Disabled", 3)
    end
end

local function toggleUnlimitedSatchel()
    unlimitedSatchelEnabled = not unlimitedSatchelEnabled
    updateButtonColor(UnlimitedSatchelButton, unlimitedSatchelEnabled)
    
    if unlimitedSatchelEnabled then
        -- Create a hook that forces satchel availability
        local foundSatchel = false
        
        -- Try to modify satchel inventory directly
        pcall(function()
            local weaponsFolder = game:GetService("ReplicatedStorage"):FindFirstChild("Weapons")
            if weaponsFolder and weaponsFolder:FindFirstChild("Satchel") then
                foundSatchel = true
                
                -- Clone and modify satchel properties if possible
                local satchel = weaponsFolder.Satchel:Clone()
                for _, v in pairs(satchel:GetDescendants()) do
                    if v:IsA("NumberValue") and 
                      (v.Name:lower():find("ammo") or v.Name:lower():find("cooldown") or v.Name:lower():find("charges")) then
                        v.Value = math.huge -- Set to infinite
                    end
                end
            end
        end)
        
        if foundSatchel then
            NotificationSystem.new("Unlimited Satchel", "Enabled - Satchel charges are now unlimited", 3)
        else
            NotificationSystem.new("Unlimited Satchel", "Enabled - Equip a satchel to use this feature", 3)
        end
    else
        NotificationSystem.new("Unlimited Satchel", "Disabled", 3)
    end
end

local function toggleAutoMove()
    autoMoveEnabled = not autoMoveEnabled
    updateButtonColor(AutoMoveButton, autoMoveEnabled)
    
    if autoMoveEnabled then
        -- Enhanced movement with slide-jump techniques
        spawn(function()
            while autoMoveEnabled and wait(0.1) do
                if player.Character and player.Character:FindFirstChild("Humanoid") then
                    local humanoid = player.Character.Humanoid
                    
                    -- Combat detection - enable advanced movement when enemies are near
                    local inCombat = false
                    for _, otherPlayer in pairs(Players:GetPlayers()) do
                        if otherPlayer ~= player and otherPlayer.Character and otherPlayer.Character:FindFirstChild("HumanoidRootPart") then
                            local distance = (otherPlayer.Character.HumanoidRootPart.Position - player.Character.HumanoidRootPart.Position).Magnitude
                            if distance < 40 then -- Increased detection range
                                inCombat = true
                                break
                            end
                        end
                    end
                    
                    if inCombat then
                        -- Enhanced movement techniques
                        humanoid.Jump = true
                        wait(0.2)
                        humanoid:ChangeState(Enum.HumanoidStateType.Seated)
                        wait(0.1)
                        humanoid:ChangeState(Enum.HumanoidStateType.Running)
                    end
                end
            end
        end)
        
        NotificationSystem.new("Auto Movement", "Enabled - Advanced movement activated", 3)
    else
        NotificationSystem.new("Auto Movement", "Disabled", 3)
    end
end

-- Detect Rivals game on load and force UI creation immediately
spawn(function()
    wait(2) -- Shorter wait time for game to load
    
    -- Always create the Rivals UI section for testing
    setupRivalsUI()
    
    -- Check periodically for Rivals game (in case player joins mid-session)
    while wait(10) do
        if detectRivalsGame() and not rivalsSection then
            setupRivalsUI()
        end
    end
end)

-- Create globals for button references
_G.RivalsButtons = {}

-- Force UI refresh to make sure buttons appear
spawn(function()
    wait(3)
    if ContentFrame then
        for _, child in pairs(ContentFrame:GetChildren()) do
            if child:IsA("Frame") or child:IsA("TextButton") then
                child.Visible = true
            end
        end
    end
end)

-- Toggle functions for imported script cheats
local function toggleRivalESP()
    rivalESPEnabled = not rivalESPEnabled
    updateButtonColor(RivalESPButton, rivalESPEnabled)
    
    if rivalESPEnabled then
        NotificationSystem.new("Rival ESP", "Enabled - Players highlighted through walls", 3)
        -- Create ESP for all players
        for _, plr in pairs(Players:GetPlayers()) do
            if plr ~= player then
                pcall(function()
                    -- Create highlights
                    if plr.Character and not plr.Character:FindFirstChild("RivalHighlight") then
                        local highlight = Instance.new("Highlight")
                        highlight.Name = "RivalHighlight"
                        highlight.Adornee = plr.Character
                        highlight.DepthMode = Enum.HighlightDepthMode.AlwaysOnTop
                        highlight.FillColor = Color3.fromRGB(255, 0, 0)
                        highlight.FillTransparency = 0.5
                        highlight.OutlineColor = Color3.fromRGB(255, 255, 255)
                        highlight.OutlineTransparency = 0
                        highlight.Parent = plr.Character
                    end
                end)
            end
        end
    else
        NotificationSystem.new("Rival ESP", "Disabled", 3)
        -- Remove all ESP highlights
        for _, plr in pairs(Players:GetPlayers()) do
            pcall(function()
                if plr.Character and plr.Character:FindFirstChild("RivalHighlight") then
                    plr.Character.RivalHighlight:Destroy()
                end
            end)
        end
    end
end

local function toggleAutoFarm()
    autoFarmEnabled = not autoFarmEnabled
    updateButtonColor(AutoFarmButton, autoFarmEnabled)
    
    if autoFarmEnabled then
        NotificationSystem.new("Auto Farm", "Enabled - Automatically target and kill enemies", 3)
        -- Start auto farm logic
        spawn(function()
            local targetConnection = nil
            
            local function getClosestTarget()
                local closestPlayer = nil
                local closestDistance = math.huge
                
                for _, otherPlayer in pairs(Players:GetPlayers()) do
                    if otherPlayer ~= player and otherPlayer.Character and 
                       otherPlayer.Character:FindFirstChild("HumanoidRootPart") and
                       otherPlayer.Character:FindFirstChild("Humanoid") and
                       otherPlayer.Character.Humanoid.Health > 0 then
                        
                        local distance = (player.Character.HumanoidRootPart.Position - 
                                         otherPlayer.Character.HumanoidRootPart.Position).Magnitude
                        
                        if distance < closestDistance then
                            closestDistance = distance
                            closestPlayer = otherPlayer
                        end
                    end
                end
                
                return closestPlayer
            end
            
            local function followTarget()
                if player.Character and player.Character:FindFirstChild("HumanoidRootPart") then
                    local target = getClosestTarget()
                    if target and target.Character and target.Character:FindFirstChild("HumanoidRootPart") then
                        local targetHRP = target.Character.HumanoidRootPart
                        local playerHRP = player.Character.HumanoidRootPart
                        
                        -- Position slightly behind target
                        local targetPos = targetHRP.Position + Vector3.new(0, 2, 0)
                        playerHRP.CFrame = CFrame.new(playerHRP.Position, targetPos)
                        playerHRP.CFrame = CFrame.new(targetPos, targetHRP.Position)
                        
                        -- Try to attack
                        local tool = nil
                        for _, item in pairs(player.Character:GetChildren()) do
                            if item:IsA("Tool") then
                                tool = item
                                break
                            end
                        end
                        
                        if tool then
                            -- Simulate firing the weapon
                            local fireEvent = tool:FindFirstChild("Fire") or tool:FindFirstChild("Shoot") or
                                              tool:FindFirstChild("MouseButton1") or tool:FindFirstChild("Attack")
                            
                            if fireEvent and fireEvent:IsA("RemoteEvent") then
                                pcall(function()
                                    fireEvent:FireServer(targetHRP.Position)
                                end)
                            end
                        end
                    end
                end
            end
            
            -- Start the auto farm loop
            while autoFarmEnabled do
                if player.Character and player.Character:FindFirstChild("HumanoidRootPart") then
                    followTarget()
                end
                wait(0.1)
            end
            
            if targetConnection then
                targetConnection:Disconnect()
                targetConnection = nil
            end
        end)
    else
        NotificationSystem.new("Auto Farm", "Disabled", 3)
    end
end

local function toggleInfiniteJump()
    infiniteJumpEnabled = not infiniteJumpEnabled
    updateButtonColor(InfiniteJumpButton, infiniteJumpEnabled)
    
    if infiniteJumpEnabled then
        NotificationSystem.new("Infinite Jump", "Enabled - Jump without limits", 3)
    else
        NotificationSystem.new("Infinite Jump", "Disabled", 3)
    end
end

local function toggleBHop()
    bHopEnabled = not bHopEnabled
    updateButtonColor(BHopButton, bHopEnabled)
    
    if bHopEnabled then
        NotificationSystem.new("Bunny Hop", "Enabled - Auto jump while moving", 3)
    else
        NotificationSystem.new("Bunny Hop", "Disabled", 3)
    end
end

local function toggleNoClip()
    noClipEnabled = not noClipEnabled
    updateButtonColor(NoClipButton, noClipEnabled)
    
    if noClipEnabled then
        NotificationSystem.new("No Clip", "Enabled - Walk through walls", 3)
    else
        NotificationSystem.new("No Clip", "Disabled", 3)
    end
end

local function toggleCFrameWalk()
    cFrameWalkEnabled = not cFrameWalkEnabled
    updateButtonColor(CFrameWalkButton, cFrameWalkEnabled)
    
    if cFrameWalkEnabled then
        NotificationSystem.new("Speed Walk", "Enabled - Move faster", 3)
    else
        NotificationSystem.new("Speed Walk", "Disabled", 3)
    end
end

-- Toggle functions for new Rivals cheats
local function togglePreTimerMove()
    preTimerMoveEnabled = not preTimerMoveEnabled
    updateButtonColor(PreTimerMoveButton, preTimerMoveEnabled)
    
    if preTimerMoveEnabled then
        NotificationSystem.new("Pre-Timer Move", "Enabled - Now you can move before the round timer starts", 3)
    else
        NotificationSystem.new("Pre-Timer Move", "Disabled", 3)
    end
end

local function toggleBackTeleport()
    backTeleportEnabled = not backTeleportEnabled
    updateButtonColor(BackTeleportButton, backTeleportEnabled)
    
    if backTeleportEnabled then
        NotificationSystem.new("Back Teleport", "Enabled - You'll teleport behind the closest enemy", 3)
    else
        NotificationSystem.new("Back Teleport", "Disabled", 3)
        -- Also disable stick if it was enabled
        if stickToTargetEnabled then
            toggleStickToTarget()
        end
    end
end

local function toggleStickToTarget()
    stickToTargetEnabled = not stickToTargetEnabled
    updateButtonColor(StickToTargetButton, stickToTargetEnabled)
    
    if stickToTargetEnabled then
        NotificationSystem.new("Stick To Target", "Enabled - You'll stay behind your target", 3)
        -- Enable back teleport if it wasn't already on
        if not backTeleportEnabled then
            toggleBackTeleport()
        end
    else
        NotificationSystem.new("Stick To Target", "Disabled", 3)
        currentStickTarget = nil
    end
end

local function toggleAllHeadshots()
    allHeadshotsEnabled = not allHeadshotsEnabled
    updateButtonColor(AllHeadshotsButton, allHeadshotsEnabled)
    
    if allHeadshotsEnabled then
        NotificationSystem.new("All Headshots", "Enabled - All your shots will be headshots for maximum damage", 3)
    else
        NotificationSystem.new("All Headshots", "Disabled", 3)
    end
end

local function toggleRivalAimbot()
    rivalAimbotEnabled = not rivalAimbotEnabled
    updateButtonColor(RivalAimbotButton, rivalAimbotEnabled)
    
    if rivalAimbotEnabled then
        NotificationSystem.new("Rivals Aimbot", "Enabled - Advanced aimbot optimized for Rivals", 3)
    else
        NotificationSystem.new("Rivals Aimbot", "Disabled", 3)
    end
end

-- Pre-Timer Move Function
local function updatePreTimerMove()
    if not preTimerMoveEnabled or not player.Character then return end
    
    -- Detect round timer/pre-round state
    local gameStates = {
        workspace:FindFirstChild("RoundTimerGUI"),
        workspace:FindFirstChild("RoundTimer"),
        workspace:FindFirstChild("PreRound"),
        workspace:FindFirstChild("GameState"),
        player:FindFirstChild("PlayerGui"):FindFirstChild("RoundTimer")
    }
    
    local inPreRound = false
    for _, state in pairs(gameStates) do
        if state then
            if typeof(state) == "Instance" and state:IsA("StringValue") and state.Value:lower():find("wait") then
                inPreRound = true
                break
            elseif typeof(state) == "Instance" and state:IsA("NumberValue") and state.Value <= 0 then
                inPreRound = true
                break
            elseif typeof(state) == "Instance" and state:IsA("BoolValue") and state.Value == true then
                inPreRound = true
                break
            elseif typeof(state) == "Instance" and state:FindFirstChild("Text") and state.Text:lower():find("start") then
                inPreRound = true
                break
            end
        end
    end
    
    -- If in pre-round, enable movement
    if inPreRound then
        -- Try to enable movement during freeze time
        pcall(function()
            if player.Character:FindFirstChild("Humanoid") then
                -- Override restricted movement
                player.Character.Humanoid.WalkSpeed = 16  -- Normal walk speed
                
                -- Attempt to bypass movement restriction scripts
                for _, script in pairs(player.Character:GetDescendants()) do
                    if script:IsA("Script") or script:IsA("LocalScript") then
                        pcall(function()
                            script.Disabled = true
                        end)
                    end
                end
                
                -- Override any game state control
                for _, freeze in pairs(player.Character:GetDescendants()) do
                    if freeze:IsA("StringValue") and (freeze.Name:lower():find("freeze") or freeze.Name:lower():find("state")) then
                        freeze.Value = "Active"
                    elseif freeze:IsA("BoolValue") and (freeze.Name:lower():find("freeze") or freeze.Name:lower():find("state")) then
                        freeze.Value = false
                    end
                end
            end
        end)
    end
end

-- Back Teleport + Stick To Target Functions
local function getClosestRivalsTarget()
    if not player.Character or not player.Character:FindFirstChild("HumanoidRootPart") then
        return nil
    end
    
    local closestDistance = math.huge
    local closestPlayer = nil
    local hrp = player.Character.HumanoidRootPart
    
    for _, otherPlayer in pairs(Players:GetPlayers()) do
        if otherPlayer ~= player and otherPlayer.Character and 
           otherPlayer.Character:FindFirstChild("HumanoidRootPart") and 
           otherPlayer.Character:FindFirstChild("Humanoid") and 
           otherPlayer.Character.Humanoid.Health > 0 then
           
            local distance = (hrp.Position - otherPlayer.Character.HumanoidRootPart.Position).Magnitude
            if distance < closestDistance and distance < 100 then  -- Only within 100 studs
                closestDistance = distance
                closestPlayer = otherPlayer
            end
        end
    end
    
    return closestPlayer
end

local function updateBackTeleport()
    if not backTeleportEnabled or not player.Character then return end
    
    local currentTime = tick()
    if currentTime - lastBackTeleport < teleportCooldown then return end
    
    -- Find the closest player to teleport behind
    local target = getClosestRivalsTarget()
    if not target then return end
    
    local targetHRP = target.Character.HumanoidRootPart
    local targetCF = targetHRP.CFrame
    
    -- Calculate position behind target
    local behind = targetCF * CFrame.new(0, 0, 3)  -- 3 studs behind
    
    -- Save our current position to restore after teleport
    local originalPosition = player.Character.HumanoidRootPart.Position
    
    -- Teleport behind the target
    pcall(function()
        player.Character:SetPrimaryPartCFrame(behind)
    end)
    
    lastBackTeleport = currentTime
    
    -- If stick to target is enabled, keep following
    if stickToTargetEnabled then
        currentStickTarget = target
    else
        currentStickTarget = nil
    end
end

local function updateStickToTarget()
    if not stickToTargetEnabled or not currentStickTarget or not player.Character then return end
    
    -- Check if target still exists and is alive
    if not currentStickTarget.Character or 
       not currentStickTarget.Character:FindFirstChild("HumanoidRootPart") or
       not currentStickTarget.Character:FindFirstChild("Humanoid") or
       currentStickTarget.Character.Humanoid.Health <= 0 then
        currentStickTarget = nil
        return
    end
    
    -- Stay behind target
    local targetHRP = currentStickTarget.Character.HumanoidRootPart
    local targetCF = targetHRP.CFrame
    local behind = targetCF * CFrame.new(0, 0, 3)  -- 3 studs behind
    
    -- Smooth follow
    pcall(function()
        player.Character:SetPrimaryPartCFrame(behind)
    end)
end

-- All Headshots Function
local function updateAllHeadshots()
    if not allHeadshotsEnabled or not player.Character then return end
    
    -- Find all weapons and modify them to always hit the head
    for _, tool in pairs(player.Character:GetChildren()) do
        if tool:IsA("Tool") then
            -- Search for any fire events
            for _, obj in pairs(tool:GetDescendants()) do
                if obj:IsA("RemoteEvent") and (obj.Name:lower():find("fire") or obj.Name:lower():find("shoot")) then
                    -- Hook the fire event
                    local oldNamecall
                    oldNamecall = hookmetamethod(game, "__namecall", function(self, ...)
                        local args = {...}
                        local method = getnamecallmethod()
                        
                        if self == obj and method == "FireServer" and allHeadshotsEnabled then
                            -- Find if any argument is a position vector
                            for i, arg in pairs(args) do
                                if typeof(arg) == "Vector3" then
                                    -- Get closest player
                                    local target = getClosestRivalsTarget()
                                    if target and target.Character and target.Character:FindFirstChild("Head") then
                                        -- Replace aim position with head position
                                        args[i] = target.Character.Head.Position
                                    end
                                    break
                                end
                            end
                            
                            return oldNamecall(self, unpack(args))
                        end
                        
                        return oldNamecall(self, ...)
                    end)
                end
            end
        end
    end
    
    -- Also attempt to modify damage values
    pcall(function()
        for _, v in pairs(getgc(true)) do
            if type(v) == "table" then
                -- Look for headshot multipliers
                for k, val in pairs(v) do
                    if type(k) == "string" and k:lower():find("head") and type(val) == "number" then
                        -- Increase headshot multiplier
                        v[k] = headShotMultiplier
                    end
                end
            end
        end
    end)
end

-- Rivals Aimbot Function
local function updateRivalAimbot()
    if not rivalAimbotEnabled or not player.Character then return end
    
    -- Get the closest target
    local target = getClosestRivalsTarget()
    if not target then return end
    
    -- Aim at the target's head for optimal damage
    local head = target.Character:FindFirstChild("Head")
    if not head then return end
    
    -- Calculate aim direction
    local camera = workspace.CurrentCamera
    local cameraPosition = camera.CFrame.Position
    local headPosition = head.Position
    local aimDirection = (headPosition - cameraPosition).Unit
    
    -- Set camera to look at target's head
    camera.CFrame = CFrame.new(cameraPosition, headPosition)
    
    -- Auto fire if possible
    if UserInputService:IsMouseButtonPressed(Enum.UserInputType.MouseButton1) then
        -- Find equipped weapon
        for _, tool in pairs(player.Character:GetChildren()) do
            if tool:IsA("Tool") then
                for _, obj in pairs(tool:GetDescendants()) do
                    if obj:IsA("RemoteEvent") and (obj.Name:lower():find("fire") or obj.Name:lower():find("shoot")) then
                        pcall(function()
                            obj:FireServer(headPosition)
                        end)
                        break
                    end
                end
                break
            end
        end
    end
end

-- Imported script function implementations
local function updateRivalESP()
    if not rivalESPEnabled then return end
    
    -- Make sure all players have ESP
    for _, plr in pairs(Players:GetPlayers()) do
        if plr ~= player and plr.Character then
            pcall(function()
                if not plr.Character:FindFirstChild("RivalHighlight") then
                    local highlight = Instance.new("Highlight")
                    highlight.Name = "RivalHighlight"
                    highlight.Adornee = plr.Character
                    highlight.DepthMode = Enum.HighlightDepthMode.AlwaysOnTop
                    highlight.FillColor = Color3.fromRGB(255, 0, 0)
                    highlight.FillTransparency = 0.5
                    highlight.OutlineColor = Color3.fromRGB(255, 255, 255)
                    highlight.OutlineTransparency = 0
                    highlight.Parent = plr.Character
                end
            end)
        end
    end
end

local function updateBHop()
    if not bHopEnabled or not player.Character then return end
    
    local humanoid = player.Character:FindFirstChildOfClass("Humanoid")
    if not humanoid then return end
    
    -- Auto-jump when touching ground
    if (humanoid.FloorMaterial ~= Enum.Material.Air) and not inBhopJump then
        humanoid:ChangeState(Enum.HumanoidStateType.Jumping)
        inBhopJump = true
    elseif humanoid.FloorMaterial == Enum.Material.Air then
        inBhopJump = false
    end
end

local function updateNoClip()
    if not noClipEnabled or not player.Character then return end
    
    -- Make all parts non-collidable
    for _, part in pairs(player.Character:GetDescendants()) do
        if part:IsA("BasePart") and part.CanCollide then
            part.CanCollide = false
        end
    end
end

local function updateCFrameWalk()
    if not cFrameWalkEnabled or not player.Character then return end
    
    local humanoid = player.Character:FindFirstChildOfClass("Humanoid")
    if not humanoid then return end
    
    -- Get movement direction and boost it
    local moveDirection = humanoid.MoveDirection
    if moveDirection.Magnitude > 0 then
        local hrp = player.Character.HumanoidRootPart
        local targetPosition = hrp.Position + (moveDirection * cFrameWalkSpeed)
        
        -- Check if there's an obstacle
        local ray = Ray.new(hrp.Position, targetPosition - hrp.Position)
        local hit, position = workspace:FindPartOnRay(ray, player.Character)
        
        if not hit then
            -- Move player faster
            humanoid:Move(moveDirection, false)
            hrp.CFrame = hrp.CFrame + (moveDirection * cFrameWalkSpeed)
        end
    end
end

-- Handle infinite jump
game:GetService("UserInputService").JumpRequest:Connect(function()
    if infiniteJumpEnabled and player.Character then
        local humanoid = player.Character:FindFirstChildOfClass("Humanoid")
        if humanoid then
            humanoid:ChangeState(Enum.HumanoidStateType.Jumping)
        end
    end
end)

-- Main game loops
RunService.RenderStepped:Connect(function()
    if speedEnabled then updateSpeed() end
    if espEnabled then updateESP() end
    if aimbotEnabled then updateAimbot() end
    
    -- Kill aura, auto hit, and wall penetration
    updateKillAura()
    updateSavannahAutoHit()
    if wallPenetrationEnabled then updateWallPenetration() end
    
    -- Rivals specific features
    if rivalsSection then
        if autoFreezeEnabled then updateAutoFreeze() end
        if instaKillEnabled then updateInstaKill() end
        if rapidFireEnabled then updateRapidFire() end
        if unlimitedSatchelEnabled then updateUnlimitedSatchel() end
        if autoMoveEnabled then updateAutoMove() end
        
        -- New Rivals cheats
        if preTimerMoveEnabled then updatePreTimerMove() end
        if backTeleportEnabled then updateBackTeleport() end
        if stickToTargetEnabled then updateStickToTarget() end
        if allHeadshotsEnabled then updateAllHeadshots() end
        if rivalAimbotEnabled then updateRivalAimbot() end
        
        -- Imported script cheats
        if rivalESPEnabled then updateRivalESP() end
        if bHopEnabled then updateBHop() end
        if noClipEnabled then updateNoClip() end
        if cFrameWalkEnabled then updateCFrameWalk() end
        if infiniteJumpEnabled then --[[ handled by input event ]] end
    end
end)

-- ESP System
local espDrawings = {}

-- Optimized ESP System (single loop, minimal lag, same visuals)
local function createESPForPlayer(plr)
    if plr == player then return end
    if espDrawings[plr.Name] then return end
    local esp = {
        box = Drawing.new("Square"),
        name = Drawing.new("Text"),
        distance = Drawing.new("Text"),
        health = Drawing.new("Text")
    }
    esp.box.Thickness = 1
    esp.box.Filled = false
    esp.box.Color = Color3.new(1, 1, 1)
    esp.box.Transparency = 0.7
    esp.box.Visible = false
    esp.name.Size = 13
    esp.name.Center = true
    esp.name.Outline = true
    esp.name.Color = Color3.new(1, 1, 1)
    esp.name.Visible = false
    esp.distance.Size = 12
    esp.distance.Center = true
    esp.distance.Outline = true
    esp.distance.Color = Color3.new(1, 1, 1)
    esp.distance.Visible = false
    esp.health.Size = 12
    esp.health.Center = true
    esp.health.Outline = true
    esp.health.Color = Color3.new(1, 1, 1)
    esp.health.Visible = false
    espDrawings[plr.Name] = esp
    plr.CharacterRemoving:Connect(function()
        if espDrawings[plr.Name] then
            for _, drawing in pairs(espDrawings[plr.Name]) do
                drawing.Visible = false
            end
        end
    end)
end

local function removeESPForPlayer(plr)
    if espDrawings[plr.Name] then
        for _, drawing in pairs(espDrawings[plr.Name]) do
            drawing:Remove()
        end
        espDrawings[plr.Name] = nil
    end
end

local function updateESP()
    if not espEnabled then
        for _, esp in pairs(espDrawings) do
            for _, drawing in pairs(esp) do
                drawing.Visible = false
            end
        end
        return
    end
    -- Add ESP for new players
    for _, plr in pairs(Players:GetPlayers()) do
        if plr ~= player then
            if not espDrawings[plr.Name] then
                createESPForPlayer(plr)
            end
        end
    end
    -- Remove ESP for players who left
    for name, _ in pairs(espDrawings) do
        local found = false
        for _, plr in pairs(Players:GetPlayers()) do
            if plr.Name == name then found = true break end
        end
        if not found then
            removeESPForPlayer({Name = name})
        end
    end
end

-- Efficient ESP rendering (single RenderStepped loop)
RunService.RenderStepped:Connect(function()
    if not espEnabled then return end
    local camera = workspace.CurrentCamera
    for _, plr in pairs(Players:GetPlayers()) do
        if plr ~= player and espDrawings[plr.Name] and plr.Character and plr.Character:FindFirstChild("HumanoidRootPart") and plr.Character:FindFirstChild("Humanoid") then
            local esp = espDrawings[plr.Name]
            local humanoid = plr.Character:FindFirstChild("Humanoid")
            local hrp = plr.Character.HumanoidRootPart
            local head = plr.Character:FindFirstChild("Head")
            if not humanoid or not hrp or not head then
                esp.box.Visible = false
                esp.name.Visible = false
                esp.distance.Visible = false
                esp.health.Visible = false
            else
                local vector, onScreen = camera:WorldToViewportPoint(hrp.Position)
                local distance = (hrp.Position - camera.CFrame.Position).Magnitude
                if not onScreen or distance > 100000 then
                    esp.box.Visible = false
                    esp.name.Visible = false
                    esp.distance.Visible = false
                    esp.health.Visible = false
                else
                    local size = math.clamp(2000 / distance, 8, 50)
                    local boxSize = Vector2.new(size * 1.5, size * 3)
                    esp.box.Size = boxSize
                    esp.box.Position = Vector2.new(vector.X - boxSize.X / 2, vector.Y - boxSize.Y / 2)
                    esp.box.Visible = true
                    esp.name.Position = Vector2.new(vector.X, vector.Y - boxSize.Y / 2 - 15)
                    esp.name.Text = string.format("%s [%dm]", plr.Name, math.floor(distance))
                    esp.name.Visible = true
                    esp.health.Position = Vector2.new(vector.X, vector.Y + boxSize.Y / 2 + 5)
                    esp.health.Text = string.format("HP: %d", humanoid.Health)
                    esp.health.Color = Color3.new(1 - humanoid.Health/humanoid.MaxHealth, humanoid.Health/humanoid.MaxHealth, 0)
                    esp.health.Visible = true
                    local distanceAlpha = math.clamp(1 - (distance / 100000), 0.3, 1)
                    local healthColor = Color3.new(1 - humanoid.Health/humanoid.MaxHealth, humanoid.Health/humanoid.MaxHealth, 0)
                    esp.box.Color = healthColor
                    esp.box.Transparency = distanceAlpha
                    esp.name.Transparency = distanceAlpha
                    esp.health.Transparency = distanceAlpha
                end
            end
        elseif plr ~= player and espDrawings[plr.Name] then
            local esp = espDrawings[plr.Name]
            esp.box.Visible = false
            esp.name.Visible = false
            esp.distance.Visible = false
            esp.health.Visible = false
        end
    end
end)

-- Fly function with improved controls and camera-based movement
local function activateFly()
    if not player.Character or not player.Character:FindFirstChild("HumanoidRootPart") then return end
    
    local hrp = player.Character.HumanoidRootPart
    local humanoid = player.Character:FindFirstChild("Humanoid")
    if not humanoid then return end
    
    -- Create velocity instance for movement
    local velocity = Instance.new("BodyVelocity")
    velocity.Name = "FlyVelocity"
    velocity.Parent = hrp
    velocity.MaxForce = Vector3.new(math.huge, math.huge, math.huge)
    
    -- Create gyro for orientation control
    local gyro = Instance.new("BodyGyro")
    gyro.Name = "FlyGyro"
    gyro.Parent = hrp
    gyro.MaxTorque = Vector3.new(math.huge, math.huge, math.huge)
    gyro.P = 10000
    gyro.D = 100
    
    RunService.RenderStepped:Connect(function()
        if not isFlying then
            -- Clean up if flying is disabled
            if velocity and velocity.Parent then velocity:Destroy() end
            if gyro and gyro.Parent then gyro:Destroy() end
            return
        end
        
        -- Update gyro orientation to match camera
        gyro.CFrame = camera.CFrame
        
        local moveDirection = Vector3.new(0, 0, 0)
        
        -- Forward/Backward movement in camera direction
        if UserInputService:IsKeyDown(Enum.KeyCode.W) then
            moveDirection = moveDirection + camera.CFrame.LookVector
        end
        if UserInputService:IsKeyDown(Enum.KeyCode.S) then
            moveDirection = moveDirection - camera.CFrame.LookVector
        end
        
        -- Left/Right movement relative to camera
        if UserInputService:IsKeyDown(Enum.KeyCode.A) then
            moveDirection = moveDirection - camera.CFrame.RightVector
        end
        if UserInputService:IsKeyDown(Enum.KeyCode.D) then
            moveDirection = moveDirection + camera.CFrame.RightVector
        end
        
        -- Up/Down movement
        if UserInputService:IsKeyDown(Enum.KeyCode.Space) then
            moveDirection = moveDirection + camera.CFrame.UpVector
        end
        if UserInputService:IsKeyDown(Enum.KeyCode.LeftShift) then
            moveDirection = moveDirection - camera.CFrame.UpVector
        end
        
        -- Normalize movement direction and apply speed
        if moveDirection.Magnitude > 0 then
            moveDirection = moveDirection.Unit
        end
        
        -- Apply velocity with smooth acceleration
        local targetVelocity = moveDirection * (customSpeedValue * 30)
        velocity.Velocity = targetVelocity
        
        -- Disable standard character rotation while flying
        if humanoid then
            humanoid.AutoRotate = false
        end
    end)
    
    -- Clean up when flying is disabled
    local function cleanUpFly()
        if velocity and velocity.Parent then velocity:Destroy() end
        if gyro and gyro.Parent then gyro:Destroy() end
        if humanoid then
            humanoid.AutoRotate = true
        end
    end
    
    -- Connect cleanup to character events
    if player.Character then
        player.Character.Humanoid.Died:Connect(cleanUpFly)
    end
    
    player.CharacterAdded:Connect(function(char)
        if isFlying then
            cleanUpFly()
            wait(0.1)
            activateFly()
        end
    end)
end

-- Anti AFK Function
local function toggleAntiAFK()
    if antiAfkEnabled then
        local virtualUser = game:GetService("VirtualUser")
        player.Idled:Connect(function()
            if antiAfkEnabled then
                virtualUser:Button2Down(Vector2.new(0,0), workspace.CurrentCamera.CFrame)
                wait(1)
                virtualUser:Button2Up(Vector2.new(0,0), workspace.CurrentCamera.CFrame)
            end
        end)
    end
end

-- Additional Button Functionality
FlyButton.MouseButton1Click:Connect(function()
    isFlying = not isFlying
    updateButtonColor(FlyButton, isFlying)
    
    if isFlying then
        activateFly()
        NotificationSystem.new("Fly Enabled", "Current speed: " .. customSpeedValue, 3)
    else
        local fly = player.Character and player.Character:FindFirstChild("HumanoidRootPart"):FindFirstChild("FlyVelocity")
        if fly then fly:Destroy() end
        NotificationSystem.new("Fly Disabled", "Flight mode turned off", 3)
    end
end)

JumpButton.MouseButton1Click:Connect(function()
    infiniteJumpEnabled = not infiniteJumpEnabled
    updateButtonColor(JumpButton, infiniteJumpEnabled)
    
    if infiniteJumpEnabled then
        NotificationSystem.new("Infinite Jump", "You can now jump infinitely", 3)
    else
        NotificationSystem.new("Jump Normal", "Jump system returned to normal", 3)
    end
end)

-- Updated Teleport Button Functionality
TpButton.MouseButton1Click:Connect(function()
    isTpEnabled = not isTpEnabled
    updateButtonColor(TpButton, isTpEnabled)
    
    if isTpEnabled then
        NotificationSystem.new("Click Teleport", "Hold CTRL and click to teleport", 3)
        
        local mouse = player:GetMouse()
        mouse.Button1Down:Connect(function()
            if not isTpEnabled or not player.Character then return end
            
            -- Check if CTRL is being held
            if UserInputService:IsKeyDown(Enum.KeyCode.LeftControl) or UserInputService:IsKeyDown(Enum.KeyCode.RightControl) then
                -- Get the hit position and add a small offset
                local hitPos = mouse.Hit.Position
                local character = player.Character
                local humanoidRootPart = character:FindFirstChild("HumanoidRootPart")
                
                if humanoidRootPart then
                    -- Smooth teleport with small animation
                    local targetCFrame = CFrame.new(hitPos + Vector3.new(0, 3, 0))
                    TweenService:Create(humanoidRootPart, 
                        TweenInfo.new(0.2, Enum.EasingStyle.Quad, Enum.EasingDirection.Out),
                        {CFrame = targetCFrame}
                    ):Play()
                end
            end
        end)
    else
        NotificationSystem.new("Teleport Disabled", "Click teleport turned off", 3)
    end
end)

AntiAfkButton.MouseButton1Click:Connect(function()
    antiAfkEnabled = not antiAfkEnabled
    updateButtonColor(AntiAfkButton, antiAfkEnabled)
    
    if antiAfkEnabled then
        toggleAntiAFK()
        NotificationSystem.new("Anti AFK", "You will not be kicked for being AFK", 3)
    else
        NotificationSystem.new("Anti AFK Disabled", "AFK system returned to normal", 3)
    end
end)

-- ESP Button Functionality
ESPButton.MouseButton1Click:Connect(function()
    espEnabled = not espEnabled
    updateButtonColor(ESPButton, espEnabled)
    
    if espEnabled then
        updateESP()
        NotificationSystem.new("ESP Enabled", "Player ESP activated with extended render distance", 3)
    else
        -- Clear ESP
        for _, esp in pairs(espDrawings) do
            for _, drawing in pairs(esp) do
                drawing:Remove()
            end
        end
        espDrawings = {}
        NotificationSystem.new("ESP Disabled", "ESP system turned off", 3)
    end
end)

-- Handle player joining/leaving for ESP
Players.PlayerAdded:Connect(function(plr)
    if espEnabled then
        createESPForPlayer(plr)
    end
end)

Players.PlayerRemoving:Connect(function(plr)
    if espDrawings[plr.Name] then
        for _, drawing in pairs(espDrawings[plr.Name]) do
            drawing:Remove()
        end
        espDrawings[plr.Name] = nil
    end
end)

-- Infinite Jump
UserInputService.JumpRequest:Connect(function()
    if infiniteJumpEnabled and player.Character and player.Character:FindFirstChild("Humanoid") then
        player.Character.Humanoid:ChangeState(Enum.HumanoidStateType.Jumping)
    end
end)

-- Create FOV Control Frame (between speed control and buttons)
local FOVControlFrame = Instance.new("Frame")
FOVControlFrame.Name = "FOVControlFrame"
FOVControlFrame.Size = UDim2.new(1, -20, 0, 35)
FOVControlFrame.Position = UDim2.new(0, 10, 0, 85) -- Positioned right after speed control
FOVControlFrame.BackgroundColor3 = Color3.fromRGB(25, 25, 25)
FOVControlFrame.BorderSizePixel = 0
FOVControlFrame.Parent = MainFrame

-- FOV Label
local FOVLabel = Instance.new("TextLabel")
FOVLabel.Size = UDim2.new(0.2, 0, 1, 0)
FOVLabel.BackgroundTransparency = 1
FOVLabel.Text = "FOV:"
FOVLabel.TextColor3 = Color3.fromRGB(255, 255, 255)
FOVLabel.TextSize = 14
FOVLabel.Font = Enum.Font.GothamSemibold
FOVLabel.Parent = FOVControlFrame

-- FOV Slider
local FOVSlider = Instance.new("Frame")
FOVSlider.Size = UDim2.new(0.6, 0, 0.3, 0)
FOVSlider.Position = UDim2.new(0.2, 0, 0.35, 0)
FOVSlider.BackgroundColor3 = Color3.fromRGB(40, 40, 40)
FOVSlider.BorderSizePixel = 0
FOVSlider.Parent = FOVControlFrame

-- Slider Fill
local SliderFill = Instance.new("Frame")
SliderFill.Size = UDim2.new(0.5, 0, 1, 0)
SliderFill.BackgroundColor3 = Color3.fromRGB(0, 120, 255)
SliderFill.BorderSizePixel = 0
SliderFill.Name = "Fill"
SliderFill.Parent = FOVSlider

-- FOV Value Label
local FOVValue = Instance.new("TextLabel")
FOVValue.Size = UDim2.new(0.2, 0, 1, 0)
FOVValue.Position = UDim2.new(0.8, 0, 0, 0)
FOVValue.BackgroundTransparency = 1
FOVValue.Text = "70°"
FOVValue.TextColor3 = Color3.fromRGB(255, 255, 255)
FOVValue.TextSize = 14
FOVValue.Font = Enum.Font.GothamSemibold
FOVValue.Parent = FOVControlFrame

-- Add corners
addCorners(FOVControlFrame)
addCorners(FOVSlider)
addCorners(SliderFill)

-- FOV Slider Functionality
local isDragging = false
local defaultFOV = 70
local minFOV = 30
local maxFOV = 120

-- Initialize FOV
camera.FieldOfView = defaultFOV

-- Update FOV display
local function updateFOVDisplay(value)
    FOVValue.Text = math.floor(value) .. "°"
    SliderFill.Size = UDim2.new((value - minFOV) / (maxFOV - minFOV), 0, 1, 0)
    camera.FieldOfView = value
end

-- Slider interaction
FOVSlider.InputBegan:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseButton1 then
        isDragging = true
    end
end)

FOVSlider.InputEnded:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseButton1 then
        isDragging = false
    end
end)

UserInputService.InputChanged:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseMovement and isDragging then
        local sliderPosition = FOVSlider.AbsolutePosition
        local sliderSize = FOVSlider.AbsoluteSize
        local mousePosition = input.Position.X
        
        local relativeX = math.clamp((mousePosition - sliderPosition.X) / sliderSize.X, 0, 1)
        local newFOV = minFOV + (maxFOV - minFOV) * relativeX
        
        updateFOVDisplay(newFOV)
    end
end)

-- Update hide button functionality
local isHidden = false
HideButton.MouseButton1Click:Connect(function()
    isHidden = not isHidden
    ContentFrame.Visible = not isHidden
    FOVControlFrame.Visible = not isHidden
    MainFrame.Size = isHidden and UDim2.new(0, 250, 0, 30) or UDim2.new(0, 250, 0, 400)
    HideButton.Text = isHidden and "+" or "-"
end)

-- Reset FOV when GUI is closed
CloseButton.MouseButton1Click:Connect(function()
    camera.FieldOfView = 70 -- Reset to default FOV
    ScreenGui:Destroy()
end)

-- Aimbot Function
local function getClosestPlayerToMouse()
    local maxDistance = 1000
    local closestPlayer = nil
    local closestDistance = maxDistance
    local mousePos = UserInputService:GetMouseLocation()
    
    for _, otherPlayer in pairs(Players:GetPlayers()) do
        if otherPlayer ~= player and otherPlayer.Character and otherPlayer.Character:FindFirstChild("HumanoidRootPart") 
            and otherPlayer.Character:FindFirstChild("Humanoid") and otherPlayer.Character.Humanoid.Health > 0 then
            
            local screenPos, onScreen = camera:WorldToViewportPoint(otherPlayer.Character.HumanoidRootPart.Position)
            if onScreen then
                local distance = (Vector2.new(screenPos.X, screenPos.Y) - mousePos).Magnitude
                if distance < closestDistance then
                    closestDistance = distance
                    closestPlayer = otherPlayer
                end
            end
        end
    end
    
    return closestPlayer
end

-- Aimbot Update Function
local function updateAimbot()
    if not aimbotEnabled then return end
    
    local target = getClosestPlayerToMouse()
    if not target or not target.Character or not target.Character:FindFirstChild("Head") then return end
    
    local targetPos = target.Character.Head.Position
    local targetCFrame = CFrame.new(camera.CFrame.Position, targetPos)
    
    -- Smooth aim
    camera.CFrame = camera.CFrame:Lerp(targetCFrame, aimbotSmoothness)
end

-- Anti-cheat bypass system
local function bypassAnticheat(target)
    if not target or not target.Character then return end
    
    -- Get character components
    local humanoid = target.Character:FindFirstChild("Humanoid")
    local hrp = target.Character:FindFirstChild("HumanoidRootPart")
    if not humanoid or not hrp then return end
    
    -- Method 1: Legitimate Combat Sequence
    pcall(function()
        for _, v in pairs(game:GetDescendants()) do
            if v:IsA("RemoteEvent") then
                -- Simulate legitimate combat sequence
                v:FireServer("combat_start", target.Character)
                task.wait(0.01)
                v:FireServer("combat_hit", target.Character, hrp.Position)
                task.wait(0.01)
                v:FireServer("combat_damage", target.Character, damageAmount)
            end
        end
    end)
    
    -- Method 2: Weapon State Manipulation
    pcall(function()
        for _, tool in pairs(player.Character:GetChildren()) do
            if tool:IsA("Tool") then
                -- Simulate legitimate weapon use
                tool:Activate()
                task.wait(0.01)
                
                -- Find and trigger all possible damage events
                for _, v in pairs(tool:GetDescendants()) do
                    if v:IsA("RemoteEvent") then
                        v:FireServer(target.Character)
                        v:FireServer(hrp)
                        v:FireServer(hrp.Position)
                        v:FireServer("damage", target.Character)
                    end
                end
                
                tool:Deactivate()
            end
        end
    end)
    
    -- Method 3: Network Timing Manipulation
    pcall(function()
        local remotes = game:GetService("ReplicatedStorage"):GetDescendants()
        for _, remote in pairs(remotes) do
            if remote:IsA("RemoteEvent") then
                -- Simulate legitimate network timing
                task.spawn(function()
                    remote:FireServer(target.Character)
                    task.wait(0.02)
                    remote:FireServer(hrp)
                    task.wait(0.02)
                    remote:FireServer("damage")
                end)
            end
        end
    end)
    
    -- Method 4: Combat System Integration
    pcall(function()
        local combatRemotes = {}
        -- Find combat system remotes
        for _, v in pairs(game:GetDescendants()) do
            if v:IsA("RemoteEvent") and 
               (v.Name:lower():match("combat") or v.Name:lower():match("damage") or v.Name:lower():match("hit")) then
                table.insert(combatRemotes, v)
            end
        end
        
        -- Try to use combat system
        for _, remote in pairs(combatRemotes) do
            remote:FireServer(target.Character)
            remote:FireServer(target.Character, "damage")
            remote:FireServer("hit", target.Character)
            remote:FireServer(target.Character, damageAmount)
        end
    end)
    
    -- Method 5: Touch Interest Bypass
    pcall(function()
        if player.Character and player.Character:FindFirstChildOfClass("Tool") then
            local tool = player.Character:FindFirstChildOfClass("Tool")
            if tool:FindFirstChild("Handle") then
                -- Create temporary hitbox
                local hitbox = Instance.new("Part")
                hitbox.Size = Vector3.new(5, 5, 5)
                hitbox.Transparency = 1
                hitbox.CanCollide = false
                hitbox.CFrame = hrp.CFrame
                hitbox.Parent = workspace
                
                -- Simulate multiple hit points
                local hitPoints = {
                    target.Character.Head,
                    target.Character.HumanoidRootPart,
                    target.Character:FindFirstChild("UpperTorso") or target.Character:FindFirstChild("Torso")
                }
                
                for _, hitPoint in pairs(hitPoints) do
                    if hitPoint then
                        -- Simulate touch with proper timing
                        firetouchinterest(tool.Handle, hitPoint, 0)
                        task.wait(0.01)
                        firetouchinterest(tool.Handle, hitPoint, 1)
                    end
                end
                
                hitbox:Destroy()
            end
        end
    end)
end

-- Kill Aura Variables
local killAuraRange = 50
local lastKillAuraAttack = 0
local killAuraCooldown = 0.1
local maxKillAuraTargets = 5
local damageAmount = 25

-- Cache frequently used services
local virtualInputManager = game:GetService("VirtualInputManager")
local contextActionService = game:GetService("ContextActionService")
local replicatedStorage = game:GetService("ReplicatedStorage")
local runService = game:GetService("RunService")
local tweenService = game:GetService("TweenService")

-- Optimization: Cache more values
local savannahCache = {
    attackRemotes = {},
    hitboxes = {},
    animations = {},
    lastTargets = {},
    lastPositions = {},
    attackAnimations = {},
    successfulPatterns = {}  -- Track which patterns work best
}

-- Optimization: Pre-define common patterns
local attackPatterns = {
    "damage",
    "hit",
    "attack",
    "strike",
    "bite",
    "claw",
    "attack1",
    "attack2",
    "attack3",
    "combat_hit"
}

-- Initialize Savannah Life combat system with optimizations
local function initializeSavannahCombat()
    -- Clear existing cache
    table.clear(savannahCache.attackRemotes)
    table.clear(savannahCache.attackAnimations)
    
    -- Find all possible combat remotes
    for _, v in pairs(game:GetDescendants()) do
        if v:IsA("RemoteEvent") or v:IsA("RemoteFunction") then
            local name = v.Name:lower()
            if name:match("attack") or name:match("hit") or name:match("damage") or 
               name:match("combat") or name:match("hurt") or name:match("strike") then
                table.insert(savannahCache.attackRemotes, v)
            end
        elseif v:IsA("Animation") and 
              (v.Name:lower():match("attack") or v.Name:lower():match("hit") or v.Name:lower():match("strike")) then
            table.insert(savannahCache.attackAnimations, v)
        end
    end
end

-- Optimization: Create a smoother teleport function
local function smoothTeleport(character, targetCFrame)
    if not character or not character:FindFirstChild("HumanoidRootPart") then return end
    
    local hrp = character.HumanoidRootPart
    local originalPos = hrp.CFrame
    
    -- Use tweening for smoother movement
    local tween = tweenService:Create(hrp, 
        TweenInfo.new(0.05, Enum.EasingStyle.Linear), 
        {CFrame = targetCFrame}
    )
    tween:Play()
    tween.Completed:Wait()
    
    return originalPos
end

-- Optimization: Improved hit registration with better timing
local function registerSavannahHit(target)
    if not target or not target.Character then return end
    
    local humanoid = target.Character:FindFirstChild("Humanoid")
    local hrp = target.Character:FindFirstChild("HumanoidRootPart")
    if not humanoid or not hrp then return end

    -- Method 1: Optimized Direct Damage Application
    pcall(function()
        for _, remote in pairs(savannahCache.attackRemotes) do
            if remote:IsA("RemoteEvent") then
                -- Use successful patterns more frequently
                for pattern, success in pairs(savannahCache.successfulPatterns) do
                    if success then
                        remote:FireServer(pattern, target.Character)
                    end
                end
                
                -- Try all patterns
                for _, pattern in ipairs(attackPatterns) do
                    remote:FireServer(pattern, target.Character)
                    remote:FireServer(target.Character, pattern)
                end
            elseif remote:IsA("RemoteFunction") then
                pcall(function()
                    remote:InvokeServer(target.Character)
                    remote:InvokeServer("damage", target.Character)
                end)
            end
        end
    end)

    -- Method 2: Optimized Tool-Based Attack
    if player.Character then
        for _, tool in pairs(player.Character:GetChildren()) do
            if tool:IsA("Tool") then
                pcall(function()
                    tool:Activate()
                    
                    -- Optimize tool remotes
                    for _, v in pairs(tool:GetDescendants()) do
                        if v:IsA("RemoteEvent") then
                            v:FireServer(target.Character)
                            v:FireServer("hit", target.Character)
                        end
                    end

                    if tool:FindFirstChild("Handle") then
                        -- Optimized hitbox
                        local hitbox = Instance.new("Part")
                        hitbox.Size = tool.Handle.Size * hitboxExpansion
                        hitbox.CFrame = hrp.CFrame
                        hitbox.Transparency = 1
                        hitbox.CanCollide = false
                        hitbox.Parent = workspace

                        -- Optimized hit points
                        local hitPoints = {
                            hrp,
                            target.Character.Head,
                            target.Character:FindFirstChild("Torso") or target.Character:FindFirstChild("UpperTorso")
                        }

                        -- Quick hits
                        for _, point in pairs(hitPoints) do
                            if point then
                                firetouchinterest(tool.Handle, point, 0)
                                task.wait(0.01)  -- Reduced wait time
                                firetouchinterest(tool.Handle, point, 1)
                            end
                        end

                        hitbox:Destroy()
                    end
                    
                    tool:Deactivate()
                end)
            end
        end
    end

    -- Method 3: Optimized Animation Attack
    pcall(function()
        if player.Character and player.Character:FindFirstChild("Humanoid") then
            local animator = player.Character.Humanoid:FindFirstChild("Animator")
            if animator then
                -- Use cached animations
                for _, anim in ipairs(savannahCache.attackAnimations) do
                    local animTrack = animator:LoadAnimation(anim)
                    animTrack:Play()
                    task.wait(0.05)  -- Reduced animation time
                    animTrack:Stop()
                end
            end
        end
    end)

    -- Method 4: Optimized Character Interaction
    pcall(function()
        if player.Character and player.Character:FindFirstChild("HumanoidRootPart") then
            -- Use smooth teleport: pick one of five random positions behind the target
            local backPositions = {
                hrp.CFrame * CFrame.new(0, 0, -3), -- directly behind
                hrp.CFrame * CFrame.new(-1.5, 0, -3.5), -- behind left
                hrp.CFrame * CFrame.new(1.5, 0, -3.5), -- behind right
                hrp.CFrame * CFrame.new(0, 0, -5), -- further directly behind
                hrp.CFrame * CFrame.new(0, 0.5, -4), -- slightly above and behind
            }
            local chosenCFrame = backPositions[math.random(1, #backPositions)]
            local originalPos = smoothTeleport(player.Character, chosenCFrame)
            
            -- Quick attacks
            for _, remote in pairs(savannahCache.attackRemotes) do
                if remote:IsA("RemoteEvent") then
                    remote:FireServer(target.Character)
                end
            end
            
            -- Smooth return
            task.wait(returnDelay)
            smoothTeleport(player.Character, originalPos)
        end
    end)
end

-- Improved Smart Auto Hit (Stick & Multi-Hit)
local autoHitTarget = nil
local autoHitOriginalPos = nil
local autoHitSticking = false
local autoHitStickDistance = -2.5 -- Always stick right at the back of the target
local autoHitHitsPerSecond = 2 -- 2 hits per second (once every 0.5 seconds)
local autoHitUnstickKey = Enum.KeyCode.LeftAlt -- Key to unstick
local autoHitSwitchCooldown = 1.5 -- Cooldown before switching to a new target (seconds)
local lastAutoHitTargetSwitch = 0
local autoHitTweenSpeed = 20 -- Studs per second for smooth movement

-- === AUTOHIT MODES ===
local autoHitMode = "Stealth" -- Default mode
local autoHitModes = {"Stealth", "Aggressive"}
local autoHitModeColors = {
    Stealth = Color3.fromRGB(100, 200, 255),
    Aggressive = Color3.fromRGB(255, 90, 90)
}

-- === UI: Add Mode Switch Buttons ===
local function createModeButton(name, parent, pos, color)
    local btn = Instance.new("TextButton")
    btn.Name = name .. "ModeButton"
    btn.Size = UDim2.new(0, 20, 0, 20)
    btn.Position = pos
    btn.BackgroundColor3 = color
    btn.Text = string.sub(name, 1, 1)
    btn.TextColor3 = Color3.fromRGB(255,255,255)
    btn.TextSize = 12
    btn.BorderSizePixel = 0
    btn.ZIndex = 3
    btn.AutoButtonColor = true
    btn.Parent = parent
    return btn
end

-- Find the main autohit button (assume it's named 'AutoHitButton' or similar)
local autoHitButton = nil
for _, obj in pairs(ScreenGui:GetDescendants()) do
    if obj:IsA("TextButton") and obj.Name:lower():find("autohit") then
        autoHitButton = obj
        break
    end
end

if autoHitButton then
    -- Place two small mode buttons behind the main button
    local stealthBtn = createModeButton("Stealth", autoHitButton.Parent, autoHitButton.Position + UDim2.new(0, -25, 0, 0), autoHitModeColors.Stealth)
    local aggressiveBtn = createModeButton("Aggressive", autoHitButton.Parent, autoHitButton.Position + UDim2.new(0, -50, 0, 0), autoHitModeColors.Aggressive)
    
    local function updateModeUI()
        stealthBtn.BackgroundColor3 = (autoHitMode == "Stealth") and autoHitModeColors.Stealth or Color3.fromRGB(60,60,60)
        aggressiveBtn.BackgroundColor3 = (autoHitMode == "Aggressive") and autoHitModeColors.Aggressive or Color3.fromRGB(60,60,60)
        autoHitButton.Text = "AutoHit ("..autoHitMode..")"
    end
    
    stealthBtn.MouseButton1Click:Connect(function()
        autoHitMode = "Stealth"
        updateModeUI()
    end)
    aggressiveBtn.MouseButton1Click:Connect(function()
        autoHitMode = "Aggressive"
        updateModeUI()
    end)
    updateModeUI()
end

-- === AUTOHIT LOGIC WITH MODES ===
local function getRandomStealthDelay()
    -- Human-like: 0.35 to 0.7s, sometimes skip
    if math.random() < 0.13 then
        return math.random(65, 110) / 100 -- 0.65-1.1s (simulate hesitation)
    end
    return math.random(35, 70) / 100
end

local function getRandomAggressiveDelay()
    -- Aggressive but not robotic: 0.1 to 0.25s
    return math.random(10, 25) / 100
end

local function shouldSkipStealthHit()
    -- 10% chance to skip a hit (simulate human error)
    return math.random() < 0.1
end

local function shuffle(tbl)
    -- Fisher-Yates shuffle
    for i = #tbl, 2, -1 do
        local j = math.random(i)
        tbl[i], tbl[j] = tbl[j], tbl[i]
    end
end

local function getTargets()
    local hrp = player.Character and player.Character:FindFirstChild("HumanoidRootPart")
    if not hrp then return {} end
    -- If a player is selected, only return that player if in range and valid
    if selectedAutoHitPlayer and selectedAutoHitPlayer.Character and selectedAutoHitPlayer.Character:FindFirstChild("HumanoidRootPart") and selectedAutoHitPlayer.Character:FindFirstChild("Humanoid") and selectedAutoHitPlayer.Character.Humanoid.Health > 0 then
        local dist = (hrp.Position - selectedAutoHitPlayer.Character.HumanoidRootPart.Position).Magnitude
        if dist <= autoHitRange then
            return {selectedAutoHitPlayer}
        end
    end
    -- Otherwise, return all valid targets
    local targets = {}
    for _, otherPlayer in pairs(Players:GetPlayers()) do
        if otherPlayer ~= player and otherPlayer.Character and otherPlayer.Character:FindFirstChild("HumanoidRootPart") and otherPlayer.Character:FindFirstChild("Humanoid") and otherPlayer.Character.Humanoid.Health > 0 then
            local dist = (hrp.Position - otherPlayer.Character.HumanoidRootPart.Position).Magnitude
            if dist <= autoHitRange then
                table.insert(targets, otherPlayer)
            end
        end
    end
    return targets
end

spawn(function()
    while true do
        if autoHitEnabled then
            local targets = getTargets()
            if #targets > 0 then
                if autoHitMode == "Stealth" then
                    shuffle(targets)
                    for _, target in ipairs(targets) do
                        if shouldSkipStealthHit() then
                            task.wait(getRandomStealthDelay())
                        else
                            registerSavannahHit(target)
                            task.wait(getRandomStealthDelay())
                        end
                    end
                elseif autoHitMode == "Aggressive" then
                    -- Always attack closest first, fast
                    table.sort(targets, function(a,b)
                        local hrp = player.Character and player.Character:FindFirstChild("HumanoidRootPart")
                        return (hrp.Position - a.Character.HumanoidRootPart.Position).Magnitude < (hrp.Position - b.Character.HumanoidRootPart.Position).Magnitude
                    end)
                    for _, target in ipairs(targets) do
                        registerSavannahHit(target)
                        task.wait(getRandomAggressiveDelay())
                    end
                end
            else
                task.wait(0.3)
            end
        else
            task.wait(0.5)
        end
    end
end)

local function findClosestTarget()
    local hrp = player.Character and player.Character:FindFirstChild("HumanoidRootPart")
    if not hrp then return nil end
    local minDist, closest = math.huge, nil
    for _, otherPlayer in pairs(Players:GetPlayers()) do
        if otherPlayer ~= player and otherPlayer.Character and otherPlayer.Character:FindFirstChild("HumanoidRootPart") and otherPlayer.Character:FindFirstChild("Humanoid") and otherPlayer.Character.Humanoid.Health > 0 then
            local dist = (hrp.Position - otherPlayer.Character.HumanoidRootPart.Position).Magnitude
            if dist < minDist and dist <= autoHitRange then
                minDist = dist
                closest = otherPlayer
            end
        end
    end
    return closest
end

local function stickToTarget(target)
    if not player.Character or not player.Character:FindFirstChild("HumanoidRootPart") or not target or not target.Character or not target.Character:FindFirstChild("HumanoidRootPart") then return end
    autoHitSticking = true
    autoHitOriginalPos = player.Character.HumanoidRootPart.CFrame
    NotificationSystem.new("Auto Hit", "Stuck to "..(target.DisplayName or target.Name), 2)
end

local function unstickFromTarget()
    if autoHitSticking and autoHitOriginalPos and player.Character and player.Character:FindFirstChild("HumanoidRootPart") then
        player.Character.HumanoidRootPart.CFrame = autoHitOriginalPos
        NotificationSystem.new("Auto Hit", "Unstuck from target", 2)
    end
    autoHitTarget = nil
    autoHitSticking = false
    autoHitOriginalPos = nil
end

UserInputService.InputBegan:Connect(function(input, gp)
    if input.KeyCode == autoHitUnstickKey and autoHitSticking then
        unstickFromTarget()
    end
end)

local lastAutoHitTick = 0
local lastMissTick = 0
local missChance = 0.08 -- 8% chance to skip a hit for anti-detection
local jitterAmount = 0.6 -- max studs to jitter position
local humanDelayMin, humanDelayMax = 0.02, 0.09 -- random delay between hits
local targetSwitchCooldown = 0.7 -- seconds before switching to new target
local lastAutoHitTargetSwitch = 0

local function applyJitter(cframe)
    local dx = (math.random() - 0.5) * 2 * jitterAmount
    local dy = (math.random() - 0.5) * 2 * (jitterAmount * 0.4)
    local dz = (math.random() - 0.5) * 2 * jitterAmount
    return cframe * CFrame.new(dx, dy, dz)
end

local function updateSavannahAutoHit()
    if not autoHitEnabled then
        if autoHitSticking then unstickFromTarget() end
        return
    end
    if not player.Character or not player.Character:FindFirstChild("HumanoidRootPart") then return end
    local now = tick()
    -- Humanized random delay
    local delay = math.random() * (humanDelayMax - humanDelayMin) + humanDelayMin
    if now - lastAutoHitTick < delay then return end
    lastAutoHitTick = now

    -- Only attack one target at a time, with switch cooldown
    if (not autoHitSticking or not autoHitTarget) or (autoHitTarget and now - lastAutoHitTargetSwitch > targetSwitchCooldown and (not autoHitTarget.Character or not autoHitTarget.Character:FindFirstChild("Humanoid") or autoHitTarget.Character.Humanoid.Health <= 0)) then
        local target = findClosestTarget()
        if target then
            autoHitTarget = target
            stickToTarget(target)
            lastAutoHitTargetSwitch = now
        else
            unstickFromTarget()
            return
        end
    end

    -- Validate target
    local targetHRP = autoHitTarget.Character and autoHitTarget.Character:FindFirstChild("HumanoidRootPart")
    local targetHum = autoHitTarget.Character and autoHitTarget.Character:FindFirstChild("Humanoid")
    if not targetHRP or not targetHum or targetHum.Health <= 0 then
        unstickFromTarget()
        return
    end

    -- Smart stick: smoothly follow optimal side
    local hrp = player.Character.HumanoidRootPart
    local targetVelocity = targetHRP.Velocity.Magnitude
    local speedThreshold = 3 -- studs/sec, adjust as needed
    local offsetCFrame
    if targetVelocity <= speedThreshold then
        -- Standing or slow: pick random front/left/right (not back)
        local choices = {
            CFrame.new(0, 0, -autoHitStickDistance), -- front
            CFrame.new(-autoHitStickDistance, 0, 0), -- left
            CFrame.new(autoHitStickDistance, 0, 0), -- right
        }
        local choice = choices[math.random(1, #choices)]
        offsetCFrame = targetHRP.CFrame * choice
    else
        -- Moving fast: teleport behind as before
        local backOffset = Vector3.new(0, 0, autoHitStickDistance)
        offsetCFrame = targetHRP.CFrame * CFrame.new(backOffset)
    end
    -- Apply jitter for anti-detection
    local jitteredCFrame = applyJitter(offsetCFrame)
    hrp.CFrame = CFrame.new(jitteredCFrame.Position, targetHRP.Position)

    -- Only attack if close enough and behind
    local distance = (hrp.Position - targetHRP.Position).Magnitude
    local rel = hrp.CFrame:ToObjectSpace(targetHRP.CFrame)
    -- Add random miss chance for anti-detection
    if rel.Z > 0 and distance < autoHitStickDistance + 1.5 then
        if math.random() > missChance then
            registerSavannahHit(autoHitTarget)
            -- Visual feedback (optional)
            pcall(function()
                local hitEffect = Instance.new("Part")
                hitEffect.Size = Vector3.new(0.5, 0.5, 0.5)
                hitEffect.Transparency = 0.9
                hitEffect.BrickColor = BrickColor.new("Really red")
                hitEffect.Material = Enum.Material.Neon
                hitEffect.CFrame = targetHRP.CFrame
                hitEffect.Anchored = true
                hitEffect.CanCollide = false
                hitEffect.Parent = workspace
                game:GetService("Debris"):AddItem(hitEffect, 0.1)
                tweenService:Create(hitEffect, TweenInfo.new(0.1), {
                    Size = Vector3.new(1.5, 1.5, 1.5),
                    Transparency = 1
                }):Play()
            end)
        end
    end
end

-- Hotkey "L" toggles AutoHit ON/OFF (same as On/Off button)
UserInputService.InputBegan:Connect(function(input, gp)
    if input.KeyCode == Enum.KeyCode.L and not gp then
        autoHitEnabled = not autoHitEnabled
        updateButtonColor(AutoHitButton, autoHitEnabled)
        if autoHitEnabled then
            NotificationSystem.new("Auto Hit Enabled", "Range: " .. autoHitRange .. " studs (Hotkey)", 3)
        else
            NotificationSystem.new("Auto Hit Disabled", "Auto hitting stopped (Hotkey)", 3)
        end
    end
end)

-- Initialize the system
initializeSavannahCombat()

-- Create Auto Hit button
local AutoHitButton = createButton("Auto Hit", UDim2.new(0.05, 0, 0.93, 0))

-- === ENSURE MODE BUTTONS ARE CREATED AND VISIBLE ===
local autoHitModeColors = {
    Stealth = Color3.fromRGB(100, 200, 255),
    Aggressive = Color3.fromRGB(255, 90, 90)
}
local function createModeButton(name, parent, pos, color)
    local btn = Instance.new("TextButton")
    btn.Name = name .. "ModeButton"
    btn.Size = UDim2.new(0, 20, 0, 20)
    btn.Position = pos
    btn.BackgroundColor3 = color
    btn.Text = string.sub(name, 1, 1)
    btn.TextColor3 = Color3.fromRGB(255,255,255)
    btn.TextSize = 12
    btn.BorderSizePixel = 0
    btn.ZIndex = AutoHitButton.ZIndex + 2
    btn.AutoButtonColor = true
    btn.Parent = parent
    return btn
end

-- Place two small mode buttons to the left of the main button
local stealthBtn = createModeButton("Stealth", AutoHitButton.Parent, AutoHitButton.Position + UDim2.new(0, -25, 0, 0), autoHitModeColors.Stealth)
local aggressiveBtn = createModeButton("Aggressive", AutoHitButton.Parent, AutoHitButton.Position + UDim2.new(0, -50, 0, 0), autoHitModeColors.Aggressive)

-- === SELECT TARGET BUTTON BELOW MODE BUTTONS ===
local selectedAutoHitPlayer = nil
local selectTargetBtn = Instance.new("TextButton")
selectTargetBtn.Name = "SelectTargetButton"
selectTargetBtn.Size = UDim2.new(0, 110, 0, 22)
selectTargetBtn.Position = AutoHitButton.Position + UDim2.new(0, -50, 0, 30)
selectTargetBtn.BackgroundColor3 = Color3.fromRGB(80, 80, 200)
selectTargetBtn.Text = "Select Target"
selectTargetBtn.TextColor3 = Color3.fromRGB(255,255,255)
selectTargetBtn.TextSize = 13
selectTargetBtn.BorderSizePixel = 0
selectTargetBtn.ZIndex = AutoHitButton.ZIndex + 3
selectTargetBtn.Parent = AutoHitButton.Parent

local selectingTarget = false
local highlightInstance = nil

local function clearHighlight()
    if highlightInstance then
        highlightInstance:Destroy()
        highlightInstance = nil
    end
end

local selectedHighlights = {}
local function clearSelectedHighlights()
    for _, h in pairs(selectedHighlights) do
        if h and h.Parent then h:Destroy() end
    end
    selectedHighlights = {}
end

local function setSelectedPlayer(plr)
    selectedAutoHitPlayer = plr
    clearSelectedHighlights()
    if plr and plr.Character then
        selectTargetBtn.Text = "Target: "..(plr.DisplayName or plr.Name)
        selectTargetBtn.BackgroundColor3 = Color3.fromRGB(60, 200, 100)
        -- Add persistent white highlight to all parts
        for _, part in ipairs(plr.Character:GetDescendants()) do
            if part:IsA("BasePart") then
                local h = Instance.new("Highlight")
                h.Adornee = part
                h.FillColor = Color3.fromRGB(255,255,255)
                h.OutlineColor = Color3.fromRGB(255,255,255)
                h.DepthMode = Enum.HighlightDepthMode.AlwaysOnTop
                h.Parent = part
                table.insert(selectedHighlights, h)
            end
        end
    else
        selectTargetBtn.Text = "Select Target"
        selectTargetBtn.BackgroundColor3 = Color3.fromRGB(80, 80, 200)
    end
end
setSelectedPlayer(nil)

-- Username input and confirm button: always visible, but created only when selectTargetBtn is parented
spawn(function()
    while not (selectTargetBtn and selectTargetBtn.Parent) do wait(0.1) end
    -- Prevent duplicate UI
    if selectTargetBtn.Parent:FindFirstChild("UsernameInput") then return end

    -- Username input box below the selectTargetBtn
    local usernameInput = Instance.new("TextBox")
    usernameInput.Name = "UsernameInput"
    usernameInput.Size = UDim2.new(0, 110, 0, 22)
    usernameInput.Position = selectTargetBtn.Position + UDim2.new(0, 0, 0, 26)
    usernameInput.BackgroundColor3 = Color3.fromRGB(40, 40, 40)
    usernameInput.Text = "Type username..."
    usernameInput.TextColor3 = Color3.fromRGB(255,255,255)
    usernameInput.TextSize = 13
    usernameInput.BorderSizePixel = 0
    usernameInput.ZIndex = selectTargetBtn.ZIndex + 1
    usernameInput.Parent = selectTargetBtn.Parent

    -- Confirm button below the username input
    local confirmBtn = Instance.new("TextButton")
    confirmBtn.Name = "ConfirmUsernameButton"
    confirmBtn.Size = UDim2.new(0, 60, 0, 20)
    confirmBtn.Position = usernameInput.Position + UDim2.new(0, 25, 0, 24)
    confirmBtn.BackgroundColor3 = Color3.fromRGB(60, 200, 100)
    confirmBtn.Text = "Confirm"
    confirmBtn.TextColor3 = Color3.fromRGB(255,255,255)
    confirmBtn.TextSize = 12
    confirmBtn.BorderSizePixel = 0
    confirmBtn.ZIndex = usernameInput.ZIndex + 1
    confirmBtn.Parent = usernameInput.Parent

    -- Helper to find player by username or display name (case-insensitive)
    local function findPlayerByName(name)
        name = name:lower()
        for _, plr in pairs(Players:GetPlayers()) do
            if plr.Name:lower() == name or (plr.DisplayName and plr.DisplayName:lower() == name) then
                return plr
            end
        end
        return nil
    end

    -- Teleport to target function
    local function teleportToTarget(plr)
        if plr and plr.Character and plr.Character:FindFirstChild("HumanoidRootPart") and player.Character and player.Character:FindFirstChild("HumanoidRootPart") then
            player.Character.HumanoidRootPart.CFrame = plr.Character.HumanoidRootPart.CFrame * CFrame.new(0, 0, -2.5)
        end
    end

    -- Confirm button logic
    confirmBtn.MouseButton1Click:Connect(function()
        local inputName = usernameInput.Text:lower()
        local found = findPlayerByName(inputName)
        if found then
            setSelectedPlayer(found)
            teleportToTarget(found)
            autoHitEnabled = true -- Enable AutoHit if a valid player is selected
            NotificationSystem.new("AutoHit", "Selected: "..(found.DisplayName or found.Name), 2)
        else
            NotificationSystem.new("AutoHit", "No player found for '"..usernameInput.Text.."'", 2)
            setSelectedPlayer(nil)
        end
    end)

    -- Optional: Enter key triggers confirm
    usernameInput.FocusLost:Connect(function(enterPressed)
        if enterPressed then
            confirmBtn:MouseButton1Click()
        end
    end)
end)

local function getPlayerFromPart(part)
    if not part then return nil end
    for _, plr in pairs(Players:GetPlayers()) do
        if plr ~= player and plr.Character and part:IsDescendantOf(plr.Character) then
            return plr
        end
    end
    return nil
end

local activeHighlights = {}
local function clearAllHighlights()
    for _, h in pairs(activeHighlights) do
        if h and h.Parent then h:Destroy() end
    end
    activeHighlights = {}
end

selectTargetBtn.MouseButton1Click:Connect(function()
    selectingTarget = true
    setSelectedPlayer(nil)
    NotificationSystem.new("AutoHit", "Click any player to select as target", 2)
    spawn(function()
        while selectingTarget do
            local mouse = player:GetMouse()
            local target = mouse.Target
            local hoveredPlayer = getPlayerFromPart(target)
            clearAllHighlights()
            if hoveredPlayer and hoveredPlayer.Character then
                -- Highlight all parts of the player's character
                for _, part in ipairs(hoveredPlayer.Character:GetDescendants()) do
                    if part:IsA("BasePart") then
                        local h = Instance.new("Highlight")
                        h.Adornee = part
                        h.FillColor = Color3.fromRGB(255, 255, 255)
                        h.OutlineColor = Color3.fromRGB(255, 255, 255)
                        h.DepthMode = Enum.HighlightDepthMode.AlwaysOnTop
                        h.Parent = part
                        table.insert(activeHighlights, h)
                    end
                end
            end
            task.wait(0.03)
        end
        clearAllHighlights()
    end)
end)

UserInputService.InputBegan:Connect(function(input, gp)
    if selectingTarget then
        local mouse = player:GetMouse()
        if input.UserInputType == Enum.UserInputType.MouseButton1 then
            local hoveredPlayer = getPlayerFromPart(mouse.Target)
            if hoveredPlayer then
                setSelectedPlayer(hoveredPlayer)
                NotificationSystem.new("AutoHit", "Selected: "..(hoveredPlayer.DisplayName or hoveredPlayer.Name), 2)
                selectingTarget = false
            else
                NotificationSystem.new("AutoHit", "No player selected", 2)
                selectingTarget = false
            end
        elseif input.UserInputType == Enum.UserInputType.MouseButton2 or input.KeyCode == Enum.KeyCode.Escape then
            NotificationSystem.new("AutoHit", "Selection cancelled", 2)
            selectingTarget = false
        end
    end
end)

local function updateModeUI()
    stealthBtn.BackgroundColor3 = (autoHitMode == "Stealth") and autoHitModeColors.Stealth or Color3.fromRGB(60,60,60)
    aggressiveBtn.BackgroundColor3 = (autoHitMode == "Aggressive") and autoHitModeColors.Aggressive or Color3.fromRGB(60,60,60)
    AutoHitButton.Text = "AutoHit ("..autoHitMode..")"
end

stealthBtn.MouseButton1Click:Connect(function()
    autoHitMode = "Stealth"
    updateModeUI()
end)
aggressiveBtn.MouseButton1Click:Connect(function()
    autoHitMode = "Aggressive"
    updateModeUI()
end)
updateModeUI()

-- Create tiny ON/OFF buttons below AutoHitButton
local function createTinyButton(name, parent, pos)
    local btn = Instance.new("TextButton")
    btn.Name = name .. "Button"
    btn.Size = UDim2.new(0.18, 0, 0.4, 0) -- Small width, short height
    btn.Position = pos
    btn.BackgroundColor3 = name == "On" and Color3.fromRGB(0, 200, 0) or Color3.fromRGB(200, 0, 0)
    btn.Text = name
    btn.TextColor3 = Color3.fromRGB(255,255,255)
    btn.TextSize = 10
    btn.Font = Enum.Font.GothamBold
    btn.BorderSizePixel = 0
    btn.ZIndex = AutoHitButton.ZIndex + 1
    btn.Parent = parent
    local corner = Instance.new("UICorner")
    corner.CornerRadius = UDim.new(0, 4)
    corner.Parent = btn
    return btn
end

-- Position tiny buttons just below and behind the main AutoHitButton
local yOffset = 0.36 -- Slightly below main button
local OnButton = createTinyButton("On", AutoHitButton, UDim2.new(0, 0, 1 + yOffset, 0))
local OffButton = createTinyButton("Off", AutoHitButton, UDim2.new(0.82, 0, 1 + yOffset, 0))

OnButton.MouseButton1Click:Connect(function()
    autoHitEnabled = true
    updateButtonColor(AutoHitButton, true)
    NotificationSystem.new("Auto Hit Enabled", "Range: " .. autoHitRange .. " studs", 3)
end)
OffButton.MouseButton1Click:Connect(function()
    autoHitEnabled = false
    updateButtonColor(AutoHitButton, false)
    NotificationSystem.new("Auto Hit Disabled", "Auto hitting stopped", 3)
end)

-- Auto Hit button functionality
AutoHitButton.MouseButton1Click:Connect(function()
    autoHitEnabled = not autoHitEnabled
    updateButtonColor(AutoHitButton, autoHitEnabled)
    
    if autoHitEnabled then
        NotificationSystem.new("Auto Hit Enabled", "Range: " .. autoHitRange .. " studs", 3)
    else
        NotificationSystem.new("Auto Hit Disabled", "Auto hitting stopped", 3)
    end
end)

-- Connect Auto Hit to RunService
RunService.Heartbeat:Connect(updateSavannahAutoHit)

-- Button Functionality for Aimbot
AimbotButton.MouseButton1Click:Connect(function()
    aimbotEnabled = not aimbotEnabled
    updateButtonColor(AimbotButton, aimbotEnabled)
    
    if aimbotEnabled then
        NotificationSystem.new("Aimbot Enabled", "Press right mouse button to lock onto targets", 3)
    else
        NotificationSystem.new("Aimbot Disabled", "Aiming returned to normal", 3)
    end
end)

-- Button Functionality for Kill Aura
KillAuraButton.MouseButton1Click:Connect(function()
    killAuraEnabled = not killAuraEnabled
    updateButtonColor(KillAuraButton, killAuraEnabled)
    
    if killAuraEnabled then
        NotificationSystem.new("Kill Aura Enabled", "Range: " .. killAuraRange .. " studs", 3)
    else
        NotificationSystem.new("Kill Aura Disabled", "Attack range returned to normal", 3)
    end
end)

-- Right Mouse Button check for Aimbot
UserInputService.InputBegan:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseButton2 then
        if aimbotEnabled then
            aimbotTarget = getClosestPlayerToMouse()
        end
    end
end)

UserInputService.InputEnded:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseButton2 then
        aimbotTarget = nil
    end
end) 
