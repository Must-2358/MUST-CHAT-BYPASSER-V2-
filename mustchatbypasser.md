local Players = game:GetService("Players")
local ReplicatedStorage = game:GetService("ReplicatedStorage")
local TextChatService = game:GetService("TextChatService")
local TweenService = game:GetService("TweenService")
local Lighting = game:GetService("Lighting")
local CoreGui = game:GetService("CoreGui")

local LocalPlayer = Players.LocalPlayer

-- Destroy existing GUI if found
pcall(function()
	if CoreGui:FindFirstChild("MiniFancyBypasser") then
		CoreGui.MiniFancyBypasser:Destroy()
	end
end)

-- Blocked word replacements (your original filter)
local replacements = {
	["Rndi"] = "rnd1", ["Tmkc"] = "tmkx", ["Maa"] = "ma", ["Lwda"] = "lnd",
	["Madarchod"] = "mc", ["Bhenchod"] = "bc", ["Gand"] = "gnd", ["Lund"] = "lnd",
    ["Chut"] = "chxt", ["Chuda"] = "cuda", ["Chud"] = "cud", ["Teri"] = "ter1", ["Chodu"] = "codu"
}

-- Simulated RHFC blocked words
local blockedWords = {"badword", "testblock", "filterme"}

-- Replace blocked words from replacements table
local function replaceBlockedWords(text)
	local modified = false
	local words = {}
	for word in text:gmatch("%S+") do
		local replacedWord = word
		for bad, safe in pairs(replacements) do
			if word:lower() == bad:lower() then
				replacedWord = safe
				modified = true
				break
			end
		end
		table.insert(words, replacedWord)
	end
	return table.concat(words, " "), modified
end

-- Fancy font table (your style)
local fontMap = {
	a = "α҉", b = "ҍ҉", c = "ƈ҉", d = "ԃ҉", e = "ҽ҉", f = "ϝ҉", g = "ɠ҉",
	h = "ԋ҉", i = "ι҉", j = "ʝ҉", k = "ƙ҉", l = "ʅ҉", m = "ɱ҉", n = "ɳ҉",
	o = "σ҉", p = "ρ҉", q = "ϙ҉", r = "ɾ҉", s = "ʂ҉", t = "ƚ҉", u = "υ҉",
	v = "ʋ҉", w = "ɯ҉", x = "Ӽ҉", y = "Ƴ҉", z = "ȥ҉"
}

-- Convert to fancy
local function toFancy(text)
	local words = {}
	for word in text:gmatch("%S+") do
		local fancyWord = {}
		for i = 1, #word do
			local c = word:sub(i, i)
			local lower = c:lower()
			local stylized = fontMap[lower] or c
			table.insert(fancyWord, stylized)
		end
		table.insert(words, table.concat(fancyWord, " "))
	end
	return table.concat(words, "   ")
end

-- Roblox chat send function (Delta safe)
local function sendChat(msg)
	if TextChatService.ChatVersion == Enum.ChatVersion.TextChatService then
		local success, err = pcall(function()
			TextChatService.TextChannels.RBXGeneral:SendAsync(msg)
		end)
		if not success then
			warn("Failed new chat send:", err)
		end
	else
		local chatRemote = ReplicatedStorage:FindFirstChild("DefaultChatSystemChatEvents")
		if chatRemote and chatRemote:FindFirstChild("SayMessageRequest") then
			chatRemote.SayMessageRequest:FireServer(msg, "All")
		else
			local fireChat = ReplicatedStorage:FindFirstChild("FireChatRemote", true)
			if fireChat then
				fireChat:FireServer(msg)
			else
				warn("No chat remote found.")
			end
		end
	end
end

-- GUI
local gui = Instance.new("ScreenGui")
gui.Name = "MiniFancyBypasser"
pcall(function() syn.protect_gui(gui) end)
gui.ResetOnSpawn = false
gui.Parent = CoreGui
gui.Enabled = false

local main = Instance.new("Frame")
main.Size = UDim2.new(0, 250, 0, 180)
main.Position = UDim2.new(0.5, -125, 0.5, -90)
main.BackgroundTransparency = 0.3
main.BackgroundColor3 = Color3.fromRGB(40, 40, 40)
main.BorderSizePixel = 0
main.AnchorPoint = Vector2.new(0.5, 0.5)
main.Active = true
main.Draggable = true
main.Visible = false
main.Parent = gui
Instance.new("UICorner", main).CornerRadius = UDim.new(0, 12)

