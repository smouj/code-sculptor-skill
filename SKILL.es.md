name: Code Sculptor
slug: code-sculptor
version: 1.2.0
description: Motor automatizado de refactorización y optimización de código para mantenibilidad y rendimiento
author: SMOUJBOT
tags: [refactoring, optimization, code-quality, automated, maintainability, performance]
introduced_in: 2025.3
dependencies:
  - python>=3.9
  - radon>=5.0
  - libcst>=1.0
  - black>=22.0
  - isort>=5.10
  - rope>=1.0
  - autoflake>=2.0
  - bandit>=1.7
  - mypy>=0.960
  - pytest>=7.0
platforms: [linux, darwin]
conflicts: [manual-edit-conflict, performance-regression]
requires_network: false
resource_requirements:
  cpu: "1"
  memory_mb: 512
  disk_mb: 100
---

# Code Sculptor

Motor automatizado de refactorización y optimización de código que transforma codebases para mejorar su mantenibilidad, rendimiento y consistencia. Opera de forma destructiva en las rutas especificadas (se recomienda usar código ya confirmado en repositorio).

## Propósito

Casos de uso reales:
- **Reducción de deuda técnica**: Extraer automáticamente funciones de lógica profundamente anidada (más de 3 niveles), convertir condicionales complejos en guard clauses, y dividir god classes (>500 líneas)
- **Reforzamiento de consistencia**: Aplicar estilo de código unificado después de divergencias con linter, estandarizar patrones de manejo de errores entre microservicios
- **Optimización de rendimiento**: Reemplazar algoritmos O(n²) por O(n log n) cuando sea detectable, eliminar cálculos redundantes en bucles ajustados, convertir E/S síncrona a asíncrona en codebases Python asyncio
- **Endurecimiento de seguridad**: Auto-escapar consultas SQL usando parametrización, reemplazar `pickle` con `json`, inyectar validación de entrada
- **Asistencia en migración**: Convertir sentencias print de Python 2 a 3, actualizar APIs de bibliotecas obsoletas (ej. `asyncio.get_event_loop()` → `asyncio.new_event_loop()`)
- **Eliminación de código muerto**: Remover imports no usados, funciones con cero llamadas, bloques de código legado comentados

## Alcance

### Comandos

#### `refactor`
Aplica operaciones automatizadas de refactorización.

**Uso:** `code-sculptor refactor [RUTA] [FLAGS]`

**Flags:**
- `--style=black|yapf|none` (predeterminado: black) - Formateador de código
- `--imports=isort|auto|skip` (predeterminado: isort) - Organización de imports
- `--deadcode` - Remover código no usado (requiere `--commit`)
- `--complexity-threshold=N` - Extraer funciones que superen complejidad ciclomática (predeterminado: 10)
- `--max-lines=N` - Dividir funciones más largas que N líneas (predeterminado: 50)
- `--security=bandit|strict` - Aplicar correcciones de seguridad
- `--asyncify` - Convertir patrones de E/S síncronos a asíncronos (solo Python)
- `--dry-run` - Mostrar cambios sin aplicarlos
- `--commit` - Crear commit de git después del refactor exitoso
- `--backup-dir=DIR` - Respaldar archivos originales antes de modificar (predeterminado: `.code-sculptor-backup`)

**Ejemplos:**
```bash
code-sculptor refactor ./src --complexity-threshold=8 --max-lines=40 --deadcode --commit
code-sculptor refactor ./app --style=none --security=strict --dry-run
```

#### `optimize`
Aplica transformaciones enfocadas en rendimiento.

**Uso:** `code-sculptor optimize [RUTA] [FLAGS]`

**Flags:**
- `--loops` - Optimizar patrones de bucles (caché de cálculos elevados, sugerencias de vectorización)
- `--data-structures` - Reemplazar estructuras ineficientes (list→set para pruebas de membresía, cadenas dict.get())
- `--caching` - Inyectar `@lru_cache` en funciones puras, memorizar llamadas repetidas
- `--algorithm=N` - Aplicar mejoras algorítmicas (1=suave, 2=moderada, 3=agresiva)
- `--benchmark=ARCHIVO` - Comparar rendimiento antes/después usando pytest-benchmark
- `--threshold=%` - Velocidad mínima requerida para aplicar cambio (predeterminado: 10%)
- `--ignore-tests` - No modificar archivos de test

**Ejemplos:**
```bash
code-sculptor optimize ./core --loops --data-structures --caching --algorithm=2
code-sculptor optimize ./pkg --benchmark=bench_perf.py --threshold=15
```

#### `audit`
Analiza calidad de código y sugiere refactorizaciones sin aplicarlas.

