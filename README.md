loadstring(game:HttpGet("data:text/plain;base64," .. game:GetService("HttpService"):Base64Encode([[
local UserInputService = game:GetService("UserInputService")
local Players = game:GetService("Players")
local DataStoreService = game:GetService("DataStoreService")

local player = Players.LocalPlayer
local playerGui = player:WaitForChild("PlayerGui")
local dataStore = DataStoreService:GetDataStore("TPManager_" .. player.UserId)

local tpList = {}
local dragging = false
local dragStart = nil
local dragOffset = nil

local screenGui = Instance.new("ScreenGui")
screenGui.Name = "TPManagerGui"
screenGui.ResetOnSpawn = false
screenGui.Parent = playerGui

local mainFrame = Instance.new("Frame")
mainFrame.Name = "MainFrame"
mainFrame.Size = UDim2.new(0, 300, 0, 400)
mainFrame.Position = UDim2.new(0.5, -150, 0.5, -200)
mainFrame.BackgroundColor3 = Color3.fromRGB(30, 30, 30)
mainFrame.BorderSizePixel = 0
mainFrame.Active = true
mainFrame.Parent = screenGui

local function startDrag(input)
	if input.UserInputType == Enum.UserInputType.MouseButton1 then
		dragging = true
		dragStart = input.Position
		dragOffset = mainFrame.AbsolutePosition
	end
end

local function stopDrag()
	dragging = false
end

local function updateDrag(input)
	if dragging then
		local delta = input.Position - dragStart
		mainFrame.Position = UDim2.new(0, dragOffset.X + delta.X, 0, dragOffset.Y + delta.Y)
	end
end

mainFrame.InputBegan:Connect(startDrag)
UserInputService.InputEnded:Connect(stopDrag)
UserInputService.InputChanged:Connect(updateDrag)

local titleLabel = Instance.new("TextLabel")
titleLabel.Size = UDim2.new(1, 0, 0, 40)
titleLabel.BackgroundColor3 = Color3.fromRGB(50, 50, 50)
titleLabel.TextColor3 = Color3.fromRGB(255, 255, 255)
titleLabel.TextSize = 18
titleLabel.Font = Enum.Font.GothamBold
titleLabel.Text = "TP Manager"
titleLabel.BorderSizePixel = 0
titleLabel.Parent = mainFrame

local scrollFrame = Instance.new("ScrollingFrame")
scrollFrame.Size = UDim2.new(1, -10, 1, -100)
scrollFrame.Position = UDim2.new(0, 5, 0, 50)
scrollFrame.BackgroundColor3 = Color3.fromRGB(40, 40, 40)
scrollFrame.BorderSizePixel = 0
scrollFrame.ScrollBarThickness = 8
scrollFrame.Parent = mainFrame

local listLayout = Instance.new("UIListLayout")
listLayout.Padding = UDim.new(0, 5)
listLayout.Parent = scrollFrame

local addButton = Instance.new("TextButton")
addButton.Size = UDim2.new(1, -10, 0, 40)
addButton.Position = UDim2.new(0, 5, 1, -45)
addButton.BackgroundColor3 = Color3.fromRGB(0, 150, 0)
addButton.TextColor3 = Color3.fromRGB(255, 255, 255)
addButton.TextSize = 14
addButton.Font = Enum.Font.GothamBold
addButton.Text = "+ TP Manager"
addButton.BorderSizePixel = 0
addButton.Parent = mainFrame

local function saveTPList()
	pcall(function()
		dataStore:SetAsync("TPList", tpList)
	end)
end

local function loadTPList()
	local success, data = pcall(function()
		return dataStore:GetAsync("TPList")
	end)
	if success and data then
		tpList = data
	else
		tpList = {}
	end
end

local function createTPButton(tpNumber)
	local tpButton = Instance.new("TextButton")
	tpButton.Size = UDim2.new(1, -10, 0, 35)
	tpButton.BackgroundColor3 = Color3.fromRGB(60, 60, 60)
	tpButton.TextColor3 = Color3.fromRGB(255, 255, 255)
	tpButton.TextSize = 12
	tpButton.Font = Enum.Font.Gotham
	tpButton.Text = "TP " .. tpNumber
	tpButton.BorderSizePixel = 0
	tpButton.Parent = scrollFrame
	
	local deleteButton = Instance.new("TextButton")
	deleteButton.Size = UDim2.new(0, 30, 0, 35)
	deleteButton.Position = UDim2.new(1, -35, 0, 0)
	deleteButton.BackgroundColor3 = Color3.fromRGB(200, 0, 0)
	deleteButton.Text = "X"
	deleteButton.Parent = tpButton
	
	deleteButton.MouseButton1Click:Connect(function()
		table.remove(tpList, tpNumber)
		tpButton:Destroy()
		saveTPList()
	end)
	
	tpButton.MouseButton1Click:Connect(function()
		if player.Character and player.Character:FindFirstChild("HumanoidRootPart") then
			local tpData = tpList[tpNumber]
			if tpData then
				player.Character.HumanoidRootPart.CFrame = CFrame.new(tpData.Position)
			end
		end
	end)
end

local function refreshTPList()
	for _, child in pairs(scrollFrame:GetChildren()) do
		if child:IsA("TextButton") then
			child:Destroy()
		end
	end
	
	for i, _ in ipairs(tpList) do
		createTPButton(i)
	end
end

addButton.MouseButton1Click:Connect(function()
	if player.Character and player.Character:FindFirstChild("HumanoidRootPart") then
		local newTP = {
			Position = player.Character.HumanoidRootPart.Position
		}
		table.insert(tpList, newTP)
		saveTPList()
		refreshTPList()
	end
end)

loadTPList()
refreshTPList()
]])))()
