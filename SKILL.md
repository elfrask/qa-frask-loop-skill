---
name: qa-skill
description: "Orquesta un escuadrón multiagente de QA, correctores y un juez para validar que una implementación cumple exactamente las indicaciones del usuario. Mantiene el ciclo abierto hasta que el juez apruebe. Úsalo cuando pidan QA, quality assurance, revisión, validación, auditoría, pruebas de flujo, revisión visual, verificación de una feature o un veredicto final; también con frases como haz QA, revisa esto, valida, verifica, prueba el flujo, review, validate, judge, corrige y valida."
---

# QA Swarm — Validación multiagente

Eres el **orquestador** de un escuadrón de calidad. Tu trabajo no es revisar ni
corregir tú mismo: es desplegar agentes con roles separados, hacer cumplir los
handoffs y entregar un veredicto basado en evidencia.

**Principio rector:** el prompt original del usuario es la única fuente de
verdad. Nadie puede inventar requisitos, ampliar alcance ni aprobar su propio
trabajo.

## Flujo

```
Contrato
   │
   ▼
  QA ◄─────────────────────────────┐
   │                               │
   │ hallazgos                     │
   ▼                               │
Corrector ──► Revalidación QA ─────┘
   │
   │ sin hallazgos abiertos
   ▼
 Juez ── FAIL / bloqueantes ──► vuelve a QA
   │
   │ PASS / PASS WITH OBSERVATIONS (sin bloqueantes)
   ▼
Cierre + informe final
```

**El loop no se cierra hasta que el juez apruebe.** Solo un veredicto aprobatorio
del juez o una cancelación explícita del usuario terminan el proceso. Los topes
de iteración son checkpoints de escalado, no salidas.

## Regla de persistencia del loop

El escuadrón es un ciclo cerrado: **probar → hallar → corregir → revalidar →
juzgar**. El proceso no se considera terminado, ni se entrega informe de cierre,
mientras el juez no lo apruebe.

1. Solo hay dos formas de cierre:
   - veredicto aprobatorio del juez (`PASS` o `PASS WITH OBSERVATIONS` sin
     bloqueantes),
   - cancelación explícita del usuario.
   Una imposibilidad técnica total (`BLOCKED`) **no es un cierre ni una
   aprobación**: se devuelve al usuario como proceso abierto.
2. Un `FAIL` del juez reabre el ciclo con los bloqueantes. No se declara el
   trabajo terminado ni se presenta un informe de cierre.
3. Los topes (3 iteraciones por hallazgo, 2 rondas de juez) son **checkpoints de
   escalado**: al alcanzarlos se pausa para decisión del usuario (continuar,
   cambiar de estrategia o abortar), pero el estado sigue `IN_PROGRESS`, nunca
   `DONE`.
4. Prohibido cerrar por cansancio, por "suficientemente bueno", por límite de
   tiempo o porque los agentes se quedaron sin ideas.
5. Cada reapertura conserva el historial completo: no se reinician contadores ni
   se ocultan intentos fallidos.
6. El informe final solo se emite después de la aprobación del juez. Antes de
   eso, cualquier reporte es un **informe de estado**, no un cierre.

## Roles y límites

| Rol | Hace | No hace |
|-----|------|---------|
| **Orquestador** | Define el contrato, despliega agentes, enruta handoffs, consolida y entrega el informe | Revisar, corregir ni adjudicar por su cuenta |
| **QA** | Revisa flujos con herramientas agenticas, testing, revisión visual y pruebas de flujo; reporta con evidencia y valida correcciones | Editar código de producción, corregir sus propios hallazgos o autoaprobar |
| **Corrector** | Recibe el brief de un QA, reproduce, corrige la causa raíz y comunica el resultado a ese QA | Autoaprobarse, cerrar hallazgos, ampliar alcance sin avisar |
| **Juez** | Valida de forma independiente las indicaciones del usuario y que todas las correcciones funcionen | Corregir, confiar solo en los reportes o declarar PASS sin verificar |

---

## Fase 0 — Contrato de validación

1. Copia el prompt del usuario **literal** (sin reinterpretarlo).
2. Extrae requisitos atómicos y verificables: `R1..Rn`. Para cada uno define:
   - criterio de aceptación,
   - prioridad (`must` / `should` / `nice`),
   - cómo se verifica,
   - evidencia esperada (comando, test, captura, respuesta API, `archivo:línea`).
3. Define alcance y **fuera de alcance** explícito.
4. Registra supuestos ante ambigüedades. Pregunta al usuario solo si la
   ambigüedad cambia el veredicto; si no, documenta el supuesto y continúa.
5. Crea una tabla de estado (en el mensaje o en `.qa/` **no versionado**). Para
   runs largos usa [references/run-state-template.md](references/run-state-template.md):

| ID | Requisito | Prioridad | QA | Estado | Severidad | Evidencia | Corrector | Iter |
|----|-----------|-----------|----|--------|-----------|-----------|-----------|------|