**Uso:** `code-sculptor audit [RUTA] [FLAGS]`

**Flags:**
- `--report=ARCHIVO` - Salida de reporte JSON
- `--format=html|json|terminal` (predeterminado: terminal)
- `--metrics=complexity,duplication,maintainability` (predeterminado: todos)
- `--severity=low|medium|high` (predeterminado: medium)

**Ejemplos:**
```bash
code-sculptor audit ./src --format=html --report=audit.html
code-sculptor audit ./ --severity=high --metrics=complexity
```

#### `rollback`
Restaura archivos desde el respaldo creado por refactor anterior.

**Uso:** `code-sculptor rollback [DIR_RESPALDO]`

**Ejemplos:**
```bash
code-sculptor rollback .code-sculptor-backup-20250302_143022
code-sculptor rollback --latest
```

## Proceso de Trabajo

1. **Descubrimiento**: Escanea RUTA objetivo para tipos de archivo soportados (.py, .js, .ts, .go según detección de lenguaje)
2. **Análisis**: Usa libcst/ts-morph para construir árboles de sintaxis concreta, calcula complejidad ciclomática, identifica code smells
3. **Transformación**:
   - Aplica eliminación de código muerto con `autoflake` (remover imports/variables no usados)
   - Reformatea con `black` (longitud de línea 88, normalización de strings)
   - Reordena imports con `isort` (terceros primero, luego propios)
   - Divide funciones grandes usando API de refactorización de rope
   - Inserta caché vía transformación AST (detectar funciones puras)
4. **Seguridad**: Ejecuta escaneo `bandit`, aplica correcciones automáticas para:
   - `B105: contraseña hardcodeada` → raise ValueError
   - `B110: try/except/pass` → añadir logging
   - `B201: flask_debug_true` → establecer en False
5. **Conversión asíncrona** (solo Python):
   - Reemplazar `time.sleep()` con `asyncio.sleep()`
   - Convertir `requests.get()` a `aiohttp` si está dentro de `async def`
   - Envolver funciones síncronas llamadas desde contexto asíncrono con `asyncio.to_thread()`
6. **Commit**: Si `--commit`, crear commit de git con mensaje: `refactor(CodeSculptor): mejoras automatizadas`
7. **Reporte**: Imprimir resumen de archivos cambiados, líneas añadidas/eliminadas, reducción de complejidad

## Reglas de Oro

1. **Nunca ejecutar en código no confirmado** - requerir árbol de trabajo limpio de git a menos que se pase `--no-commit-check` explícitamente
2. **Preservar comportamiento** - las transformaciones deben ser de preservación semántica; si hay duda, omitir transformación
3. **Integración de tests obligatoria** - después del refactor, ejecutar automáticamente `pytest` (o comando de test especificado); abortar en fallos
4. **Sin conflictos de merge** - stash cambios no confirmados antes de la operación, restaurar después
5. **Respaldo por defecto** - crear directorio de respaldo con marca de tiempo; no eliminar hasta 30 días
6. **Respetar type hints** - si `mypy` falla después de la transformación, revertir ese archivo
7. **Soporte de archivo de configuración** - leer `.codesculptor.yaml` para reglas específicas del proyecto (patrones de exclusión, umbrales de complejidad personalizados)
8. **Guarda contra regresión de rendimiento** - si se proporciona `--benchmark` y algún test se ralentiza > umbral, abortar y rollback

## Ejemplos

### Ejemplo 1: Extraer Función Compleja
**Entrada del usuario:**
```bash
code-sculptor refactor ./services/user.py --complexity-threshold=12 --max-lines=60 --commit
```

**Qué sucede:**
- Detecta función `process_user_data` con complejidad 18, 120 líneas
- Divide en: `validate_input`, `transform_data`, `persist_changes`, `send_notification`
- Crea PR si está en modo CI, de lo contrario commitea directamente
- Salida:
```
[CodeSculptor] Refactored ./services/user.py
  - Extracted 4 functions from process_user_data (complexity: 18 → 4,3,5,4)
  - Reformatted with black (88 chars)
  - Removed 3 unused imports
  - Created git commit: refactor(CodeSculptor): automated improvements
```

### Ejemplo 2: Endurecimiento de Seguridad
**Entrada del usuario:**
```bash
code-sculptor refactor ./legacy --security=strict --style=none --commit
```

**Qué sucede:**
- Bandit encuentra concatenación SQL en `db_query(sql = "SELECT * FROM users WHERE id = " + user_id)`
- Transforma a consulta parametrizada con placeholders
- Reemplaza `pickle.loads()` con `json.loads()` donde los datos sean compatibles con JSON
- Salida:
```
[CodeSculptor] Security fixes applied:
  ./legacy/db.py:42 → parameterized query
  ./legacy/utils.py:15 → pickle → json
```

