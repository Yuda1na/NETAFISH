local env = (getgenv and getgenv()) or _G
if env.NetAFish then
    pcall(env.NetAFish.unload)
end

local Players = game:GetService("Players")
local RS = game:GetService("ReplicatedStorage")
local VirtualUser = game:GetService("VirtualUser")
local plr = Players.LocalPlayer
local remotes = RS:WaitForChild("Remotes", 10)
assert(remotes, "Net a Fish remotes not found")
local Shared = require(RS:WaitForChild("FishingShared"))
local Tree = require(RS:WaitForChild("SkillTree"))
local Rayfield = (isfile and isfile("rayfield_src.lua") and loadstring(readfile("rayfield_src.lua"))(
)) or loadstring(game:HttpGet("https://sirius.menu/gen2"))()

local alive, connections = true, {}
local S = {
    fish = false, resolve = false, keepRarities = {}, best = false, upgrades = false, nets = false,
    spear = false, helper = false, boat = false, skills = false, skill = "Weave",
    skillMode = "Smart", skillBranch = "Catch", quests = false, gifts = false, spin = false,
    index = false, rebirth = false, antiAFK = false, pass = false, collect = false, traits = false,
    buyPotions = false, drinkPotions = false,
    potionKinds = {Net = true, Cash = true, Caught = true, Size = true, Mut = true}
}
local rarityNames, rarityById, defaultRarities = {}, {}, {}
for i, rarity in ipairs(Shared.Rarities or {}) do
    local name = rarity.Name or tostring(i)
    rarityNames[#rarityNames + 1] = name
    rarityById[i] = name
    if i >= 3 then
        S.keepRarities[name] = true;
        defaultRarities[#defaultRarities + 1] = name
    end
end
local lastCast, lastReel, lastBest = 0, 0, 0
local window = Rayfield:CreateWindow(
    {
        name = "Net a Fish", subtitle = "by z3bd", sidebarLayout = true,
        configuration = {autoSave = true, autoLoad = true, fileName = "NetAFish"}
    }
)

local function notify(title, text)
    pcall(
        function()
            window:Notify({title = title, content = tostring(text), duration = 4})
        end
    )
end
local function call(name, ...)
    local remote = remotes:FindFirstChild(name)
    if not remote then
        return nil
    end
    local ok, result
    if remote:IsA("RemoteFunction") then
        ok, result = pcall(remote.InvokeServer, remote, ...)
    elseif remote:IsA("RemoteEvent") then
        ok, result = pcall(remote.FireServer, remote, ...)
    else
        return nil
    end
    if not ok then
        warn("Net a Fish / " .. name .. ": " .. tostring(result));
        return nil
    end
    return result
end
local function connect(signal, fn)
    local conn = signal:Connect(fn)
    table.insert(connections, conn)
    return conn
end
local function ownBase()
    local map = workspace:FindFirstChild("Map")
    local bases = map and map:FindFirstChild("Bases")
    if not bases then
        return nil
    end
    for _, base in ipairs(bases:GetChildren()) do
        if base:GetAttribute("OwnerUserId") == plr.UserId then
            return base
        end
    end
    return nil
end

local fishTab = window:CreateTab({name = "Fishing", icon = "anchor"})
fishTab:CreateToggle(
    {
        name = "Auto Cast & Reel Full Net", value = false,
        callback = function(v)
            S.fish = v
        end
    }
)
fishTab:CreateToggle(
    {
        name = "Auto Resolve Catch", value = false,
        callback = function(v)
            S.resolve = v
        end
    }
)
fishTab:CreateDropdown(
    {
        name = "Keep Rarities", options = rarityNames, value = defaultRarities, multiSelect = true,
        callback = function(values)
            table.clear(S.keepRarities)
            if type(values) == "table" then
                for _, name in ipairs(values) do
                    S.keepRarities[name] = true
                end
            end
        end
    }
)
fishTab:CreateText(
    {
        name = "Catch Decisions",
        text = "Keeps new, perfect, locked, unknown-rarity and selected rare fish. Other fish go to the cooler."
    }
)
fishTab:CreateToggle(
    {
        name = "Auto Best Setup", value = false,
        callback = function(v)
            S.best = v
        end
    }
)
fishTab:CreateToggle(
    {
        name = "Auto Collect Cash", value = false,
        callback = function(v)
            S.collect = v
        end
    }
)

local upTab = window:CreateTab({name = "Upgrades", icon = "trending-up"})
upTab:CreateToggle(
    {
        name = "Auto Rebirth", value = false,
        callback = function(v)
            S.rebirth = v
        end
    }
)
upTab:CreateToggle(
    {
        name = "Auto Upgrade Net", value = false,
        callback = function(v)
            S.nets = v
        end
    }
)
upTab:CreateToggle(
    {
        name = "Fishing & Market Upgrades", value = false,
        callback = function(v)
            S.upgrades = v
        end
    }
)
upTab:CreateToggle(
    {
        name = "Spear Gun Upgrades", value = false,
        callback = function(v)
            S.spear = v
        end
    }
)
upTab:CreateToggle(
    {
        name = "Fisherman Upgrades", value = false,
        callback = function(v)
            S.helper = v
        end
    }
)
upTab:CreateToggle(
    {
        name = "Boat Upgrades", value = false,
        callback = function(v)
            S.boat = v
        end
    }
)
local skillIds = {}
for _, node in ipairs(Tree.Nodes or {}) do
    if type(node.Id) == "string" and not node.Legacy then
        table.insert(skillIds, node.Id)
    end
end
table.sort(skillIds)
if #skillIds > 0 then
    S.skill = skillIds[1]
end
upTab:CreateDropdown(
    {
        name = "Skill To Upgrade", options = skillIds, value = S.skill,
        callback = function(v)
            local value = type(v) == "table" and v[1] or v
            if type(value) == "string" then
                S.skill = value
            end
        end
    }
)
upTab:CreateDropdown(
    {
        name = "Skill Mode", options = {"Smart", "Branch", "Single"}, value = "Smart",
        callback = function(v)
            v = type(v) == "table" and v[1] or v
            if v == "Smart" or v == "Branch" or v == "Single" then
                S.skillMode = v
            end
        end
    }
)
local branchNames = {}
for name in pairs(Tree.ByBranch or {}) do
    branchNames[#branchNames + 1] = name
end
table.sort(branchNames)
if #branchNames > 0 then
    S.skillBranch = branchNames[1]
end
upTab:CreateDropdown(
    {
        name = "Skill Branch", options = branchNames, value = S.skillBranch,
        callback = function(v)
            v = type(v) == "table" and v[1] or v
            if type(v) == "string" then
                S.skillBranch = v
            end
        end
    }
)
upTab:CreateToggle(
    {
        name = "Auto Upgrade Selected Skill", value = false,
        callback = function(v)
            S.skills = v
        end
    }
)
upTab:CreateText(
    {
        name = "Spending",
        text = "Upgrade toggles spend in-game currency or skill points. The server checks costs and unlocks."
    }
)

local rewardTab = window:CreateTab({name = "Rewards", icon = "gift"})
local potionTab = window:CreateTab({name = "Potions", icon = "flask-conical"})
local potionOptions = {"Net", "Cash", "Caught", "Size", "Mut"}
potionTab:CreateDropdown(
    {
        name = "Potion Types", options = potionOptions, value = table.clone(potionOptions),
        multiSelect = true,
        callback = function(values)
            table.clear(S.potionKinds)
            if type(values) == "table" then
                for _, value in ipairs(values) do
                    S.potionKinds[value] = true
                end
            end
        end
    }
)
potionTab:CreateToggle(
    {
        name = "Auto Buy Potions", value = false,
        callback = function(v)
            S.buyPotions = v
        end
    }
)
potionTab:CreateToggle(
    {
        name = "Auto Drink Potions", value = false,
        callback = function(v)
            S.drinkPotions = v
        end
    }
)
rewardTab:CreateToggle(
    {
        name = "Auto Claim Quests", value = false,
        callback = function(v)
            S.quests = v
        end
    }
)
rewardTab:CreateText(
    {name = "Quests", text = "Claims completed contracts and Today challenge rewards."}
)
rewardTab:CreateToggle(
    {
        name = "Auto Claim Online & Daily", value = false,
        callback = function(v)
            S.gifts = v
        end
    }
)
rewardTab:CreateToggle(
    {
        name = "Auto Spin", value = false,
        callback = function(v)
            S.spin = v
        end
    }
)
rewardTab:CreateToggle(
    {
        name = "Auto Claim Index Pages", value = false,
        callback = function(v)
            S.index = v
        end
    }
)
rewardTab:CreateToggle(
    {
        name = "Auto Claim Leviathan Pass", value = false,
        callback = function(v)
            S.pass = v
        end
    }
)
rewardTab:CreateToggle(
    {
        name = "Auto Roll Traits", value = false,
        callback = function(v)
            S.traits = v
        end
    }
)
local code = ""
rewardTab:CreateInput(
    {
        name = "Code", value = "",
        callback = function(v)
            code = tostring(v or "")
        end
    }
)
rewardTab:CreateButton(
    {
        name = "Redeem Code",
        callback = function()
            if code:match("%S") then
                local result = call("Rewards", "redeem", code);
                notify(
                    "Code",
                    type(result) == "table" and (result.message or result.err or (result.ok and "Request accepted") or "Request rejected") or "No response"
                )
            end
        end
    }
)
local miscTab = window:CreateTab({name = "Settings", icon = "settings"})
local oldStayAFK, stayAFKOwned
miscTab:CreateToggle(
    {
        name = "Anti AFK", value = false,
        callback = function(v)
            S.antiAFK = v
            if v then
                if not stayAFKOwned then
                    oldStayAFK, stayAFKOwned = plr:GetAttribute("Set_StayAFK"), true
                end
                plr:SetAttribute("Set_StayAFK", true)
            elseif stayAFKOwned then
                plr:SetAttribute("Set_StayAFK", oldStayAFK)
                stayAFKOwned = false
            end
        end
    }
)
local creditsTab = window:CreateTab({name = "Credits", icon = "award"})
creditsTab:CreateButton(
    {
        name = "Made by z3bd",
        callback = function()
            local copy = setclipboard or toclipboard
            if copy then
                copy("@z3bd");
                notify("Credits", "Copied @z3bd")
            end
        end
    }
)
creditsTab:CreateButton(
    {
        name = "Discord Server",
        callback = function()
            local copy = setclipboard or toclipboard
            if copy then
                copy("https://discord.gg/S832pF3RCT");
                notify("Credits", "Copied Discord link")
            end
        end
    }
)
local eventTab = window:CreateTab({name = "Events", icon = "calendar"})
local eventInfo = eventTab:CreateText({name = "River Events", text = "Waiting for event updates."})
local riverEvents = remotes:FindFirstChild("EventRecap")
if riverEvents and riverEvents:IsA("RemoteEvent") then
    connect(
        riverEvents.OnClientEvent,
        function(...)
            local values = {...}
            local out = {}
            for _, value in ipairs(values) do
                if type(value) == "string" or type(value) == "number" then
                    out[#out + 1] = tostring(value)
                end
            end
            if #out > 0 then
                pcall(eventInfo.Set, eventInfo, table.concat(out, " / "))
            end
        end
    )
end
connect(
    plr.Idled,
    function()
        if S.antiAFK then
            pcall(
                function()
                    VirtualUser:CaptureController();
                    VirtualUser:ClickButton2(Vector2.zero)
                end
            )
        end
    end
)
local catchRemote = remotes:FindFirstChild("CatchReady")
if catchRemote and catchRemote:IsA("RemoteEvent") then
    connect(
        catchRemote.OnClientEvent,
        function(fish)
            if not alive or not S.resolve or type(fish) ~= "table" then
                return
            end
            local decisions = {}
            for _, item in ipairs(fish) do
                if type(item) == "table" and item.uid ~= nil then
                    local rarity = tonumber(item.r)
                    local name = rarity and rarityById[rarity]
                    local keep = not name or S.keepRarities[name] or item.locked or item.lock or item.pb or item.mu or item.new or item.perfect
                    decisions[tostring(item.uid)] = keep and "keep" or "sell"
                end
            end
            if next(decisions) then
                call("ResolveCatch", decisions)
            end
        end
    )
end

task.spawn(
    function()
        while alive do
            if S.fish then
                local base = ownBase()
                if base then
                    local state = base:GetAttribute("NetState")
                    local now = os.clock()
                    if state == "Idle" and now - lastCast > 2 then
                        lastCast = now;
                        call("ThrowNet")
                    elseif state == "Fishing" and (base:GetAttribute("NetCount") or 0) >= (base:GetAttribute(
                        "NetCap"
                    ) or math.huge) and now - lastReel > 2 then
                        lastReel = now;
                        call("ReelIn")
                    end
                end
            end
            if S.best and os.clock() - lastBest > 15 then
                lastBest = os.clock();
                call("BestSetup")
            end
            task.wait(0.5)
        end
    end
)
task.spawn(
    function()
        local last = {}
        local currentSlot
        while alive do
            if S.collect then
                local base = ownBase()
                local pedestals = base and base:FindFirstChild("Pedestals")
                local char = plr.Character
                local hrp = char and char:FindFirstChild("HumanoidRootPart")
                local hum = char and char:FindFirstChildOfClass("Humanoid")
                if pedestals and hrp and hum and hum.Health > 0 and not hrp.Anchored and not plr:GetAttribute(
                    "InMiniGame"
                ) then
                    local lower = -math.max(5, (hum and hum.HipHeight or 2) + hrp.Size.Y / 2 + 2.5)
                    local target, targetSlot, nearest = nil, nil, math.huge
                    for _, ped in ipairs(pedestals:GetChildren()) do
                        local collect = ped:FindFirstChild("Collect")
                        local pad = collect and collect:FindFirstChild("Pad")
                        local slot = ped:GetAttribute("Slot")
                        if pad and pad:IsA("BasePart") and slot ~= nil then
                            local delta = pad.Position - hrp.Position
                            local distance = Vector2.new(delta.X, delta.Z).Magnitude
                            if os.clock() - (last[slot] or 0) >= 15 then
                                if distance < 2.6 and delta.Y > lower and delta.Y < 1 then
                                    last[slot] = os.clock()
                                    call("CollectPadReq", slot)
                                elseif distance < nearest then
                                    target, targetSlot, nearest = pad, slot, distance
                                end
                            end
                        end
                    end
                    if target then
                        currentSlot = targetSlot
                        hum:MoveTo(target.Position)
                    else
                        currentSlot = nil
                    end
                end
            elseif currentSlot then
                local char = plr.Character
                local hum = char and char:FindFirstChildOfClass("Humanoid")
                local hrp = char and char:FindFirstChild("HumanoidRootPart")
                if hum and hrp then
                    hum:MoveTo(hrp.Position)
                end
                currentSlot = nil
            end
            task.wait(0.5)
        end
    end
)
task.spawn(
    function()
        while alive do
            local data = (S.upgrades or S.boat or S.skills) and call("GetData") or nil
            if S.nets then
                local tier = plr:GetAttribute("NetTier")
                if type(tier) == "number" and Shared.Nets and Shared.Nets[tier + 1] then
                    call("BuyItem", "Net", tier + 1)
                end
            end
            if S.upgrades and type(data) == "table" then
                local priorities = {
                    Weave = 1, Reel = 2, Luck = 3, BigBait = 4, Twin = 5, Golden = 6, Bulk = 7,
                    Treasure = 8, Sell = 9, Walk = 10
                }
                local best, score
                for _, upgrade in ipairs(Shared.Upgrades or {}) do
                    local id = upgrade.Id
                    local rank = priorities[id] or 14
                    local level = type(data.Upgrades) == "table" and (data.Upgrades[id] or 0) or 0
                    if not upgrade.Hidden and (upgrade.Tab == "Fishing" or upgrade.Tab == "Market") and level < (upgrade.Max or 0) and (data.Rebirths or 0) >= (upgrade.Rebirth or 0) then
                        local ok, cost = pcall(Shared.UpgradeCost, id, level)
                        if ok and type(cost) == "number" and cost <= (data.Cash or 0) then
                            local value = rank * 0.25 + level + math.log10(math.max(cost, 1)) * 0.08
                            if not score or value < score then
                                best, score = id, value
                            end
                        end
                    end
                end
                if best then
                    call("BuyItem", "Upgrade", best)
                end
            end
            if S.boat and type(data) == "table" and type(data.Boat) == "table" then
                local hull = Shared.Boat and Shared.Boat.Hulls and Shared.Boat.Hulls[(data.Boat.Hull or 0) + 1]
                if hull and (data.Rebirths or 0) >= (hull.Rebirth or 0) and (data.Cash or 0) >= (hull.Cost or math.huge) then
                    call("BoatAction", "buy", "Hull")
                end
            end
            if S.skills and type(data) == "table" then
                local choice = nil
                if S.skillMode == "Single" then
                    if Tree.canBuy(data, S.skill) then
                        choice = S.skill
                    end
                else
                    local bestScore
                    local weights = {
                        Weave = 0, Reel = 1, Luck = 2, BigBait = 3, Drills = 5, Aim = 6, Haggle = 7
                    }
                    for _, node in ipairs(Tree.Nodes or {}) do
                        if not node.Legacy and (S.skillMode ~= "Branch" or node.Branch == S.skillBranch) and Tree.canBuy(
                            data, node.Id
                        ) then
                            local level = Tree.level(data, node.Id)
                            local score = (weights[node.Id] or (12 + (node.Order or 0) * 0.1)) + level * 2
                            if not bestScore or score < bestScore then
                                choice, bestScore = node.Id, score
                            end
                        end
                    end
                end
                if choice then
                    call("SkillTree", "buy", choice)
                end
            end
            task.wait(2)
        end
    end
)
for _, spec in ipairs(
    {
        {"Spear", "SpearAction", "spear"}, {"Helper", "HelperAction", "helper"},
        {"Boat", "BoatAction", "boat"}
    }
) do
    task.spawn(
        function()
            local index = 1
            while alive do
                local upgrades = Shared[spec[1]] and Shared[spec[1]].Upgrades
                if S[spec[3]] and upgrades and #upgrades > 0 then
                    local upgrade = upgrades[index]
                    if upgrade then
                        call(spec[2], "buy", upgrade.Id)
                    end
                    index = index % #upgrades + 1
                end
                task.wait(1)
            end
        end
    )
end
task.spawn(
    function()
        while alive do
            if S.rebirth then
                local data = call("GetData")
                if type(data) == "table" then
                    local ok, eligible = pcall(Shared.CanRebirth, data)
                    if ok and eligible and alive and S.rebirth then
                        call("RebirthRequest")
                    end
                end
            end
            task.wait(2)
        end
    end
)
task.spawn(
    function()
        while alive do
            if S.quests then
                local state = call("Contracts", "state")
                if type(state) == "table" and type(state.list) == "table" then
                    for i = 1, #state.list do
                        if not alive or not S.quests then
                            break
                        end
                        local quest = state.list[i]
                        local progress = type(quest) == "table" and tonumber(quest.p)
                        local goal = type(quest) == "table" and tonumber(quest.goal)
                        if quest and quest.k ~= "wait" and progress and goal and progress >= goal then
                            local result = call("Contracts", "claim", i)
                            if type(result) == "table" and type(result.state) == "table" and type(
                                result.state.list
                            ) == "table" then
                                state = result.state
                            end
                            task.wait(0.5)
                        end
                    end
                end
            end
            task.wait(5)
        end
    end
)
task.spawn(
    function()
        while alive do
            if S.quests then
                local state = call("Challenge", "state")
                if type(state) == "table" and type(state.tiers) == "table" then
                    for i, tier in ipairs(state.tiers) do
                        if not alive or not S.quests then
                            break
                        end
                        if type(tier) == "table" and tier.done and not tier.claimed then
                            local result = call("Challenge", "claim", i)
                            if type(result) == "table" and result.ok == false then
                                warn(
                                    "Net a Fish / Today claim: " .. tostring(
                                        result.err or "rejected"
                                    )
                                )
                            end
                            task.wait(0.5)
                        end
                    end
                end
            end
            task.wait(5)
        end
    end
)
task.spawn(
    function()
        while alive do
            if S.gifts then
                local state = call("Rewards", "state")
                if type(state) == "table" then
                    if state.daily and state.daily.claimable then
                        call("Rewards", "claimDaily")
                    end
                    for i, gift in ipairs(state.gifts or {}) do
                        if not alive or not S.gifts then
                            break
                        end
                        if gift.ready and not gift.claimed then
                            call("Rewards", "claimGift", i);
                            task.wait(0.5)
                        end
                    end
                end
            end
            if S.spin then
                local state = call("Retention", "state")
                if type(state) == "table" and state.spin and (state.spin.spins or 0) > 0 then
                    call("Retention", "spin")
                end
            end
            if S.index then
                local data = call("GetData")
                if type(data) == "table" then
                    local ok, pages = pcall(Shared.IndexPagesDone, data)
                    if ok and type(pages) == "table" then
                        for _, id in ipairs(pages) do
                            if not alive or not S.index then
                                break
                            end
                            if type(data.IndexClaims) ~= "table" or not data.IndexClaims[tostring(
                                id
                            )] then
                                call("Rewards", "claimPage", id);
                                task.wait(0.5)
                            end
                        end
                    end
                end
                local index = call("Rewards", "index")
                if type(index) == "table" then
                    for id, page in pairs(index) do
                        if not alive or not S.index then
                            break
                        end
                        if type(page) == "table" and page.claimable then
                            call("Rewards", "claimIndex", id);
                            task.wait(0.5)
                        end
                    end
                end
            end
            if S.pass then
                local pass = call("SeasonPass", "state")
                if type(pass) == "table" and type(pass.tier) == "number" then
                    local free = type(pass.free) == "table" and pass.free or {}
                    local gold = type(pass.gold) == "table" and pass.gold or {}
                    local claimable = false
                    for i = 1, pass.tier do
                        if not free[tostring(i)] or (pass.vip and not gold[tostring(i)]) then
                            claimable = true;
                            break
                        end
                    end
                    if claimable and alive and S.pass then
                        call("SeasonPass", "claimAll")
                    end
                end
            end
            task.wait(15)
        end
    end
)
task.spawn(
    function()
        while alive do
            if S.traits then
                local state = call("Traits", "state")
                if type(state) == "table" and (tonumber(state.rolls) or 0) > 0 and alive and S.traits then
                    call("Traits", "roll", 1)
                end
            end
            task.wait(3)
        end
    end
)
task.spawn(
    function()
        while alive do
            if S.buyPotions or S.drinkPotions then
                local shop = call("Potions", "state")
                if type(shop) == "table" then
                    if S.buyPotions and type(shop.stock) == "table" then
                        for index, item in ipairs(shop.stock) do
                            if not alive or not S.buyPotions then
                                break
                            end
                            if type(item) == "table" and S.potionKinds[item.kind] and not item.bought and type(
                                item.price
                            ) == "number" and item.price <= (tonumber(shop.cash) or 0) then
                                local result = call("Potions", "buy", index)
                                if type(result) == "table" and type(result.state) == "table" then
                                    shop = result.state
                                end
                                task.wait(0.6)
                            end
                        end
                    end
                    if S.drinkPotions then
                        local data = call("GetData")
                        local inv = type(data) == "table" and data.PotionInv
                        if type(inv) == "table" then
                            local best = {}
                            for key, count in pairs(inv) do
                                local kind, tier = tostring(key):match("^(%a+)_(%d+)$")
                                tier = tonumber(tier)
                                if kind and tier and S.potionKinds[kind] and (tonumber(count) or 0) > 0 and (not best[kind] or tier > best[kind].tier) then
                                    best[kind] = {key = key, tier = tier}
                                end
                            end
                            for kind, item in pairs(best) do
                                if not alive or not S.drinkPotions then
                                    break
                                end
                                local active = type(data.Potions) == "table" and data.Potions[kind]
                                if type(active) ~= "table" or (tonumber(active.e) or 0) <= workspace:GetServerTimeNow(
                                ) then
                                    call("Potions", "use", item.key);
                                    task.wait(0.6)
                                end
                            end
                        end
                    end
                end
            end
            task.wait(15)
        end
    end
)
local Script = {settings = S}
function Script.unload()
    if not alive then
        return
    end
    alive = false
    if stayAFKOwned then
        plr:SetAttribute("Set_StayAFK", oldStayAFK);
        stayAFKOwned = false
    end
    for _, conn in ipairs(connections) do
        conn:Disconnect()
    end
    table.clear(connections)
    if window and window.Unload then
        pcall(window.Unload, window)
    end
    if env.NetAFish == Script then
        env.NetAFish = nil
    end
end
miscTab:CreateButton({name = "Unload", callback = Script.unload})
env.NetAFish = Script
