# Cuentas — Nu Amor

PWA instalable para llevar el registro de **ventas** (lo que me deben) y **deudas** (lo que debo), con desglose por artículo, pagos parciales y saldos por moneda (AUD / EUR / USD).

Sin backend, sin cuentas: los datos viven en el navegador (`localStorage`) del dispositivo. Funciona offline una vez instalada.

## Estructura

```
nu-amor-cuentas/
├── index.html              # toda la app (HTML + CSS + JS en un archivo)
├── manifest.webmanifest    # metadatos PWA (nombre, iconos, colores)
├── sw.js                   # service worker (offline + cache + actualización)
├── netlify.toml            # config de deploy en Netlify
└── icons/
    ├── icon-192.png
    ├── icon-512.png
    ├── icon-512-maskable.png
    └── apple-touch-icon.png
```

## Desarrollo local

Un service worker necesita servirse por HTTP (no abrir el archivo con `file://`):

```bash
python3 -m http.server 8080
# abrir http://localhost:8080
```

## Cómo versionar (IMPORTANTE al desplegar cambios)

Cuando edites `index.html` (o cualquier asset cacheado), **sube el número de versión** del
cache en `sw.js`:

```js
const CACHE_VERSION = "cuentas-v2";   // → "cuentas-v3", "cuentas-v4", ...
```

Qué pasa al hacerlo y desplegar:

1. El navegador detecta un `sw.js` distinto y descarga el nuevo service worker.
2. En `install` el SW llama a `skipWaiting()`; en `activate` borra los caches viejos
   (`CACHE_VERSION` antiguos) y hace `clients.claim()`.
3. `index.html` escucha el evento `controllerchange` y **recarga la página una sola vez**
   (con guarda `hadController` para no recargar en la primera instalación), así el usuario
   ya instalado ve la versión nueva sin tener que reinstalar.

Regla práctica: **un cambio visible = un bump de `CACHE_VERSION`**. Si no lo subes, los
usuarios instalados pueden seguir viendo la versión anterior (servida desde su cache).

> El HTML usa estrategia **network-first** (si hay red, trae la última versión; si no, sirve
> la copia cacheada), y el resto de assets **cache-first**. `netlify.toml` además marca
> `sw.js`, `index.html` y `/` como `no-cache` para que el CDN no sirva un HTML viejo.

## Deploy

### Opción A — Netlify (recomendada)

**Por la web (conectando el repo de GitHub):**

1. Entra en <https://app.netlify.com> → **Add new site → Import an existing project**.
2. Elige **GitHub** y autoriza; selecciona el repositorio `nu-amor-cuentas`.
3. Build settings: **Build command** vacío · **Publish directory** `.` (ya viene en `netlify.toml`).
4. **Deploy site**. Netlify te da una URL `https://<nombre>.netlify.app`.
5. (Opcional) **Site settings → Change site name** para una URL más bonita, o añade un dominio propio.

A partir de ahí, cada `git push` a `main` redepliega automáticamente.

**Por CLI (alternativa):**

```bash
npm i -g netlify-cli
netlify login
netlify deploy --prod   # publish dir "." ya está en netlify.toml
```

### Opción B — GitHub Pages

1. `Settings → Pages` del repo.
2. **Source: Deploy from a branch**, rama `main`, carpeta `/ (root)`.
3. Guarda; la URL será `https://<usuario>.github.io/nu-amor-cuentas/`.

Las rutas son relativas (`./`), así que funciona también bajo un subdirectorio.
(Nota: en GitHub Pages no se aplican las cabeceras de `netlify.toml`; el versionado por
`CACHE_VERSION` sigue funcionando igual.)

## Instalar como app

- **iPhone (Safari):** Compartir → **Añadir a pantalla de inicio**.
- **Android (Chrome):** menú ⋮ → **Instalar app** / **Añadir a pantalla de inicio**.

### Checklist de instalación en iPhone

1. Abre la **URL desplegada** en **Safari** (no Chrome: en iOS solo Safari instala PWAs).
2. Toca **Compartir** (cuadrado con flecha) → **Añadir a pantalla de inicio** → **Añadir**.
3. Abre la app desde el icono de la pantalla de inicio.
4. Verifica que **abre en pantalla completa** (modo standalone, sin barra de Safari).
5. Prueba **offline**: activa modo avión y vuelve a abrirla; debe cargar y mostrar tus datos.
6. Añade un pago/venta, ciérrala y reábrela: los datos persisten (`localStorage`).

## Copias de seguridad

Como los datos son locales al navegador, usa **Exportar copia** (descarga un `.json`) para
respaldar, e **Importar copia** para restaurar o migrar a otro dispositivo. La importación
sanea los datos automáticamente (tolera campos faltantes).

## Notas

- Al arrancar sin datos previos, la app siembra las entradas iniciales (venta de Tristan y
  deudas). Usa **Empezar vacío** para borrarlas y empezar limpio, o **Importar copia** para
  cargar datos reales.
- Persistencia: solo `localStorage` (robusto ante datos vacíos o corruptos; si falla el
  parseo, la app reinicia con la semilla en lugar de quedarse en blanco).
- Paleta: lima `#CCFF00` sobre fondo `#1C1C1C` (identidad Nu Amor).
