-- Player ESP Script with Team Colors and Health Bars
-- Place this script in a LocalScript inside StarterPlayerScripts

local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local LocalPlayer = Players.LocalPlayer
local Camera = workspace.CurrentCamera

-- ESP Configuration
local ESP = {
    Enabled = true,
    ShowTeamColor = true,
    ShowHealthBar = true,
    TextSize = 14,
    BoxThickness = 2,
    BoxTransparency = 0.5,
    TextTransparency = 0,
    HealthBarThickness = 2,
    HealthBarWidth = 50,
    HealthBarHeight = 5,
    MaxDistance = 5000, -- Increased max distance to 5000 studs
}

-- Storage for ESP drawings
local ESPObjects = {}

-- Function to create ESP objects for a player
local function CreateESPForPlayer(player)
    if player == LocalPlayer then return end
    
    local espObject = {}
    
    -- Box outline for player
    espObject.Box = Drawing.new("Square")
    espObject.Box.Visible = false
    espObject.Box.Thickness = ESP.BoxThickness
    espObject.Box.Transparency = ESP.BoxTransparency
    espObject.Box.Filled = false
    
    -- Player name and health text
    espObject.Text = Drawing.new("Text")
    espObject.Text.Visible = false
    espObject.Text.Size = ESP.TextSize
    espObject.Text.Center = true
    espObject.Text.Outline = true
    espObject.Text.Transparency = ESP.TextTransparency
    
    -- Health bar background (gray)
    espObject.HealthBG = Drawing.new("Square")
    espObject.HealthBG.Visible = false
    espObject.HealthBG.Thickness = ESP.HealthBarThickness
    espObject.HealthBG.Filled = true
    espObject.HealthBG.Color = Color3.fromRGB(100, 100, 100)
    espObject.HealthBG.Transparency = 0.7
    
    -- Health bar fill (green to red depending on health)
    espObject.HealthBar = Drawing.new("Square")
    espObject.HealthBar.Visible = false
    espObject.HealthBar.Thickness = ESP.HealthBarThickness
    espObject.HealthBar.Filled = true
    espObject.HealthBar.Transparency = 0.7
    
    ESPObjects[player] = espObject
    
    -- Clear ESP when player leaves
    player.AncestryChanged:Connect(function()
        if not player:IsDescendantOf(game) then
            if ESPObjects[player] then
                for _, object in pairs(ESPObjects[player]) do
                    object:Remove()
                end
                ESPObjects[player] = nil
            end
        end
    end)
end

-- Create ESP for all existing players
for _, player in pairs(Players:GetPlayers()) do
    if player ~= LocalPlayer then
        CreateESPForPlayer(player)
    end
end

-- Create ESP for new players who join
Players.PlayerAdded:Connect(CreateESPForPlayer)

-- Clear ESP for players who leave
Players.PlayerRemoving:Connect(function(player)
    if ESPObjects[player] then
        for _, object in pairs(ESPObjects[player]) do
            object:Remove()
        end
        ESPObjects[player] = nil
    end
end)

-- Function to get player team color
local function GetPlayerTeamColor(player)
    if player.Team then
        return player.TeamColor.Color
    end
    return Color3.fromRGB(255, 255, 255) -- White for no team
end

-- Function to get health color based on percentage
local function GetHealthColor(health, maxHealth)
    local healthPercent = health / maxHealth
    
    -- Gradient from red to green based on health percentage
    local r = math.clamp(2 * (1 - healthPercent), 0, 1)
    local g = math.clamp(2 * healthPercent, 0, 1)
    
    return Color3.new(r, g, 0)
end

-- Update ESP on each frame
RunService.RenderStepped:Connect(function()
    if not ESP.Enabled then
        -- Hide all ESP elements if disabled
        for _, espObject in pairs(ESPObjects) do
            for _, drawing in pairs(espObject) do
                drawing.Visible = false
            end
        end
        return
    end
    
    for player, espObject in pairs(ESPObjects) do
        -- Get character and make sure it's valid
        local character = player.Character
        if not character or not character:FindFirstChild("HumanoidRootPart") or not character:FindFirstChild("Humanoid") then
            for _, drawing in pairs(espObject) do
                drawing.Visible = false
            end
            continue
        end
        
        local humanoidRootPart = character:FindFirstChild("HumanoidRootPart")
        local humanoid = character:FindFirstChild("Humanoid")
        
        -- Get health information
        local health = humanoid.Health
        local maxHealth = humanoid.MaxHealth
        
        -- Check if character is on screen and within max distance
        local vector, onScreen = Camera:WorldToViewportPoint(humanoidRootPart.Position)
        local distance = (humanoidRootPart.Position - Camera.CFrame.Position).Magnitude
        
        -- Only show ESP if player is on screen and within max distance
        if onScreen and distance <= ESP.MaxDistance then
            -- Get player size for box ESP (using character bounds)
            local topPosition = humanoidRootPart.Position + Vector3.new(0, 3, 0)
            local bottomPosition = humanoidRootPart.Position - Vector3.new(0, 3, 0)
            
            local topVector = Camera:WorldToViewportPoint(topPosition)
            local bottomVector = Camera:WorldToViewportPoint(bottomPosition)
            
            local height = math.abs(topVector.Y - bottomVector.Y)
            local width = height * 0.6
            
            local boxPosition = Vector2.new(vector.X - width / 2, vector.Y - height / 2)
            local boxSize = Vector2.new(width, height)
            
            -- Update box
            espObject.Box.Position = boxPosition
            espObject.Box.Size = boxSize
            espObject.Box.Color = GetPlayerTeamColor(player)
            espObject.Box.Visible = true
            
            -- Update player name and health text
            espObject.Text.Position = Vector2.new(vector.X, boxPosition.Y - 20)
            espObject.Text.Text = string.format("%s [%.0f%%]", player.Name, (health / maxHealth) * 100)
            espObject.Text.Color = GetPlayerTeamColor(player)
            espObject.Text.Visible = true
            
            -- Update health bar background
            espObject.HealthBG.Position = Vector2.new(vector.X - ESP.HealthBarWidth / 2, boxPosition.Y - 10)
            espObject.HealthBG.Size = Vector2.new(ESP.HealthBarWidth, ESP.HealthBarHeight)
            espObject.HealthBG.Visible = ESP.ShowHealthBar
            
            -- Update health bar fill
            local healthPercent = health / maxHealth
            espObject.HealthBar.Position = Vector2.new(vector.X - ESP.HealthBarWidth / 2, boxPosition.Y - 10)
            espObject.HealthBar.Size = Vector2.new(ESP.HealthBarWidth * healthPercent, ESP.HealthBarHeight)
            espObject.HealthBar.Color = GetHealthColor(health, maxHealth)
            espObject.HealthBar.Visible = ESP.ShowHealthBar
        else
            -- Hide ESP elements if player is off screen or too far
            for _, drawing in pairs(espObject) do
                drawing.Visible = false
            end
        end
    end
end)

-- Add toggle functionality (can be bound to a key)
local UserInputService = game:GetService("UserInputService")

UserInputService.InputBegan:Connect(function(input)
    if input.KeyCode == Enum.KeyCode.RightAlt then
        ESP.Enabled = not ESP.Enabled
    end
end)

-- Instructions on how to use this script
print("ESP Script loaded! Press Right Alt to toggle ESP.")
print("ESP shows player boxes by team color and health bars.")
print("Maximum ESP rendering distance: 5000 studs")
