local Library = loadstring(game:HttpGetAsync("https://github.com/1dontgiveaf/Fluent-Renewed/releases/download/v1.0/Fluent.luau"))()
local SaveManager = loadstring(game:HttpGetAsync("https://raw.githubusercontent.com/1dontgiveaf/Fluent-Renewed/refs/heads/main/Addons/SaveManager.luau"))()
local InterfaceManager = loadstring(game:HttpGetAsync("https://raw.githubusercontent.com/1dontgiveaf/Fluent-Renewed/refs/heads/main/Addons/InterfaceManager.luau"))()
 
local Players = game:GetService("Players")
local rs = game:GetService("ReplicatedStorage")
local packets = require(rs.Modules.Packets)
local plr = Players.LocalPlayer
local char = plr.Character or plr.CharacterAdded:Wait()
local root = char and char:WaitForChild("HumanoidRootPart", 5)
local hum = char and char:WaitForChild("Humanoid", 5)
local runs = game:GetService("RunService")
local httpService = game:GetService("HttpService")
 
-- Sigma Parts
local function createSigmaPart(name, position, size, shape, orientation)
    local part = Instance.new("Part")
    part.Name = name
    part.Parent = workspace
    part.Anchored = true
    part.Shape = shape or Enum.PartType.Block
    part.Position = position
    part.Size = size
    part.Color = Color3.fromRGB(0, 255, 130)
    part.Orientation = orientation or Vector3.new(0, 0, 0)
    part.Transparency = 0.9
end
 
createSigmaPart("SigmaPart", Vector3.new(-122, -28, -193), Vector3.new(4, 30, 25), Enum.PartType.Wedge, Vector3.new(0, 180, 0))
createSigmaPart("SigmaPart2", Vector3.new(-202, 5, -616), Vector3.new(4, 30, 25), Enum.PartType.Wedge, Vector3.new(0, 200, 0))
createSigmaPart("SigmaPart3", Vector3.new(-214, 18, -627), Vector3.new(12, 1, 12))
createSigmaPart("SigmaPart4", Vector3.new(-44, -104, -392), Vector3.new(6, 20, 17), Enum.PartType.Wedge)
createSigmaPart("SigmaPart5", Vector3.new(-45, -94, -374), Vector3.new(13, 1, 13))
 
-- Create Window
local Window = Library:CreateWindow{
    Title = "VHub",
    SubTitle = "Booga Booga",
    TabWidth = 160,
    Size = UDim2.fromOffset(830, 525),
    Resize = true,
    MinSize = Vector2.new(470, 380),
    Acrylic = true,
    Theme = "Dark",
    MinimizeKey = Enum.KeyCode.RightControl
}
 
local Tabs = {
    Settings = Window:AddTab({ Title = "Settings", Icon = "settings" }),
    Main = Window:AddTab({ Title = "Main", Icon = "menu" }),
    Combat = Window:AddTab({ Title = "Heal", Icon = "pizza" }),
    WaypointTab = Window:AddTab({ Title = "Waypoints", Icon = "waypoints" })
}
 
local Options = Library.Options
local MainSection = Tabs.Main:Section("Main")
local HealSection = Tabs.Combat:Section("Eating")
local CFGSect = Tabs.WaypointTab:Section("Configs")
local MoveSection = Tabs.WaypointTab:Section("Moving")
local WaypointSection = Tabs.WaypointTab:Section("Waypoints")
local PickSection = Tabs.WaypointTab:Section("Item PickUp")
local AutoHitSect = Tabs.Main:Section("....")
local AutoCampFire = Tabs.Main:Selection("Main")

-- Delete Old Boards
WaypointSection:CreateButton({Title = "Delete OldBoards", Callback = function() 
    game:GetService("RunService").RenderStepped:Connect(function()
        for i, v in pairs(workspace.Resources:GetDescendants()) do
            if v.Name == "Old Boards" and workspace.Resources:FindFirstChild("Old Boards") then
                v:Destroy()
            end
        end
    end)
end})
 
-- Walkspeed
local wstoggle = Tabs.Main:CreateToggle("wstoggle", { Title = "Walkspeed", Default = false })
local wsslider = Tabs.Main:CreateSlider("wsslider", { Title = "Value", Min = 16, Max = 23, Rounding = 1, Default = 16 })
 
-- MaxSlopeAngle
local msatoggle = Tabs.Main:CreateToggle("msatoggle", { Title = "MountainWalker", Default = false })
 
