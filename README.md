# ========== Configuração ==========
$wallpaperUrl = "https://raw.githubusercontent.com/engguzanelato-source/Wallpaper-ENG-/main/ENG_Imagem.jpeg"
$wallpaperPath = "C:\ProgramData\ENG_Imagem.jpeg"
$logFile = "C:\ProgramData\wallpaper_apply.log"

function Log($text) {
    $line = "$(Get-Date -Format o) - $text"
    Add-Content -Path $logFile -Value $line -Force
}

Log "=== Início do script ==="

# Baixa a imagem
try {
    Log "Baixando imagem de $wallpaperUrl ..."
    Invoke-WebRequest -Uri $wallpaperUrl -OutFile $wallpaperPath -UseBasicParsing -ErrorAction Stop
    Log "Imagem salva em $wallpaperPath"
}
catch {
    Log "ERRO ao baixar imagem: $_"
    exit 1
}

# Detecta usuário logado via processo explorer (mais confiável para sessão interativa)
try {
    $expl = Get-Process -Name explorer -ErrorAction SilentlyContinue | Select-Object -First 1
    if ($expl) {
        $procInfo = Get-CimInstance -Query "SELECT * FROM Win32_Process WHERE ProcessId=$($expl.Id)"
        $owner = $procInfo.GetOwner()
        $activeUser = "$($owner.Domain)\$($owner.User)"
        Log "Usuário detectado via explorer: $activeUser"
    } else {
        $activeUser = (Get-CimInstance -ClassName Win32_ComputerSystem).UserName
        Log "Nenhum explorer encontrado, usuário via Win32_ComputerSystem: $activeUser"
    }

    if (-not $activeUser) {
        Log "Nenhum usuário ativo encontrado. Saindo."
        exit 1
    }
}
catch {
    Log "ERRO detectando usuário: $_"
    exit 1
}

# Traduz usuário em SID
try {
    $nt = New-Object System.Security.Principal.NTAccount($activeUser)
    $sid = $nt.Translate([System.Security.Principal.SecurityIdentifier]).Value
    Log "SID do usuário: $sid"
}
catch {
    Log "ERRO ao traduzir usuário para SID: $_"
    exit 1
}

$regPath = "Registry::HKEY_USERS\$sid\Control Panel\Desktop"

# Garante a chave
if (-not (Test-Path $regPath)) {
    try {
        New-Item -Path $regPath -Force | Out-Null
        Log "Criada chave de registro: $regPath"
    } catch {
        Log "ERRO criando chave no registro: $_"
    }
}

# Escreve as chaves necessárias (Wallpaper + estilo)
try {
    Set-ItemProperty -Path $regPath -Name Wallpaper -Value $wallpaperPath -Force
    # WallpaperStyle e TileWallpaper - ajustar conforme necessidade:
    #  WallpaperStyle = 10 (Fill), TileWallpaper = 0
    Set-ItemProperty -Path $regPath -Name WallpaperStyle -Value "10" -Force
    Set-ItemProperty -Path $regPath -Name TileWallpaper -Value "0" -Force
    Log "Registro atualizado (Wallpaper, WallpaperStyle, TileWallpaper)."
}
catch {
    Log "ERRO escrevendo no registro do usuário: $_"
    exit 1
}

# Mata o(s) explorer.exe do usuário para forçar reload
try {
    $explProcs = Get-Process -Name explorer -ErrorAction SilentlyContinue | Where-Object {
        try {
            $pinfo = Get-CimInstance -Query "SELECT * FROM Win32_Process WHERE ProcessId=$($_.Id)"
            $o = $pinfo.GetOwner()
            "$($o.Domain)\$($o.User)" -eq $activeUser
        } catch { $false }
    }

    if ($explProcs) {
        foreach ($p in $explProcs) {
            Log "Finalizando explorer (PID $($p.Id)) do usuário $activeUser"
            Stop-Process -Id $p.Id -Force -ErrorAction SilentlyContinue
        }
        Start-Sleep -Seconds 2
        Log "Esperando reinício do explorer (se o Windows reiniciar automaticamente)..."
    } else {
        Log "Nenhum processo explorer do usuário encontrado para finalizar."
    }
}
catch {
    Log "ERRO ao tentar finalizar explorer: $_"
}

Log "Script finalizado. Verifique se o explorer do usuário reiniciou e pegou o novo wallpaper."
