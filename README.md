--[[
    VOLLEYBALL LEGENDS - OPTIMIZED EXPLOIT
    Versão: 2.1.0
    Atualizado: 2026
    Anti-detecção aprimorada
--]]

-- ============================================
-- CONFIGURAÇÕES (EDITÁVEIS)
-- ============================================
local CONFIG = {
    -- Delays (segundos)
    minDelay = 1.8,
    maxDelay = 4.2,
    minPause = 8,
    maxPause = 20,
    attemptsBeforePause = 5,
    
    -- UI
    uiName = "🛡️ Optimized Collector",
    uiIcon = "rbxassetid://4483345998",
    
    -- Anti-detecção
    jitter = true,
    randomMethodRotation = true,
    
    -- Webhook para logs (opcional, deixe vazio para desativar)
    webhookURL = "", -- Coloque seu webhook do Discord aqui se quiser logs
    
    -- Modo silencioso (menos notificações)
    silentMode = false
}

-- ============================================
-- BIBLIOTECA DE UI (ORION)
-- ============================================
local OrionLib = loadstring(game:HttpGet('https://raw.githubusercontent.com/shlexware/Orion/main/source'))()

-- ============================================
-- FUNÇÃO PRINCIPAL DO SCRIPT
-- ============================================
local function LoadMainScript()
    -- Variáveis de controle
    local exploitSkill = false
    local exploitStyle = false
    local attemptCounter = 0
    local isPaused = false
    local totalClaims = 0
    
    -- Serviços
    local RS = game:GetService("ReplicatedStorage")
    local Players = game:GetService("Players")
    local LP = Players.LocalPlayer
    
    -- Webhook logger (opcional)
    local function sendLog(message)
        if CONFIG.webhookURL and CONFIG.webhookURL ~= "" then
            pcall(function()
                local data = {
                    content = "**[VolleyLegends Exploit]** " .. message,
                    username = "Exploit Logger"
                }
                game:HttpGet(CONFIG.webhookURL .. "?content=" .. game:HttpService:JSONEncode(data))
            end)
        end
    end
    
    -- Delay humano aprimorado
    local function humanDelay()
        local baseDelay = math.random(CONFIG.minDelay * 100, CONFIG.maxDelay * 100) / 100
        
        if CONFIG.jitter then
            local jitter = baseDelay * math.random(-15, 15) / 100
            baseDelay = baseDelay + jitter
        end
        
        return math.max(0.5, baseDelay)
    end
    
    -- Pausa longa
    local function longPause()
        if isPaused then return end
        
        local pauseDuration = math.random(CONFIG.minPause, CONFIG.maxPause)
        isPaused = true
        
        warn("[ANTI-DETECT] Pausa de " .. pauseDuration .. "s")
        
        task.wait(pauseDuration)
        isPaused = false
    end
    
    -- Verifica atividade do jogador
    local function isPlayerActive()
        local inGame = LP.PlayerGui:FindFirstChild("GameUI") ~= nil
        return inGame
    end
    
    -- Encontrar Knit
    local Knit = nil
    pcall(function()
        for _, v in pairs(RS:GetChildren()) do
            if v.Name and v.Name:lower():find("knit") then
                Knit = require(v)
                break
            end
        end
        if not Knit and debug.getregistry and debug.getregistry().Knit then
            Knit = debug.getregistry().Knit
        end
    end)
    
    -- Executar exploit
    local function executeExploit(type)
        if isPaused then return false end
        if not isPlayerActive() then return false end
        
        local success = false
        
        pcall(function()
            local methodIndex = 1
            if CONFIG.randomMethodRotation then
                methodIndex = math.random(1, 4)
            end
            
            -- MÉTODO 1: Remote direto
            if methodIndex == 1 then
                local remote = RS:FindFirstChild("AwardDailyReward")
                if remote then remote:InvokeServer() end
                
            -- MÉTODO 2: Via Knit
            elseif methodIndex == 2 then
                if Knit and Knit.Services and Knit.Services.GameService then
                    local svc = Knit.Services.GameService
                    if svc.RF and svc.RF.AwardDailyReward then
                        svc.RF.AwardDailyReward:InvokeServer()
                    end
                end
                
            -- MÉTODO 3: Remote alternativo
            elseif methodIndex == 3 then
                local altRemote = RS:FindFirstChild("ClaimLevelReward")
                if altRemote then altRemote:FireServer() end
                
            -- MÉTODO 4: Evento genérico
            elseif methodIndex == 4 then
                local genericEvent = RS:FindFirstChild("RewardClaim")
                if genericEvent then genericEvent:FireServer() end
            end
            
            success = true
            totalClaims = totalClaims + 1
        end)
        
        return success
    end
    
    -- Exploit Skill
    local function exploitSkillTokens()
        if not exploitSkill then return end
        
        attemptCounter = attemptCounter + 1
        
        if attemptCounter >= CONFIG.attemptsBeforePause then
            longPause()
            attemptCounter = 0
        end
        
        local result = executeExploit("skill")
        
        if result then
            task.wait(humanDelay())
        end
    end
    
    -- Exploit Style
    local function exploitStyleTokens()
        if not exploitStyle then return end
        
        pcall(function()
            local styleRemote = RS:FindFirstChild("AwardStyleReward") or RS:FindFirstChild("StyleReward")
            if styleRemote then
                styleRemote:InvokeServer()
            elseif Knit and Knit.Services and Knit.Services.StyleService then
                Knit.Services.StyleService.RF.ClaimReward:InvokeServer()
            end
        end)
        
        task.wait(humanDelay())
    end
    
    -- Criar UI
    local Window = OrionLib:MakeWindow({
        Name = CONFIG.uiName,
        HidePremium = false,
        SaveConfig = true,
        ConfigFolder = "OptimizedCollector"
    })
    
    local MainTab = Window:MakeTab({
        Name = "Collectors",
        Icon = CONFIG.uiIcon
    })
    
    -- Seção de aviso
    MainTab:AddSection({ Name = "⚠️ AVISO DE RISCO ⚠️" })
    
    MainTab:AddParagraph({
        Name = "Este script é DETECTÁVEL",
        Content = "Use por no máximo 10 minutos por sessão.\nNunca deixe ligado enquanto AFK.\nDesative antes de sair de partidas."
    })
    
    -- Botões principais
    MainTab:AddSection({ Name = "🛡️ EXPLOIT MODE" })
    
    MainTab:AddToggle({
        Name = "🔴 Exploit Tokens de Habilidade",
        Default = false,
        Callback = function(v)
            exploitSkill = v
            if v and not CONFIG.silentMode then
                OrionLib:MakeNotification({
                    Name = "⚠️ RISCO ATIVADO",
                    Content = "Modo exploit de habilidade ativo!",
                    Time = 2
                })
            end
            sendLog("Habilidade exploit: " .. tostring(v))
        end
    })
    
    MainTab:AddToggle({
        Name = "🔵 Exploit Tokens de Estilo",
        Default = false,
        Callback = function(v)
            exploitStyle = v
            if v and not CONFIG.silentMode then
                OrionLib:MakeNotification({
                    Name = "⚠️ RISCO ATIVADO",
                    Content = "Modo exploit de estilo ativo!",
                    Time = 2
                })
            end
            sendLog("Estilo exploit: " .. tostring(v))
        end
    })
    
    -- Estatísticas
    MainTab:AddSection({ Name = "📊 ESTATÍSTICAS" })
    
    local statsLabel = MainTab:AddParagraph({
        Name = "Total de coletas",
        Content = "0"
    })
    
    -- Loop de atualização das estatísticas
    spawn(function()
        while true do
            statsLabel:SetContent(tostring(totalClaims) .. " tokens coletados")
            task.wait(2)
        end
    end)
    
    -- Informações de delay
    MainTab:AddSection({ Name = "⚙️ CONFIGURAÇÕES ATUAIS" })
    
    MainTab:AddParagraph({
        Name = "Delays e pausas",
        Content = "Delay: " .. CONFIG.minDelay .. "-" .. CONFIG.maxDelay .. "s\n" ..
                  "Pausa a cada: " .. CONFIG.attemptsBeforePause .. " tentativas\n" ..
                  "Duração pausa: " .. CONFIG.minPause .. "-" .. CONFIG.maxPause .. "s"
    })
    
    -- Inicializar UI
    OrionLib:Init()
    
    -- Notificação de início
    if not CONFIG.silentMode then
        OrionLib:MakeNotification({
            Name = "✅ Script Carregado",
            Content = "Versão otimizada com anti-detecção",
            Time = 3
        })
    end
    
    sendLog("Script iniciado - Versão " .. "2.1.0")
    
    -- LOOPS PRINCIPAIS
    
    -- Loop Skill
    spawn(function()
        local lastTime = tick()
        while true do
            if exploitSkill then
                local now = tick()
                if now - lastTime >= CONFIG.minDelay then
                    exploitSkillTokens()
                    lastTime = tick()
                end
            end
            task.wait(0.3)
        end
    end)
    
    -- Loop Style
    spawn(function()
        local lastTime = tick()
        while true do
            if exploitStyle then
                local now = tick()
                if now - lastTime >= CONFIG.minDelay + 0.5 then
                    exploitStyleTokens()
                    lastTime = tick()
                end
            end
            task.wait(0.3)
        end
    end)
end

-- ============================================
-- EXECUÇÃO PRINCIPAL COM ERROR HANDLING
-- ============================================
local success, err = pcall(LoadMainScript)
if not success then
    warn("Erro ao carregar script: " .. tostring(err))
    -- Fallback UI simples
    local OrionLib = loadstring(game:HttpGet('https://raw.githubusercontent.com/shlexware/Orion/main/source'))()
    local Window = OrionLib:MakeWindow({ Name = "Erro" })
    local Tab = Window:MakeTab({ Name = "Erro" })
    Tab:AddParagraph({
        Name = "❌ Falha ao carregar",
        Content = "Erro: " .. tostring(err) .. "\n\nTente executar novamente."
    })
    OrionLib:Init()
end# Lucky-vbl