-- AutoHeal
local EatToggle = HealSection:AddToggle("EatToggle", { Title = "Heal", Default = false })
local eatdropdown = HealSection:AddDropdown("eatdropdown", {Title = "Food", Values = {"Bloodfruit", "Bluefruit", "Lemon", "Coconut", "Jelly", "Banana", "Orange", "Oddberry", "Berry", "Strangefruit", "Strawberry", "Sunjfruit", "Pumpkin", "Prickly Pear", "Apple", "Barley", "Cloudberry", "Carrot"}, Default = "Bloodfruit"})
local HealthCount = HealSection:CreateSlider("HealthCount", { Title = "Health", Min = 1, Max = hum and hum.Health or 100, Rounding = 1, Default = 2 })
 
-- AutoHit Resources
local resourceauratoggle = AutoHitSect:CreateToggle("resourceauratoggle", { Title = "Hit Resources", Default = false })
 
-- AutoPickup
local autopickuptoggle = PickSection:CreateToggle("autopickuptoggle", { Title = "Auto Pickup Raw Gold", Default = false })
 
-- Move To Points
local startMOVE = MoveSection:CreateToggle("StartMoving", { Title = "Start Moving", Default = false })
 
-- Goldfarm Config
local javaUrl = 'https://raw.githubusercontent.com/XDSCRIPTER/DefaultCFG.json/refs/heads/main/TweensCFG1.json'

--Auto CampFire
local AutoCamFire = main:CreateToggle("AutoCamFire", { Title = "Auto Camp Fire", Default = false })
local ItemsToCamp = main:CreateDropdown("ItemsToCamp", {Title = "Select Items To Campfire",Values = {"Log",'Leaves', 'Wood', 'Coal'}, Default = "Wood"})
 
if not workspace:FindFirstChild("DotsFolder") then 
    local foldFordots = Instance.new("Model")
    foldFordots.Name = "DotsFolder"
    foldFordots.Parent = workspace
end
 
if not workspace.DotsFolder:FindFirstChild("Counter") then 
    local Counter = Instance.new("IntValue")
    Counter.Name = "Counter"
    Counter.Parent = workspace.DotsFolder
    Counter.Value = 0
end
 
CFGSect:CreateButton({Title = "GoldFarm Config", Callback = function() 
    local receivedData = request({ Url = javaUrl, Method = 'GET' })
    local body = receivedData.Body
    local WaypointData = httpService:JSONDecode(body)
    for index, position in WaypointData.position do
        position = Vector3.new(position.X, position.Y, position.Z)
        if position ~= Vector3.new(0, 0, 0) then
            local Dot = Instance.new("Part")
            Dot.Parent = workspace.DotsFolder
            Dot.Anchored = true 
            Dot.Position = position
            Dot.Name = "Dot"
            Dot.Size = Vector3.new(1, 1, 1)
            Dot.Color = Color3.fromRGB(0, 255, 130)
            Dot.CanCollide = false
            Dot.Material = Enum.Material.Neon
            Dot.Shape = Enum.PartType.Ball
            Dot.Transparency = 0
            workspace.DotsFolder.Counter.Value = workspace.DotsFolder.Counter.Value + 1
        end
    end
end})
 
local function swingtool(entityid)
    if packets.SwingTool and packets.SwingTool.send and plr.Character then
        packets.SwingTool.send(entityid)
        local animation = Instance.new("Animation")
        animation.AnimationId = "rbxassetid://10761451679"
        local humanoid = plr.Character:FindFirstChild("Humanoid")
        if humanoid then
            local track = humanoid:LoadAnimation(animation)
            track:Play()
        end
    end
end
 
local function pickup(entityid)
    if packets.Pickup and packets.Pickup.send then
        packets.Pickup.send(entityid)
    end
end
 
local function moveToWaypointStand(waypoint)
    local humanoid = game:GetService("Players").LocalPlayer.Character and game:GetService("Players").LocalPlayer.Character:FindFirstChild("Humanoid")
    if humanoid then
        humanoid:MoveTo(waypoint.Position)
    end
end
 
local function moveToWaypointsStand()
    if game:GetService("Players").LocalPlayer and game:GetService("Players").LocalPlayer.Character and workspace:FindFirstChild("DotsFolder") then
        for _, waypoint in ipairs(workspace.DotsFolder:GetChildren()) do
            if waypoint.Name ~= "Counter" and waypoint.Name ~= "Highlight" and waypoint.Name ~= "HowMuch" then
                local humanoid = game:GetService("Players").LocalPlayer.Character:FindFirstChild("Humanoid")
                if humanoid then
                    moveToWaypointStand(waypoint)
                    humanoid.MoveToFinished:Wait()
                end
            end
        end
    end
end
 
