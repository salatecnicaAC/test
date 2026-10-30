# WaterPartners Consultor

Agente experto en desalinización, plantas desalinizadoras, ósmosis inversa, automatización, instrumentación, potencia, puesta a tierra, química de procesos y potabilización de agua. Incluye una interfaz de chat bilingüe (ES/EN) y un widget embebible para cualquier página web.

## Requisitos

- Node.js 20.9 o superior
- Una clave de API de un proveedor compatible (Gemini gratis recomendado, OpenAI, OpenRouter o Groq) o Ollama local

## Puesta en marcha

```powershell
Copy-Item .env.example .env.local
# Edita .env.local: AI_PROVIDER=gemini y pega tu AI_API_KEY
npm install
npm run dev
```

Para obtener la clave gratuita de Gemini: https://aistudio.google.com/apikey (basta una cuenta de Google). Abre `http://localhost:3000`.

Para producción:

```powershell
npm run build
npm start
```

## Proveedores

| Proveedor | `AI_PROVIDER` | Coste | Notas |
| --- | --- | --- | --- |
| Google Gemini | `gemini` | Nivel gratuito | Recomendado. Clave en AI Studio, modelo `gemini-3.5-flash-lite` con respaldo a `gemini-3-flash-preview` y `gemini-3.1-flash-lite`. |
| OpenRouter | `openrouter` | Modelos `:free` | Requiere clave. Los identificadores de modelos gratuitos cambian con frecuencia. |
| Groq | `groq` | Nivel gratuito | Muy rápido. Requiere clave. |
| Ollama | `ollama` | Gratis, local | Sin clave. Requiere instalar Ollama y descargar un modelo; necesita RAM y CPU/GPU. |
| OpenAI | `openai` | De pago | Usa créditos de la cuenta. |

La app habla el formato OpenAI Chat Completions en todos los casos, así que puedes cambiar de proveedor solo con variables de entorno. Los límites y la disponibilidad de los niveles gratuitos dependen de cada proveedor.

Gemini retira modelos para cuentas nuevas y a veces satura los más recientes con errores `503`. Por eso la app prueba `AI_MODEL` y, si falla, los modelos de `AI_FALLBACK_MODELS`, recordando durante 10 minutos el último que funcionó. Si quieres usar un modelo más potente cuando haya capacidad, pon `AI_MODEL=gemini-3.8-flash` y deja los ligeros como respaldo.

Para producción:

```powershell
npm run build
npm start
```

## Variables de entorno

| Variable | Obligatoria | Descripción |
| --- | --- | --- |
| `AI_PROVIDER` | No | `gemini` (por defecto en `.env.example`), `openai`, `openrouter`, `groq` u `ollama`. |
| `AI_API_KEY` | Según proveedor | Clave del proveedor. Solo se usa en el servidor. Ollama no la necesita. |
| `AI_MODEL` | No | Modelo a usar. Si se omite se aplica el predeterminado del proveedor. |
| `AI_FALLBACK_MODELS` | No | Modelos de respaldo separados por comas; se usan si el principal devuelve 503, 404 o 429. |
| `AI_BASE_URL` | No | Endpoint personalizado compatible con OpenAI. |
| `CHAT_RATE_LIMIT` | No | Peticiones por minuto y por IP. Por defecto `20`. |
| `OPENAI_API_KEY`, `OPENAI_MODEL`, `OPENAI_BASE_URL` | No | Compatibilidad con la configuración anterior; solo se usan con `AI_PROVIDER=openai`. |

Claves alternativas reconocidas automáticamente: `GEMINI_API_KEY`/`GOOGLE_API_KEY`, `OPENROUTER_API_KEY`, `GROQ_API_KEY` y `OPENAI_API_KEY`.

## Integrar el agente en una página web

### Opción 1: widget flotante (recomendado)

Copia `public/embed.js` a tu sitio o publícalo junto con la app y añade antes de `</body>`:

```html
<script
  src="https://TU-DOMINIO.com/embed.js"
  data-base="https://TU-DOMINIO.com"
  data-position="right"
  data-label="WaterPartners Consultor"
  defer
></script>
```

- `data-base`: origen donde está desplegada esta app (obligatorio si el script se sirve desde otro dominio).
- `data-position`: `right` (por defecto) o `left`.
- `data-label`: texto visible junto a la nube y texto accesible del botón. Por defecto `WaterPartners Consultor`.

### Opción 2: iframe directo

```html
<iframe
  src="https://TU-DOMINIO.com/embed"
  title="WaterPartners Consultor"
  style="width: 400px; height: 640px; border: 0; border-radius: 18px"
  allow="clipboard-write"
></iframe>
```

La página `/embed` es una versión compacta del chat, sin portada ni pie de página.

## API

`POST /api/chat`

```json
{
  "lang": "es",
  "messages": [
    { "role": "user", "content": "¿Cómo dimensiono una planta de ósmosis inversa?" }
  ]
}
```

La respuesta es `text/plain` en streaming. Los errores se devuelven como JSON `{ "error": "..." }` con el código HTTP correspondiente. Límites: 30 mensajes por petición, 12.000 caracteres por mensaje y 60.000 en total.

## Estructura

```
app/
  page.tsx            Portada con chat
  embed/page.tsx      Vista compacta para incrustar
  api/chat/route.ts   Endpoint de streaming y validación
  globals.css         Estilos de la interfaz
components/
  Chat.tsx            Interfaz de conversación
  Logo.tsx            Logotipo SVG
lib/
  systemPrompt.ts     Perfil y criterios del agente experto
  aiConfig.ts         Proveedor, endpoint, modelo y clave
  i18n.ts             Textos, especialidades y consultas sugeridas
public/
  embed.js            Widget flotante embebible
```

## Personalización

- **Conocimiento y comportamiento del agente:** edita `lib/systemPrompt.ts`.
- **Especialidades, sugerencias y textos de la interfaz:** edita `lib/i18n.ts`.
- **Modelo o proveedor:** cambia `AI_PROVIDER` y `AI_MODEL` en `.env.local`.
- **Apariencia:** variables y estilos en `app/globals.css`.
- **Dominio permitido para el widget:** si necesitas restringir qué sitios pueden incrustarlo, añade una comprobación de `Origin` o cabeceras `Content-Security-Policy: frame-ancestors` en `next.config.mjs`.

## Despliegue

En Vercel: importa el repositorio, configura `AI_PROVIDER` y `AI_API_KEY` (y `AI_MODEL` si aplica) y despliega. En otros entornos, ejecuta `npm run build` y `npm start` detrás de HTTPS. Para Ollama, el servidor debe poder alcanzar `http://127.0.0.1:11434` o el `AI_BASE_URL` configurado.

## Notas

- La clave de API nunca se expone al navegador: todas las llamadas pasan por `/api/chat`.
- El límite de peticiones es en memoria; en despliegues con varias instancias usa un almacén compartido (Redis, Upstash) para un límite global.
- Las respuestas son orientación técnica y no sustituyen normativa local, cálculos firmados ni supervisión de personal cualificado.
