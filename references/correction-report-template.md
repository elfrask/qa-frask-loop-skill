# Correction Report — <C-ID>

## 1. Brief recibido
- QA de origen: <QA-ID>
- Hallazgos: <F-..., F-...>
- Requisitos afectados: <R...>
- Criterio de aceptación: <...>
- Iteración: <n>

## 2. Reproducción
- Comando / pasos: <...>
- Resultado antes de corregir: <...>
- ¿Reproducido? Sí / No (si no, explica)

## 3. Causa raíz
<Explicación técnica breve y concreta.>

## 4. Cambios

| Archivo | Cambio | Motivo |
|---------|--------|--------|
| <ruta> | <...> | <...> |

- Diff resumido: <...>
- ¿Se tocó algo fuera del brief? Sí/No — <detalle>

## 5. Verificación

| Verificación | Comando | Resultado |
|--------------|---------|-----------|
| Test unitario | <...> | PASS / FAIL |
| Build | <...> | PASS / FAIL |
| Lint | <...> | PASS / FAIL |
| Flujo manual | <...> | PASS / FAIL |

- Evidencia antes/después: <capturas, logs, salidas>

## 6. Riesgo residual
- <...>

## 7. Nuevos hallazgos (no corregidos)
- <descripción, severidad sugerida, evidencia> / Ninguno

## 8. Estado
- **READY_FOR_QA** — pendiente de validación por <QA-ID>.
- No autoaprobado.
- **El loop sigue abierto** hasta la validación del QA y la aprobación final del
  juez. READY_FOR_QA no es un cierre.