startMOVE:OnChanged(function()
    if Options.StartMoving.Value then
        moveToWaypointsStand()
        task.spawn(function()
            while Options.StartMoving.Value do
                moveToWaypointsStand()
                task.wait(0.1)
            end
            local humanoid = game:GetService("Players").LocalPlayer.Character and game:GetService("Players").LocalPlayer.Character:FindFirstChild("Humanoid")
            if humanoid and game:GetService("Players").LocalPlayer.Character:FindFirstChild("HumanoidRootPart") then
                humanoid:MoveTo(game:GetService("Players").LocalPlayer.Character.HumanoidRootPart.Position)
            end
        end)
    end
end)
 
local wscon
local function updws()
    if wscon then wscon:Disconnect() end
    if Options.wstoggle.Value and hum then
        wscon = runs.RenderStepped:Connect(function()
            hum.WalkSpeed = Options.wsslider.Value
        end)
    elseif hum then
        hum.WalkSpeed = 16
    end
end
 
local slopecon
local function updmsa()
    if slopecon then slopecon:Disconnect() end
    if Options.msatoggle.Value and hum then
        slopecon = runs.RenderStepped:Connect(function()
            hum.MaxSlopeAngle = 90
        end)
    elseif hum then
        hum.MaxSlopeAngle = 46
    end
end
 
local function onplradded(newChar)
    char = newChar
    root = char:WaitForChild("HumanoidRootPart", 5)
    hum = char:WaitForChild("Humanoid", 5)
    if hum then
        updws()
        updmsa()
    end
end
 
plr.CharacterAdded:Connect(onplradded)
if hum then
    updws()
    updmsa()
end
Options.wstoggle:OnChanged(updws)
Options.msatoggle:OnChanged(updmsa)
 
local function getlayout(itemname)
    local inventory = plr.PlayerGui:FindFirstChild("MainGui") and plr.PlayerGui.MainGui:FindFirstChild("RightPanel") and plr.PlayerGui.MainGui.RightPanel:FindFirstChild("Inventory") and plr.PlayerGui.MainGui.RightPanel.Inventory:FindFirstChild("List")
    if inventory then
        for _, child in ipairs(inventory:GetChildren()) do
            if child:IsA("ImageLabel") and child.Name == itemname then
                return child.LayoutOrder
            end
        end
    end
end
 
local function useItem(itemname)
    local layout = getlayout(itemname)
    if layout and packets.UseBagItem and packets.UseBagItem.send then
        packets.UseBagItem.send(layout)
    end
end
 
runs.Heartbeat:Connect(function()
    if Options.EatToggle.Value and hum and hum.Health <= Options.HealthCount.Value then
        useItem(Options.eatdropdown.Value)
    end
end)
 
task.spawn(function()
    while true do
        if not Options.resourceauratoggle.Value then
            task.wait(0.1)
            continue
        end
        local range = 20
        local targetcount = 6
        local cooldown = 0.1
        local targets = {}
        for _, res in ipairs(workspace.Resources:GetChildren()) do
            if res:IsA("Model") and res:GetAttribute("EntityID") then
                local eid = res:GetAttribute("EntityID")
                local ppart = res.PrimaryPart or res:FindFirstChildWhichIsA("BasePart")
                if ppart then
                    local dist = (ppart.Position - root.Position).Magnitude
                    if dist <= range then
                        table.insert(targets, { eid = eid, dist = dist })
                    end
                end
            end
        end
        if #targets > 0 then
            table.sort(targets, function(a, b)
                return a.dist < b.dist
            end)
            local selectedtargets = {}
            for i = 1, math.min(targetcount, #targets) do
                table.insert(selectedtargets, targets[i].eid)
            end
            swingtool(selectedtargets)
        end
        task.wait(cooldown)
    end
end)
 
task.spawn(function()
    while true do
        local range = 35
        if Options.autopickuptoggle.Value then
            for _, item in ipairs(workspace.Items:GetChildren()) do
                if (item:IsA("BasePart") or item:IsA("MeshPart")) and item.Name == "Raw Gold" then
                    local entityid = item:GetAttribute("EntityID")
                    if entityid then
                        local dist = (item.Position - root.Position).Magnitude
                        if dist <= range then
                            pickup(entityid)
                        end
                    end
                end
            end
        end
        task.wait(0.01)
    end
end)

SaveManager:SetLibrary(Library)
InterfaceManager:SetLibrary(Library)
SaveManager:IgnoreThemeSettings()
SaveManager:SetIgnoreIndexes{}
InterfaceManager:SetFolder("FluentScriptHub")
SaveManager:SetFolder("FluentScriptHub/specific-game")
InterfaceManager:BuildInterfaceSection(Tabs.Settings)
SaveManager:BuildConfigSection(Tabs.Settings)
Window:SelectTab(1)
SaveManager:LoadAutoloadConfig()
