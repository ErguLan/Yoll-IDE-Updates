# Yoll IDE — descargas y actualizaciones

Canal público de distribución de Yoll IDE. Este repositorio contiene documentación y las versiones compiladas; el repositorio de desarrollo se mantiene privado.

## Descargar

[Descargar la versión estable más reciente](https://github.com/ErguLan/Yoll-IDE-Updates/releases/latest)

En la Release elige `Yoll-IDE_Setup_vVERSION.exe`. Los instaladores actuales son para Windows x64.

## Actualización desde versiones antiguas

Si tienes una versión cuyo actualizador apunta al repositorio privado, instala la versión 1.0.7 una vez desde esta página. A partir de esta versión, el IDE consultará este canal público sin pedir una cuenta o token de GitHub.

Las descargas se hacen en segundo plano. El reinicio para instalar es explícito y requiere guardar los proyectos abiertos y detener Python. Que una versión esté publicada no significa que todos los usuarios ya la hayan instalado.

## Integridad

Cada Release incluye el instalador, su `.blockmap`, `latest.yml` y `SHA256SUMS.txt`. Puedes comparar el hash de un instalador descargado en PowerShell:

```powershell
Get-FileHash -Algorithm SHA256 .\Yoll-IDE_Setup_v1.0.7.exe
```

Compara el resultado con `SHA256SUMS.txt` de esa misma Release. El programa no tiene firma comercial de Windows configurada; no desactives SmartScreen o el antivirus para ocultar un aviso.

## Para mantenimiento

[Crear instaladores y publicar futuras actualizaciones](./ACTUALIZACIONES_E_INSTALADORES.md)

No se publican aquí tokens, credenciales ni código fuente del IDE.