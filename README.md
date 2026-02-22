-- [[ DOORS CRYSTAL OMNI-HUB V3 BY GEM FOR ORICHIIN-SAN ]]
local Library = loadstring(game:HttpGet("https://raw.githubusercontent.com/xHeptc/Kavo-UI-Library/main/source.lua"))()
local Window = Library.CreateLib("CRYSTAL MATRIX - GOD & SLAYER", "DarkScene")

-- === CÁC BIẾN HỆ THỐNG ===
local Players = game:GetService("Players")
local LocalPlayer = Players.LocalPlayer
local Character = LocalPlayer.Character or LocalPlayer.CharacterAdded:Wait()
local RootPart = Character:WaitForChild("HumanoidRootPart")
local Hum = Character:WaitForChild("Humanoid")

-- Biến Trạng Thái
local AutoLootEnabled = true
local AutoHideEnabled = true
local SlayerMode = false
local GodModeEnabled = false

-- === TAB 1: NGƯỜI CHƠI (PLAYER) ===
local PlayerTab = Window:NewTab("Người Chơi")
local PlayerSection = PlayerTab:NewSection("Chỉ Số Cơ Thể")

PlayerSection:NewSlider("Tốc Độ (WalkSpeed)", "Chỉnh tốc độ chạy cho Orichiin-san", 100, 16, function(s)
    Hum.WalkSpeed = s
end)

PlayerSection:NewButton("Reset Tốc Độ", "Về lại 16", function()
    Hum.WalkSpeed = 16
end)

-- === TAB 2: THẦN THÁNH (GOD & SLAYER) ===
local GodTab = Window:NewTab("Thần Thánh")
local GodSection = GodTab:NewSection("Vô Hiệu Hóa Thực Thể")

GodSection:NewToggle("Slayer Mode (Diệt Quái)", "Xóa sổ Rush, Ambush, Screech ngay lập tức", function(state)
    SlayerMode = state
    if state then
        game.StarterGui:SetCore("SendNotification", {Title = "SLAYER", Text = "Chế độ đồ tể đã bật!"})
    end
end)

GodSection:NewButton("Bất Tử (God Mode)", "Lưu ý: Sẽ đổi Humanoid (Cần dùng cẩn thận)", function()
    local oldHum = Character:FindFirstChildOfClass("Humanoid")
    if oldHum then
        local newHum = oldHum:Clone()
        newHum.Parent = Character
        newHum.Name = "GodHumanoid"
        oldHum:Destroy()
        LocalPlayer.Character = Character
        game.StarterGui:SetCore("SendNotification", {Title = "GOD MODE", Text = "Orichiin-san đã bất tử!"})
    end
end)

-- === TAB 3: TỰ ĐỘNG (AUTOMATION) ===
local AutoTab = Window:NewTab("Tự Động")
local AutoSection = AutoTab:NewSection("Farm & Trốn")

AutoSection:NewToggle("Auto Loot (Nhặt Đồ)", "Tự động hút Key, Gold, Pin", function(state)
    AutoLootEnabled = state
end)

AutoSection:NewToggle("Auto Hide (Trốn Quái)", "Tự động tìm tủ khi Rush/Ambush", function(state)
    AutoHideEnabled = state
end)

-- === TAB 4: TỐI ƯU (BOOSTER) ===
local BoostTab = Window:NewTab("Tối Ưu")
local BoostSection = BoostTab:NewSection("Performance & Vision")

BoostSection:NewButton("Kích Hoạt Turbo FPS", "Giảm lag cực mạnh", function()
    local lighting = game:GetService("Lighting")
    lighting.GlobalShadows = false
    lighting.FogEnd = 9e9
    for _, v in pairs(game:GetDescendants()) do
        if v:IsA("Part") or v:IsA("MeshPart") or v:IsA("UnionOperation") then
            v.Material = Enum.Material.Plastic
            v.Reflectance = 0
        elseif v:IsA("ParticleEmitter") or v:IsA("Trail") or v:IsA("PostEffect") then
            v.Enabled = false
        end
    end
end)

BoostSection:NewButton("FullBright (Sáng Hầm)", "Làm sáng toàn bộ bản đồ", function()
    local lighting = game:GetService("Lighting")
    lighting.Brightness = 2
    lighting.ClockTime = 14
    lighting.OutdoorAmbient = Color3.fromRGB(255, 255, 255)
end)

-- === HẬU ĐÀI (CORE LOGIC) ===

local function CreateESP(part, color, name)
    if not part or part:FindFirstChild("CrystalESP") then return end
    local billboard = Instance.new("BillboardGui", part)
    billboard.Name = "CrystalESP"; billboard.Size = UDim2.new(0, 150, 0, 40); billboard.AlwaysOnTop = true
    local label = Instance.new("TextLabel", billboard)
    label.Size = UDim2.new(1, 0, 1, 0); label.BackgroundTransparency = 1
    label.TextColor3 = color; label.Text = name; label.Font = Enum.Font.Code; label.TextSize = 15
end

-- Xử lý Thực thể (Slayer & Anti-Eyes)
game.Workspace.ChildAdded:Connect(function(child)
    -- Nếu bật Slayer Mode
    if SlayerMode then
        local targets = {"RushMoving", "AmbushMoving", "Screech", "Eyes", "A60", "A120"}
        for _, name in pairs(targets) do
            if child.Name:find(name) then
                task.wait(0.1)
                child:Destroy()
                return
            end
        end
    end

    -- Né quái (Nếu không xóa kịp hoặc không bật Slayer)
    if child.Name:find("Rush") or child.Name:find("Ambush") then
        if AutoHideEnabled then
            -- Logic tìm tủ và trốn (Giống bản cũ)
            local targetHide = nil
            for _, v in pairs(game.Workspace.CurrentRooms:GetDescendants()) do
                if (v.Name == "Closet" or v.Name == "Bed") and v:FindFirstChildWhichIsA("ProximityPrompt", true) then
                    targetHide = v; break
                end
            end
            if targetHide then
                RootPart.CFrame = targetHide.PrimaryPart.CFrame * CFrame.new(0, 0, 2)
                task.wait(0.1)
                fireproximityprompt(targetHide:FindFirstChildWhichIsA("ProximityPrompt", true))
                task.wait(6)
                fireproximityprompt(targetHide:FindFirstChildWhichIsA("ProximityPrompt", true))
            end
        end
    end
end)

-- Vòng lặp quét đồ & ESP
task.spawn(function()
    while task.wait(0.7) do
        for _, room in pairs(game.Workspace.CurrentRooms:GetChildren()) do
            if room:FindFirstChild("Door") and room.Door:FindFirstChild("Client") then
                CreateESP(room.Door.Client, Color3.fromRGB(0, 255, 0), "DOOR")
            end
            if AutoLootEnabled then
                for _, obj in pairs(room:GetDescendants()) do
                    if obj.Name == "Key" or obj.Name == "GoldPile" or obj.Name == "Battery" then
                        CreateESP(obj, Color3.fromRGB(255, 255, 0), obj.Name:upper())
                        local prompt = obj:FindFirstChildWhichIsA("ProximityPrompt", true)
                        if prompt and (RootPart.Position - (obj:IsA("Model") and obj.PrimaryPart or obj).Position).Magnitude < 15 then
                            fireproximityprompt(prompt)
                        end
                    end
                end
            end
        end
    end
end)

game.StarterGui:SetCore("SendNotification", {Title = "CRYSTAL MATRIX V3", Text = "God & Slayer đã sẵn sàng!"})
