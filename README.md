-- [[ Roblox AI: 強制発言・デバッグ版 ]]
local BRAIN_URL = "https://script.google.com/macros/s/AKfycbwFgZpdIaTTKyZPQYhFEu1mIl_aeauQlCLjwAWerM--hqh7pKXjZDCXGaI-s_elJ9WInw/exec"
local IsActive = false
local LastSpeech = ""

-- // あらゆる手段で発言を試みる関数
local function Speak(message)
    if not message or message == "" then return end
    print("AI発言試行: " .. message)

    -- 1. 新しいチャット形式 (TextChatService)
    pcall(function()
        local tcs = game:GetService("TextChatService")
        if tcs.ChatVersion == Enum.ChatVersion.TextChatService then
            tcs.TextChannels.RBXGeneral:SendAsync(message)
        end
    end)

    -- 2. 古いチャット形式 (Legacy)
    pcall(function()
        game:GetService("ReplicatedStorage").DefaultChatSystemChatEvents.SayMessageRequest:FireServer(message, "All")
    end)
    
    -- 3. 頭上に吹き出しを出す (必ず見えるはずです)
    pcall(function()
        game:GetService("Chat"):Chat(game.Players.LocalPlayer.Character.Head, message)
    end)
end

local function Think()
    if not IsActive then return end
    local success, response = pcall(function()
        return game:GetService("HttpService"):PostAsync(BRAIN_URL, game:GetService("HttpService"):JSONEncode({situation = "Active"}))
    end)
    
    if success and response then
        local data = game:GetService("HttpService"):JSONDecode(response)
        if data.reply and data.reply ~= LastSpeech then
            Speak(data.reply)
            LastSpeech = data.reply
        end
    end
end

-- UI
local sg = Instance.new("ScreenGui", game:GetService("CoreGui") or game.Players.LocalPlayer.PlayerGui)
local btn = Instance.new("TextButton", sg)
btn.Size = UDim2.new(0, 120, 0, 50)
btn.Position = UDim2.new(0.5, -60, 0.1, 0)
btn.Text = "AI OFF"
btn.BackgroundColor3 = Color3.new(0.2, 0.2, 0.2)
btn.TextColor3 = Color3.new(1, 1, 1)

btn.MouseButton1Click:Connect(function()
    IsActive = not IsActive
    btn.Text = IsActive and "AI ON" or "AI OFF"
    btn.BackgroundColor3 = IsActive and Color3.new(0, 0.8, 0) or Color3.new(0.2, 0.2, 0.2)
end)

task.spawn(function()
    while true do
        task.wait(10)
        Think()
    end
end)
