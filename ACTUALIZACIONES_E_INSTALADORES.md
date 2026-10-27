# Yoll IDE: crear instaladores y publicar actualizaciones

Guía del canal estable a partir de la versión 1.0.7. Actualizada el 27 de octubre de 2026.

## Dónde está cada cosa

- Código fuente privado: https://github.com/ErguLan/Yoll-IDE-Golden
- Instaladores y actualizaciones públicas: https://github.com/ErguLan/Yoll-IDE-Updates/releases
- Configuración del instalador y del canal: `package.json`, sección `build`.
- Arranque: `src/main.tsx`, `src/App.tsx`, `index.html`.
- Actualizador: `electron/services/UpdateManager.ts`, `electron/main/main.ts`, preload, `src/services/updateClient.js` y `src/components/Common/UpdateNotification.jsx`.

No es obligatorio tener dos repositorios. Se separan porque el código de este proyecto es privado y los usuarios deben poder descargar versiones sin iniciar sesión en GitHub. Una rama de un repositorio privado también es privada. Nunca distribuyas un token personal de GitHub dentro del programa.

GitHub Releases está diseñado para distribuir versiones de software y binarios. Cada archivo debe medir menos de 2 GiB. Adjunta los instaladores a una Release: no los subas como archivos del historial Git. Documentación oficial: https://docs.github.com/en/repositories/releasing-projects-on-github/about-releases

## 1. Preparar el equipo

Para el instalador Windows x64 usa Windows, Git, Node.js LTS, npm y GitHub CLI (`gh`). Necesitas permiso de escritura en ambos repositorios.

```powershell
git clone https://github.com/ErguLan/Yoll-IDE-Golden.git
cd Yoll-IDE-Golden
npm ci
gh auth login
```

Usa las versiones del `package-lock.json`. El proyecto ya incluye Electron, Monaco, electron-updater y electron-builder. Si aparece un error del módulo nativo de serialport, ejecuta `npm run rebuild:serialport`; compilar módulos nativos puede requerir las herramientas de compilación de Visual Studio.

No copies `.env`, credenciales ni tokens al instalador. Los archivos `.env` están excluidos del paquete. Los binarios de Electron contienen código compilado del cliente; mantener el repositorio privado no convierte al cliente en un lugar seguro para secretos.

## 2. Cambiar la versión

Trabaja con un checkout actualizado y conserva tus cambios antes de integrar los de otras ramas. Para la siguiente versión después de 1.0.7:

```powershell
npm version patch --no-git-tag-version
```

Esto actualiza `package.json` y `package-lock.json`. No reutilices una versión que ya publicaste. Escribe las notas con los cambios y limitaciones reales.

## 3. Ejecutar comprobaciones y crear el instalador

```powershell
npm run test:updates
npm run test:editor
npm run test:integration
npm run build:installer
npm run verify:release
npx electron test_scripts/packaged.native.cjs
npm run test:updates:native
npm run test:editor:native
npm run test:integration:native
```

`build:installer` comprueba TypeScript del renderer y de Electron, compila con `vite.config.ts` y crea un instalador NSIS. No publica ni instala automáticamente nada en tu equipo. La salida del instalador es `release/`; el renderer y Electron compilados siguen en `dist/` y `dist-electron/`. No ejecutes dos builds sobre el mismo checkout a la vez.

Archivos que debes conservar y publicar juntos, usando el número de versión recién generado:

```text
release/Yoll-IDE_Setup_v1.0.7.exe
release/Yoll-IDE_Setup_v1.0.7.exe.blockmap
release/latest.yml
release/SHA256SUMS.txt
```

`verify:release` comprueba versión, tamaño, SHA512 del instalador, blockmap, canal público, recursos y exclusión de archivos de entorno. También genera `SHA256SUMS.txt`. No edites a mano la versión o el hash de `latest.yml`.

La prueba nativa del actualizador usa Electron y electron-updater reales con un servidor HTTP local y el instalador recién construido. Prueba fallos de red, reintento, descarga, rechazo por SHA512, progreso, varias ventanas, cambios sin guardar, Python activo y reinicio. Intercepta la ejecución final del instalador para no reemplazar tu aplicación ni alterar el registro del equipo durante las pruebas.

