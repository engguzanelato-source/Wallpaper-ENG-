# Cambia el fondo de pantalla del escritorio

# crea la carpeta si no existe
$carpeta = "C:\ENG Imagem.jpeg"
If(!(test-path -PathType container $carpeta)){
  New-Item -ItemType Directory -Path $carpeta
}
# Se descarga la imagen
$address = "https://github.com/engguzanelato-source/Wallpaper-ENG-/edit/main/README.md"
$fileName = "$carpeta\ENG Imagem.jpeg"
(New-Object System.Net.WebClient).DownloadFile($address, $fileName)

# Informacion del fondo de escritorio del registro del usuario actual
# Importante: este script debe ejecutarse con la cuenta del usuario, por lo del HKCU.
$path = "HKCU:\Control Panel\Desktop"

# Cambia el wallpaper al cambiar el registro de windows
Set-ItemProperty -Path $path -Name WallPaper -Value $fileName

# es necesario reiniciar la computadora para que muestre los cambios
Restart-Computer -Force
