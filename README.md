# StockAbarrotes v2

Sistema de inventario offline para tiendas de abarrotes.
Desarrollado para San Cristóbal de las Casas, Chiapas — funciona sin internet.

---

## ¿Qué hace esta app?

| Función | Descripción |
|---|---|
| 📦 **Productos** | Registra nombre, marca, unidad, precio y código de barras |
| 🗃️ **Lotes** | Controla fechas de caducidad por lote |
| ⚠️ **Alertas** | Avisa cuando algo vence en 7 o 15 días, o cuando el stock está bajo |
| 📥 **Entradas** | Registra compras por unidad o por caja (calcula automáticamente) |
| 📤 **Salidas** | Descuenta stock del lote correcto |
| 🔍 **Escáner** | Lector de código de barras USB — registra varias salidas a la vez |
| 📄 **PDF** | Exporta el resumen de alertas para imprimir |
| 💾 **Respaldo** | Guarda una copia de la base de datos en un clic |

---

## Lo que necesitas instalar (una sola vez)

### 1. Node.js — versión 20 LTS (obligatorio)

Instala la versión **20 LTS** específicamente.
Versiones más nuevas (v21, v22, v23, v24) causan un error al compilar `better-sqlite3` y el `npm install` falla.

👉 https://nodejs.org/en/download — elige la pestaña **"Previous Releases"** y descarga la **v20 LTS**.

> **Windows:** ejecuta el instalador `.msi` y acepta todo.
> **Ubuntu/Debian:**
> ```bash
> curl -fsSL https://deb.nodesource.com/setup_20.x | sudo -E bash -
> sudo apt install nodejs -y
> ```

Verifica que quedó la versión correcta:
```bash
node --version   # debe mostrar v20.x.x  ← importante que sea 20
npm --version    # debe mostrar 10.x.x
```

> **¿Ya tienes Node instalado pero en versión 21, 22, 23 o 24?**
> Si tienes **nvm**, cambia a Node 20 con tres comandos:
> ```bash
> nvm install 20
> nvm use 20
> node --version   # confirma v20.x.x
> ```
> Si no tienes nvm, desinstala tu Node actual desde el Panel de Control y reinstala la v20 desde el enlace de arriba.

### 2. Git (solo si vas a clonar desde GitHub)

- **Windows:** https://git-scm.com/download/win
- **Ubuntu:** `sudo apt install git -y`

---

## Cómo ejecutar el proyecto

### Paso 1 — Descarga el código

**Opción A — desde GitHub:**
```bash
git clone https://github.com/tu-usuario/stockabarrotes.git
cd stockabarrotes
```

**Opción B — desde el ZIP:**
Descomprime el archivo y abre una terminal dentro de la carpeta `stockabarrotes`.

---

### Paso 2 — Instala las dependencias

```bash
npm install
```

Esto descarga todas las dependencias **y recompila automáticamente `better-sqlite3` para Electron** (el script `postinstall` lo hace solo). Puede tardar 3-6 minutos la primera vez.

Cuando termine verás:
```
✔ Rebuild Complete
added 4XX packages
```

