# 🚀 Ruff de 0 a Pro en Cursor

> **Objetivo** — Esta guía paso a paso te lleva desde instalar Python + Ruff en **Cursor** hasta aplicar las mejores prácticas de adopción en equipo (Módulo 8). Incluye comandos listos para copiar, fragmentos de configuración y explicaciones para que cualquiera pueda reproducir el proceso.

---

## 📑 Tabla de contenidos

1. [Prerrequisitos](#prerrequisitos)
2. [Paso 0 · Entorno base](#paso-0--entorno-base)
3. [Instalación de Ruff](#instalación-de-ruff)
4. [Integración con Cursor](#integración-con-cursor)
5. [Módulo 2 · Comandos básicos](#módulo-2--comandos-básicos)
6. [Módulo 3 · Uso diario en proyectos](#módulo-3--uso-diario-en-proyectos)
7. [Módulo 4 · Ruff como herramienta todo‑en‑uno](#módulo-4--ruff-como-herramienta-todo‑en‑uno)
8. [Módulo 5 · Rendimiento y caché](#módulo-5--rendimiento-y-caché)
9. [Módulo 6 · Flujos modernos (uv, CI/CD…)](#módulo-6--flujos-modernos-uv-cicd…)
10. [Módulo 7 · Casos avanzados y escala](#módulo-7--casos-avanzados-y-escala)
11. [Módulo 8 · Buenas prácticas y adopción](#módulo-8--buenas-prácticas-y-adopción)
12. [Recursos extra](#recursos-extra)

---

## Prerrequisitos <a name="prerrequisitos"></a>

| Herramienta      | Versión sugerida | ¿Para qué sirve?                                                  |
| ---------------- | ---------------- | ----------------------------------------------------------------- |
| Python           | ≥ 3.11           | Ejecución de tu código y de algunos wrappers de Ruff.             |
| **uv**           | ≥ 0.2            | Gestor de paquetes ultra‑rápido en Rust (reemplaza `pip`/`pipx`). |
| **Cursor**       | ≥ 0.40           | Editor derivado de VS Code, con IA integrada.                     |
| Git              | Cualquiera       | Control de versiones y hooks `pre‑commit`.                        |
| Cargo (opcional) | nightly          | Compilar Ruff desde fuente, sólo si quieres hackearlo.            |

> 💡 **Tip** — Si ya tienes Python instalado por tu sistema operativo, instala una versión independiente con pyenv o asdf para evitar permisos de root.

---

## Paso 0 · Entorno base <a name="paso-0--entorno-base"></a>

1. **Instala uv con Homebrew (macOS/Linux) o Scoop (Windows):**

   ```bash
   brew install astral-sh/astral/uv   # macOS/Linux
   scoop bucket add astral https://github.com/astral-sh/scoop-bucket.git
   scoop install uv                   # Windows
   ```
2. **Crea un directorio de proyecto de prueba y entra en él:**

   ```bash
   mkdir hello-ruff && cd hello-ruff
   ```
3. **Inicializa un entorno virtual “invisible” con uv:**

   ```bash
   uv venv
   uv pip install --upgrade pip setuptools
   ```

   > uv guarda el venv en `~/.cache/uv/venvs` y luego lo detecta según `pyproject.toml`; no ensucia tu directorio.

---

## Instalación de Ruff <a name="instalación-de-ruff"></a>

### Opción A · **uv tool install** (recomendada)

Instala Ruff global y crea un shim en tu `$PATH`:

```bash
uv tool install ruff
ruff --version   # → 0.x.x
```

### Opción B · **pipx** (aislado, global)

```bash
pipx install ruff
```

### Opción C · **pip en venv local** (rápido, sólo para el proyecto)

```bash
uv pip install --dev ruff
```

> ⚙️ **Config ruta global**: Linux/macOS → `~/.config/ruff/ruff.toml`; Windows → `%APPDATA%\ruff\ruff.toml`.

---

## Integración con Cursor <a name="integración-con-cursor"></a>

### 1 · Instalar la extensión

1. Abre `⌘⇧X` (Extensiones).
2. Busca **“Ruff”** → *charliermarsh.ruff* → **Install**.

### 2 · Ajustes recomendados (`settings.json`)

```jsonc
{
  // Formateador y linting principales
  "editor.defaultFormatter": "charliermarsh.ruff",
  "editor.formatOnSave": true,
  "editor.codeActionsOnSave": {
    "source.organizeImports": true
  },
  // Desactiva otros linters para evitar duplicados
  "python.linting.enabled": false,
  "python.formatting.provider": "none",
  // Ruff LSP (opcional para diagnósticos en tiempo real)
  "ruff.lsp.enabled": true
}
```

> 🪄 **FixAll** — Si quieres que Ruff arregle todo al guardar, añade
>
> ```jsonc
> "editor.codeActionsOnSave": {
>   "source.fixAll": true,
>   "source.organizeImports": true
> }
> ```

### 3 · Tareas integradas en Cursor (`.vscode/tasks.json`)

```jsonc
{
  "version": "2.0.0",
  "tasks": [
    {
      "label": "Lint & Format (Ruff)",
      "type": "shell",
      "command": "uv run ruff check . --fix && uv run ruff format .",
      "problemMatcher": []
    }
  ]
}
```

Ejecuta con `⇧⌘P → Run Task → Lint & Format (Ruff)`.

---

## Módulo 2 · Comandos básicos <a name="módulo-2--comandos-básicos"></a>

| Acción        | Comando                | ¿Qué hace?                                      |
| ------------- | ---------------------- | ----------------------------------------------- |
| Lint proyecto | `ruff check .`         | Escanea todos los `.py` y muestra diagnósticos. |
| Autocorregir  | `ruff check . --fix`   | Aplica arreglos seguros in‑place.               |
| Simular fix   | `ruff check . --diff`  | Muestra diff sin modificar archivos.            |
| Formatear     | `ruff format .`        | Reemplaza Black, mantiene estilo coherente.     |
| Feedback vivo | `ruff check . --watch` | Rerun en cada guardado; ideal para TDD.         |

> 🧪 **Prueba rápida** — Crea `ugly.py` con `import os, sys` y ejecuta `ruff check ugly.py --fix` para ver la magia.

---

## Módulo 3 · Uso diario en proyectos <a name="módulo-3--uso-diario-en-proyectos"></a>

### 3.1 · Ciclo local típico

```bash
uv run ruff check . --fix   # limpia estilo y errores
uv run ruff format .        # formatea a lo Black
pytest -q                   # corre tests
```

### 3.2 · Configuración mínima (`pyproject.toml`)

```toml
[tool.ruff]
line-length = 100          # ancho de línea
fix = true                 # allow --fix por defecto
select = ["E", "F", "I"]  # errores, fallos, imports ordenados
```

### 3.3 · Editor + LSP

Arranca servidor:

```bash
ruff server --preview
```

Cursor detecta y muestra errores en tiempo real.

### 3.4 · Hooks pre‑commit

`.pre-commit-config.yaml`:

```yaml
repos:
  - repo: https://github.com/astral-sh/ruff-pre-commit
    rev: v0.4.6
    hooks:
      - id: ruff
```

```bash
pre-commit install
```

Sólo analiza archivos modificados en cada `git commit`.

---

## Módulo 4 · Ruff como herramienta todo‑en‑uno <a name="módulo-4--ruff-como-herramienta-todo‑en‑uno"></a>

### 4.1 · Sustituciones

| Herramienta tradicional | Grupo de reglas Ruff     |
| ----------------------- | ------------------------ |
| flake8                  | `E`, `F`, `W`            |
| isort                   | `I`                      |
| black                   | `ruff format`            |
| pylint (parcial)        | `B`, `C4`, `PLE`, `PLR`… |
| pyupgrade               | `UP`                     |

### 4.2 · Ejemplo integral de configuración

```toml
[tool.ruff]
line-length = 100
fix = true
exclude = ["migrations/*", "docs/_build/*"]

# Lint groups
select = [
  "E",  # estilo
  "F",  # fallos
  "I",  # imports ordenados
  "B",  # bugbear
  "C90",# complejidad ciclomática
  "D",  # docstrings (pydocstyle)
  "UP", # pyupgrade
  "PTH",# pathlib > open/os.path
  "S",  # seguridad (bandit)
]

[tool.ruff.per-file-ignores]
"tests/*" = ["D", "S101"]  # Relaja docstrings y assertions en tests
```

---

## Módulo 5 · Rendimiento y caché <a name="módulo-5--rendimiento-y-caché"></a>

### 5.1 · Cómo funciona

* **Rust + rayon** → análisis multihilo.
* **Hash de archivo + mtime** guardado en `~/.cache/ruff` → evita re‑analizar sin cambios.

### 5.2 · Fuerza recomputo total

```bash
ruff check . --no-cache
```

### 5.3 · Benchmarks personales

| Proyecto | Flake8 + Black | Ruff check + format | Aceleración |
| -------- | -------------- | ------------------- | ----------- |
| 2 k LOC  | 2.8 s          | 0.15 s              | ×18         |
| 50 k LOC | 55 s           | 2.4 s               | ×23         |

> ⚡ **Conclusion** — Ruff cabe en tu guardado automático sin notar lag.

---

## Módulo 6 · Flujos modernos (uv, CI/CD…) <a name="módulo-6--flujos-modernos-uv-cicd…"></a>

### 6.1 · uv y scripts

Añade a `pyproject.toml`:

```toml
[tool.uv.scripts]
lint = "ruff check . --fix"
format = "ruff format ."
```

Ahora ejecuta `uv lint` o `uv format` sin recordar flags.

### 6.2 · GitHub Actions

`.github/workflows/lint.yml`:

```yaml
name: Ruff
on: [push, pull_request]
jobs:
  lint:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: astral-sh/setup-ruff@v1  # binario pre‑compilado
      - run: ruff check . --output-format=github
```

### 6.3 · GitLab CI

```yaml
ruff:
  image: python:3.11-slim
  script:
    - pip install ruff
    - ruff check .
```

### 6.4 · VS Code/ Cursor tasks y Live Share

a) Task que falla el build si Ruff marca algo.
b) Live Share plus pre‑commit => todos ven las mismas reglas.

---

## Módulo 7 · Casos avanzados y escala <a name="módulo-7--casos-avanzados-y-escala"></a>

### 7.1 · Monorepos

Estructura:

```
repo/
  pyproject.toml   # raíz con select e ignores globales
  apps/api/pyproject.toml      # reglas extra API
  libs/utils/pyproject.toml    # reglas laxas
```

Ruff busca hacia arriba hasta encontrar la primera config => heredable.

### 7.2 · Reglas por carpeta

```toml
[tool.ruff.per-file-ignores]
"scripts/*" = ["F401", "S101"]
"migrations/*" = ["D", "E501"]
```

### 7.3 · Contribuir a Ruff

* Fork → `cargo build --release` → `target/release/ruff`.
* Añade regla en `crates/ruff_linter/src/rules/`.
* Tests: `cargo test -p ruff_linter`.

---

## Módulo 8 · Buenas prácticas y adopción <a name="módulo-8--buenas-prácticas-y-adopción"></a>

> “**Ruff no es solo un linter, es tu firewall de calidad**.”

### 8.1 · Estrategia de incorporación gradual

1. Activa sólo `E`, `F`, `I`.
2. Añade `UP`, `B`, `D` tras una semana.
3. Introduce complejidad `C90` y seguridad `S` cuando el equipo ya esté cómodo.

### 8.2 · Centraliza configuración

* **Global** (`~/.config/ruff/ruff.toml`) → reglas personales.
* **Proyecto** (`pyproject.toml`) → consenso de equipo.
* Documenta EXPLICACIONES de cada ignore para evitar “ruido por costumbre”.

### 8.3 · Automatiza todo

* Hook `pre‑commit` obligatorio.
* Step de CI que bloquee merge si Ruff falla.
* Alias de make/uv (`make lint`, `make format`).

### 8.4 · Mantenimiento

* Revisa CHANGELOG de Ruff cada 2 semanas.
* Añade reglas nuevas en rama aparte, corrige código con `--fix`, y mergea.

### 8.5 · Formación del equipo

* Demo de 30 min mostrando `--diff` antes/después.
* Cheat‑sheet interno con los errores comunes (E501, F401, etc.).
* Pareja Ruff + mypy: lint + type‑check ≠ solapados.

---

## Recursos extra <a name="recursos-extra"></a>

* Docs oficiales: [https://docs.astral.sh/ruff/](https://docs.astral.sh/ruff/)
* Lista completa de reglas: [https://docs.astral.sh/ruff/rules/](https://docs.astral.sh/ruff/rules/)
* Vídeo Corey Schafer: “*Super‑fast Python Linter*”
* Plantilla `ruff.toml` de CoreyMSchafer: [https://github.com/CoreyMSchafer/dotfiles/blob/master/settings/ruff.toml](https://github.com/CoreyMSchafer/dotfiles/blob/master/settings/ruff.toml)
* Blog Astral: benchmarks y comparativas.

---

## 🏁 ¡A lintear se ha dicho!

Con esta plantilla:

1. Clona repo,
2. `uv tool install ruff`,
3. Copia las secciones de configuración,
4. Pulsa **Guardar** en Cursor y disfruta del código limpio.