La prueba `packaged.native.cjs` carga el `app.asar` exacto que se distribuye, incluyendo sus dependencias, preload y renderer, y comprueba el arranque sin red con un perfil temporal. No instala el programa sobre tu copia existente.

Antes de publicar, revisa también una instalación/actualización real en una VM Windows limpia. La firma de código comercial no está configurada en este proyecto; Windows puede mostrar el aviso de editor desconocido. No desactives SmartScreen o el antivirus para ocultarlo.

## 4. Guardar el código y publicar un borrador

En el repositorio privado, revisa el diff, guarda el código probado en un commit y súbelo. No agregues `node_modules`, `.env`, `dist`, `dist-electron`, `release` ni instaladores antiguos al commit. Usa un push normal: no fuerces el historial.

Ejemplo para una versión nueva: cambia `1.0.8` por la versión real de `package.json`.

```powershell
git tag -a v1.0.8 -m "Yoll IDE v1.0.8"
git push origin HEAD
git push origin v1.0.8

gh release create v1.0.8 --repo ErguLan/Yoll-IDE-Updates --draft --title "Yoll IDE v1.0.8" --notes-file NOTAS_VERSION.md
gh release upload v1.0.8 --repo ErguLan/Yoll-IDE-Updates release/Yoll-IDE_Setup_v1.0.8.exe release/Yoll-IDE_Setup_v1.0.8.exe.blockmap release/latest.yml release/SHA256SUMS.txt
```

`NOTAS_VERSION.md` debe contener las notas de esa versión. El repositorio público solo necesita documentación y las Releases; no copies el repositorio privado ni sus ramas/historial. GitHub creará archivos “Source code” con el contenido de ese repositorio público, no con el código privado.

Descarga los cuatro archivos del borrador a una carpeta de comprobación y compara sus hashes antes de hacerlo público. Una Release publicada e indicada como Latest activa el canal estable:

```powershell
gh release edit v1.0.8 --repo ErguLan/Yoll-IDE-Updates --draft=false --prerelease=false --latest
```

## 5. Rollout a todos

Sin `stagingPercentage` en `latest.yml`, electron-updater ofrece la versión a todos los clientes compatibles del canal estable. La distribución no obliga a cerrar programas: los usuarios reciben la descarga en segundo plano y eligen “Reiniciar e instalar”.

El IDE busca al arrancar y cada cuatro horas. También permite buscar desde Ayuda y Ajustes. Una caída de Internet no bloquea el arranque. El estado se conserva en el proceso principal y se comparte con las ventanas abiertas después de la descarga.

Antes de instalar comprueba todas las ventanas, exige guardar los documentos y detener Python, y vacía las escrituras pendientes del workspace. También revisa si hubo cambios durante la preparación del reinicio. “Más tarde” conserva la descarga; cerrar el IDE normalmente no instala una actualización sin pedirlo.

Las versiones antiguas que apunten al repositorio privado no pueden descubrir anónimamente este nuevo canal. Esos usuarios deben instalar 1.0.7 una vez desde la Release pública; las siguientes versiones ya usarán el canal público. No afirmes que todos actualizaron solo porque la Release esté publicada.

## Si una versión sale mal

No reemplaces los binarios de una versión publicada ni reutilices su tag. Corrige el problema y publica una versión superior, por ejemplo 1.0.9. `allowDowngrade` está desactivado: cambiar Latest a una versión inferior no es un mecanismo de rollback para clientes ya actualizados.

Para un rollout por porcentaje futuro, consulta antes las restricciones del proveedor GitHub de la versión de electron-updater que use el proyecto; no cambies el mecanismo durante una publicación sin probarlo. En el canal actual se distribuye al 100 % de clientes compatibles, sin filtro de porcentaje.

## Plataformas

Esta receta y los binarios publicados aquí son para Windows x64. macOS y Linux necesitan builds y pruebas en sus sistemas reales; no publiques un instalador de esas plataformas solo por tener su sección en `package.json`.
