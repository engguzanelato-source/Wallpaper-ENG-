# Script para trocar o papel de parede remotamente via Deep Freeze Cloud

# Caminho RAW da imagem hospedada no GitHub
# ⚠️ Depois que subir o repositório, troque SEU_USUARIO e SEU_REPOSITORIO abaixo:
$wallpaperUrl = "https://raw.githubusercontent.com/engguzanelato-source/Wallpaper-ENG-/main/ENG_Imagem.jpeg"

# Caminho local onde a imagem será salva
$wallpaperPath = "$env:ProgramData\ENG_Imagem.jpeg"

# Baixar a imagem do GitHub para a máquina remota
Invoke-WebRequest -Uri $wallpaperUrl -OutFile $wallpaperPath -UseBasicParsing

# Configurar a imagem como papel de parede no Registro
Set-ItemProperty -Path "HKCU:\Control Panel\Desktop" -Name Wallpaper -Value $wallpaperPath

# Força a atualização usando SystemParametersInfo
Add-Type @"
using System;
using System.Runtime.InteropServices;
public class Wallpaper {
  [DllImport("user32.dll", CharSet = CharSet.Auto)]
  public static extern int SystemParametersInfo(int uAction, int uParam, string lpvParam, int fuWinIni);
}
"@

# 20 = SPI_SETDESKWALLPAPER
# 0 = uParam
# 3 = SPIF_UPDATEINIFILE | SPIF_SENDWININICHANGE (atualiza imediatamente)
[Wallpaper]::SystemParametersInfo(20, 0, $wallpaperPath, 3)

# Atualizar o papel de parede sem reiniciar
RUNDLL32.EXE user32.dll,UpdatePerUserSystemParameters


