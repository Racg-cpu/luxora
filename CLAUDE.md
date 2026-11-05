# Luxora — instrucciones para Claude

Sitio de Luxora, agencia de marketing digital de Raúl en Guasave, Sinaloa. Raúl no programa: explícale en español, claro y corto, y haz el trabajo tú.

## Qué hay aquí
- `index.html` — landing pública (marketing). HTML/CSS en un solo archivo, sin JavaScript de datos. Los logos van incrustados como `data:` PNG.
- `portal/index.html` — portal de clientes y panel de administrador, todo en un solo archivo (HTML + CSS + JS, sin build).
- `vendor/supabase.js` y `vendor/591.supabase.js` — `@supabase/supabase-js` 2.45.4 (UMD) incluido localmente para no depender de un CDN. No lo edites; para actualizarlo, baja la versión nueva de npm (`npm pack @supabase/supabase-js@<versión>`) y copia `dist/umd/*`.
- `.nojekyll` — GitHub Pages sirve los archivos tal cual.

## Cómo se publica
GitHub Pages publica la rama `main` (carpeta raíz) en https://racg-cpu.github.io/luxora . Cada push a `main` se publica solo en 1–2 minutos (workflow "pages build and deployment"; revísalo con la API de Actions si necesitas confirmar). Para regresar a una versión anterior: `git revert` del commit y push.

## Backend: Supabase
- Proyecto: `https://jhzmopnzwfpicbvnuxcz.supabase.co`. La clave en el portal es la **publicable** (`sb_publishable_…`): es pública por diseño. **Nunca** pongas en el repo la `service_role`/`secret key` ni contraseñas, y nunca se las pidas a Raúl.
- La seguridad la dan las políticas RLS del servidor, no la página. El esquema y las políticas están en el repo `Racg-cpu/app_planner`, carpeta `luxora/supabase/` (`1-esquema.sql`, `2-datos.sql`, `3-hacerme-admin.sql`, `4-ajustes-app.sql`).
- Tablas: `perfiles` (rol admin/cliente), `negocios`, `accesos` (qué negocios ve cada cliente), `imagenes`, `notas_internas` (solo admin), `comentarios`, `solicitudes`, `reportes`, `tareas` (solo admin), `ajustes` (`marca` se lee sin sesión).
- Storage: bucket privado `archivos` con ruta `<negocio_id>/<archivo>` (el cliente solo ve carpetas de sus negocios; se usan enlaces firmados de 1 h) y bucket público `marca` para el logo.
- Cambios de esquema: escribe un archivo SQL nuevo e idempotente (`if not exists`, sin borrar datos) y pídele a Raúl que lo corra en Supabase → SQL Editor. Tú no puedes correrlo con la clave publicable.
- Sin sesión, la API solo debe devolver `ajustes`; el resto responde 42501. Puedes comprobarlo con `curl "$URL/rest/v1/<tabla>?select=*" -H "apikey: <publicable>"`.

## Cómo funciona el portal (`portal/index.html`)
- Inicio de sesión con correo y contraseña (Supabase Auth). Los clientes se registran solos ("Soy cliente nuevo"); no ven nada hasta que el admin les asigna negocios en **Accesos**.
- El código de la app viene del antiguo artefacto de claude.ai: usa un adaptador (`crearDb`, `MAPAS`) que imita la API `collection().onSnapshot/add/doc().update` y la traduce a tablas de Supabase (camelCase ↔ snake_case). Si agregas un campo, agrégalo en `MAPAS` y en la tabla.
- No hay tiempo real: relee todo cada 20 s mientras la pestaña está visible.
- **Importar lote** (botón en Inicio del admin, función `importarLote`): sube muchos archivos de una vez. Nombres: `<idDeImagen>__borrador-1.png`, `__borrador-2.png`… o `<idDeImagen>__final.png`. Los borradores importados reemplazan a los anteriores; opcionalmente pasa a "Por aprobar" las pendientes o en creación. Úsalo para cargar exportaciones de Canva: tú preparas y nombras los archivos y Raúl los selecciona con su sesión de admin.
- **Campos opcionales al pedir una imagen** (`camposDetalles`): tipo, producto, qué incluye, precios confirmados, vigencia, fecha del primer borrador, dónde se publica, número de opciones, texto obligatorio y referencia. Se guardan en `detalles` (jsonb) y `fecha_borrador` de `solicitudes` e `imagenes` (`app_planner/luxora/supabase/5-campos-extra.sql`). El portal detecta si esas columnas existen (`S.v5`); si no, mete esos datos como texto en `detalle`.
- Todo el texto de usuario se pinta con `textContent` mediante el helper `h()`. No uses `innerHTML` con datos.
- Diseño: marino `#001F3F`, oro `#E2C27A`/`#9D791C`, cristal con filo dorado; títulos en Cormorant Garamond y texto en Hanken Grotesk. Mantén ese estilo.

## Reglas de contenido
- Ejemplos y placeholders (`Ej. …`) siempre ficticios: nunca nombres de clientes reales (Farmacias Campero, Laboratorios Económicos), sus productos, precios ni teléfonos reales.
- Sin precios, resultados, testimonios ni afirmaciones médicas inventadas. Publicidad sanitaria responsable.

## Antes de subir un cambio
1. Revisa la sintaxis del script: extrae el último `<script>` y corre `node --check`.
2. Sirve la carpeta (`python3 -m http.server`) y ábrela con Playwright (Chromium ya está instalado) en 1440×900 y 390×844: sin errores en consola y sin desplazamiento horizontal.
3. Commit con mensaje claro en español y push a `main`.
4. Si cambiaste algo que Raúl debe hacer en Supabase, dale el SQL listo y dile exactamente dónde pegarlo.

## Relacionado
- Mesa de Mando (dashboard privado de Raúl) y el agente viven fuera de este repo: artefacto de claude.ai `https://claude.ai/artifact/JynmQTT3ff6389LvRz9NZV` y la sesión "Agente Luxora". Hoy siguen leyendo la base vieja del artefacto, no Supabase.
