local Players = game:GetService("Players")
local ReplicatedStorage = game:GetService("ReplicatedStorage")
local LocalPlayer = Players.LocalPlayer
local mt = getrawmetatable(game)

-- Anti-NotifyGui
local function BlockNotify()
    for _, v in pairs(LocalPlayer.PlayerGui:GetDescendants()) do
        if v:IsA("TextLabel") and string.find(v.Text, "Espere um pouco antes") then
            v.Text = "" -- Esconde a mensagem
        end
    end
    -- Bloquear futuras notificações
    LocalPlayer.PlayerGui.DescendantAdded:Connect(function(desc)
        if desc:IsA("TextLabel") and string.find(desc.Text, "Espere um pouco antes") then
            desc.Text = "" -- Bloqueia novas mensagens
        end
    end)
end

-- Forçar o Remote de pegar peça ignorando o cooldown
local function ForcePickup()
    for _,v in pairs(ReplicatedStorage:GetDescendants()) do
        if v:IsA("RemoteEvent") and string.find(string.lower(v.Name), "pickup") then -- Tenta achar um Remote chamado "Pickup" ou parecido
            print("Forçando remote:", v.Name)
            pcall(function()
                v:FireServer() -- Tenta disparar o Remote ignorando o cooldown
            end)
        end
    end
end

-- Hook de Namecall para filtrar mensagens (caso sejam enviadas como Remotes)
setreadonly(mt, false)
local old = mt.__namecall
mt.__namecall = newcclosure(function(self, ...)
    local args = {...}
    if self.Name == "NotifyGui" and typeof(self) == "Instance" then
        for _,arg in pairs(args) do
            if typeof(arg) == "string" and string.find(arg, "Espere um pouco antes") then
                print("Bloqueando NotifyGui de cooldown")
                return -- Bloqueia o envio
            end
        end
    end
    return old(self, unpack(args))
end)

-- Loop para bloquear constantemente o NotifyGui
task.spawn(function()
    while true do
        BlockNotify()
        task.wait(0.2)
    end
end)

-- Loop para forçar a coleta a cada 1 segundo (cuidado, pode causar kick se o servidor for muito rígido)
task.spawn(function()
    while true do
        ForcePickup()
        task.wait(1)
    end
end)

print("✅ Anti-Countdown e Anti-Notify ativado.")