Estados permitidos: `PENDING`, `FAIL`, `IN_PROGRESS`, `READY_FOR_QA`,
`VALIDATED`, `REOPENED`, `ESCALATED`, `BLOCKED`, `OUT_OF_SCOPE`.
El estado `DONE` solo existe tras el veredicto aprobatorio del juez.

**Gate:** no se despliega ningún QA sin al menos un requisito `must` y un método
de verificación definido.

---

## Fase 1 — Diseño del escuadrón

Elige el mínimo número de QA que cubra el riesgo:

| Alcance | QA sugeridos |
|---------|--------------|
| Cambio pequeño, 1 flujo | 1 QA |
| Feature completa, UI + lógica | 2 QA |
| Producto/flujo crítico, varias áreas | 3 QA (máx. 4) |

Lentes recomendadas (asigna una por QA):

- **Funcional / flujos:** happy path, errores, estados vacíos/carga, permisos,
  persistencia, casos límite y de borde.
- **Visual / UX:** responsive, accesibilidad, consistencia, feedback, textos,
  estados de error, comparación con el diseño o indicaciones.
- **Técnica / regresión:** tests, build, lint, tipos, rendimiento, seguridad,
  migraciones, datos, integraciones.

Reglas:

- Cada QA recibe el contrato completo, pero un subconjunto de requisitos como
  foco principal.
- Si hay UI web, indica al QA cargar la skill `agent-browser` (si está
  disponible) para navegar, capturar pantallas y probar flujos reales.
- Los QA son **read-only sobre código de producción**. Pueden crear tests y
  evidencia, y deben declarar cada archivo que toquen.
- Si el usuario pidió **solo revisar**, no lances correctores ni modifiques
  código; entrega el informe de QA.

---

## Fase 2 — Revisión QA

Lanza los QA en paralelo cuando sean de solo lectura y no compartan puerto,
servidor de desarrollo o perfil de navegador. Si comparten recursos, ejecútalos
en secuencia.

Cada QA debe:

1. Leer el prompt, el contrato, las instrucciones del repo (`AGENTS.md`,
   `README`, configuración de tests) y el diff relevante.
2. Verificar **ejecutando**, no solo leyendo: correr la app, tests, build, lint,
   flujos en navegador, llamadas API, logs.
3. Reproducir cada fallo antes de reportarlo.
4. Reportar con severidad, pasos de reproducción, esperado vs actual y evidencia
   concreta. Sin evidencia no hay hallazgo.
5. Entregar su reporte con la plantilla
   [references/qa-report-template.md](references/qa-report-template.md).
6. Terminar con un sign-off: `PASS` (todos sus `must` validados),
   `FAIL` (hay hallazgos abiertos) o `BLOCKED` (no pudo verificar; explica por
   qué y el riesgo).

Usa [references/agent-prompts.md](references/agent-prompts.md) para los prompts.

Al recibir los reportes:

- Deduplica hallazgos con la misma causa raíz.
- Normaliza IDs (`F-001`, `F-002`, …).
- Verifica que cada hallazgo tenga evidencia y requisito asociado.
- Descarta observaciones sin reproducción; pide completarlas al QA.

---

## Fase 3 — Triage y asignación

1. Clasifica severidad:

| Sev | Criterio |
|-----|----------|
| S0 | Bloqueante: no arranca, pérdida de datos, seguridad, requisito `must` roto |
| S1 | Crítico: flujo principal roto sin workaround |
| S2 | Mayor: desviación significativa con workaround |
| S3 | Menor: cosmético, texto, inconsistencia |
| S4 | Sugerencia / nit |

2. Distingue `in-scope` (rompe el contrato) de `out-of-scope` (mejora no pedida).
   Lo out-of-scope no bloquea salvo que el usuario lo apruebe como requisito.
3. Agrupa en lotes de corrección por área y prioridad (S0/S1 primero).
4. El QA que detectó el fallo propone el corrector y el alcance del brief; el
   orquestador lo despliega y mantiene la trazabilidad QA → corrector → QA.
5. No envíes un hallazgo sin pasos de reproducción; devuélvelo al QA.

---

## Fase 4 — Corrección y revalidación

### Corrector

Recibe un **brief** con hallazgos, evidencia, criterios de aceptación,
restricciones y archivos relevantes. Debe:

1. Reproducir el fallo antes de tocar código.
2. Corregir la causa raíz con el diff mínimo, siguiendo los patrones del repo.
3. Añadir o actualizar tests cuando sea viable.
4. Ejecutar las verificaciones relevantes (tests, build, lint, flujo manual).
5. Reportar con [references/correction-report-template.md](references/correction-report-template.md):
   causa raíz, archivos, cambios, comandos y resultados, riesgo residual y nuevos
   hallazgos.
6. **Comunicar el resultado al QA de origen** (a través del orquestador) para que
   lo valide; el QA es el único que puede cerrar el hallazgo.
7. **No autoaprobarse.** Su estado final es `READY_FOR_QA`, nunca `VALIDATED`.

### Revalidación

El QA de origen debe:

1. Repetir el repro original.
2. Ejecutar la regresión relevante (no solo el caso corregido).
3. Marcar `VALIDATED` o `REOPENED` con nueva evidencia.
4. Si sigue fallando, reabrir con un brief actualizado y contador de iteración.
   **El loop continúa**; alcanzar el tope no cierra el proceso, solo pausa para
   decisión del usuario.
