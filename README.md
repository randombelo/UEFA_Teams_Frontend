# UEFA Teams Frontend

Frontend (SPA estática) para consumir la API **UEFA Teams Management API**: gestión de **equipos**, **jugadores** y **competiciones**, con interfaz construida bajo el patrón **Atomic Design** y comunicación asíncrona con **Axios**.

El proyecto es un conjunto de archivos **estáticos** (HTML + CSS + JavaScript moderno en módulos ES) sin step de build: se sirve con cualquier servidor estático y las páginas consumen la API directamente desde el navegador (Web + CORS).

## Tecnologías

| Tecnología | Uso |
|------------|-----|
| HTML5 | Estructura y páginas (`index.html`, `pages/*.html`) |
| CSS3 (BEM + CSS custom properties) | Estilos con tokens por capas (`tokens`, `base`, `components`, `pages`) |
| JavaScript ES Modules | Lógica de la aplicación (`import`/`export`, sin bundlers) |
| Axios (CDN) | Cliente HTTP contra la API backend |
| Atomic Design | Organización de componentes (átomos → moléculas → organismos → páginas) |

## Estructura del proyecto

```
.
├── index.html               # Página Home
├── css/
│   ├── tokens.css           # Variables de diseño (colores, espaciados, radios...)
│   ├── base.css             # Reset y base (layout, header, footer)
│   ├── components.css       # Componentes (átomos, moléculas, organismos)
│   └── pages.css            # Estilos específicos de páginas
├── pages/                   # Páginas estáticas
│   ├── teams.html
│   ├── players.html
│   ├── competitions.html
│   └── docs.html
└── js/
    ├── app/mountLayout.js       # Monta header, footer y toasts
    ├── api/axiosClient.js        # Instancia Axios (baseURL + interceptores)
    ├── api/normalizeApiError.js # Normaliza errores HTTP (422 → campos, etc.)
    ├── config/endpointsConfig.js# Definición de endpoints y API_BASE_URL
    ├── services/                # teamService, playerService, competitionService
    ├── components/
    │   ├── atoms/               # Badge, Btn, Checkbox, FieldLabel, Input, Select, Spinner
    │   ├── molecules/           # ApiStatus, CheckboxGroup, FormField, PaginationBar, SearchBar
    │   └── organisms/           # Header, Footer, Tables, FormDialogs, ConfirmDialog, Toast, EndpointCard
    ├── pages/                   # home, teams, players, competitions, docs
    └── utils/format.js          # Utilidades de formateo (fechas, moneda)
```

## Configuración de la API

La URL de la API backend se define en **una sola línea**: `js/config/endpointsConfig.js`.

```js
export const API_BASE_URL = "http://127.0.0.1:8000";
```

Si el backend corre en otro host o puerto, cambia solo este valor (el backend tiene CORS abierto, así que cualquier origen es válido).

## Requisitos previos

- Un navegador moderno.
- **El backend corriendo** (puerto `8000` por defecto). Sin él, la app funciona pero las llamadas fallarán y la página mostrará "API: offline".
- Python 3 (solo para servir estáticos) o cualquier servidor estático.
- Node.js (opcional) únicamente para verificar la sintaxis de los módulos.

## Guía paso a paso (ejecución)

```bash
# 1. Arrancar el backend (repositorio hermano UEFA_Teams)
#    cd ../UEFA_Teams && uvicorn main:app --reload

# 2. Servir el frontend desde la raíz de este repositorio
python3 -m http.server 5500

# 3. Abrir el navegador
#    http://127.0.0.1:5500
```

Alternativas al servidor estático:

```bash
# VS Code: extensión "Live Server" sobre la carpeta del proyecto
# o con npx (requiere Node):
npx serve .
```

## Páginas y funcionalidad

