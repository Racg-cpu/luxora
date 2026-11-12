# Luxora

Sitio de Luxora, agencia de marketing digital en Guasave, Sinaloa.

- `index.html`: landing pública.
- `portal/index.html`: portal de clientes conectado a Supabase (inicio de sesión con correo y contraseña; cada cliente solo ve sus negocios por reglas RLS del servidor).
- `vendor/`: librería `@supabase/supabase-js` 2.45.4 (licencia MIT, ver `vendor/SUPABASE-LICENSE`), incluida aquí para no depender de un CDN.

Publicado con Cloudflare Pages en https://luxora-3c8.pages.dev (el link viejo de GitHub Pages redirige aquí)

La clave de Supabase incluida en el portal es la clave **publicable**, hecha para ir en el navegador. Nunca subas aquí la `service_role` ni la `secret key`.