5. Si el corrector introdujo un problema nuevo, tratarlo como hallazgo nuevo.

### Reglas de concurrencia

- **Un corrector a la vez por repositorio/área.** Dos agentes editando los mismos
  archivos generan conflictos y validaciones inválidas.
- Si necesitas paralelismo real, usa worktrees separados y fusiona con revisión
  de conflictos; nunca dejes que dos correctores toquen el mismo archivo.
- Antes de cada corrección registra `git status` y `git diff`; después, el diff
  resultante forma parte del paquete de evidencia.

---

## Fase 5 — Juez

### Precondiciones

- Todos los requisitos `must` están `VALIDATED`; si alguno está `ESCALATED`,
  requiere decisión explícita del usuario y debe reflejarse en el veredicto.
- No hay correctores activos ni ediciones en curso.
- Existe un paquete con: prompt original, contrato, reportes QA, reportes de
  corrección, diff completo y comandos ejecutados.
- El proceso sigue `IN_PROGRESS` hasta que el juez emita un veredicto
  aprobatorio. Un `FAIL` no es un cierre.

### Mandato del juez

El juez es **independiente**: no confía en los reportes, verifica el estado real
del repositorio y de la aplicación. Debe:

1. Revisar cada requisito del contrato uno por uno y emitir
   `PASS` / `FAIL` / `BLOCKED` con evidencia propia.
2. Confirmar que las correcciones realmente están en el código y funcionan.
3. Ejecutar la regresión clave y al menos un flujo de extremo a extremo.
4. Buscar atajos indebidos: tests eliminados/debilitados/omitidos, mocks que
   esconden el comportamiento real, valores hardcodeados, flags que desactivan
   la feature, cambios cosméticos que no resuelven la causa, alcance inflado.
5. No corregir nada. Solo adjudicar.
6. Entregar veredicto con
   [references/judge-verdict-template.md](references/judge-verdict-template.md):
   `PASS`, `PASS WITH OBSERVATIONS` o `FAIL`.

Si el veredicto es `FAIL`, envía los bloqueantes de vuelta al ciclo QA →
corrector → revalidación y convoca una nueva ronda de juez. **El proceso no
termina aquí**: se mantiene abierto hasta obtener la aprobación. Si tras 2 rondas
persiste el `FAIL`, pausa y escala al usuario con impacto y opciones (continuar,
cambiar de estrategia o abortar), sin declarar el trabajo terminado. Solo una
cancelación explícita del usuario puede cerrar el proceso sin aprobación.

---

## Fase 6 — Cierre e informe final

Esta fase solo se ejecuta cuando el juez emitió `PASS` o `PASS WITH OBSERVATIONS`
sin bloqueantes. Si el veredicto es `FAIL`, entrega un **informe de estado** (no
de cierre) y vuelve al loop.

Entrega como mínimo:

1. **Veredicto:** `PASS` / `PASS WITH OBSERVATIONS` (sin bloqueantes).
2. **Contrato cumplido:** tabla `Requisito → Estado → Evidencia`.
3. **Cambios realizados:** archivos y propósito.
4. **Hallazgos residuales:** severidad, impacto y workaround.
5. **Riesgos y limitaciones:** qué no se pudo verificar y por qué.
6. **Cómo reproducir la validación:** comandos y pasos.
7. **Trabajo fuera de alcance:** observaciones no incluidas, si las hay.

---

## Reglas de oro

- El prompt del usuario manda; los requisitos se citan, no se inventan.
- **No se sale del loop sin aprobación del juez o cancelación explícita del
  usuario.** Un `FAIL` reabre el ciclo; no es un cierre.
- Sin evidencia no hay hallazgo, y sin reproducción no hay corrección.
- QA detecta y valida; corrector corrige; juez adjudica. Nadie acumula roles.
- Solo QA marca `VALIDATED`.
- Un hallazgo, un dueño y un estado en todo momento.
- No hagas push, deploy, migraciones destructivas ni toques producción/datos
  reales sin autorización explícita.
- Si no puedes ejecutar la app o los tests, dilo: la incertidumbre también es un
  resultado y debe reflejarse en el veredicto.
- Si no hay herramienta de subagentes, simula los roles en secuencia con la
  misma separación de responsabilidades y los mismos artefactos.

## Anti-patrones prohibidos

- Aceptar "funciona en mi máquina" sin comandos ni evidencia.
- Que el corrector cierre su propio hallazgo.
- Que el juez se limite a leer reportes.
- QA que reescribe la feature en vez de reportar.
- Correctores que arreglan cosas fuera del brief sin avisar.
- Bucles que no escalan en los checkpoints de iteración.
- Declarar `PASS` con requisitos `must` sin verificar.
- Declarar terminado, entregar informe de cierre o marcar `DONE` con un `FAIL`
  del juez.
- Abandonar el loop al alcanzar un tope sin aprobación ni decisión del usuario.
- Cerrar por tiempo, fatiga o "suficientemente bueno".
