local Players = game:GetService("Players")
local Player = Players.LocalPlayer
local Character = Player.Character or Player.CharacterAdded:Wait()
local Root = Character:WaitForChild("HumanoidRootPart")
local TweenService = game:GetService("TweenService")
local UserInputService = game:GetService("UserInputService")

-- Create ScreenGui
local ScreenGui = Instance.new("ScreenGui")
ScreenGui.Name = "AutofarmGUI"
ScreenGui.Parent = game.Players.LocalPlayer:WaitForChild("PlayerGui")

-- Create main frame
local MainFrame = Instance.new("Frame")
MainFrame.Name = "MainFrame"
MainFrame.Size = UDim2.new(0, 200, 0, 570)
MainFrame.Position = UDim2.new(0.5, -100, 0.5, -285)
MainFrame.BackgroundColor3 = Color3.fromRGB(45, 45, 45)
MainFrame.BorderSizePixel = 0
MainFrame.Parent = ScreenGui

-- Create title
local Title = Instance.new("TextLabel")
Title.Name = "Title"
Title.Size = UDim2.new(1, 0, 0, 30)
Title.BackgroundColor3 = Color3.fromRGB(35, 35, 35)
Title.TextColor3 = Color3.fromRGB(255, 255, 255)
Title.Text = "Autofarm"
Title.TextSize = 18
Title.Font = Enum.Font.SourceSansBold
Title.Parent = MainFrame

-- Add section headers
local function createHeader(text, yPos)
    local Header = Instance.new("TextLabel")
    Header.Size = UDim2.new(1, 0, 0, 25)
    Header.Position = UDim2.new(0, 0, 0, yPos)
    Header.BackgroundColor3 = Color3.fromRGB(40, 40, 40)
    Header.TextColor3 = Color3.fromRGB(255, 255, 255)
    Header.Text = text
    Header.TextSize = 14
    Header.Font = Enum.Font.SourceSansBold
    Header.Parent = MainFrame
    return Header
end

-- Make GUI draggable
local isDragging = false
local dragStart = nil
local startPos = nil

Title.InputBegan:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseButton1 then
        isDragging = true
        dragStart = input.Position
        startPos = MainFrame.Position
    end
end)

UserInputService.InputChanged:Connect(function(input)
    if isDragging and input.UserInputType == Enum.UserInputType.MouseMovement then
        local delta = input.Position - dragStart
        MainFrame.Position = UDim2.new(
            startPos.X.Scale,
            startPos.X.Offset + delta.X,
            startPos.Y.Scale,
            startPos.Y.Offset + delta.Y
        )
    end
end)

UserInputService.InputEnded:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseButton1 then
        isDragging = false
    end
end)

