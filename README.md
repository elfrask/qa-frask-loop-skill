# QA Frask Loop Skill

Skill de orquestación multiagente para validar que una implementación cumple
**exactamente** las indicaciones del usuario.

Despliega un escuadrón con roles separados —**QA**, **correctores** y un
**juez**— y mantiene el ciclo de pruebas y corrección abierto hasta que el juez
aprueba el resultado.

## Qué hace

- Convierte el prompt del usuario en un **contrato de validación** con requisitos
  atómicos y evidencia esperada.
- Despliega uno o varios **QA** con lentes funcional, visual/UX y
  técnica/regresión.
- Los QA prueban flujos con herramientas agenticas, testing, navegador, capturas
  y revisión de código, y reportan hallazgos con reproducción y evidencia.
- Cada fallo se envía a un **corrector**, que reproduce, corrige la causa raíz y
  devuelve el resultado al QA de origen.
- El QA revalida y solo él puede marcar un hallazgo como `VALIDATED`.
- Un **juez independiente** verifica cada requisito, las correcciones y posibles
  atajos indebidos, y emite el veredicto final.

## Regla del loop

El proceso **no se cierra hasta que el juez apruebe**:

- `PASS` o `PASS WITH OBSERVATIONS` sin bloqueantes → cierre e informe final.
- `FAIL` → vuelve al ciclo QA → corrector → revalidación.
- `BLOCKED` no es aprobación: se devuelve como proceso abierto.
- Solo una cancelación explícita del usuario puede cerrar el proceso sin
  aprobación.
- Los topes de iteraciones son **checkpoints de escalado**, no salidas.

## Instalación

### Skills CLI

```bash
npx skills add elfrask/qa-frask-loop-skill
```

### Manual

Copia la carpeta a tu directorio de skills:

```bash
# OpenCode / agentes compatibles
~/.agents/skills/qa-skill/
# o en un proyecto
.opencode/skill/qa-skill/
```

La estructura debe quedar así:

```text
qa-skill/
├─ SKILL.md
└─ references/
   ├─ agent-prompts.md
   ├─ correction-report-template.md
   ├─ judge-verdict-template.md
   ├─ qa-report-template.md
   └─ run-state-template.md
```

## Uso

Invoca el skill cuando quieras QA, validación, revisión o un veredicto final:

```text
Haz QA de esta feature y no cierres hasta que el juez apruebe.
Revisa que el flujo de pago cumpla exactamente lo que pedí.
Valida esta implementación con QA, correcciones y juez.
```

El skill toma el prompt original como fuente de verdad y lo usa como contrato de
validación.

## Requisitos

- Un agente compatible con skills (OpenCode, Claude Code y similares).
- Soporte de subagentes para el despliegue en paralelo. Si no está disponible, el
  skill simula los roles en secuencia manteniendo la separación de
  responsabilidades.
- Herramientas de ejecución para tests, build, navegador y capturas según el
  proyecto.

## Notas

- No hace push, deploy ni toca producción salvo autorización explícita.
- El informe final solo se emite tras la aprobación del juez.
- Las plantillas de handoff están en `references/`.

## Licencia

Sin licencia definida. Si quieres publicarla con una licencia abierta, añade el
archivo `LICENSE` correspondiente.
