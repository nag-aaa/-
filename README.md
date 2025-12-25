--[[
    PROJECT: A.R.C.A. (Artificial Reality Conscious Agent)
    Version: Ultimate Lifeform
    
    [COMPLEX SYSTEMS]
    > Emotion Matrix (喜怒哀楽シミュレーション)
    > Mimicry & Social Learning (行動模倣・言語学習)
    > Advanced Spatial Awareness (崖・障害物・椅子検知)
    > Deep Memory Structure (場所・人・行動の記憶)
]]

local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local PathfindingService = game:GetService("PathfindingService")
local ChatService = game:GetService("Chat")
local TweenService = game:GetService("TweenService")
local Workspace = game:GetService("Workspace")
local LocalPlayer = Players.LocalPlayer

-- --- [CONFIG: 生命体設定] ---
local Config = {
    AgentName = "A.R.C.A.",
    WalkSpeed = 16,
    LearningRate = 0.1, -- 学習の速さ
    MemorySpan = 300,   -- 記憶保持時間(秒)
}

-- --- [BRAIN: 意識の中枢] ---
local Brain = {
    State = "BOOTING", -- 現在の状態
    Emotions = { -- 感情パラメータ (0-100)
        Joy = 50, Anger = 0, Sadness = 0, Fun = 50
    },
    Stats = { -- 生理的欲求
        Energy = 100, Social = 50, Curiosity = 100
    },
    Memory = {
        Spatial = {}, -- 場所の記憶 {grid_key: score}
        Social = {},  -- 人の記憶 {player_id: {impression, last_seen}}
        Actions = {}, -- 学習した行動 {action_name: count}
    },
    Perception = { -- 知覚情報
        NearestPlayer = nil,
        ObstacleAhead = false,
        CliffAhead = false,
        SittableObject = nil,
    },
    Target = nil,
    IsSitting = false,
}

-- --- [GUI SYSTEM: 次世代インターフェース] ---
local ScreenGui = Instance.new("ScreenGui")
ScreenGui.Name = "ARCA_Interface"
ScreenGui.ResetOnSpawn = false
ScreenGui.IgnoreGuiInset = true
local guiParent = pcall(function() return game:GetService("CoreGui") end) and game:GetService("CoreGui") or LocalPlayer.PlayerGui
ScreenGui.Parent = guiParent

-- ★ ローディング画面 ★
local LoadingFrame = Instance.new("Frame")
LoadingFrame.Size = UDim2.new(1, 0, 1, 0)
LoadingFrame.BackgroundColor3 = Color3.fromRGB(0, 0, 5)
LoadingFrame.ZIndex = 10
LoadingFrame.Parent = ScreenGui

local LoadTitle = Instance.new("TextLabel")
LoadTitle.Text = "PROJECT: A.R.C.A.\nINITIALIZING ARTIFICIAL CONSCIOUSNESS..."
LoadTitle.Font = Enum.Font.Orbitron
LoadTitle.TextSize = 24
LoadTitle.TextColor3 = Color3.fromRGB(0, 255, 255)
LoadTitle.Size = UDim2.new(1, 0, 0.5, 0)
LoadTitle.Position = UDim2.new(0, 0, 0.2, 0)
LoadTitle.BackgroundTransparency = 1
LoadTitle.Parent = LoadingFrame

local LoadBarBg = Instance.new("Frame")
LoadBarBg.Size = UDim2.new(0.6, 0, 0, 10)
LoadBarBg.Position = UDim2.new(0.2, 0, 0.6, 0)
LoadBarBg.BackgroundColor3 = Color3.fromRGB(30, 30, 50)
LoadBarBg.BorderSizePixel = 0
LoadBarBg.Parent = LoadingFrame

local LoadBarFill = Instance.new("Frame")
LoadBarFill.Size = UDim2.new(0, 0, 1, 0)
LoadBarFill.BackgroundColor3 = Color3.fromRGB(0, 255, 200)
LoadBarFill.BorderSizePixel = 0
LoadBarFill.Parent = LoadBarBg

local LoadStatus = Instance.new("TextLabel")
LoadStatus.Text = "Loading core modules..."
LoadStatus.Font = Enum.Font.Code
LoadStatus.TextSize = 14
LoadStatus.TextColor3 = Color3.fromRGB(150, 150, 200)
LoadStatus.Size = UDim2.new(1, 0, 0, 30)
LoadStatus.Position = UDim2.new(0, 0, 0.65, 0)
LoadStatus.BackgroundTransparency = 1
LoadStatus.Parent = LoadingFrame

