# Run State — <RUN-ID>

> Archivo de estado del loop. Mantenerlo actualizado en cada handoff. No
> versionar en el repo (usar `.qa/` o el directorio de artefactos de la sesión).

## 1. Identificación
- Run ID: <RUN-ID>
- Inicio: <fecha/hora>
- Prompt original: <enlace o copia literal>
- Directorio / rama / commit: <...>
- Orquestador: <sesión/agente>

## 2. Estado del proceso

| Campo | Valor |
|-------|-------|
| Estado | `IN_PROGRESS` / `CLOSED_APPROVED` / `CLOSED_CANCELLED` |
| Fase actual | `CONTRATO` / `QA` / `TRIAGE` / `CORRECCION` / `REVALIDACION` / `JUEZ` / `CIERRE` |
| Ronda de juez | <n> |
| Iteraciones totales de corrección | <n> |
| Reaperturas | <n> |
| Último veredicto | `—` / `PASS` / `PASS WITH OBSERVATIONS` / `FAIL` / `BLOCKED` |
| Cierre autorizado por | `—` / `JUEZ` / `USUARIO (cancelación)` |

**Invariante:** el estado no puede pasar a `CLOSED_APPROVED` sin un veredicto
aprobatorio del juez. `CLOSED_CANCELLED` solo con cancelación explícita del
usuario. `BLOCKED` no cierra el proceso.

## 3. Contrato y requisitos

| Req | Prioridad | Criterio de aceptación | Estado | Evidencia | QA dueño |
|-----|-----------|------------------------|--------|-----------|----------|
| R1 | must | <...> | `PENDING` | <...> | <QA-ID> |

## 4. Hallazgos e iteraciones

| Hallazgo | Severidad | Requisito | QA | Corrector | Iter | Estado | Evidencia |
|----------|-----------|-----------|----|-----------|------|--------|-----------|
| F-001 | S1 | R1 | QA-1 | C-1 | 1 | `OPEN` | <...> |

Estados: `OPEN`, `IN_PROGRESS`, `READY_FOR_QA`, `VALIDATED`, `REOPENED`,
`ESCALATED`, `BLOCKED`, `OUT_OF_SCOPE`.

## 5. Rondas de juez

| Ronda | Fecha | Veredicto | Bloqueantes | Acción |
|-------|-------|-----------|-------------|--------|
| 1 | <...> | `FAIL` | F-001, F-003 | Vuelve a QA → corrector |

## 6. Checkpoints y decisiones del usuario
- Checkpoint alcanzado: <iteraciones / rondas / bloqueo>
- Decisión: <continuar / cambiar estrategia / abortar>
- Fecha y contexto: <...>

## 7. Handoffs

| Fecha | De | A | Contenido | Artefacto |
|-------|----|---|-----------|-----------|
| <...> | QA-1 | C-1 | Brief F-001 | <ruta> |
| <...> | C-1 | QA-1 | Corrección READY_FOR_QA | <ruta> |

## 8. Checklist de cierre

- [ ] Todos los requisitos `must` están `VALIDATED`.
- [ ] No hay correctores activos ni ediciones en curso.
- [ ] El juez verificó cada requisito con evidencia propia.
- [ ] El juez emitió `PASS` o `PASS WITH OBSERVATIONS` sin bloqueantes.
- [ ] Se registró el veredicto y el informe final.
- [ ] Si hubo cancelación, quedó registrada explícitamente por el usuario.

Si algún punto falla, el proceso sigue **ABIERTO**.
