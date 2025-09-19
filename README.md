# ========== Configuração ==========
$wallpaperUrl = "https://raw.githubusercontent.com/engguzanelato-source/Wallpaper-ENG-/main/ENG_Imagem.jpeg"
$wallpaperPath = "$env:ProgramData\ENG_Imagem.jpeg"
$logFile = "$env:ProgramData\wallpaper_apply.log"

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

# 2️⃣ Atualiza o registro do usuário atual
try {
    $regPath = "HKCU:\Control Panel\Desktop"
    Set-ItemProperty -Path $regPath -Name Wallpaper -Value $wallpaperPath -Force
    Set-ItemProperty -Path $regPath -Name WallpaperStyle -Value "10" -Force   # 10 = Fill
    Set-ItemProperty -Path $regPath -Name TileWallpaper -Value "0" -Force
    Log "Registro atualizado para a sessão atual."
}
catch {
    Log "ERRO atualizando registro: $_"
}

# 3️⃣ Atualiza o wallpaper imediatamente na sessão atual
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
    Log "Wallpaper aplicado na sessão atual."
}
catch {
    Log "ERRO aplicando wallpaper na sessão atual: $_"
}

Log "=== Script finalizado ==="
