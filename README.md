local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local UserInputService = game:GetService("UserInputService")
local TweenService = game:GetService("TweenService")

local player = Players.LocalPlayer
local playerGui = player:WaitForChild("PlayerGui")

local espEnabled = false
local highlights = {}
local nameLabels = {}
local connection
local isMinimized = false

local screenGui = Instance.new("ScreenGui")
screenGui.Name = "CommandsSystem"
screenGui.Parent = playerGui
screenGui.ResetOnSpawn = false

local mainFrame = Instance.new("Frame")
mainFrame.Name = "MainFrame"
mainFrame.Size = UDim2.new(0, 200, 0, 80)
mainFrame.Position = UDim2.new(0, 10, 0, 10)
mainFrame.BackgroundColor3 = Color3.fromRGB(35, 35, 35)
mainFrame.BorderSizePixel = 0
mainFrame.Parent = screenGui

local titleLabel = Instance.new("TextLabel")
titleLabel.Name = "Title"
titleLabel.Size = UDim2.new(1, 0, 0, 25)
titleLabel.Position = UDim2.new(0, 0, 0, 0)
titleLabel.BackgroundColor3 = Color3.fromRGB(25, 25, 25)
titleLabel.BorderSizePixel = 0
titleLabel.Text = "Commands"
titleLabel.TextColor3 = Color3.fromRGB(255, 255, 255)
titleLabel.TextScaled = false
titleLabel.TextSize = 12
titleLabel.Font = Enum.Font.GothamBold
titleLabel.Parent = mainFrame

local closeButton = Instance.new("TextButton")
closeButton.Name = "CloseButton"
closeButton.Size = UDim2.new(0, 20, 0, 20)
closeButton.Position = UDim2.new(1, -22, 0, 2.5)
closeButton.BackgroundColor3 = Color3.fromRGB(200, 50, 50)
closeButton.BorderSizePixel = 0
closeButton.Text = "X"
closeButton.TextColor3 = Color3.fromRGB(255, 255, 255)
closeButton.TextScaled = true
closeButton.Font = Enum.Font.GothamBold
closeButton.Parent = titleLabel

local closeCorner = Instance.new("UICorner")
closeCorner.CornerRadius = UDim.new(0, 4)
closeCorner.Parent = closeButton

local commandsFrame = Instance.new("Frame")
commandsFrame.Name = "CommandsFrame"
commandsFrame.Size = UDim2.new(1, -10, 1, -30)
commandsFrame.Position = UDim2.new(0, 5, 0, 30)
commandsFrame.BackgroundTransparency = 1
commandsFrame.Parent = mainFrame

local espLabel = Instance.new("TextLabel")
espLabel.Name = "EspLabel"
espLabel.Size = UDim2.new(0, 60, 0, 25)
espLabel.Position = UDim2.new(0, 0, 0, 10)
espLabel.BackgroundTransparency = 1
espLabel.Text = " Esp :"
espLabel.TextColor3 = Color3.fromRGB(255, 255, 255)
espLabel.TextScaled = true
espLabel.Font = Enum.Font.Gotham
espLabel.TextXAlignment = Enum.TextXAlignment.Left
espLabel.Parent = commandsFrame

local espButton = Instance.new("TextButton")
espButton.Name = "EspButton"
espButton.Size = UDim2.new(0, 60, 0, 25)
espButton.Position = UDim2.new(0, 70, 0, 10)
espButton.BackgroundColor3 = Color3.fromRGB(200, 50, 50)
espButton.BorderSizePixel = 0
espButton.Text = "OFF"
espButton.TextColor3 = Color3.fromRGB(255, 255, 255)
espButton.TextScaled = true
espButton.Font = Enum.Font.GothamBold
espButton.Parent = commandsFrame

local function createHighlight(character)
    if not character or highlights[character] then return end 
    local highlight = Instance.new("Highlight")
    highlight.Name = "ESPHighlight"
    highlight.FillColor = Color3.fromRGB(100, 255, 100)
    highlight.FillTransparency = 0.6
    highlight.OutlineColor = Color3.fromRGB(100, 255, 100)
    highlight.OutlineTransparency = 0
    highlight.Parent = character
    highlights[character] = highlight
end

local function createNameLabel(character, playerName)
    if not character or nameLabels[character] or not character:FindFirstChild("Head") then return end 
    local billboardGui = Instance.new("BillboardGui")
    billboardGui.Name = "ESPNameLabel"
    billboardGui.Adornee = character.Head
    billboardGui.Size = UDim2.new(0, 100, 0, 50)
    billboardGui.StudsOffset = Vector3.new(0, 2, 0)
    billboardGui.AlwaysOnTop = true
    billboardGui.Parent = character.Head 
    local nameLabel = Instance.new("TextLabel")
    nameLabel.Size = UDim2.new(1, 0, 1, 0)
    nameLabel.BackgroundTransparency = 1
    nameLabel.Text = playerName
    nameLabel.TextColor3 = Color3.fromRGB(255, 255, 255)
    nameLabel.TextScaled = true
    nameLabel.Font = Enum.Font.GothamBold
    nameLabel.TextStrokeTransparency = 0
    nameLabel.TextStrokeColor3 = Color3.fromRGB(0, 0, 0)
    nameLabel.Parent = billboardGui
    nameLabels[character] = billboardGui
