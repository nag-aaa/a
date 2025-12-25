-- [[ Roblox AI: 動作＆チャット完全対応版 ]]
local BRAIN_URL = "https://script.google.com/macros/s/AKfycbwFgZpdIaTTKyZPQYhFEu1mIl_aeauQlCLjwAWerM--hqh7pKXjZDCXGaI-s_elJ9WInw/exec"
local IsActive = false
local LastSpeech = ""

-- // どんな環境でも強制的にチャットを流す関数
local function Speak(message)
    if not message or message == "" then return end
    print("AIが発言を試みます: " .. message)

    -- 1. 最新のTextChatServiceに対応
    pcall(function()
        local textService = game:GetService("TextChatService")
        if textService.ChatVersion == Enum.ChatVersion.TextChatService then
            local channel = textService.TextChannels.RBXGeneral
            channel:SendAsync(message)
        end
    end)

    -- 2. 従来のチャットシステム(Legacy)に対応
    pcall(function()
        game:GetService("ReplicatedStorage").DefaultChatSystemChatEvents.SayMessageRequest:FireServer(message, "All")
    end)
    
    -- 3. 自分の頭上に吹き出しを出す（念のため）
    pcall(function()
        game:GetService("Chat"):Chat(game.Players.LocalPlayer.Character.Head, message, Enum.ChatSpeakerType.User)
    end)
end

-- // 思考ルーチン
local function Think()
    if not IsActive then return end
    
    local success, response = pcall(function()
        return game:GetService("HttpService"):PostAsync(BRAIN_URL, game:GetService("HttpService"):JSONEncode({situation = "Communicating"}))
    end)
    
    if success and response then
        local data = game:GetService("HttpService"):JSONDecode(response)
        if data.reply and data.reply ~= LastSpeech then
            Speak(data.reply)
            LastSpeech = data.reply
        end
    else
        warn("脳(GAS)との通信に失敗しました")
    end
end

-- // 移動・ジャンプ制御 (動作部分はそのまま維持)
game:GetService("RunService").Heartbeat:Connect(function()
    if not IsActive then return end
    local char = game.Players.LocalPlayer.Character
    if not char or not char:FindFirstChild("HumanoidRootPart") then return end
    
    -- 最も近いプレイヤーを探す
    local target = nil
    local minDist = 50
    for _, p in pairs(game.Players:GetPlayers()) do
        if p ~= game.Players.LocalPlayer and p.Character and p.Character:FindFirstChild("HumanoidRootPart") then
            local d = (p.Character.HumanoidRootPart.Position - char.HumanoidRootPart.Position).Magnitude
            if d < minDist then minDist = d; target = p end
        end
    end

    if target then
        char.Humanoid:MoveTo(target.Character.HumanoidRootPart.Position)
    end
end)

-- // UIボタン（中央上部）
local sg = Instance.new("ScreenGui", game:GetService("CoreGui") or game.Players.LocalPlayer.PlayerGui)
local btn = Instance.new("TextButton", sg)
btn.Size = UDim2.new(0, 120, 0, 40)
btn.Position = UDim2.new(0.5, -60, 0.05, 0)
btn.Text = "AI: OFF"
btn.BackgroundColor3 = Color3.fromRGB(50, 50, 50)
btn.TextColor3 = Color3.new(1, 1, 1)

btn.MouseButton1Click:Connect(function()
    IsActive = not IsActive
    btn.Text = IsActive and "AI: ON" or "AI: OFF"
    btn.BackgroundColor3 = IsActive and Color3.fromRGB(0, 150, 0) or Color3.fromRGB(50, 50, 50)
end)

task.spawn(function()
    while true do
        task.wait(12)
        Think()
    end
end)
