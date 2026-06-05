# constitucion-mexicana

Texto vigente de la **Constitución Política de los Estados Unidos Mexicanos**,
un archivo por artículo, versionado en git para dejar un **rastro auditable de
las reformas**.

> Repo de **datos**. La lógica que genera estos archivos vive en
> [`constitucion-extractor`](../constitucion-extractor). Aquí no hay código:
> cada commit corresponde a una versión del texto (idealmente, a una reforma
> publicada en el Diario Oficial de la Federación).

## Estructura

```
articulos/
  001.md   002.md   ...   136.md     # un artículo por archivo
metadata/
  articulos.json                     # índice: artículo → título/capítulo/reformas
  reformas.json                      # fecha DOF → artículos afectados
```

Cada `articulos/NNN.md` lleva frontmatter con su metadata y el texto en
párrafos. Las notas oficiales de reforma van en *cursiva*:

```markdown
---
articulo: 1
clave: "001"
titulo: "Título Primero"
capitulo: "Capítulo I"
ultima_reforma: 2011-06-10
reformas: [2001-08-14, 2006-12-04, 2011-06-10]
fuente: "Diario Oficial de la Federación — CPEUM, H. Cámara de Diputados"
---

# Artículo 1o.

En los Estados Unidos Mexicanos todas las personas gozarán de los derechos
humanos reconocidos en esta Constitución...

_Párrafo reformado DOF 10-06-2011_
```

## Para qué sirve el historial de git

- `git log articulos/004.md` → todas las reformas del Artículo 4o.
- `git blame articulos/004.md` → en qué reforma cambió cada párrafo.
- `git diff <reforma_anterior> HEAD -- articulos/` → qué cambió entre versiones.

## Cómo se actualiza

No se edita a mano. Cuando hay una reforma, se regenera desde el PDF oficial
con el extractor y se commitea el diff resultante. Ver el README de
[`constitucion-extractor`](../constitucion-extractor#flujo-de-actualización-rastro-de-reformas).

## Fuente

Diario Oficial de la Federación — CPEUM, H. Cámara de Diputados del H. Congreso
de la Unión. Este repositorio es una reproducción para fines de consulta y
seguimiento de cambios; el texto oficial es el publicado en el DOF.
