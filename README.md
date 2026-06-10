# Constitucion Mexicana 🇲🇽

Texto vigente de la **Constitución Política de los Estados Unidos Mexicanos**,
un archivo por artículo, versionado en git para dejar un **rastro auditable de
las reformas**.

> ### 🔎 ¿Quieres hacer preguntas en lenguaje natural?
> Entra a **[constitucion.app](https://constitucion.app)** — pregunta como hablas
> (“¿puedo tener doble nacionalidad?”, “¿qué dice sobre la libertad de
> expresión?”) y obtén respuestas **con la cita exacta** al artículo y la página
> del PDF oficial.

> Repo de **datos**. La lógica que genera estos archivos vive en
> [`constitucion-extractor`](https://github.com/OpenCodice/constitucion-extractor). Aquí no hay código:
> cada commit corresponde a una versión del texto (idealmente, a una reforma
> publicada en el Diario Oficial de la Federación).

## Arquitectura: texto vs. metadata

El principio es simple: **el texto es la fuente de verdad y git es el detector
de cambios.** No usamos string matching para "detectar reformas"; eso lo hace
git. Las fechas de reforma son metadata *derivada* y best-effort, y viven
**aparte** para que nunca contaminen el historial del texto.

```
articulos/                    ← CAPA 1: TEXTO (fuente de verdad)
  001.md  002.md ... 136.md   solo encabezado + texto fiel. Cambia SOLO con la ley.

metadata/                     ← CAPA 3: DERIVADO (regenerable, no canónico)
  articulos.json              artículo → título/capítulo/fechas de reforma
  reformas.json               fecha DOF → artículos afectados
  segmentos/NNN.json          artículo descompuesto en bloques, con cada reforma
                              ya ENLAZADA a su bloque (pensado para LLMs)
  pasajes.jsonl               índice plano para ingesta de un RAG: un pasaje
                              citable por bloque (cita legal, provenance, temporal)
```

### Para consumo por LLM: `metadata/segmentos/`

En el `.md`, qué nota corresponde a qué párrafo se infiere por la posición (la
nota va debajo del bloque que modifica) y por su primera palabra (`Párrafo`,
`Fracción`, `Artículo`...). Para que un LLM **no tenga que inferir nada**, cada
`segmentos/NNN.json` ya trae la asociación explícita:

```json
{
  "reformas_articulo": [                       // notas de alcance "Artículo"
    { "nota": "Artículo reformado DOF 14-08-2001", "fechas": ["2001-08-14"] }
  ],
  "bloques": [
    { "id": "001.p1", "tipo": "parrafo",
      "ruta": "Artículo 1o. › párrafo 1",
      "texto": "En los Estados Unidos Mexicanos...",
      "reformas": [ { "nota": "Párrafo reformado DOF 10-06-2011",
                      "fechas": ["2011-06-10"] } ] }
  ]
}
```

Reglas que evitan la confusión típica: las notas de alcance **Artículo** van en
`reformas_articulo` (no cuelgan de un párrafo); una nota de **Fracción** se ancla
a la fracción completa aunque tenga sub-párrafos; cada bloque tiene `id` único y
una `ruta` legible. El 100% de las notas quedan enlazadas.

### Para RAG con citas: `metadata/pasajes.jsonl`

Un registro por bloque citable (1353 en total), listo para embeber e indexar.
Cada pasaje trae lo necesario para **recuperar, responder y citar de forma
verificable**:

```json
{ "id": "002.A.I",
  "cita": "Artículo 2o., Apartado A, fracción I",   // cita legal formal
  "ruta": "Artículo 2o. › Apartado A › Fracción I",
  "texto": "I. Decidir, conforme a sus sistemas normativos...",
  "texto_embedding": "<ruta + título>\n<texto>",     // contexto para retrieval
  "reformas": ["2024-09-30"],
  "vigente_desde": "2024-09-30", "vigente_hasta": null,
  "version": "2026-06-02",                            // versión del PDF
  "fuente": "...", "url_fuente": "...",
  "archivo_texto": "articulos/002.md" }
```

**Patrón para no alucinar citas:** no dejes que el LLM *escriba* la cita. Que
devuelva `id`s de pasaje; tu aplicación renderiza la cita desde `cita`/`ruta`/
`texto` del registro confiable. Así toda cita apunta a texto real y verificable.

Garantía: regenerar `metadata/` (p. ej. al mejorar la extracción de fechas)
**no modifica ningún `.md`**. Un `git diff` en `articulos/` es siempre un
cambio real en la ley.

Cada `articulos/NNN.md` es texto puro, sin frontmatter. Las notas oficiales de
reforma (que vienen en el PDF) se conservan en *cursiva*:

```markdown
# Artículo 1o.

En los Estados Unidos Mexicanos todas las personas gozarán de los derechos
humanos reconocidos en esta Constitución...

_Párrafo reformado DOF 10-06-2011_
```

La metadata de ese artículo se consulta en `metadata/articulos.json`.

## Para qué sirve el historial de git

- `git log articulos/004.md` → todas las reformas del Artículo 4o.
- `git blame articulos/004.md` → en qué reforma cambió cada párrafo.
- `git diff <reforma_anterior> HEAD -- articulos/` → qué cambió entre versiones.

## Cómo se mantiene al día

Este texto **nunca se edita a mano**: se regenera desde el PDF oficial de la
Cámara de Diputados, y cada reforma publicada en el DOF queda registrada como un
commit. El proceso está automatizado, de modo que el repositorio sigue a la
versión vigente de la Constitución conservando su historial completo de cambios.

## Fuente

Diario Oficial de la Federación — CPEUM, H. Cámara de Diputados del H. Congreso
de la Unión. Este repositorio es una reproducción para fines de consulta y
seguimiento de cambios; el texto oficial es el publicado en el DOF.
