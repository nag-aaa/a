--[[
    Roblox AI v21 (Linguistic & Instinct Interface)
    - 外部の脳: Google Apps Script (GAS)
    - 知能レベル: IQ 1000 (外部演算)
    - 実行環境: モバイル (スマホ) 対応
]]

local HttpService = game:GetService("HttpService")
local RunService = game:GetService("RunService")
local Players = game:GetService("Players")
local LocalPlayer = Players.LocalPlayer

-- ■ あなたが作成した「脳」のURL
local BRAIN_URL = "https://script.google.com/macros/s/AKfycbwFgZpdIaTTKyZPQYhFEu1mIl_aeauQlCLjwAWerM--hqh7pKXjZDCXGaI-s_elJ9WInw/exec"

local CurrentMode = "友好"
local LastSpeech = ""
local IsActive = false

-- // 外部の脳（GAS）に状況を報告し、指示を仰ぐ
local function Think()
    if not IsActive then return end
    local char = LocalPlayer.Character; if not char then return end
    local root = char:FindFirstChild("HumanoidRootPart")
    if not root then return end

    -- 周囲の状況をデータ化して送信
    local situation = ""
    for _, p in pairs(Players:GetPlayers()) do
        if p ~= LocalPlayer and p.Character and p.Character:FindFirstChild("HumanoidRootPart") then
            local d = (p.Character.HumanoidRootPart.Position - root.Position).Magnitude
            if d < 30 then 
                situation = situation .. "Near:" .. p.Name .. " Dist:" .. math.floor(d) .. " " 
            end
        end
    end

    -- GASサーバーへPOSTリクエスト
    local success, response = pcall(function()
        return HttpService:PostAsync(BRAIN_URL, HttpService:JSONEncode({
            situation = situation ~= "" and situation or "No targets nearby."
        }))
    end)

    if success and response then
        local data = HttpService:JSONDecode(response)
        -- 言葉の出力
        if data.reply and data.reply ~= LastSpeech then
            if game:GetService("TextChatService").ChatVersion == Enum.ChatVersion.TextChatService then
                game:GetService("TextChatService").TextChannels.RBXGeneral:SendAsync(data.reply)
            else
                game:GetService("ReplicatedStorage").DefaultChatSystemChatEvents.SayMessageRequest:FireServer(data.reply, "All")
            end
            LastSpeech = data.reply
        end
        -- 行動モードの更新
        if data.mode then CurrentMode = data.mode end
    end
end

-- // 移動・ジャンプ制御 (基本動作の徹底)
RunService.Heartbeat:Connect(function()
    if not IsActive then return end
    local char = LocalPlayer.Character; if not char then return end
    local root = char:FindFirstChild("HumanoidRootPart"); local hum = char:FindFirstChild("Humanoid")
    if not root or not hum then return end

    -- 最も近いターゲットを検索
    local target = nil; local minDist = 100
    for _, p in pairs(Players:GetPlayers()) do
        if p ~= LocalPlayer and p.Character and p.Character:FindFirstChild("HumanoidRootPart") then
            local d = (p.Character.HumanoidRootPart.Position - root.Position).Magnitude
            if d < minDist then minDist = d; target = p end
        end
    end

    if target then
        local tRoot = target.Character.HumanoidRootPart
        local tPos = tRoot.Position
        local tVel = tRoot.AssemblyLinearVelocity
        
        hum.WalkSpeed = 32
        
        if CurrentMode == "追跡" or CurrentMode == "遮断" then
            -- 0.3秒後の未来位置へ移動
            hum:MoveTo(tPos + (tVel * 0.3))
        elseif CurrentMode == "友好" then
            -- 適度な距離(10スタッド)を保つ
            if (tPos - root.Position).Magnitude > 10 then
                hum:MoveTo(tPos)
            end
        elseif CurrentMode == "逃走" then
            local fleeDir = (root.Position - tPos).Unit
            hum:MoveTo(root.Position + (fleeDir * 20))
            if (tPos - root.Position).Magnitude < 15 then hum.Jump = true end
        end
    end
end)

-- // UI作成
local ScreenGui = Instance.new("ScreenGui", game:GetService("CoreGui") or LocalPlayer:WaitForChild("PlayerGui"))
local MainBtn = Instance.new("TextButton", ScreenGui)
MainBtn.Size = UDim2.new(0, 100, 0, 50)
MainBtn.Position = UDim2.new(0.5, -50, 0.1, 0)
MainBtn.Text = "AI OFF"
MainBtn.BackgroundColor3 = Color3.fromRGB(50, 50, 50)
MainBtn.TextColor3 = Color3.new(1,1,1)

MainBtn.MouseButton1Click:Connect(function()
    IsActive = not IsActive
    MainBtn.Text = IsActive and "AI ON" or "AI OFF"
    MainBtn.BackgroundColor3 = IsActive and Color3.fromRGB(0, 150, 0) or Color3.fromRGB(50, 50, 50)
end)

-- 12秒ごとに「脳」に相談
spawn(function()
    while true do
        wait(12)
        if IsActive then Think() end
    end
end)