-- Function to create toggle buttons
local function createToggleButton(name, yPos, callback)
    local Button = Instance.new("TextButton")
    Button.Name = name
    Button.Size = UDim2.new(0.9, 0, 0, 30)
    Button.Position = UDim2.new(0.05, 0, 0, yPos)
    Button.BackgroundColor3 = Color3.fromRGB(60, 60, 60)
    Button.TextColor3 = Color3.fromRGB(255, 255, 255)
    Button.Text = name
    Button.TextSize = 14
    Button.Font = Enum.Font.SourceSans
    Button.Parent = MainFrame
    
    local enabled = false
    local connection = nil
    
    Button.MouseButton1Click:Connect(function()
        enabled = not enabled
        Button.BackgroundColor3 = enabled and Color3.fromRGB(0, 170, 0) or Color3.fromRGB(60, 60, 60)
        
        if enabled then
            if name == "Train Mobility" then
                -- Mobility training logic
                local forward = true
                connection = game:GetService("RunService").Heartbeat:Connect(function()
                    if Character and Root then
                        local currentPos = Root.Position
                        if forward then
                            Root.CFrame = CFrame.new(currentPos + Vector3.new(10, 0, 0))
                        else
                            Root.CFrame = CFrame.new(currentPos + Vector3.new(-10, 0, 0))
                        end
                        forward = not forward
                        wait(0.1)
                    end
                end)
            elseif name == "Train Defense" then
                -- Teleport to sand pile
                local sandPile = workspace.Lobby.Extras.SandPile
                Root.CFrame = sandPile.CFrame + Vector3.new(0, 5, 0)
                
                -- Start defense training
                connection = game:GetService("RunService").Heartbeat:Connect(function()
                    local args = {[1] = 1}
                    game:GetService("ReplicatedStorage"):WaitForChild("Events"):WaitForChild("Train"):WaitForChild("TrainDefense"):FireServer(unpack(args))
                    wait(0.1)
                end)
            elseif name == "Auto Start Main Task" then
                connection = game:GetService("RunService").Heartbeat:Connect(function()
                    local args = {[1] = "MainTask"}
                    game:GetService("ReplicatedStorage"):WaitForChild("Events"):WaitForChild("Other"):WaitForChild("StartMainTask"):FireServer(unpack(args))
                    wait(0.1)
                end)
            elseif name == "Auto Claim Main Task" then
                connection = game:GetService("RunService").Heartbeat:Connect(function()
                    local args = {[1] = 1}
                    game:GetService("ReplicatedStorage"):WaitForChild("Events"):WaitForChild("Other"):WaitForChild("ClaimMainTask"):FireServer(unpack(args))
                    wait(0.1)
                end)
            elseif name == "Auto Claim Weekly Tasks" then
                local currentTask = 1
                connection = game:GetService("RunService").Heartbeat:Connect(function()
                    local args = {[1] = currentTask}
                    game:GetService("ReplicatedStorage"):WaitForChild("Events"):WaitForChild("Other"):WaitForChild("ClaimWeeklyTask"):FireServer(unpack(args))
                    
                    -- Cycle through tasks 1-10
                    currentTask = currentTask + 1
                    if currentTask > 10 then
                        currentTask = 1
                    end
                    wait(0.1)
                end)
            elseif name == "Destroy Train Effects" then
                connection = game:GetService("RunService").Heartbeat:Connect(function()
                    local playerGui = game:GetService("Players").LocalPlayer.PlayerGui
                    local hud = playerGui:FindFirstChild("HUD")
                    if hud then
                        local effects = hud:FindFirstChild("TEMPE")
                        if effects then
                            effects:Destroy()
                        end
                    end
                    wait(10)
                end)
            elseif name:match("Upgrade") then
                -- Handle stat upgrades
                local statTypes = {
                    ["Upgrade Strength"] = 1,
                    ["Upgrade Health"] = 2,
                    ["Upgrade Immunity"] = 3,
                    ["Upgrade Psychics"] = 4,
                    ["Upgrade Magic"] = 5,
                    ["Upgrade Mobility"] = 6
                }
                local statValue = statTypes[name]
                if statValue then
                    connection = game:GetService("RunService").Heartbeat:Connect(function()
                        local args = {[1] = statValue}
                        game:GetService("ReplicatedStorage"):WaitForChild("Events"):WaitForChild("Spent"):WaitForChild("UpgradeStat"):FireServer(unpack(args))
                        wait(0.1)
                    end)
                end
            else
                -- Other training types
                local eventName = name:gsub("Train ", "")
                connection = game:GetService("RunService").Heartbeat:Connect(function()
                    local args = {[1] = 0}
                    game:GetService("ReplicatedStorage"):WaitForChild("Events"):WaitForChild("Train"):WaitForChild("Train" .. eventName):FireServer(unpack(args))
                    wait(0.1)
                end)
            end
        else
            if connection then
                connection:Disconnect()
                connection = nil
            end
        end
    end)
end

-- Create section headers and buttons
createHeader("Training", 40)
local trainingButtons = {
    "Train Power", 
    "Train Psychics", 
    "Train Health", 
    "Train Defense", 
    "Train Mobility",
    "Destroy Train Effects"
}

local currentY = 65
for i, name in ipairs(trainingButtons) do
    createToggleButton(name, currentY)
    currentY = currentY + 40
end

createHeader("Tasks", currentY)
currentY = currentY + 25
local taskButtons = {
    "Auto Start Main Task",
    "Auto Claim Main Task",
    "Auto Claim Weekly Tasks"
}

for i, name in ipairs(taskButtons) do
    createToggleButton(name, currentY)
    currentY = currentY + 40
end

createHeader("Auto Upgrade Stats", currentY)
currentY = currentY + 25
local upgradeButtons = {
    "Upgrade Strength",
    "Upgrade Health",
    "Upgrade Immunity",
    "Upgrade Psychics",
    "Upgrade Magic",
    "Upgrade Mobility"
}

for i, name in ipairs(upgradeButtons) do
    createToggleButton(name, currentY)
    currentY = currentY + 40
end
