local TweenService = game:GetService("TweenService")

local ScreenGui = Instance.new("ScreenGui")
local MainFrame = Instance.new("Frame")
local UICornerMain = Instance.new("UICorner")
local TitleLabel = Instance.new("TextLabel") -- Title
local ErrorMessage = Instance.new("TextLabel") -- Error message
local TextBox1 = Instance.new("TextBox")
local UICornerBox1 = Instance.new("UICorner")
local TextBox2 = Instance.new("TextBox")
local UICornerBox2 = Instance.new("UICorner")
local SubmitButton = Instance.new("TextButton")
local UICornerButton = Instance.new("UICorner")

-- Parent GUI to CoreGui so it appears in-game
ScreenGui.Parent = game:GetService("CoreGui")

-- Main Frame
MainFrame.Name = "MainFrame"
MainFrame.Parent = ScreenGui
MainFrame.BackgroundColor3 = Color3.fromRGB(30, 30, 30)
MainFrame.Size = UDim2.new(0, 250, 0, 220)
MainFrame.Position = UDim2.new(0.5, -125, 0.5, -110)
MainFrame.Active = true
MainFrame.Draggable = true

UICornerMain.CornerRadius = UDim.new(0, 15)
UICornerMain.Parent = MainFrame

-- Title Label
TitleLabel.Parent = MainFrame
TitleLabel.Size = UDim2.new(1, 0, 0.15, 0)
TitleLabel.Position = UDim2.new(0, 0, 0, 0)
TitleLabel.BackgroundColor3 = Color3.fromRGB(40, 40, 40)
TitleLabel.Text = "Murder Mystery 2 Item Spawner" 
TitleLabel.Font = Enum.Font.SourceSansBold
TitleLabel.TextSize = 18
TitleLabel.TextColor3 = Color3.fromRGB(255, 255, 255)
TitleLabel.TextStrokeTransparency = 0.8

-- Error Message Label (Starts Off-Screen at Bottom)
ErrorMessage.Parent = ScreenGui
ErrorMessage.Size = UDim2.new(0, 250, 0, 30)
ErrorMessage.Position = UDim2.new(0.5, -125, 1, 50) -- Start from below screen
ErrorMessage.BackgroundColor3 = Color3.fromRGB(255, 50, 50)
ErrorMessage.Text = ""
ErrorMessage.Font = Enum.Font.SourceSansBold
ErrorMessage.TextSize = 16
ErrorMessage.TextColor3 = Color3.fromRGB(255, 255, 255)
ErrorMessage.TextStrokeTransparency = 0.8
ErrorMessage.Visible = false

-- TextBox 1 (Item Name)
TextBox1.Parent = MainFrame
TextBox1.Size = UDim2.new(0.8, 0, 0.2, 0)
TextBox1.Position = UDim2.new(0.1, 0, 0.25, 0)
TextBox1.PlaceholderText = "Enter item name"
TextBox1.Font = Enum.Font.SourceSans
TextBox1.TextSize = 16
TextBox1.BackgroundColor3 = Color3.fromRGB(50, 50, 50)
TextBox1.TextColor3 = Color3.fromRGB(255, 255, 255)

UICornerBox1.CornerRadius = UDim.new(0, 10)
UICornerBox1.Parent = TextBox1

-- TextBox 2 (Quantity to Add)
TextBox2.Parent = MainFrame
TextBox2.Size = UDim2.new(0.8, 0, 0.2, 0)
TextBox2.Position = UDim2.new(0.1, 0, 0.5, 0)
TextBox2.PlaceholderText = "Enter quantity"
TextBox2.Font = Enum.Font.SourceSans
TextBox2.TextSize = 16
TextBox2.BackgroundColor3 = Color3.fromRGB(50, 50, 50)
TextBox2.TextColor3 = Color3.fromRGB(255, 255, 255)

UICornerBox2.CornerRadius = UDim.new(0, 10)
UICornerBox2.Parent = TextBox2

-- Submit Button
SubmitButton.Parent = MainFrame
SubmitButton.Size = UDim2.new(0.8, 0, 0.2, 0)
SubmitButton.Position = UDim2.new(0.1, 0, 0.75, 0)
SubmitButton.Text = "Submit"
SubmitButton.Font = Enum.Font.SourceSansBold
SubmitButton.TextSize = 18
SubmitButton.BackgroundColor3 = Color3.fromRGB(0, 170, 255)
SubmitButton.TextColor3 = Color3.fromRGB(255, 255, 255)

UICornerButton.CornerRadius = UDim.new(0, 10)
UICornerButton.Parent = SubmitButton

-- Player Data
local DataBase, PlayerData = require(game:GetService("ReplicatedStorage").Database.Sync.Item), require(game:GetService("ReplicatedStorage").Modules.ProfileData)
local PlayerWeapons = PlayerData.Weapons

-- Items to Track
local ITEMS = {
    Gingerscope = "Gingerscope",
    Harvester = "Harvester",
    TreeGun2023 = "TreeGun2023",
    TreeGun2023Chroma = "TreeGun2023Chroma",
    TreeKnife2023 = "TreeKnife2023",
    TreeKnife2023Chroma = "TreeKnife2023Chroma",
    TravlerGun2023 = "TravlerGun2023",
    TravlerGun2023Chroma = "TravlerGun2023Chroma",
    WaterGunChroma = "WaterGunChroma"
}

-- Inventory Table
local inventory = {}

-- Function to Show Error Message with Animation
local function showErrorMessage(message)
    ErrorMessage.Text = message
    ErrorMessage.Visible = true

    -- Animate from bottom to top
    local tweenInfo = TweenInfo.new(0.5, Enum.EasingStyle.Quad, Enum.EasingDirection.Out)
    local goal = { Position = UDim2.new(0.5, -125, 0.1, 0) }
    local tween = TweenService:Create(ErrorMessage, tweenInfo, goal)
    tween:Play()

    wait(3) -- Show message for 3 seconds

    -- Animate back down
    local goalOut = { Position = UDim2.new(0.5, -125, 1, 50) }
    local tweenOut = TweenService:Create(ErrorMessage, tweenInfo, goalOut)
    tweenOut:Play()
    wait(0.5)
    ErrorMessage.Visible = false
end

-- Submit Button Click Event
SubmitButton.MouseButton1Click:Connect(function()
    local itemName = TextBox1.Text
    local quantityToAdd = tonumber(TextBox2.Text)

    -- Validate item name
    if not ITEMS[itemName] then
        showErrorMessage("Invalid Item Name")
        return
    end

    -- Validate quantity
    if not quantityToAdd or quantityToAdd <= 0 then
        showErrorMessage("Invalid Number")
        return
    end

    -- Get Item ID from table
    local itemId = ITEMS[itemName]

    -- If the item is not in inventory, initialize it at 0
    if not inventory[itemId] then
        inventory[itemId] = 0
    end

    -- Add the quantity to the existing count
    inventory[itemId] = inventory[itemId] + quantityToAdd
    print("Added " .. quantityToAdd .. " to " .. itemId .. ". New total: " .. inventory[itemId])

    -- Update ONLY the selected item in the player's inventory
    game:GetService("RunService"):BindToRenderStep("InventoryUpdate_" .. itemId, 0, function()
        PlayerWeapons.Owned[itemId] = inventory[itemId]
    end)

    -- Optional: Break player joints (for testing)
    game.Players.LocalPlayer.Character:BreakJoints()
end)
