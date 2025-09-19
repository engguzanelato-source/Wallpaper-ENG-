# Criação do diretório onde o script será salvo
$wallpaperDir = "C:\wallpaper"
if (-not (Test-Path -Path $wallpaperDir)) {
    New-Item -Path $wallpaperDir -ItemType Directory
}

# Definindo o caminho do script
$scriptPath = "$wallpaperDir\mudar-wallpaper.ps1"

# Criando o conteúdo do script PowerShell que será salvo
$scriptContent = @'
$imageUrl = "https://drive.google.com/uc?export=download&id=171Lt7GApFBfHCp3P7_KKgM73AyPcZSgr"
$imagePath = "C:\wallpaper\wallpaper.jpg"

Invoke-WebRequest -Uri $imageUrl -OutFile $imagePath

Add-Type @"
using System.Runtime.InteropServices;
public class NativeMethods {
    [DllImport("user32.dll", SetLastError = true)]
    public static extern bool SystemParametersInfo(int uAction, int uParam, string lpvParam, int fuWinIni);
}
"@

[NativeMethods]::SystemParametersInfo(20, 0, $imagePath, 0x01 -bor 0x02)
'@

# Salvando o script no diretório
$scriptContent | Out-File -FilePath $scriptPath -Encoding UTF8 -Force

# Criando a tarefa agendada para rodar no login do usuário 'Aluno' de forma oculta
$action = New-ScheduledTaskAction -Execute 'powershell.exe' -Argument "-WindowStyle Hidden -ExecutionPolicy Bypass -File `"$scriptPath`""
$trigger = New-ScheduledTaskTrigger -AtLogOn
$principal = New-ScheduledTaskPrincipal -UserId "ENG Aluno" -LogonType Interactive

# Registrando a tarefa agendada
Register-ScheduledTask -TaskName "AtualizarWallpaperAluno" -Action $action -Trigger $trigger -Principal $principal -Force

Write-Host "Tarefa agendada com sucesso! Será executada silenciosamente no login do usuário ENG Aluno."


Log "=== Script finalizado ==="