### Ejemplo 3: Optimización de Rendimiento
**Entrada del usuario:**
```bash
code-sculptor optimize ./api --loops --caching --algorithm=2 --benchmark=benchmarks/api_bench.py
```

**Qué sucede:**
- Encuentra bucle `for item in items: results.append(process(item))` → convierte a list comprehension
- Detecta función pura `calculate_metric(data)` sin efectos secundarios → añade `@lru_cache(maxsize=128)`
- Ejecuta benchmarks antes/después; verifica ≥10% de mejora; si no, revierte cambios
- Salida:
```
[CodeSculptor] Optimizations applied (avg speedup: 23%):
  ./api/endpoints.py:67 → list comprehension (1.8x faster)
  ./api/metrics.py:22 → @lru_cache (3.2x faster)
```

## Verificación

Después de operación exitosa:
1. Ejecutar tests: `pytest tests/` (o comando personalizado desde configuración del proyecto)
2. Type check: `mypy src/` (proyectos Python)
3. Lint: `flake8 src/` o `eslint src/` (JS/TS)
4. Para rendimiento: `pytest --benchmark-only`

Verificar:
- Todos los tests pasan
- No hay nuevas advertencias de lint
- Reporte de complejidad muestra reducción
- Benchmark muestra mejora (si se benchmarkeó)

## Rollback

### Rollback Automático
Si algún paso de verificación falla, Code Sculptor automáticamente:
1. Restaura archivos desde directorio de respaldo
2. Elimina cambios incompletos
3. Sale con código de error 1

### Rollback Manual
```bash
# Listar respaldos disponibles
ls -1t .code-sculptor-backup-*

# Restaurar último respaldo
code-sculptor rollback --latest

# Restaurar respaldo específico
code-sculptor rollback .code-sculptor-backup-20250302_143022
```

**Proceso de rollback:**
- Copia todos los archivos desde respaldo a ubicaciones originales
- Verifica que el checksum coincida con el respaldo
- Elimina directorio de respaldo después de restauración exitosa (opcional `--keep`)

## Dependencias y Requisitos

**Sistema:**
- Python 3.9+ para transformaciones Python
- Node.js 16+ para transformaciones JS/TS (vía ts-morph)
- Git instalado y en PATH

**Variables de entorno:**
- `CODESCULPTOR_MAX_FILE_SIZE` - Omitir archivos mayores a N bytes (predeterminado: 1M)
- `CODESCULPTOR_EXCLUDE` - Patrones glob para excluir (predeterminado: `["__pycache__", "node_modules", "*.min.js", "dist"]`)
- `CODESCULPTOR_GIT_AUTHOR` - Sobrescribir autor de commit git (predeterminado: inferido de git config)
- `CODESCULPTOR_DRY_RUN` - Si se establece en `true`, nunca hacer cambios (para testing)

**Archivo de configuración:**
`.codesculptor.yaml` (opcional):
```yaml
exclude:
  - "**/migrations/**"
  - "**/test_*.py"
complexity_threshold: 8
max_lines: 40
commit_message_prefix: "[auto-refactor]"
security_level: medium
backup_retention_days: 30
```

## Solución de Problemas

**Problema:** `Error: Cambios no confirmados detectados. Abortando.`
- **Solución:** Confirmar o hacer stash de cambios: `git stash push -m "code-sculptor backup"` antes de ejecutar. Usar `--no-commit-check` para omitir (no recomendado).

**Problema:** `Transformation failed: mypy errors after formatting`
- **Solución:** Code Sculptor no preservó type hints. Ejecutar con `--style=none` o corregir errores de tipo manualmente. Verificar si faltan stubs de bibliotecas.

**Problema:** `Performance regression detected (benchmark -5%)`
- **Solución:** Optimización revertida automáticamente. Probar nivel `--algorithm` menor o excluir funciones problemáticas via `.codesculptor.yaml` `skip_transformations`.

**Problema:** `Backup directory not found`
- **Solución:** Operación anterior pudo haber limpiado. Usar `git reflog` para recuperar. Siempre pushear antes de ejecutar en producción.

**Problema:** `Unsupported language: Go`
- **Solución:** Code Sculptor actualmente soporta Python, JavaScript, TypeScript. Para Go, usar solo modo audit o esperar futura versión.

**Problema:** `bandit failed to install`
- **Solución:** Instalar dependencias de seguridad: `pip install bandit`. O ejecutar con `--security=skip` para deshabilitar.
```