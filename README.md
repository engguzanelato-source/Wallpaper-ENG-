# ========== Configuração ==========
$wallpaperUrl = "https://raw.githubusercontent.com/engguzanelato-source/Wallpaper-ENG-/main/ENG_Imagem.jpeg"
$wallpaperPath = "C:\ProgramData\ENG_Imagem.jpeg"
$logFile = "C:\ProgramData\wallpaper_apply.log"

function Log($text) {
    $line = "$(Get-Date -Format o) - $text"
    Add-Content -Path $logFile -Value $line -Force
}

Log "=== Início do script ==="

# 1️⃣ Baixa a imagem
try {
    Log "Baixando imagem de $wallpaperUrl ..."
    Invoke-WebRequest -Uri $wallpaperUrl -OutFile $wallpaperPath -UseBasicParsing -ErrorAction Stop
    Log "Imagem salva em $wallpaperPath"
}
catch {
    Log "ERRO ao baixar imagem: $_"
    exit 1
}

# 2️⃣ Detecta usuários ativos e/ou carregados
try {
    $loggedUsers = Get-CimInstance Win32_LoggedOnUser | ForEach-Object {
        $_.Antecedent -replace '.*Domain="([^"]+)",Name="([^"]+)".*','$1\$2'
    } | Sort-Object -Unique

    if (-not $loggedUsers) {
        Log "Nenhum usuário ativo encontrado."
        exit 1
    }
    Log "Usuários ativos detectados: $($loggedUsers -join ', ')"
}
catch {
    Log "ERRO detectando usuários: $_"
    exit 1
}

# 3️⃣ Atualiza o registro para cada usuário
foreach ($user in $loggedUsers) {
    try {
        $nt = New-Object System.Security.Principal.NTAccount($user)
        $sid = $nt.Translate([System.Security.Principal.SecurityIdentifier]).Value
        $regPath = "Registry::HKEY_USERS\$sid\Control Panel\Desktop"

        if (-not (Test-Path $regPath)) {
            New-Item -Path $regPath -Force | Out-Null
            Log "Criada chave de registro: $regPath"
        }

        Set-ItemProperty -Path $regPath -Name Wallpaper -Value $wallpaperPath -Force
        Set-ItemProperty -Path $regPath -Name WallpaperStyle -Value "10" -Force
        Set-ItemProperty -Path $regPath -Name TileWallpaper -Value "0" -Force
        Log "Wallpaper aplicado para $user (SID: $sid)"
    }
    catch {
        Log "ERRO atualizando registro de $user: $_"
    }
}

# 4️⃣ Força atualização do wallpaper na sessão atual (se possível)
try {
    Add-Type @"
using System.Runtime.InteropServices;
public class WinAPI {
    [DllImport("user32.dll",SetLastError=true)]
    public static extern bool SystemParametersInfo(int uAction,int uParam,string lpvParam,int fuWinIni);
}
"@

    $SPI_SETDESKWALLPAPER = 20
    $SPIF_UPDATEINIFILE = 1
    $SPIF_SENDWININICHANGE = 2

    [WinAPI]::SystemParametersInfo($SPI_SETDESKWALLPAPER, 0, $wallpaperPath, $SPIF_UPDATEINIFILE -bor $SPIF_SENDWININICHANGE) | Out-Null
    Log "Wallpaper atualizado na sessão atual (se houver)."
}
catch {
    Log "ERRO atualizando wallpaper na sessão atual: $_"
}

Log "Script finalizado."
