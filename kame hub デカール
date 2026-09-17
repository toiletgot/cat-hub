local Players = game:GetService("Players")
local LocalPlayer = Players.LocalPlayer

local StarterGui = game:GetService("StarterGui")
local function Notify(title, text, duration)
    StarterGui:SetCore("SendNotification", {
        Title = title,
        Text = text,
        Duration = duration or 2
    })
end

-- ===== 固定設定 =====
local DECAL_ID = "rbxassetid://89027170969335"
local TARGET_TRANSPARENCY = 1.0  -- 完全透明（体・武器用）
local WALL_TRANSPARENCY = 0.0    -- 壁は透明にしない
local UPDATE_INTERVAL = 5        -- 5秒ごとに更新

local enabled = false
local loopRunning = false

-- 元のプロパティ保存用
local originalProps = {}
-- デカール貼り付け済み管理
local hasDecal = {}

-- ===== 壁判定関数 =====
local function isWall(part)
    if not part:IsA("BasePart") then return false end
    if part.Transparency >= 0.95 then return false end
    if part.CanCollide == false then return false end
    local size = part.Size
    if size.X < 0.3 or size.Y < 0.3 or size.Z < 0.3 then return false end
    return true
end

-- ===== デカールクリア（OFF時のみ） =====
local function clearDecals()
    for part, data in pairs(originalProps) do
        if part and part.Parent then
            part.Material = data.Material
            part.Color = data.Color
            part.Transparency = data.Transparency
        end
    end
    originalProps = {}

    for part, _ in pairs(hasDecal) do
        if part and part.Parent then
            for _, decal in ipairs(part:GetChildren()) do
                if decal:IsA("Decal") then
                    decal:Destroy()
                end
            end
        end
    end
    hasDecal = {}
end

-- ===== デカール貼り付け（強制透明度適用） =====
local function applyDecalsToPart(part, transparency)
    if not part or not part.Parent then return end
    
    -- 元のプロパティ保存（初回のみ）
    if not originalProps[part] then
        originalProps[part] = {
            Material = part.Material,
            Color = part.Color,
            Transparency = part.Transparency
        }
    end

    -- 強制的に透明度を適用
    part.Transparency = transparency

    -- デカールがなければ作成
    local hasDecalAlready = false
    for _, child in ipairs(part:GetChildren()) do
        if child:IsA("Decal") then
            hasDecalAlready = true
            break
        end
    end
    
    if not hasDecalAlready then
        local faces = {
            Enum.NormalId.Front,
            Enum.NormalId.Back,
            Enum.NormalId.Left,
            Enum.NormalId.Right,
            Enum.NormalId.Top,
            Enum.NormalId.Bottom
        }

        for _, face in ipairs(faces) do
            local decal = Instance.new("Decal")
            decal.Texture = DECAL_ID
            decal.Face = face
            decal.Parent = part
        end

        hasDecal[part] = true
    end
end

-- ===== 武器を再帰的に取得 =====
local function getWeaponParts(container)
    local parts = {}
    if not container then return parts end
    
    for _, obj in ipairs(container:GetDescendants()) do
        if obj:IsA("Tool") or obj:IsA("Accessory") then
            for _, part in ipairs(obj:GetDescendants()) do
                if part:IsA("BasePart") then
                    parts[part] = true
                end
            end
        end
        
        if obj:IsA("BasePart") then
            local nameLower = obj.Name:lower()
            if nameLower:find("knife") or nameLower:find("gun") or
               nameLower:find("weapon") or nameLower:find("blade") or
               nameLower:find("mag") or nameLower:find("ammo") or
               nameLower:find("handle") or nameLower:find("barrel") or
               nameLower:find("stock") or nameLower:find("trigger") or
               nameLower:find("sight") or nameLower:find("grip") then
                
                parts[obj] = true
                
                local parent = obj.Parent
                if parent and (parent:IsA("Model") or parent:IsA("Folder")) then
                    for _, sibling in ipairs(parent:GetDescendants()) do
                        if sibling:IsA("BasePart") and sibling ~= obj then
                            parts[sibling] = true
                        end
                    end
                end
            end
        end
    end
    
    return parts
end