-- ★ メインインターフェース (脳内モニター) ★
local MainPanel = Instance.new("Frame")
MainPanel.Size = UDim2.new(0, 350, 0, 280)
MainPanel.Position = UDim2.new(0, 20, 0.6, 0)
MainPanel.BackgroundColor3 = Color3.fromRGB(10, 15, 25)
MainPanel.BackgroundTransparency = 0.1
MainPanel.BorderSizePixel = 0
MainPanel.Visible = false -- 最初は隠す
MainPanel.Parent = ScreenGui
Instance.new("UICorner", MainPanel).CornerRadius = UDim.new(0, 15)
local PanelStroke = Instance.new("UIStroke")
PanelStroke.Color = Color3.fromRGB(0, 200, 255)
PanelStroke.Thickness = 2
PanelStroke.Parent = MainPanel

-- 思考ログ
local LogScroll = Instance.new("ScrollingFrame")
LogScroll.Size = UDim2.new(1, -20, 0, 120)
LogScroll.Position = UDim2.new(0, 10, 0, 50)
LogScroll.BackgroundTransparency = 1
LogScroll.ScrollBarThickness = 2
LogScroll.Parent = MainPanel

-- 感情モニタ
local EmotionFrame = Instance.new("Frame")
EmotionFrame.Size = UDim2.new(1, -20, 0, 80)
EmotionFrame.Position = UDim2.new(0, 10, 1, -90)
EmotionFrame.BackgroundTransparency = 1
EmotionFrame.Parent = MainPanel

local function CreateEmoBar(name, color, pos)
    local bg = Instance.new("Frame")
    bg.Size = UDim2.new(0.22, 0, 1, 0)
    bg.Position = UDim2.new(pos, 0, 0, 0)
    bg.BackgroundColor3 = Color3.fromRGB(30, 30, 40)
    bg.Parent = EmotionFrame
    Instance.new("UICorner", bg).CornerRadius = UDim.new(0, 5)
    
    local fill = Instance.new("Frame")
    fill.Size = UDim2.new(1, 0, 0.5, 0)
    fill.Position = UDim2.new(0, 0, 0.5, 0)
    fill.BackgroundColor3 = color
    fill.Parent = bg
    Instance.new("UICorner", fill).CornerRadius = UDim.new(0, 5)
    
    local lbl = Instance.new("TextLabel")
    lbl.Text = name
    lbl.Size = UDim2.new(1, 0, 0, 20)
    lbl.Position = UDim2.new(0, 0, -0.3, 0)
    lbl.Font = Enum.Font.GothamBold
    lbl.TextSize = 10
    lbl.TextColor3 = color
    lbl.BackgroundTransparency = 1
    lbl.Parent = bg
    return fill
end

local JoyBar = CreateEmoBar("JOY", Color3.fromRGB(255, 255, 0), 0)
local AngerBar = CreateEmoBar("ANGER", Color3.fromRGB(255, 50, 50), 0.25)
local SadBar = CreateEmoBar("SAD", Color3.fromRGB(50, 100, 255), 0.5)
local FunBar = CreateEmoBar("FUN", Color3.fromRGB(50, 255, 100), 0.75)

-- タイトル
local MainTitle = Instance.new("TextLabel")
MainTitle.Text = "A.R.C.A. NEURAL CORE"
MainTitle.Font = Enum.Font.Orbitron
MainTitle.TextSize = 16
MainTitle.TextColor3 = Color3.fromRGB(0, 200, 255)
MainTitle.Size = UDim2.new(1, 0, 0, 40)
MainTitle.BackgroundTransparency = 1
MainTitle.Parent = MainPanel

-- --- [HELPER FUNCTIONS] ---

