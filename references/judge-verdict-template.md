# Judge Verdict — <RUN-ID>

## 0. Estado del loop
- Estado del proceso: `IN_PROGRESS` / `CLOSED_APPROVED` / `CLOSED_CANCELLED`
- Ronda de juez: <n> de <máx. 2 antes de checkpoint>
- Iteraciones de corrección acumuladas: <...>
- Reaperturas: <...>
- Veredictos previos: <...>

## 1. Entradas revisadas
- Prompt original: <...>
- Contrato: <R1..Rn>
- Reportes QA: <...>
- Reportes de corrección: <...>
- Diff / commit: <...>

## 2. Verificación independiente
- Comandos ejecutados por el juez: <...>
- App/flujos probados: <...>
- Capturas / logs: <...>
- Diferencias entre reportes y estado real: <...>

## 3. Adjudicación por requisito

| Req | Prioridad | Veredicto | Evidencia propia | Notas |
|-----|-----------|-----------|------------------|-------|
| R1 | must | PASS / FAIL / BLOCKED | <...> | <...> |

## 4. Verificación de correcciones
- Correcciones aplicadas y funcionales: <...>
- Correcciones incompletas o cosméticas: <...>
- Regresiones detectadas: <...>

## 5. Controles anti-cheat

| Control | Resultado | Evidencia |
|---------|-----------|-----------|
| Tests eliminados / debilitados / omitidos | OK / ALERTA | <...> |
| Mocks o stubs que ocultan comportamiento | OK / ALERTA | <...> |
| Hardcodes o valores de prueba | OK / ALERTA | <...> |
| Flags que desactivan la feature | OK / ALERTA | <...> |
| Alcance inflado / cambios no pedidos | OK / ALERTA | <...> |
| Evidencia incongruente | OK / ALERTA | <...> |

## 6. Veredicto global
**PASS / PASS WITH OBSERVATIONS / FAIL**

- Justificación: <...>
- Bloqueantes (si FAIL): <F-..., impacto, evidencia>
- Observaciones no bloqueantes: <...>

## 7. Riesgos residuales y limitaciones
- <...>

## 8. Siguiente acción
- **FAIL:** el proceso permanece `IN_PROGRESS` y **ABIERTO**; vuelve al ciclo
  QA → corrector → revalidación con estos bloqueantes. No es un cierre.
- **PASS / PASS WITH OBSERVATIONS sin bloqueantes:** autoriza el cierre y el
  informe final.
- **Checkpoint alcanzado:** pausa para decisión del usuario sin cerrar el
  proceso.
- **Cancelación:** solo si el usuario la pidió explícitamente; registrar
  `CLOSED_CANCELLED` y no declarar aprobación.
