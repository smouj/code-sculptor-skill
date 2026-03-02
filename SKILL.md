---
name: Code Sculptor
slug: code-sculptor
version: 1.2.0
description: Automated code refactoring and optimization engine for maintainability and performance
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

Automated code refactoring and optimization engine that transforms codebases for maintainability, performance, and consistency. Operates destructively on specified paths (committed code recommended).

## Purpose

Real-world use cases:
- **Technical debt reduction**: Automatically extract functions from deeply nested logic (more than 3 levels), convert complex conditionals into guard clauses, and split god classes (>500 lines)
- **Consistency enforcement**: Enforce team-wide code style after linter divergence, standardize error handling patterns across microservices
- **Performance optimization**: Replace O(n²) algorithms with O(n log n) where detectable, eliminate redundant calculations in tight loops, convert synchronous I/O to async in Python asyncio codebases
- **Security hardening**: Auto-escape SQL queries using parameterization, replace `pickle` with `json`, inject input validation
- **Migration assistance**: Convert Python 2 to 3 print statements, update deprecated library APIs (e.g., `asyncio.get_event_loop()` → `asyncio.new_event_loop()`)
- **Dead code elimination**: Remove unused imports, functions with zero call sites, commented-out legacy code blocks

## Scope

### Commands

#### `refactor`
Apply automated refactoring operations.

**Usage:** `code-sculptor refactor [PATH] [FLAGS]`

**Flags:**
- `--style=black|yapf|none` (default: black) - Code formatter
- `--imports=isort|auto|skip` (default: isort) - Import organization
- `--deadcode` - Remove unused code (requires `--commit`)
- `--complexity-threshold=N` - Extract functions exceeding cyclomatic complexity (default: 10)
- `--max-lines=N` - Split functions longer than N lines (default: 50)
- `--security=bandit|strict` - Apply security fixes
- `--asyncify` - Convert sync I/O patterns to async (Python only)
- `--dry-run` - Show changes without applying
- `--commit` - Create git commit after successful refactor
- `--backup-dir=DIR` - Backup original files before modification (default: `.code-sculptor-backup`)

**Examples:**
```bash
code-sculptor refactor ./src --complexity-threshold=8 --max-lines=40 --deadcode --commit
code-sculptor refactor ./app --style=none --security=strict --dry-run
```

#### `optimize`
Apply performance-focused transformations.

**Usage:** `code-sculptor optimize [PATH] [FLAGS]`

**Flags:**
- `--loops` - Optimize loop patterns (caching hoisted calculations, vectorization hints)
- `--data-structures` - Replace inefficient structures (list→set for membership tests, dict.get() chains)
- `--caching` - Inject `@lru_cache` on pure functions, memoize repeated calls
- `--algorithm=N` - Apply algorithmic improvements (1=mild, 2=moderate, 3=aggressive)
- `--benchmark=FILE` - Compare performance before/after using pytest-benchmark
- `--threshold=%` - Minimum speedup required to apply change (default: 10%)
- `--ignore-tests` - Don't modify test files

**Examples:**
```bash
code-sculptor optimize ./core --loops --data-structures --caching --algorithm=2
code-sculptor optimize ./pkg --benchmark=bench_perf.py --threshold=15
```

#### `audit`
Analyze code quality and suggest refactorings without applying.

**Usage:** `code-sculptor audit [PATH] [FLAGS]`

**Flags:**
- `--report=FILE` - Output JSON report
- `--format=html|json|terminal` (default: terminal)
- `--metrics=complexity,duplication,maintainability` (default: all)
- `--severity=low|medium|high` (default: medium)

**Examples:**
```bash
code-sculptor audit ./src --format=html --report=audit.html
code-sculptor audit ./ --severity=high --metrics=complexity
```

#### `rollback`
Restore files from backup created by previous refactor.

**Usage:** `code-sculptor rollback [BACKUP_DIR]`

**Examples:**
```bash
code-sculptor rollback .code-sculptor-backup-20250302_143022
code-sculptor rollback --latest
```

## Work Process

1. **Discovery**: Scans target PATH for supported file types (.py, .js, .ts, .go based on language detection)
2. **Analysis**: Uses libcst/ts-morph to build concrete syntax trees, calculates cyclomatic complexity, identifies code smells
3. **Transformation**:
   - Apply dead code elimination with `autoflake` (remove unused imports/variables)
   - Reformat with `black` (line length 88, string normalization)
   - Reorder imports with `isort` (third-party first, then first-party)
   - Split large functions using rope's refactoring API
   - Insert caching via AST transformation (detect pure functions)
4. **Security**: Run `bandit` scan, auto-apply fixes for:
   - `B105: hardcoded password` → raise ValueError
   - `B110: try/except/pass` → add logging
   - `B201: flask_debug_true` → set to False
5. **Async conversion** (Python only):
   - Replace `time.sleep()` with `asyncio.sleep()`
   - Convert `requests.get()` to `aiohttp` if inside `async def`
   - Wrap sync functions called from async context with `asyncio.to_thread()`
6. **Commit**: If `--commit`, create git commit with message: `refactor(CodeSculptor): automated improvements`
7. **Report**: Print summary of changed files, lines added/removed, complexity reduction

## Golden Rules