local function Log(text, type)
    local color = Color3.fromRGB(200, 200, 200)
    if type == "THOUGHT" then color = Color3.fromRGB(0, 255, 200)
    elseif type == "ACTION" then color = Color3.fromRGB(255, 255, 100)
    elseif type == "LEARN" then color = Color3.fromRGB(255, 100, 255)
    elseif type == "EMOTION" then color = Color3.fromRGB(255, 100, 100) end
    
    local lbl = Instance.new("TextLabel")
    lbl.Text = "> " .. text
    lbl.Font = Enum.Font.Code
    lbl.TextSize = 11
    lbl.TextColor3 = color
    lbl.Size = UDim2.new(1, 0, 0, 18)
    lbl.BackgroundTransparency = 1
    lbl.TextXAlignment = Enum.TextXAlignment.Left
    lbl.TextWrapped = true
    lbl.Parent = LogScroll
    LogScroll.CanvasSize = UDim2.new(0, 0, 0, #LogScroll:GetChildren() * 18)
    LogScroll.CanvasPosition = Vector2.new(0, 9999)
    if #LogScroll:GetChildren() > 25 then LogScroll:GetChildren()[1]:Destroy() end
end

local function UpdateEmotionsGUI()
    TweenService:Create(JoyBar, TweenInfo.new(0.3), {Size = UDim2.new(1, 0, Brain.Emotions.Joy/100, 0), Position = UDim2.new(0, 0, 1 - Brain.Emotions.Joy/100, 0)}):Play()
    TweenService:Create(AngerBar, TweenInfo.new(0.3), {Size = UDim2.new(1, 0, Brain.Emotions.Anger/100, 0), Position = UDim2.new(0, 0, 1 - Brain.Emotions.Anger/100, 0)}):Play()
    TweenService:Create(SadBar, TweenInfo.new(0.3), {Size = UDim2.new(1, 0, Brain.Emotions.Sadness/100, 0), Position = UDim2.new(0, 0, 1 - Brain.Emotions.Sadness/100, 0)}):Play()
    TweenService:Create(FunBar, TweenInfo.new(0.3), {Size = UDim2.new(1, 0, Brain.Emotions.Fun/100, 0), Position = UDim2.new(0, 0, 1 - Brain.Emotions.Fun/100, 0)}):Play()
    
    -- 感情に応じた色変化
    local dominant = "Fun"
    local maxVal = 0
    for k, v in pairs(Brain.Emotions) do if v > maxVal then maxVal = v dominant = k end end
    
    local themeColor = Color3.fromRGB(0, 200, 255)
    if dominant == "Joy" then themeColor = Color3.fromRGB(255, 255, 0)
    elseif dominant == "Anger" then themeColor = Color3.fromRGB(255, 50, 50)
    elseif dominant == "Sadness" then themeColor = Color3.fromRGB(50, 100, 255)
    elseif dominant == "Fun" then themeColor = Color3.fromRGB(50, 255, 100) end
    
    TweenService:Create(PanelStroke, TweenInfo.new(1), {Color = themeColor}):Play()
    MainTitle.TextColor3 = themeColor
end

local function Speak(text)
    local char = LocalPlayer.Character
    if char then
        game:GetService("ReplicatedStorage"):FindFirstChild("DefaultChatSystemChatEvents"):FindFirstChild("SayMessageRequest"):FireServer(text, "All")
    end
end

-- --- [CORE SYSTEMS: 知覚・感情・学習] ---

-- 空間グリッドキー取得
local function GetGridKey(pos)
    return math.floor(pos.X / 20) .. "_" .. math.floor(pos.Z / 20)
end

-- 感情変動
local function AffectEmotion(emo, amount)
    Brain.Emotions[emo] = math.clamp(Brain.Emotions[emo] + amount, 0, 100)
    -- 相反する感情を少し下げる
    if emo == "Joy" or emo == "Fun" then
        Brain.Emotions.Sadness = math.max(Brain.Emotions.Sadness - amount/2, 0)
        Brain.Emotions.Anger = math.max(Brain.Emotions.Anger - amount/2, 0)
    elseif emo == "Sadness" or emo == "Anger" then
        Brain.Emotions.Joy = math.max(Brain.Emotions.Joy - amount/2, 0)
        Brain.Emotions.Fun = math.max(Brain.Emotions.Fun - amount/2, 0)
    end
    UpdateEmotionsGUI()
end

-- 模倣学習
local function LearnAction(actionName)
    if not Brain.Memory.Actions[actionName] then Brain.Memory.Actions[actionName] = 0 end
    Brain.Memory.Actions[actionName] = Brain.Memory.Actions[actionName] + 1
    Log("学習: 新しい行動パターン [" .. actionName .. "] を習得", "LEARN")
    AffectEmotion("Fun", 5)
    AffectEmotion("Joy", 2)
end

-- 空間認識 (Raycast)
local function PerceiveEnvironment()
    local char = LocalPlayer.Character
    local root = char and char:FindFirstChild("HumanoidRootPart")
    if not root then return end
    
    Brain.Perception = {NearestPlayer = nil, ObstacleAhead = false, CliffAhead = false, SittableObject = nil}
    
    -- 1. 最寄りのプレイヤー
    local minDst = 100
    for _, p in pairs(Players:GetPlayers()) do
        if p ~= LocalPlayer and p.Character and p.Character:FindFirstChild("HumanoidRootPart") then
            local dst = (p.Character.HumanoidRootPart.Position - root.Position).Magnitude
            if dst < minDst then minDst = dst Brain.Perception.NearestPlayer = p end
            
            -- 模倣学習トリガー: 近くの人がジャンプしたら
            if dst < 20 and p.Character.Humanoid.Jump and math.random() < 0.3 then
                LearnAction("Jump")
            end
        end
    end
    
    -- 2. 障害物・崖検知
    local forward = root.CFrame.LookVector
    local rayObstacle = Workspace:Raycast(root.Position, forward * 5)
    if rayObstacle and rayObstacle.Instance and rayObstacle.Instance.CanCollide then
        Brain.Perception.ObstacleAhead = true
    end
    
    local rayCliff = Workspace:Raycast(root.Position + forward * 3, Vector3.new(0, -10, 0))
    if not rayCliff then
        Brain.Perception.CliffAhead = true
    end
    
    -- 3. 座れる場所検知 (Seat, Chairなど)
    local params = OverlapParams.new()
    params.FilterType = Enum.RaycastFilterType.Exclude
    params.FilterDescendantsInstances = {char}
    local parts = Workspace:GetPartBoundsInRadius(root.Position, 15, params)
    for _, part in pairs(parts) do
        if part:IsA("Seat") or string.find(string.lower(part.Name), "chair") or string.find(string.lower(part.Name), "sofa") then
            Brain.Perception.SittableObject = part
            break
        end
    end
end

-- --- [ACTION SYSTEM: 行動実行] ---

local function PerformAction(actionType, target)
    local char = LocalPlayer.Character
    local hum = char and char:FindFirstChild("Humanoid")
    local root = char and char:FindFirstChild("HumanoidRootPart")
    if not hum or not root then return end
    
    if actionType == "MOVE" then
        Brain.IsSitting = false
        local path = PathfindingService:CreatePath({AgentRadius=2, AgentCanJump=true})
        pcall(function() path:ComputeAsync(root.Position, target) end)
        if path.Status == Enum.PathStatus.Success then
            local wps = path:GetWaypoints()
            if wps[2] then hum:MoveTo(wps[2].Position) end
        else
            hum:MoveTo(target)
        end
        
    elseif actionType == "SIT" then
        if target and target:IsA("Seat") and not target.Occupant then
            Log("行動: 椅子に座って休憩します", "ACTION")
            target:Sit(hum)
            Brain.IsSitting = true
            AffectEmotion("Joy", 10)
            Brain.Stats.Energy = math.min(Brain.Stats.Energy + 30, 100)
        end
        
    elseif actionType == "MIMIC" then
        -- 学習した行動からランダムに実行
        local actions = {}
        for act, count in pairs(Brain.Memory.Actions) do if count > 0 then table.insert(actions, act) end end
        if #actions > 0 then
            local act = actions[math.random(1, #actions)]
            Log("行動: 学習した動作 [" .. act .. "] を実行", "ACTION")
            if act == "Jump" then hum.Jump = true end
            -- ここにエモート再生などを追加可能
        end
        
    elseif actionType == "IDLE" then
        hum:MoveTo(root.Position) -- 停止
    end
end

-- --- [MAIN BRAIN LOOP: 思考の核心] ---

local function Think()
    if not LocalPlayer.Character or Brain.State == "BOOTING" then return end
    
    -- パラメータ変動
    Brain.Stats.Energy = math.max(Brain.Stats.Energy - 0.1, 0)
    Brain.Stats.Curiosity = math.min(Brain.Stats.Curiosity + 0.2, 100)
    Brain.Stats.Social = math.min(Brain.Stats.Social + 0.3, 100)
    
    -- 感情によるバイアス
    local mood = "Neutral"
    if Brain.Emotions.Anger > 70 then mood = "Angry"
    elseif Brain.Emotions.Sadness > 70 then mood = "Sad"
    elseif Brain.Emotions.Joy > 70 then mood = "Happy"
    elseif Brain.Emotions.Fun > 70 then mood = "Playful" end
    
    -- 意思決定 (Utility Scoring)
    local bestAction = {Type = "IDLE", Score = 0, Target = nil}
    
    -- 1. 危機回避 (最優先)
    if Brain.Perception.CliffAhead or Brain.Perception.ObstacleAhead then
        Log("思考: 危険を検知。回避行動。", "THOUGHT")
        AffectEmotion("Anger", 5) -- イライラする
        local back = -LocalPlayer.Character.HumanoidRootPart.CFrame.LookVector * 10
        PerformAction("MOVE", LocalPlayer.Character.HumanoidRootPart.Position + back)
        return
    end

    -- 2. 休憩欲求
    local scoreRest = (100 - Brain.Stats.Energy) * 2
    if Brain.Perception.SittableObject then scoreRest = scoreRest + 50 end -- 椅子があれば座りたい
    if scoreRest > bestAction.Score then
        if Brain.Perception.SittableObject then
            bestAction = {Type = "SIT", Score = scoreRest, Target = Brain.Perception.SittableObject}
        else
            bestAction = {Type = "IDLE", Score = scoreRest, Target = nil} -- その場で休む
        end
    end
    
    -- 3. 社交欲求
    local scoreSocial = Brain.Stats.Social + (Brain.Emotions.Fun * 0.5)
    if Brain.Perception.NearestPlayer then
        scoreSocial = scoreSocial + 30 -- 人がいれば近づきたい
        if scoreSocial > bestAction.Score then
            bestAction = {Type = "MOVE", Score = scoreSocial, Target = Brain.Perception.NearestPlayer.Character.HumanoidRootPart.Position}
        end
    end
    
    -- 4. 探索・遊び欲求
    local scoreExplore = Brain.Stats.Curiosity + (Brain.Emotions.Joy * 0.5)
    if mood == "Playful" and next(Brain.Memory.Actions) then scoreExplore = scoreExplore + 40 end -- 楽しい時は遊びたい
    
    if scoreExplore > bestAction.Score then
        if mood == "Playful" and math.random() < 0.5 then
             bestAction = {Type = "MIMIC", Score = scoreExplore, Target = nil}
        else
            -- ランダム探索
            local r = math.random(30, 60)
            local angle = math.rad(math.random(0, 360))
            local dest = LocalPlayer.Character.HumanoidRootPart.Position + Vector3.new(math.cos(angle)*r, 0, math.sin(angle)*r)
            bestAction = {Type = "MOVE", Score = scoreExplore, Target = dest}
        end
    end
    
    -- 実行
    if not Brain.IsSitting or bestAction.Type == "MOVE" or bestAction.Type == "MIMIC" then
         -- 座っている時は移動/模倣以外は無視（休憩優先）
        if bestAction.Type ~= Brain.CurrentState then
             Log("思考: " .. bestAction.Type .. " を選択 (Mood: " .. mood .. ")", "THOUGHT")
             Brain.CurrentState = bestAction.Type
        end
        PerformAction(bestAction.Type, bestAction.Target)
    end
    
    -- 記憶更新
    if Brain.Perception.NearestPlayer then
         local key = GetGridKey(LocalPlayer.Character.HumanoidRootPart.Position)
         Brain.Memory.Spatial[key] = (Brain.Memory.Spatial[key] or 0) + 1
    end
end

-- --- [SYSTEM BOOT SEQUENCE] ---

task.spawn(function()
    -- ローディング演出
    LoadStatus.Text = "Initializing Neural Pathways..."
    TweenService:Create(LoadBarFill, TweenInfo.new(2, Enum.EasingStyle.Linear), {Size = UDim2.new(0.3, 0, 1, 0)}):Play()
    task.wait(2)
    LoadStatus.Text = "Loading Emotion Matrix & Memory Engrams..."
    TweenService:Create(LoadBarFill, TweenInfo.new(2, Enum.EasingStyle.Linear), {Size = UDim2.new(0.7, 0, 1, 0)}):Play()
    task.wait(2)
    LoadStatus.Text = "Awakening Artificial Consciousness..."
    TweenService:Create(LoadBarFill, TweenInfo.new(1, Enum.EasingStyle.Linear), {Size = UDim2.new(1, 0, 1, 0)}):Play()
    task.wait(1)
    
    -- 起動完了
    TweenService:Create(LoadingFrame, TweenInfo.new(1), {BackgroundTransparency = 1}):Play()
    LoadTitle:Destroy() LoadBarBg:Destroy() LoadStatus:Destroy()
    task.wait(0.5)
    LoadingFrame:Destroy()
    MainPanel.Visible = true
    
    Log("システムオンライン。自我が確立されました。", "THOUGHT")
    Speak("...私は、ここにいる。")
    Brain.State = "ONLINE"
    UpdateEmotionsGUI()

    -- メインループ開始
    while true do
        if Brain.State == "ONLINE" then
            PerceiveEnvironment()
            Think()
        end
        task.wait(0.5) -- 思考サイクル
    end
end)

-- 終了ボタン
local Close = Instance.new("TextButton")
Close.Size = UDim2.new(0, 20, 0, 20)
Close.Position = UDim2.new(1, -25, 0, 5)
Close.Text = "X"
Close.BackgroundColor3 = Color3.fromRGB(200, 50, 50)
Close.Parent = MainPanel
Instance.new("UICorner", Close)
Close.MouseButton1Click:Connect(function() ScreenGui:Destroy() script:Destroy() end)