end

local function removeHighlight(character)
    if highlights[character] then
        highlights[character]:Destroy()
        highlights[character] = nil
    end
end

local function removeNameLabel(character)
    if nameLabels[character] then
        nameLabels[character]:Destroy()
        nameLabels[character] = nil
    end
end

local function updateESP()
    if espEnabled then
        for _, otherPlayer in pairs(Players:GetPlayers()) do
            if otherPlayer ~= player and otherPlayer.Character then
                createHighlight(otherPlayer.Character)
                createNameLabel(otherPlayer.Character, otherPlayer.Name)
            end
        end
    else
        for character, highlight in pairs(highlights) do
            highlight:Destroy()
        end
        for character, nameLabel in pairs(nameLabels) do
            nameLabel:Destroy()
        end
        highlights = {}
        nameLabels = {}
    end
end

local function toggleESP()
    espEnabled = not espEnabled
    if espEnabled then
        espButton.Text = "ON"
        espButton.BackgroundColor3 = Color3.fromRGB(50, 200, 50)
        connection = RunService.Heartbeat:Connect(function()
            updateESP()
        end)
        Players.PlayerAdded:Connect(function(newPlayer)
            newPlayer.CharacterAdded:Connect(function(character)
                if espEnabled then
                    wait(0.1)
                    createHighlight(character)
                    createNameLabel(character, newPlayer.Name)
                end
            end)
        end)
    else
        espButton.Text = "OFF"
        espButton.BackgroundColor3 = Color3.fromRGB(200, 50, 50)
        if connection then
            connection:Disconnect()
            connection = nil
        end
    end
    updateESP()
end

espButton.MouseButton1Click:Connect(toggleESP)

local function toggleMinimize()
    isMinimized = not isMinimized 
    if isMinimized then
        commandsFrame.Visible = false
        mainFrame.Size = UDim2.new(0, 200, 0, 25)
        closeButton.Text = "+"
        closeButton.BackgroundColor3 = Color3.fromRGB(50, 200, 50)
    else
        commandsFrame.Visible = true
        mainFrame.Size = UDim2.new(0, 200, 0, 80)
        closeButton.Text = "X"
        closeButton.BackgroundColor3 = Color3.fromRGB(200, 50, 50)
    end
end

closeButton.MouseButton1Click:Connect(toggleMinimize)

local dragging = false
local dragStart = nil
local startPos = nil

mainFrame.Active = true
mainFrame.Draggable = true

local function updateInput(input)
    if dragging then
        local delta = input.Position - dragStart
        local position = UDim2.new(startPos.X.Scale, startPos.X.Offset + delta.X, startPos.Y.Scale, startPos.Y.Offset + delta.Y)
        mainFrame.Position = position
    end
end

titleLabel.InputBegan:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseButton1 then
        dragging = true
        dragStart = input.Position
        startPos = mainFrame.Position
    end
end)

UserInputService.InputChanged:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseMovement then
        updateInput(input)
    end
end)

UserInputService.InputEnded:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseButton1 then
        dragging = false
    end
end)

-- Limpeza quando jogadores saem
Players.PlayerRemoving:Connect(function(leavingPlayer)
    if leavingPlayer.Character then
        if highlights[leavingPlayer.Character] then
            removeHighlight(leavingPlayer.Character)
        end
        if nameLabels[leavingPlayer.Character] then
            removeNameLabel(leavingPlayer.Character)
        end
    end
end)

mainFrame.BackgroundTransparency = 1
titleLabel.BackgroundTransparency = 1
titleLabel.TextTransparency = 1
espButton.BackgroundTransparency = 1
espButton.TextTransparency = 1
closeButton.BackgroundTransparency = 1
closeButton.TextTransparency = 1

local fadeIn = TweenService:Create(mainFrame, TweenInfo.new(0.3), {BackgroundTransparency = 0})
local fadeInTitle = TweenService:Create(titleLabel, TweenInfo.new(0.3), {BackgroundTransparency = 0, TextTransparency = 0})
local fadeInEspButton = TweenService:Create(espButton, TweenInfo.new(0.3), {BackgroundTransparency = 0, TextTransparency = 0})
local fadeInClose = TweenService:Create(closeButton, TweenInfo.new(0.3), {BackgroundTransparency = 0, TextTransparency = 0})

fadeIn:Play()
fadeInTitle:Play()
fadeInEspButton:Play()
fadeInClose:Play()
