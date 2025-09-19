# Caminho RAW da imagem no GitHub
$wallpaperUrl = "https://raw.githubusercontent.com/engguzanelato-source/Wallpaper-ENG-/main/ENG_Imagem.jpeg"
$wallpaperPath = "$env:ProgramData\ENG_Imagem.jpeg"

# Baixa a imagem
Invoke-WebRequest -Uri $wallpaperUrl -OutFile $wallpaperPath -UseBasicParsing

# Pega o usuário logado na sessão ativa
$ActiveUser = (Get-CimInstance -ClassName Win32_ComputerSystem | Select-Object -ExpandProperty UserName)

if ($ActiveUser) {
    # Extrai o SID do usuário ativo
    $SID = (New-Object System.Security.Principal.NTAccount($ActiveUser)).Translate([System.Security.Principal.SecurityIdentifier]).Value
    $RegPath = "Registry::HKEY_USERS\$SID\Control Panel\Desktop"

    # Define o papel de parede no registro do usuário ativo
    Set-ItemProperty -Path $RegPath -Name Wallpaper -Value $wallpaperPath

    # Força atualização usando SystemParametersInfo no contexto do usuário
    $sig = @"
    using System;
    using System.Runtime.InteropServices;
    public class Wallpaper {
        [DllImport("user32.dll", CharSet = CharSet.Auto)]
        public static extern int SystemParametersInfo(int uAction, int uParam, string lpvParam, int fuWinIni);
    }
"@
    Add-Type $sig
    [Wallpaper]::SystemParametersInfo(20, 0, $wallpaperPath, 3)

    Write-Output "Wallpaper aplicado para o usuário $ActiveUser"
}
else {
    Write-Output "Nenhum usuário ativo encontrado. O script executou como SYSTEM."
}
