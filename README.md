-- [[ 最終完成版：左側高め配置・対話チャット ]]
local BRAIN_URL = "Https://script.google.com/macros/s/AKfycbzfH6aK-YDrexmxQWqIZLDgLYhrID4oGCDspq70s-OCf2AUdbSRkx9IG1w6s6vrYCZR_w/exec" 
local Player = game.Players.LocalPlayer
local PlayerGui = Player:WaitForChild("PlayerGui")

-- 二重起動防止
if PlayerGui:FindFirstChild("AIChatGui") then PlayerGui.AIChatGui:Destroy() end

local ScreenGui = Instance.new("ScreenGui", PlayerGui)
ScreenGui.Name = "AIChatGui"
ScreenGui.ResetOnSpawn = false

-- 位置設定：左側の中央より上（ジョイスティック回避）
local BASE_POSITION = UDim2.new(0, 15, 0.15, 0)

-- ログ表示枠
local ChatWindow = Instance.new("ScrollingFrame", ScreenGui)
ChatWindow.Size = UDim2.new(0, 280, 0, 180)
ChatWindow.Position = BASE_POSITION
ChatWindow.BackgroundColor3 = Color3.fromRGB(10, 10, 10)
ChatWindow.BackgroundTransparency = 0.4
ChatWindow.BorderSizePixel = 0
ChatWindow.AutomaticCanvasSize = Enum.AutomaticSize.Y
ChatWindow.CanvasSize = UDim2.new(0, 0, 0, 0)
ChatWindow.ScrollBarThickness = 4
Instance.new("UICorner", ChatWindow).CornerRadius = UDim.new(0, 8)

local UIList = Instance.new("UIListLayout", ChatWindow)
UIList.Padding = UDim.new(0, 4)

-- 入力枠
local InputBox = Instance.new("TextBox", ScreenGui)
InputBox.Size = UDim2.new(0, 280, 0, 45)
InputBox.Position = BASE_POSITION + UDim2.new(0, 0, 0, 185)
InputBox.BackgroundColor3 = Color3.fromRGB(35, 35, 35)
InputBox.PlaceholderText = "タップして入力..."
InputBox.Text = ""
InputBox.TextColor3 = Color3.new(1, 1, 1)
InputBox.TextSize = 16
InputBox.ClearTextOnFocus = true
Instance.new("UICorner", InputBox).CornerRadius = UDim.new(0, 8)

local function AddLog(sender, text, color)
    local Label = Instance.new("TextLabel", ChatWindow)
    Label.Size = UDim2.new(1, -10, 0, 0)
    Label.AutomaticSize = Enum.AutomaticSize.Y
    Label.BackgroundTransparency = 1
    Label.Text = " <b>[" .. sender .. "]</b>: " .. text
    Label.RichText = true
    Label.TextColor3 = color or Color3.new(1, 1, 1)
    Label.TextSize = 14
    Label.Font = Enum.Font.SourceSans
    Label.TextWrapped = true
    Label.TextXAlignment = Enum.TextXAlignment.Left
    task.wait(0.1)
    ChatWindow.CanvasPosition = Vector2.new(0, ChatWindow.AbsoluteCanvasSize.Y)
end

local function AskAI(userMessage)
    AddLog("SYSTEM", "AIに送信中...", Color3.new(0.7, 0.7, 0.7))
    local success, response = pcall(function()
        return game:GetService("HttpService"):PostAsync(
            BRAIN_URL,
            game:GetService("HttpService"):JSONEncode({situation = "Chat", message = userMessage}),
            Enum.HttpContentType.ApplicationJson
        )
    end)
    
    if success then
        local data = game:GetService("HttpService"):JSONDecode(response)
        if data.reply then
            AddLog("AI", data.reply, Color3.fromRGB(0, 255, 180))
        end
    else
        AddLog("ERROR", "通信失敗。URLを確認してください。", Color3.new(1, 0, 0))
    end
end

InputBox.FocusLost:Connect(function(enterPressed)
    if enterPressed and InputBox.Text ~= "" then
        local msg = InputBox.Text
        InputBox.Text = ""
        AddLog("YOU", msg, Color3.new(1, 1, 1))
        AskAI(msg)
    end
end)

AddLog("SYSTEM", "接続準備完了。文字を打ってみてください。", Color3.new(1, 1, 0))
