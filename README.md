# Caminho do papel de parede (pode ser local ou baixado da internet)
$wallpaperUrl = "https://share.google/p3rzS65lZgfN867ls"
$wallpaperPath = "$env:ProgramData\wallpaper.jpeg"

# Baixar o papel de parede da URL
Invoke-WebRequest - "https://share.google/p3rzS65lZgfN867ls"

# Definir o papel de parede no Registro
Set-ItemProperty -Path "HKCU:\Control Panel\Desktop\" -Name wallpaper -Value $wallpaperPath

# Forçar atualização imediata do papel de parede
RUNDLL32.EXE user32.dll,UpdatePerUserSystemParameters