1. **Never run on uncommitted code** - require clean git working tree unless `--no-commit-check` explicitly passed
2. **Preserve behavior** - transformations must be semantics-preserving; if unsure, skip transformation
3. **Test integration mandatory** - after refactor, automatically run `pytest` (or specified test command); abort on failures
4. **No merging conflicts** - stash uncommitted changes before operation, restore after
5. **Backup by default** - create timestamped backup directory; don't delete until 30 days
6. **Respect type hints** - if `mypy` fails after transformation, revert that file
7. **Configuration file support** - read `.codesculptor.yaml` for project-specific rules (exclude patterns, custom complexity thresholds)
8. **Performance regression guard** - if `--benchmark` provided and any test slows down > threshold, abort and rollback

## Examples

### Example 1: Extract Complex Function
**User input:**
```bash
code-sculptor refactor ./services/user.py --complexity-threshold=12 --max-lines=60 --commit
```

**What happens:**
- Detects function `process_user_data` with complexity 18, 120 lines
- Splits into: `validate_input`, `transform_data`, `persist_changes`, `send_notification`
- Creates PR if in CI mode, otherwise commits directly
- Output:
```
[CodeSculptor] Refactored ./services/user.py
  - Extracted 4 functions from process_user_data (complexity: 18 → 4,3,5,4)
  - Reformatted with black (88 chars)
  - Removed 3 unused imports
  - Created git commit: refactor(CodeSculptor): automated improvements
```

### Example 2: Security Hardening
**User input:**
```bash
code-sculptor refactor ./legacy --security=strict --style=none --commit
```

**What happens:**
- Bandit finds SQL concatenation in `db_query(sql = "SELECT * FROM users WHERE id = " + user_id)`
- Transforms to parameterized query with placeholders
- Replaces `pickle.loads()` with `json.loads()` where data is JSON-compatible
- Output:
```
[CodeSculptor] Security fixes applied:
  ./legacy/db.py:42 → parameterized query
  ./legacy/utils.py:15 → pickle → json
```

### Example 3: Performance Optimization
**User input:**
```bash
code-sculptor optimize ./api --loops --caching --algorithm=2 --benchmark=benchmarks/api_bench.py
```

**What happens:**
- Finds loop `for item in items: results.append(process(item))` → converts to list comprehension
- Detects pure function `calculate_metric(data)` with no side effects → adds `@lru_cache(maxsize=128)`
- Runs benchmarks before/after; verifies ≥10% improvement; if not, rollbacks changes
- Output:
```
[CodeSculptor] Optimizations applied (avg speedup: 23%):
  ./api/endpoints.py:67 → list comprehension (1.8x faster)
  ./api/metrics.py:22 → @lru_cache (3.2x faster)
```

## Verification

After successful operation:
1. Run tests: `pytest tests/` (or custom command from project config)
2. Type check: `mypy src/` (Python projects)
3. Lint: `flake8 src/` or `eslint src/` (JS/TS)
4. For performance: `pytest --benchmark-only`

Check:
- All tests pass
- No new lint warnings
- Complexity report shows reduction
- Benchmark shows improvement (if benchmarked)

## Rollback

### Automatic Rollback
If any verification step fails, Code Sculptor automatically:
1. Restores files from backup directory
2. Deletes incomplete changes
3. Exits with error code 1

### Manual Rollback
```bash
# List available backups
ls -1t .code-sculptor-backup-*

# Restore latest backup
code-sculptor rollback --latest

# Restore specific backup
code-sculptor rollback .code-sculptor-backup-20250302_143022
```

**Rollback process:**
- Copies all files from backup to original locations
- Verifies checksum matches backup
- Removes backup directory after successful restore (optional `--keep`)

## Dependencies & Requirements

**System:**
- Python 3.9+ for Python transformations
- Node.js 16+ for JS/TS transformations (via ts-morph)
- Git installed and in PATH

**Environment variables:**
- `CODESCULPTOR_MAX_FILE_SIZE` - Skip files larger than N bytes (default: 1M)
- `CODESCULPTOR_EXCLUDE` - Glob patterns to exclude (default: `["__pycache__", "node_modules", "*.min.js", "dist"]`)
- `CODESCULPTOR_GIT_AUTHOR` - Override git commit author (default: inferred from git config)
- `CODESCULPTOR_DRY_RUN` - If set to `true`, never make changes (for testing)

**Configuration file:**
`.codesculptor.yaml` (optional):
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

## Troubleshooting

**Issue:** `Error: Uncommitted changes detected. Aborting.`
- **Fix:** Commit or stash changes: `git stash push -m "code-sculptor backup"` before running. Use `--no-commit-check` to bypass (not recommended).

**Issue:** `Transformation failed: mypy errors after formatting`
- **Fix:** Code Sculptor failed to preserve type hints. Run with `--style=none` or fix type errors manually. Check if library stubs are missing.

**Issue:** `Performance regression detected (benchmark -5%)`
- **Fix:** Optimization rolled back automatically. Try lower `--algorithm` level or exclude problematic functions via `.codesculptor.yaml` `skip_transformations`.

**Issue:** `Backup directory not found`
- **Fix:** Previous operation may have cleaned up. Use `git reflog` to recover. Always push before running in production.

**Issue:** `Unsupported language: Go`
- **Fix:** Code Sculptor currently supports Python, JavaScript, TypeScript. For Go, use only audit mode or wait for future release.

**Issue:** `bandit failed to install`
- **Fix:** Install security dependencies: `pip install bandit`. Or run with `--security=skip` to disable.
```