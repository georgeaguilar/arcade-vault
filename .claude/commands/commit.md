---
description: Revisa todos los cambios pendientes y los separa en commits atómicos
argument-hint: "[contexto opcional sobre los cambios]"
allowed-tools: Bash(git status:*), Bash(git diff:*), Bash(git log:*), Bash(git add:*), Bash(git reset:*), Bash(git commit:*), Bash(git apply:*), Bash(git stash list:*), Read, Grep, Glob
---

Tu tarea: revisar **todo** lo que hay pendiente en el working tree, detectar que puede haber varias unidades de trabajo mezcladas (un feature que quedó sin commitear + otro feature + un bugfix + docs…) y separarlas en **commits atómicos, uno por cosa**. Nunca dejes que todo se vaya en un solo commit.

Contexto opcional del usuario sobre los cambios: $ARGUMENTS

## 1. Recolectar estado

En un solo bloque, en paralelo:

- `git status --porcelain=v1 -uall` — estado completo, untracked archivo por archivo
- `git diff` — cambios sin stage
- `git diff --staged` — cambios ya en stage
- `git log --oneline -15` — para imitar el estilo de mensajes que ya existe en el repo

Luego lee el contenido completo de los archivos untracked relevantes y revisa los diffs a fondo para entender **qué se hizo** en cada archivo.

Si no hay nada pendiente → dilo y termina.

## 2. Analizar y agrupar

- Una unidad lógica = un commit. Agrupa por **intención del cambio**, no por ubicación del archivo.
- Separa: feature nuevo · bugfix · refactor · estilos/formato · docs · config/deps · tests.
- Nunca mezcles un fix con un feature aunque toquen el mismo archivo.
- Un cambio de soporte (un helper nuevo que solo usa el feature X) va **con** ese feature.
- `package.json` + `package-lock.json` viajan juntos.
- Ordena los commits por dependencia: lo que otro commit necesita va primero, para que cada commit deje el árbol coherente.
- Ignora ruido no versionable (`.next/`, artefactos ya en `.gitignore`). Si aparece algo que debería estar ignorado (p. ej. `.qodo/`), avísalo en vez de commitearlo.

## 3. Presentar el plan y ESPERAR confirmación

Muestra el plan compacto:

```
1. feat: add snake game board
   app/games/snake/page.tsx, app/games/snake/board.tsx
2. fix: correct point calculation on tie
   lib/scores.ts
3. docs: update README with setup steps
   README.md
```

**Detente aquí.** No ejecutes ningún `git commit` hasta que el usuario apruebe. Puede pedir fusionar, dividir o reordenar grupos.

## 4. Ejecutar, commit por commit

- Antes de empezar: `git reset` para limpiar el staging y controlar exactamente qué entra en cada commit.
- Por grupo: `git add -- <rutas exactas>` → `git commit -m "<mensaje>"`.
- Antes de cada commit, verifica con `git status --porcelain` que quedó staged **solo** lo esperado.

### Archivo con cambios de dos temas distintos

`git add -p` es interactivo y **no funciona** en este entorno. En su lugar:

1. `git diff -- <archivo> > "$SCRATCHPAD/split.patch"` (usa tu directorio scratchpad)
2. Edita el patch dejando solo los hunks del grupo actual (ajusta los contadores `@@`)
3. `git apply --cached "$SCRATCHPAD/split.patch"`
4. Commitea; el resto del archivo queda unstaged para el siguiente commit

Si el split por hunks resulta inviable, díselo al usuario y ofrece commitear el archivo entero en el grupo dominante.

## 5. Reglas de mensaje

- `type: description` — imperativo, minúscula, sin punto final, ≤72 chars.
- Todo en **inglés**.
- Tipos: `feat` `fix` `refactor` `style` `docs` `test` `chore` `perf` `build` `ci`.
- **Sin scope** — nunca uses paréntesis. Siempre `feat: ...`, nunca `feat(x): ...`.
- Solo `-m` con el subject. Cuerpo únicamente si el cambio no se explica solo.
- **Sin** `Co-Authored-By`, **sin** `Claude-Session`, sin emojis. Mensajes limpios.

## 6. Cierre

- Muestra `git log --oneline -<n>` con los commits creados.
- **No hagas push** ni lo sugieras.
- Si `git status` sigue mostrando cosas, di explícitamente qué quedó fuera y por qué.

## 7. Guardas

- Estar en `main` no bloquea (el usuario trabaja ahí). No crees ramas por iniciativa propia.
- Nunca `git add -A` / `git add .` — siempre rutas explícitas.
- Nunca `git commit --amend`, `git reset --hard`, ni `git checkout --` sobre cambios del usuario.
- Si un commit falla (p. ej. un hook), párate, muestra el error y no sigas con los demás.