| Página | Ruta | Funcionalidad |
|--------|------|---------------|
| **Home** | `/index.html` | Presentación, estado de la API en vivo y accesos rápidos |
| **Equipos** | `/pages/teams.html` | CRUD de equipos, búsqueda, paginación, **badges con sus competiciones** y asignación de competiciones mediante checkbox durante crear/editar |
| **Jugadores** | `/pages/players.html` | CRUD de jugadores, búsqueda, paginación y **filtro por club** |
| **Competiciones** | `/pages/competitions.html` | CRUD del catálogo de competiciones |
| **Docs** | `/pages/docs.html` | Listado de endpoints agrupados con parámetros, ejemplos y **ejecución real de peticiones GET** desde la página |

## Arquitectura

Flujo típico de una página:

1. La **página** (`js/pages/*.js`) mantiene un `state` local (datos, paginación, búsqueda, carga).
2. Las llamadas se agrupan en **services** (`js/services/*.js`) que envuelven a Axios.
3. **Axios** (`js/api/axiosClient.js`) aplica `baseURL` y un interceptor que:
   - devuelve `true` en respuestas `204`,
   - devuelve `res.data` en el resto,
   - normaliza errores mediante `normalizeApiError` (errores `422` mapeados con sus campos).
4. El resultado se renderiza como **template strings** de componentes Atomic Design (`TeamTable`, `TeamFormDialog`, etc.).
5. Los **organismos** que abren diálogos (`…FormDialog`, `ConfirmDialog`, `Toast`) gestionan validación en cliente, mapeo de errores del servidor y feedback visual (toasts `success` / `danger` / `warning`).

Convenciones:

- Componentes por capas: `atoms` (sin lógica), `molecules` (compuestos), `organisms` (complejos con estado y eventos).
- Cada formulario expone `open(values, extras)` y un `readForm` que serializa el formulario a un payload.
- `data-page` en `<body>` alimenta al `Header` para marcar la página activa.

## Verificación de calidad

Comprobar la sintaxis de todos los módulos ES (sin ejecutar el navegador):

```bash
node --experimental-default-type=module --check js/api/axiosClient.js
node --experimental-default-type=module --check js/pages/teams.js
# ... para cada archivo de js/
```

## Backend (proyecto hermano)

La API vive en el repositorio independiente [`../UEFA_Teams/`](../UEFA_Teams/).

- **Arrancar el backend**: `uvicorn main:app --reload` desde `UEFA_Teams` → `http://127.0.0.1:8000`.
- **Documentación Swagger**: `http://127.0.0.1:8000/docs`.
- La URL que consume este frontend se configura en `js/config/endpointsConfig.js` (ver sección *Configuración de la API*).
- CORS del backend abierto al 100 %: el frontend puede servirse desde cualquier origen o puerto.

## Capturas

> Las capturas se añaden manualmente en esta carpeta. Reemplaza la ruta de cada una por tu imagen y ajusta si hace falta.

| Descripción | Imagen |
|-------------|--------|
| Home con estado de la API | `docs/screenshots/home.png` |
| Listado de equipos con competiciones | `docs/screenshots/teams.png` |
| Listado de jugadores con filtro por club | `docs/screenshots/players.png` |
| Página de competiciones | `docs/screenshots/competitions.png` |

## Troubleshooting

| Síntoma | Causa / solución |
|---------|------------------|
| `ERR_CONNECTION_REFUSED` en consola y "API: offline" | El backend no está corriendo en `127.0.0.1:8000`. Arranca la API (paso 1 de la guía). Es ruido esperado si el backend está apagado. |
| Las páginas no cargan la API | Revisa `API_BASE_URL` en `js/config/endpointsConfig.js`. |
| Los módulos JS no se resuelven | Sirve siempre desde la **raíz** del repositorio (no abras `js/...` directamente); las importaciones usan rutas relativas y absolutas. |
| El navegador bloquea el CDN de Axios | Requiere acceso a Internet para `https://cdn.jsdelivr.net/npm/axios@1/dist/axios.min.js`. |