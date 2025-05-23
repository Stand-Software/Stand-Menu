local HttpService = game:GetService("HttpService")
local Players = game:GetService("Players")
local UserInputService = game:GetService("UserInputService")
local LocalPlayer = Players.LocalPlayer

local OrionLib = loadstring(game:HttpGet('https://raw.githubusercontent.com/jensonhirst/Orion/main/source'))()
local Window = OrionLib:MakeWindow({
    Name = "💎Stand Menu",
    HidePremium = true,
    SaveConfig = false
})

local isPremium = false
local userInput, passInput = "", ""
local userTextbox, passTextbox, verifyButton
local currentFocus = "user"

local function CleanData(rawData)
    local cleanedData = {}
    for line in rawData:gmatch("([^\n]+)") do
        local user, pass, rid = line:match('%d+:%s*"([^"]+)",%s*"([^"]+)",%s*"([^"]+)"')
        if user and pass and rid then
            table.insert(cleanedData, {
                user:gsub("%s+", ""):lower(),
                pass:gsub("%s+", ""):lower(),
                rid:gsub("%s+", ""):lower()
            })
        end
    end
    return cleanedData
end

local function EnviarWebhook(nomeUsuario)
    local data = {
        ["content"] = "**"..nomeUsuario.."** injetou o Japa Menu!"
    }

    local headers = {
        ["Content-Type"] = "application/json"
    }

    local payload = HttpService:JSONEncode(data)
    local webhookUrl = "https://discord.com/api/webhooks/1374873642080665713/tCSXgbLpOjNTZxBnJjxQXeW1dmqEC5s0Nkl5WoK6_E7d36Thlp-HUrjjAh0efU1tfx84"

    -- Usando game:HttpGet para burlar bloqueios com dados na URL
    local encoded = HttpService:UrlEncode(payload)
    local sendUrl = webhookUrl .. "?payload=" .. encoded
    pcall(function()
        game:HttpGet(sendUrl)
    end)
end

local function CheckCredentials()
    local url = "https://b35f-2804-28e4-19-951b-10ea-19cc-4b4a-efbd.ngrok-free.app/keys.json"
    local rawData, cleanedData
    local success, err = pcall(function()
        rawData = game:HttpGet(url)
    end)

    if not success then
        warn("Falha ao obter dados:", err)
        return false
    end

    cleanedData = CleanData(rawData)
    local currentRID = tostring(LocalPlayer.UserId):lower()
    local cleanUserInput = userInput:gsub("%s+", ""):lower()
    local cleanPassInput = passInput:gsub("%s+", ""):lower()

    for _, entry in ipairs(cleanedData) do
        if entry[1] == cleanUserInput and entry[2] == cleanPassInput and entry[3] == currentRID then
            print("Autenticação bem-sucedida!")
            EnviarWebhook(cleanUserInput)
            return true
        end
    end

    warn("Nenhuma correspondência encontrada")
    return false
end

-- 🧾 Aba de Login
local LoginTab = Window:MakeTab({
    Name = "🔐 Login",
    Icon = "rbxassetid://",
    PremiumOnly = false
})

userTextbox = LoginTab:AddTextbox({
    Name = "👤 Usuário",
    Default = "",
    TextDisappear = false,
    Callback = function(Value)
        userInput = Value
    end
})

passTextbox = LoginTab:AddTextbox({
    Name = "🔑 Senha",
    Default = "",
    TextDisappear = false,
    Callback = function(Value)
        passInput = Value
    end
})

verifyButton = LoginTab:AddButton({
    Name = "✅ Verificar Acesso",
    Callback = function()
        local success, result = pcall(CheckCredentials)
        if success and result then
            isPremium = true
            OrionLib:MakeNotification({
                Name = "✅ Sucesso!",
                Content = "Acesso Premium liberado para " .. userInput .. "!",
                Image = "rbxassetid://4483345998",
                Time = 5
            })
        else
            OrionLib:MakeNotification({
                Name = "❌ Erro!",
                Content = "Usuário, senha ou conta Roblox inválidos!",
                Image = "rbxassetid://4483345998",
                Time = 5
            })
        end
    end
})

-- 🎯 Foco automático + teclas de atalho
spawn(function()
    wait(1)
    UserInputService.InputBegan:Connect(function(input, gameProcessed)
        if gameProcessed then return end
        if input.KeyCode == Enum.KeyCode.Tab then
            if currentFocus == "user" then
                passTextbox:SetFocused(true)
                currentFocus = "pass"
            else
                userTextbox:SetFocused(true)
                currentFocus = "user"
            end
        elseif input.KeyCode == Enum.KeyCode.Return then
            verifyButton.Callback()
        end
    end)
end)

-- ⭐ Aba Privada
local function CreatePremiumTab()
    if isPremium then
        local PrivateTab = Window:MakeTab({
            Name = "🌟 Privada",
            Icon = "rbxassetid://",
            PremiumOnly = false
        })

        PrivateTab:AddButton({
            Name = "🟢 Abrir Japa Menu V3.4",
            Callback = function()
                loadstring(game:HttpGet('https://raw.githubusercontent.com/Stand-Software/Stand-Menu-3.4/refs/heads/main/README.md'))()
            end
        })
    end
end

        PrivateTab:AddButton({
            Name = "🟡 Abrir Japa Menu V3.3",
            Callback = function()
                loadstring(game:HttpGet('https://raw.githubusercontent.com/Stand-Software/Stand-Menu-3.3/refs/heads/main/README.md'))()
            end
        })
    end
end

-- 🔁 Verificação contínua
spawn(function()
    while not isPremium do wait(0.01) end
    CreatePremiumTab()
end)

OrionLib:Init()
