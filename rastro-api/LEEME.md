# Cómo activar la IA real con Gemini gratuito (15 minutos)

El sitio (`index.html`) no puede guardar la clave de la API: cualquiera la vería. Por eso hay un servidor intermedio gratuito (Cloudflare Worker) que la guarda en secreto, oculta datos personales, limita el uso y valida la respuesta del modelo.

## 1. Obtener la clave de Gemini (gratis)
1. Entren a **https://aistudio.google.com** con una cuenta de Google **personal** (de preferencia de un adulto responsable; las cuentas escolares a veces bloquean estos servicios).
2. Busquen **Get API key** → **Create API key** y copien la clave (no la compartan por chat, correo ni capturas).
3. Revisen en **https://aistudio.google.com/rate-limit** qué modelos tienen nivel gratuito y cuáles son sus límites. Anoten un modelo disponible (en noviembre de 2026 funciona `gemini-3.5-flash-lite`; `gemini-2.5-flash` ya no acepta cuentas nuevas). En `MODEL` de `wrangler.toml` pueden poner dos separados por coma: el segundo se usa si el primero no existe o tarda más de 11 segundos.

## 2. Instalar y desplegar (terminal, carpeta `rastro-api`)
```bash
npm install
npx wrangler login
npx wrangler secret put GEMINI_API_KEY
npx wrangler deploy
```
- `wrangler login` abre el navegador para crear/entrar a una cuenta gratuita de Cloudflare.
- `secret put` pide pegar la clave (no se muestra ni se guarda en archivos).
- `deploy` imprime una URL como `https://rastro-api.TU-USUARIO.workers.dev`.

## 3. Conectar el sitio
En `index.html`, busca `const API_URL = "";` y pega la URL del Worker:
```js
const API_URL = "https://rastro-api.TU-USUARIO.workers.dev";
```

## 4. Antes de la entrega
- En `wrangler.toml` cambia `ALLOWED_ORIGIN` por el origen de tu sitio publicado (por ejemplo `https://tu-sitio.netlify.app`, sin `/` final) y vuelve a ejecutar `npx wrangler deploy`.
- Prueba 10 preguntas reales. Debe aparecer "🤖 Clasificación y redacción con IA (Gemini)". Si ves "redactada localmente", el Worker no respondió y se usó el respaldo.
- **Guarda capturas** de esas pruebas: son evidencia de uso de IA para la convocatoria.

## Si algo falla (el Worker devuelve un `detalle`)
Abre la consola del navegador (F12) → pestaña Network → la petición al Worker → respuesta.
| Mensaje | Qué hacer |
|---|---|
| `HTTP 404 ... model` | El modelo de `MODEL` no existe o no está en el nivel gratuito. Cambia `MODEL` en `wrangler.toml` por uno de https://aistudio.google.com/rate-limit y `npx wrangler deploy`. |
| `HTTP 400/403 ... API key` | La clave es incorrecta o está restringida. Crea otra y repite `wrangler secret put GEMINI_API_KEY`. |
| `lento` | El modelo no contestó en 11 segundos. El `detalle` dice cuál; pongan otro modelo primero en `MODEL` y `npx wrangler deploy`. |
| `cuota` | Se agotó el límite gratuito. El sitio usa el respaldo local; esperen o reduzcan pruebas seguidas. |
| `truncado` / `bloqueado` | Respuesta cortada o filtrada por seguridad. El sitio usa el respaldo local. |
| `sin_clave` | Falta el secreto: repite `wrangler secret put GEMINI_API_KEY`. |

## Pruebas automáticas (no llaman a Google)
```bash
npm test
```

## Qué debe declararse (transparencia, criterio de ética)
En el nivel gratuito, Google indica que lo enviado **puede usarse para mejorar sus productos** y que hay límites de uso. Esto ya está explicado en la sección "IA y ética" del sitio y en `ENTREGA_innovatiQ.md`. Mantengan esa declaración y no pidan a los usuarios escribir datos de ningún caso.

## Seguridad que ya incluye
- La clave vive solo en el Worker y viaja en una cabecera, no en la URL.
- Oculta teléfonos, correos y CURP antes de llamar al modelo; el Worker no registra los mensajes.
- Máximo 400 caracteres por mensaje y 6 preguntas por minuto por IP (cuida la cuota gratuita).
- El modelo solo recibe la base verificada (`src/kb.js`); su respuesta es JSON validado y **cualquier fuente que no exista se descarta**.
- Si el modelo falla, se agota la cuota o bloquea la respuesta, el sitio responde con el modo de respaldo local.
