# Reinstalar coursier

### 1. Borra rastro de instalaciones previas (Opcional)

Si quieres empezar de cero, borra esta carpeta (si es que existe):
`%LOCALAPPDATA%\Coursier`

### 2. Reinstalación mediante PowerShell

Abre una terminal de **PowerShell** (no CMD) y pega este bloque de comandos. Este script descarga el ejecutable, lo renombra y lo prepara:

```jsx
# Descarga el zip de Coursier
Invoke-WebRequest -Uri "https://github.com/coursier/launchers/raw/master/cs-x86_64-pc-win32.zip" -OutFile "cs-x86_64-pc-win32.zip"

# Descomprime el archivo
Expand-Archive -Path "cs-x86_64-pc-win32.zip" -DestinationPath "."

# Cambia el nombre al ejecutable estándar
Rename-Item -Path "cs-x86_64-pc-win32.exe" -NewName "cs.exe"

# Ejecuta el setup automático (esto configura el PATH por ti)
.\cs setup
```

### 3. ¿Qué hace `.\cs setup`?

Al ejecutar el último comando, Coursier te preguntará (en inglés) si quieres modificar el PATH y configurar el entorno.

- Escribe **`yes`** a las preguntas.
- Esto instalará los binarios usualmente en: `C:\Users\TuUsuario\AppData\Local\Coursier\data\bin`

### 4. Paso final obligatorio: Reiniciar

Para que Windows se entere de que `cs` ahora existe:

1. **Cierra VS Code** por completo.
2. **Cierra PowerShell**.
3. Vuelve a abrir VS Code y escribe `cs --version` en la terminal.