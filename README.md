# Generador de Documentos RRHH — v2 (cálculo de finiquito automatizado)

⚠️ **Esta es una versión aparte, en desarrollo, pensada para ir en un repositorio
de GitHub DISTINTO al que ya tienes funcionando.** No reemplaces tu app actual
con esta hasta que hayas comparado unos cuantos casos reales contra Excel y
confíes en el resultado.

## Qué cambia respecto a tu versión actual

En la pestaña **"Cálculo Finiquitos"**:

1. **El monto "TOTAL FINIQUITO A PAGAR" ahora se calcula automáticamente en
   Python**, reproduciendo (no aproximando) las fórmulas reales de tu planilla
   Excel: vacaciones proporcionales, días inhábiles, e indemnizaciones cuando
   la causal es un despido (Art. 161-1, 161-2, 159-5). Fue verificado fórmula
   por fórmula contra tu plantilla y validado contra un caso real tuyo
   (Alicia Nury Zepeda Araya → exactamente $10.770, igual que tu Excel).
2. **Nuevo botón de descarga: "Excel con MONTO FINIQUITO relleno"** — el mismo
   Excel de datos que subiste, con la columna `MONTO FINIQUITO` ya calculada,
   listo para usar directo en la pestaña "Finiquito + Decl. Jurada" — se acabó
   pasar el monto a mano.
3. Se corrigió un bug donde las fechas de inicio/fin se escribían como texto
   en vez de fecha real de Excel (esto hacía que si abrías la planilla en
   Excel y la recalculabas, los resultados podían salir mal).

## Limitaciones que debes conocer

- **El reconocimiento de la causal es por patrón, no por texto exacto**
  (reconoce "Art. 161-1: ...", "161-1", "161, N°1", etc. — busca el código
  tipo "161-1"). Si una fila tiene una causal que no logra reconocer, la app
  te avisa con una lista de filas — revísalas a mano, no asume $0 en silencio.
- Sueldo mínimo, valor UF, tipo de sueldo (fijo/variable), gratificación,
  colación, movilización, zona extrema, días ya tomados y remuneración
  pendiente se leen **de tu plantilla Excel** (no hay todavía una columna por
  trabajador para variarlos individualmente). Si necesitas que varíen por
  persona, es el siguiente paso natural a construir.
- El cálculo de indemnización por sueldo **variable** (comisiones) está
  portado pero mucho menos probado que el caso de sueldo fijo — úsalo con
  más cautela si aplica a tus datos.

## Recomendación antes de usarla en serio
Antes de confiar en esto para pagos reales: corre 5-10 casos reales tuyos
(incluyendo al menos un despido, no solo renuncias) y compáralos a mano
contra lo que da tu Excel abierto en Excel de verdad. Si todo calza, recién
ahí considera reemplazar tu app en producción.

## Cambios de esta ronda (fusión + ajuste de cálculo)

- **El monto ahora se calcula SIEMPRE como renuncia voluntaria** (vacaciones
  proporcionales + días inhábiles + remuneración pendiente). Ya NO incluye
  indemnización por aviso previo, años de servicio ni obra/faena, sin importar
  la causal — eso lo agregas tú aparte, a mano, cuando corresponda. Esto fue
  a pedido explícito: antes, para causales de despido (Art. 161-1, 161-2,
  159-5), el cálculo sí las incluía y podía dar un monto varias veces más
  alto que solo las vacaciones proporcionales.
- **Se fusionaron "Cálculo Finiquitos" y "Finiquito + Decl. Jurada" en una
  sola pestaña** ("Cálculo + Finiquito + Jurada"): subes el Excel de datos +
  las 3 plantillas (cálculo Excel, finiquito Word, declaración jurada Word),
  un clic, y se genera todo junto — ya no hace falta descargar el Excel con
  el monto y volver a subirlo a mano.
- Se corrigió que el texto de la causal que se escribe en la celda `D14` de
  la planilla de cálculo ahora es el texto EXACTO del desplegable de Excel
  (ej. "Art. 161-1: Necesidades de la empresa"), no el texto libre de tu
  columna de datos — así, si alguna vez abres y recalculas la planilla en
  Excel de verdad, no hay sorpresas de números que no calzan.

---

App web (Streamlit) con 5 herramientas:

1. Generador de Contratos
2. Cálculo de Finiquitos (planilla Excel)
3. Finiquito + Declaración Jurada
4. Anexo de Continuidad
5. Anexo Obrero → Capataz

En cada pestaña subes **tu Excel de datos** y **tu(s) plantilla(s)** (Word con
`«CAMPO»` o Excel con las celdas fijas), le das a generar, y descargas un ZIP.
Nada queda guardado en el servidor: todo se procesa en memoria durante esa
sesión.

## Probarla en tu computador

```bash
pip install -r requirements.txt
streamlit run app.py
```

Se abre solo en `http://localhost:8501`.

## Desplegarla gratis (recomendado: Streamlit Community Cloud)

1. Crea un repositorio en GitHub (puede ser privado) y sube estos 3 archivos:
   `app.py`, `requirements.txt`, `README.md`.
2. Entra a **share.streamlit.io** con tu cuenta de GitHub.
3. Click en "New app" → elige el repo → archivo principal `app.py` → Deploy.
4. En 1-2 minutos te da una URL pública (tipo `tuapp.streamlit.app`) que
   puedes compartir con quien necesite usarla.

Es gratis, no duerme tan agresivo como otras opciones, y cada vez que hagas
`git push` con cambios se actualiza sola.

### Alternativa: Render.com
Si prefieres no usar GitHub público: crea un "Web Service" en Render, conecta
el repo (puede ser privado), y como start command pon:
```
streamlit run app.py --server.port $PORT --server.address 0.0.0.0
```
El tier gratis de Render "duerme" el servicio tras un rato sin uso y tarda
~30 seg en despertar la primera vez que alguien entra — no es un problema
grave para uso ocasional tipo RRHH.

## Nota de seguridad
El script original traía una API key de Groq escrita directo en el código.
La eliminé por completo en esa primera versión. Si en algún momento vuelves a
meter una clave de API en un proyecto, ponla como "Secret"/variable de
entorno en la plataforma de hosting, nunca directo en el código — sobre todo
si el repo es público.

## Activar la IA (opcional)
La app tiene dos funciones de IA, ambas apagadas por defecto y que no rompen
nada si no las configuras:

- **Chat de ayuda** (barra lateral): responde preguntas de cómo usar la app.
- **"Revisar con IA"** (dentro de cada pestaña, después de subir el Excel):
  audita los datos buscando RUTs mal formados, fechas raras, nombres vacíos,
  montos con pinta de error de tipeo, etc.

Para activarlas:
1. Consigue una API key en **console.anthropic.com** (Settings → API Keys).
2. En Streamlit Community Cloud, entra a tu app → **Manage app** → **Settings**
   → **Secrets**, y pega:
   ```toml
   ANTHROPIC_API_KEY = "sk-ant-tu-key-aqui"
   ```
3. Guarda — la app se reinicia sola y ambas funciones de IA quedan activas.

**Importante:** al usar "Revisar con IA", una muestra de los datos del Excel
(nombres, fechas, montos) se envía a la API de Anthropic para el análisis.
Anthropic no entrena modelos con datos enviados por la API por defecto, pero
si vas a procesar datos de otras empresas, vale la pena que lo tengas
presente y lo transparentes a quien use la app.