-- Close button
local closeBtn = Instance.new("TextButton")
closeBtn.Size = UDim2.new(0, 24, 0, 24)
closeBtn.Position = UDim2.new(1, -28, 0, 4)
closeBtn.Text = "X"
closeBtn.TextColor3 = Color3.new(1, 1, 1)
closeBtn.BackgroundTransparency = 0.2
closeBtn.BackgroundColor3 = Color3.fromRGB(100, 0, 0)
closeBtn.Font = Enum.Font.GothamBold
closeBtn.TextSize = 14
Instance.new("UICorner", closeBtn).CornerRadius = UDim.new(0, 6)
closeBtn.MouseButton1Click:Connect(function()
	gui:Destroy()
end)
closeBtn.Parent = main

-- Input box
local inputBox = Instance.new("TextBox")
inputBox.Size = UDim2.new(1, -20, 0, 35)
inputBox.Position = UDim2.new(0, 10, 0, 30)
inputBox.PlaceholderText = "Enter text..."
inputBox.Text = ""
inputBox.TextSize = 16
inputBox.TextColor3 = Color3.new(1, 1, 1)
inputBox.BackgroundColor3 = Color3.fromRGB(30, 30, 30)
inputBox.Font = Enum.Font.Gotham
Instance.new("UICorner", inputBox).CornerRadius = UDim.new(0, 8)
inputBox.Parent = main

-- Send button
local sendBtn = Instance.new("TextButton")
sendBtn.Size = UDim2.new(1, -20, 0, 32)
sendBtn.Position = UDim2.new(0, 10, 0, 75)
sendBtn.Text = "Send"
sendBtn.TextColor3 = Color3.new(1, 1, 1)
sendBtn.BackgroundColor3 = Color3.fromRGB(0, 120, 255)
sendBtn.Font = Enum.Font.GothamBold
sendBtn.TextSize = 15
Instance.new("UICorner", sendBtn).CornerRadius = UDim.new(0, 8)
sendBtn.Parent = main

-- Output label
local outputLbl = Instance.new("TextLabel")
outputLbl.Size = UDim2.new(1, -20, 0, 40)
outputLbl.Position = UDim2.new(0, 10, 0, 115)
outputLbl.Text = ""
outputLbl.TextWrapped = true
outputLbl.TextColor3 = Color3.new(1,1,1)
outputLbl.TextSize = 14
outputLbl.Font = Enum.Font.Gotham
outputLbl.BackgroundTransparency = 1
outputLbl.Parent = main

-- Popup function
local function showPopup(text, color)
	local popup = Instance.new("TextLabel")
	popup.Size = UDim2.new(0, 200, 0, 28)
	popup.Position = UDim2.new(0, 10, 1, -10)
	popup.AnchorPoint = Vector2.new(0, 1)
	popup.BackgroundColor3 = color
	popup.Text = text
	popup.TextColor3 = Color3.new(1, 1, 1)
	popup.TextSize = 14
	popup.Font = Enum.Font.GothamBold
	popup.BackgroundTransparency = 0.2
	popup.ZIndex = 999
	popup.Parent = main
	Instance.new("UICorner", popup).CornerRadius = UDim.new(0, 6)

	TweenService:Create(popup, TweenInfo.new(2), {
		TextTransparency = 1,
		BackgroundTransparency = 1
	}):Play()

	task.delay(2, function()
		popup:Destroy()
	end)
end

-- Send button logic
sendBtn.MouseButton1Click:Connect(function()
	local msg = inputBox.Text
	if msg:match("^%s*$") then
		showPopup("❌ Please enter text!", Color3.fromRGB(200, 50, 50))
		return
	end

	-- Simulated RHFC check
	for _, bad in ipairs(blockedWords) do
		if msg:lower():find(bad) then
			showPopup("❌ Blocked by RHFC!", Color3.fromRGB(200, 50, 50))
			return
		end
	end

	local replaced, wasModified = replaceBlockedWords(msg)
	local fancy = toFancy(replaced)

	outputLbl.Text = fancy
	sendChat(fancy)

	if wasModified then
		showPopup("⚠️ Word replaced & sent!", Color3.fromRGB(255, 165, 0))
	else
		showPopup("✅ Sent!", Color3.fromRGB(50, 180, 90))
	end
end)

-- Startup animation
local function playStartup()
	local blur = Instance.new("BlurEffect")
	blur.Size = 0
	blur.Parent = Lighting

	local label = Instance.new("TextLabel")
	label.Size = UDim2.new(0, 320, 0, 60)
	label.Position = UDim2.new(0.5, -160, 0.5, -30)
	label.Text = "Must Bypasser V2"
	label.TextScaled = true
	label.Font = Enum.Font.GothamBlack
	label.TextColor3 = Color3.new(1, 1, 1)
	label.BackgroundTransparency = 1
	label.Parent = gui

	gui.Enabled = true
	TweenService:Create(blur, TweenInfo.new(1), {Size = 24}):Play()
	task.wait(4)
	TweenService:Create(blur, TweenInfo.new(1), {Size = 0}):Play()
	task.wait(1)

	blur:Destroy()
	label:Destroy()
	main.Visible = true
end

task.spawn(playStartup)