> ⚠️ **Si `npm install` falla con error de `better-sqlite3` o `NODE_MODULE_VERSION`:**
>
> **Causa más común — Node.js demasiado nuevo.** Verifica tu versión:
> ```bash
> node --version   # debe ser v20.x.x
> ```
> Si es v21 o superior, cambia a Node 20:
> ```bash
> nvm install 20
> nvm use 20
> ```
> Luego borra `node_modules` y vuelve a instalar:
> ```powershell
> # Windows PowerShell
> Remove-Item -Recurse -Force node_modules
> npm install
> ```
> ```bash
> # Linux / Mac
> rm -rf node_modules && npm install
> ```
>
> **Si ya tienes Node 20 y sigue fallando** — falta el compilador de C++:
> - **Windows:** instala [Visual Studio Build Tools](https://visualstudio.microsoft.com/visual-cpp-build-tools/) — elige "Desarrollo para escritorio con C++"
> - **Ubuntu:** `sudo apt install build-essential python3 -y`

---

### Paso 3 — Ejecutar en modo desarrollo

```bash
npm run dev
```

Esto abre la ventana de la aplicación automáticamente.
La primera vez puede tardar unos segundos mientras Vite compila.

> El comando `npm run dev` hace dos cosas al mismo tiempo:
> 1. Inicia el servidor de Vite (la interfaz React)
> 2. Abre Electron apuntando a ese servidor
>
> Los datos se guardan en:
> - **Windows:** `C:\Users\TuUsuario\AppData\Roaming\StockAbarrotes\inventario.db`
> - **Linux:** `~/.config/StockAbarrotes/inventario.db`

Para detenerlo: presiona `Ctrl + C` en la terminal.

---

## Compilar el instalador (para distribuir)

Antes de compilar, reemplaza los íconos placeholder con los definitivos:
- `public/icon.ico` — 256×256 píxeles, formato ICO (para Windows)
- `public/icon.png` — 512×512 píxeles, formato PNG (para Linux)

### Instalador para Windows (.exe)

```bash
npm run build:win
```

Genera el archivo en `release/` — algo como `StockAbarrotes Setup 1.0.0.exe`.

### Paquete para Linux (.AppImage + .deb)

```bash
npm run build:linux
```

Genera en `release/`:
- `StockAbarrotes-1.0.0.AppImage` — ejecutable portable, sin instalar
- `stockabarrotes_1.0.0_amd64.deb` — paquete de instalación para Ubuntu/Debian

---

## Estructura del proyecto

```
stockabarrotes/
│
├── electron/
│   ├── main.js          ← Proceso principal de Electron
│   │                      Abre la ventana, maneja la base de datos SQLite,
│   │                      gestiona los respaldos y el ciclo de vida de la app
│   │
│   └── preload.js       ← Puente seguro entre Electron y React
│                          Expone window.db.* para que la UI pueda
│                          leer y escribir datos sin acceso directo a Node
│
├── src/
│   ├── App.jsx          ← Toda la interfaz de usuario (1100+ líneas)
│   │                      Productos, lotes, movimientos, escáner, PDF, respaldo
│   ├── main.jsx         ← Punto de entrada de React
│   └── index.css        ← Estilos globales mínimos
│
├── public/
│   ├── icon.ico         ← Ícono para Windows (reemplazar con logo final)
│   └── icon.png         ← Ícono para Linux (reemplazar con logo final)
│
├── docs/
│   └── MANUAL_USUARIO.md
│
├── index.html           ← Punto de entrada HTML que carga React (raíz del proyecto)
├── package.json         ← Dependencias y configuración de electron-builder
└── vite.config.js       ← Configuración del bundler
```

---

## Base de datos

La app usa **SQLite** a través de `better-sqlite3` — un archivo `.db` local, sin servidor, sin configuración.

Se crea automáticamente la primera vez que abres la app:
- **Windows:** `%APPDATA%\StockAbarrotes\inventario.db`
- **Linux:** `~/.config/StockAbarrotes/inventario.db`

**Tablas:**
- `productos` — catálogo de productos con marca, unidad, precio y código de barras
- `lotes` — cada entrada de mercancía con su fecha de caducidad y cantidad actual
- `movimientos` — historial completo de entradas y salidas

El archivo WAL (`inventario.db-wal`) se fusiona automáticamente al cerrar la app, protegiendo los datos ante cortes de luz.

---

## Hacer un respaldo

Desde la app: **Resumen inteligente → botón "Hacer respaldo"**

Esto abre un diálogo para guardar una copia del archivo `.db` donde quieras (USB, escritorio, etc.).

Para restaurar: copia el archivo `.db` de respaldo a la ruta de datos y renómbralo `inventario.db`.

---

## Solución de problemas comunes

**La app no abre / pantalla en blanco**
- Asegúrate de que `npm run dev` haya terminado de compilar (espera el mensaje `ready in Xms`)
- Revisa que el puerto 5173 no esté ocupado por otra app

**Error `better-sqlite3` / `gyp ERR! build error` al hacer `npm install`**

Este es el error más común. Tiene dos causas posibles:

**Causa 1 — Node.js demasiado nuevo (la más frecuente)**
`better-sqlite3` necesita compilar código nativo y no tiene binarios listos para Node v21 o superior.
Solución: cambia a Node 20 LTS.
```bash
nvm install 20
nvm use 20
rmdir /s /q node_modules   # Windows
# rm -rf node_modules      # Linux/Mac
npm install
```

**Causa 2 — Falta el compilador de C++**
Si ya tienes Node 20 y sigue fallando:
- **Windows:** instala [Visual Studio Build Tools](https://visualstudio.microsoft.com/visual-cpp-build-tools/) — elige "Desarrollo para escritorio con C++"
- **Ubuntu:** `sudo apt install build-essential python3 -y`

Luego borra `node_modules` y vuelve a correr `npm install`.

**El escáner de código de barras no funciona**
- El lector debe estar en modo HID (keyboard emulation) — es el modo predeterminado en casi todos los lectores USB
- Asegúrate de que el cursor esté dentro del campo azul de escaneo antes de escanear

**`npm run build:win` falla con error de ícono**
- Reemplaza `public/icon.ico` con un archivo ICO real de 256×256 px

---

## Licencia

Uso privado — UNLICENSED