-- ===== メイン適用関数 =====
local function applyAll()
    if not enabled then
        return
    end

    local character = LocalPlayer.Character
    if not character then return end

    local totalCount = 0
    local wallCount = 0
    local bodyCount = 0
    local weaponCount = 0

    -- === 1. 壁・床・地面 ===
    local allParts = workspace:GetDescendants()
    for _, part in ipairs(allParts) do
        if isWall(part) then
            applyDecalsToPart(part, WALL_TRANSPARENCY)
            wallCount = wallCount + 1
            totalCount = totalCount + 1
        end
    end

    -- === 2. 自分のボディパーツ ===
    local bodyPartNames = {
        "Head", "Torso", "Left Arm", "Right Arm", "Left Leg", "Right Leg",
        "UpperTorso", "LowerTorso",
        "LeftUpperArm", "LeftLowerArm", "LeftHand",
        "RightUpperArm", "RightLowerArm", "RightHand",
        "LeftUpperLeg", "LeftLowerLeg", "LeftFoot",
        "RightUpperLeg", "RightLowerLeg", "RightFoot"
    }
    
    for _, name in ipairs(bodyPartNames) do
        local part = character:FindFirstChild(name)
        if part and part:IsA("BasePart") then
            applyDecalsToPart(part, TARGET_TRANSPARENCY)
            bodyCount = bodyCount + 1
            totalCount = totalCount + 1
        end
    end

    -- 見逃し防止（全BasePartスキャン）
    for _, part in ipairs(character:GetDescendants()) do
        if part:IsA("BasePart") and not hasDecal[part] then
            applyDecalsToPart(part, TARGET_TRANSPARENCY)
            bodyCount = bodyCount + 1
            totalCount = totalCount + 1
        end
    end

    -- === 3. 武器 ===
    local searchContainers = {
        character,
        workspace:FindFirstChild("ViewModelRig"),
        workspace.CurrentCamera,
        LocalPlayer:FindFirstChild("Backpack"),
        LocalPlayer:FindFirstChild("StarterGear"),
    }

    local weaponTargets = {}

    for _, container in ipairs(searchContainers) do
        if not container then continue end
        local parts = getWeaponParts(container)
        for part, _ in pairs(parts) do
            weaponTargets[part] = true
        end
    end

    for part, _ in pairs(weaponTargets) do
        if part and part.Parent then
            applyDecalsToPart(part, TARGET_TRANSPARENCY)
            weaponCount = weaponCount + 1
            totalCount = totalCount + 1
        end
    end

    if totalCount > 0 then
        Notify("デカール適用", "壁:" .. wallCount .. " 体:" .. bodyCount .. " 武器:" .. weaponCount .. " (計" .. totalCount .. ")", 2)
    end
end

-- ===== ループ処理（5秒ごと） =====
local function startLoop()
    if loopRunning then return end
    loopRunning = true
    
    task.spawn(function()
        while loopRunning and enabled do
            if enabled then
                applyAll()
            end
            task.wait(UPDATE_INTERVAL)
        end
    end)
end

local function stopLoop()
    loopRunning = false
end

-- ===== キャラクター再生成対応 =====
local function onCharacterAdded()
    task.wait(0.5)
    if enabled then
        applyAll()
    end
end

-- ===== GUI =====
local playerGui = LocalPlayer:WaitForChild("PlayerGui")

local ScreenGui = Instance.new("ScreenGui")
ScreenGui.ResetOnSpawn = false
ScreenGui.ZIndexBehavior = Enum.ZIndexBehavior.Sibling
ScreenGui.Parent = playerGui

local Frame = Instance.new("Frame")
Frame.Size = UDim2.new(0, 180, 0, 80)
Frame.Position = UDim2.new(1, -190, 0, 20)
Frame.BackgroundColor3 = Color3.fromRGB(20, 20, 20)
Frame.BackgroundTransparency = 0.2
Frame.Active = true
Frame.Draggable = true
Frame.ZIndex = 10
Frame.Parent = ScreenGui

local Corner = Instance.new("UICorner")
Corner.CornerRadius = UDim.new(0, 12)
Corner.Parent = Frame

local Title = Instance.new("TextLabel")
Title.Size = UDim2.new(1, 0, 0, 25)
Title.BackgroundTransparency = 1
Title.Text = "全オブジェクトデカール"
Title.TextColor3 = Color3.fromRGB(255, 255, 255)
Title.TextSize = 13
Title.Font = Enum.Font.GothamBold
Title.ZIndex = 11
Title.Parent = Frame

local ToggleButton = Instance.new("TextButton")
ToggleButton.Size = UDim2.new(1, -20, 0, 30)
ToggleButton.Position = UDim2.new(0, 10, 0, 30)
ToggleButton.BackgroundColor3 = Color3.fromRGB(200, 50, 50)
ToggleButton.Text = "OFF"
ToggleButton.TextColor3 = Color3.fromRGB(255, 255, 255)
ToggleButton.TextSize = 13
ToggleButton.Font = Enum.Font.GothamSemibold
ToggleButton.ZIndex = 12
ToggleButton.Parent = Frame

local ToggleCorner = Instance.new("UICorner")
ToggleCorner.CornerRadius = UDim.new(0, 6)
ToggleCorner.Parent = ToggleButton

-- ===== Toggleボタン =====
ToggleButton.MouseButton1Click:Connect(function()
    enabled = not enabled
    ToggleButton.BackgroundColor3 = enabled and Color3.fromRGB(50, 200, 50) or Color3.fromRGB(200, 50, 50)
    ToggleButton.Text = enabled and "ON" or "OFF"
    
    if enabled then
        applyAll()
        startLoop()
        Notify("全デカール", "ON（5秒ごと更新開始）", 2)
    else
        stopLoop()
        clearDecals()
        Notify("全デカール", "OFF", 1)
    end
end)

-- キャラクター追加イベント
LocalPlayer.CharacterAdded:Connect(onCharacterAdded)

Notify("全デカール", "右上のGUIでON/OFF（5秒ごと更新）", 2)
