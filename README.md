# Governance Pack · plantilla

Plantilla para documentar la gobernanza de IA y de datos de un proyecto de software. Se creó para el taller *Gobernanza de IA y de datos* del programa Business Solutions de Fundación Forward Costa Rica (noviembre de 2026), pero sirve para cualquier proyecto.

El pack contesta, por escrito y dentro del repositorio, cuatro preguntas: **¿quién decide?**, **¿con qué datos?**, **¿quién responde?** y **¿cómo se corrige?**

## Qué trae

```text
AGENTS.md                                   reglas para cualquier agente de código que toque el repo
docs/
├── adr/
│   ├── README.md                           índice de decisiones
│   ├── 0001-modelo-y-proveedor-de-ia.md
│   ├── 0002-que-decide-la-ia.md
│   └── 0003-donde-viven-los-datos.md
└── gobernanza/
    ├── inventario-de-datos.md
    └── registro-de-riesgos.md
.github/
├── pull_request_template.md                checklist de cumplimiento
├── workflows/governance-pack.yml           verificación automática en cada PR
└── governance-pack/verificar-pack.mjs      el verificador (Node.js, sin dependencias)
```

## Cómo usarla en su proyecto

El pack va en **el repositorio de su propio proyecto**, no en una copia de esta plantilla.

1. En la carpeta de su proyecto, creen una rama:

   ```bash
   git checkout -b docs/governance-pack
   ```

2. Copien los archivos de la plantilla sin pisar nada de lo que ya tienen:

   ```bash
   git clone --depth 1 https://github.com/gustmach-star/governance-pack-template /tmp/governance-pack
   cp -Rn /tmp/governance-pack/docs /tmp/governance-pack/.github /tmp/governance-pack/AGENTS.md .
   ```

   Si ya tenían un `AGENTS.md` o un `.github/pull_request_template.md`, no se reemplazan: agreguen a mano las secciones de la plantilla.

3. Llenen los seis archivos. Todo lo que está entre `<` y `>` es un marcador que hay que reemplazar.

4. Pruébenlo en su computadora antes de subir (necesitan Node.js):

   ```bash
   node .github/governance-pack/verificar-pack.mjs
   ```

5. Suban la rama y abran un pull request **dentro de su propio repo**, hacia su rama principal. El bot comenta en el PR qué falta.

   La descripción del PR va a salir vacía: GitHub toma la plantilla de PR de la rama principal, y ahí todavía no está. Copien en la descripción el checklist de `.github/pull_request_template.md`.

## Qué revisa el bot y qué no

Revisa la **forma**: que existan los archivos, que cada ADR tenga estado, fecha, quién decide y sus cinco secciones, que el inventario use las categorías de la Ley 8968 y tenga «Lo que NO recolectamos», que haya al menos tres riesgos completos, que `AGENTS.md` tenga al menos cinco reglas y que no queden marcadores sin llenar.

No revisa la **verdad**. Si un ADR afirma algo que el código no hace, está mal aunque el bot diga que la forma está completa. Eso lo revisa una persona.

El workflow solo se dispara cuando cambian estos archivos, así que no interfiere con el resto de su CI. Pueden borrarlo cuando quieran.

## Para ir más allá

Otras prácticas que aparecen en proyectos con IA en producción:

| Práctica | Para qué sirve |
| --- | --- |
| PRD (Product Requirements Document, documento de requisitos de producto) con los límites de la IA | Deja escrito qué debe hacer el producto y, sobre todo, qué no debe hacer. Los ADRs pueden citar los requisitos del PRD por su código |
| Runbooks | Qué hacer, paso a paso, cuando algo falla: un pico de gasto en IA, una caída, una filtración |
| Bitácora de lecciones | Qué pasó y qué hacer en su lugar. Si una lección cambia una regla, la regla va al ADR o al `AGENTS.md` |
| Banderas de funcionalidad | La función con IA está apagada hasta que alguien con autoridad la aprueba |
| Proveedor detrás de una interfaz propia | Cambiar de modelo o de proveedor sin reescribir el sistema |
| Medir el costo de la IA | Gasto por usuario, presupuesto mensual y alerta |
| Revisiones automáticas periódicas | Vulnerabilidades y dependencias revisadas cada semana |

## Referencias

- Nygard, M. (2011). [Documenting Architecture Decisions](https://cognitect.com/blog/2011/11/15/documenting-architecture-decisions).
- [AGENTS.md](https://agents.md/), estándar abierto.
- [OWASP Top 10 for LLM Applications](https://genai.owasp.org/llm-top-10/) y [OWASP Top 10 for Agentic Applications for 2026](https://genai.owasp.org/resource/2025-12-09/owasp-top-10-for-agentic-applications-for-2026).
- [Ley 8968 de Costa Rica](https://www.micitt.go.cr/sites/default/files/marco_juridico_legal/08.%20Ley%20n.%C2%B0%208968%20Ley%20de%20Protecci%C3%B3n%20de%20la%20Persona%20frente%20al%20tratamiento%20de%20sus%20datos%20personales..pdf), Protección de la Persona frente al tratamiento de sus datos personales.

Autor: Gustavo Machado Loría.
