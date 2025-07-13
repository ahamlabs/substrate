
# 📄 Software Requirements Specification (SRS)

- **Project Name**: Substrate
- **Version**: 0.1.0
- **Author**: Aham Labs
- **Date**: July 13, 2025

---

## 📘 1. Introduction

### 1.1 Purpose
The purpose of this document is to define the functional, non-functional, and system requirements for **Substrate** — a manifest-driven, memory-aware Python environment manager. It aims to provide a reproducible, optimized, conflict-resolving environment experience for developers, researchers, and teams.

### 1.2 Scope
Substrate is a command-line tool written in Go that enables users to:
- Compose layered, declarative Python environments
- Analyze dependency usage and memory footprint
- Resolve version conflicts via microenv isolation
- Support preload/lazy-load strategies for performance tuning

The tool is designed to work with both new and existing Python projects.

### 1.3 Definitions

| Term | Meaning |
|------|--------|
| **Manifest** | A `substrate.yaml` file defining the structure of an environment |
| **Microenv** | An isolated virtual environment for conflicting packages |
| **Access Graph** | A dependency-use graph built from static AST analysis |
| **Digest** | Parsing existing `requirements.txt` or environments into structured manifest |

---

## 🧩 2. Overall Description

### 2.1 Product Perspective
Substrate reimagines traditional Python env tools (like `venv`, `pipenv`, `conda`, `poetry`) by introducing structure, analysis, and system-awareness. It is standalone, written in Go, and offers cross-platform binaries.

### 2.2 Product Functions
- Initialize a manifest-based Python environment
- Convert traditional `requirements.txt` into manifest
- Analyze imports and memory use
- Suggest optimization strategies
- Compose layered environments with reusable base layers
- Run Python scripts within optimized environments

### 2.3 User Classes

| Role | Description |
|------|-------------|
| Developer | Uses CLI to create, analyze, and run envs |
| Researcher | Optimizes memory-heavy ML code |
| DevOps | Builds reproducible pipelines |
| OSS Maintainer | Publishes manifests instead of `requirements.txt` |

---

## 🧱 3. System Features

### 3.1 Manifest Parsing & Composition
- Parse `substrate.yaml` and any extended `base.yaml`
- Validate schema (env name, python version, memory budget, packages)
- Compose an internal environment graph from layers

### 3.2 Dependency Resolution
- Resolve packages and versions
- Detect transitive dependency conflicts
- Use `isolate: true` to split microenvs
- Generate `substrate.lock` for reproducibility

### 3.3 Static Code Analyzer
- Parse AST using `tree-sitter` or `go-python-ast`
- Extract import usage
- Build access graph (file → module → package)
- Identify unused dependencies

### 3.4 Memory Profiler
- Light mode: `gopsutil` for RSS, peaks
- Deep mode: call `memray`, `tracemalloc`, or custom profiler
- Map memory cost to packages/modules
- Suggest `lazy` loading for heavy but rarely used packages

### 3.5 Environment Composer
- Create `.substrate/` cache with:
  - base layer
  - overlays
  - microenvs
- Inject runtime shims (`sitecustomize.py`)
- Handle `PYTHONPATH`, virtualenv, and symlinks

### 3.6 CLI Interface

| Command | Purpose |
|---------|---------|
| `substrate init` | Create a starter manifest |
| `substrate digest` | Convert `requirements.txt` |
| `substrate analyze` | Scan imports + memory use |
| `substrate compose` | Build environment |
| `substrate run script.py` | Run with composed env |
| `substrate freeze` | Lock current config |

---

## 🎯 4. Non-Functional Requirements

### 4.1 Performance
- CLI startup time: < 50ms
- Compose time: < 2s for small envs, < 10s for large ones

### 4.2 Portability
- OS Support: macOS, Linux, Windows (via Go binary)
- Python versions: 3.7 – 3.12

### 4.3 Usability
- One-line install via `curl` or Go
- Minimal config required to get started
- Compatible with existing Python tooling

### 4.4 Extensibility
- Plugin-ready architecture for:
  - Import checkers
  - Memory profilers
  - Lockfile generators

### 4.5 Security
- Does not execute user code during analysis
- Uses isolated subprocess for profiling
- Option to sandbox envs using Docker (future)

---

## ⚙️ 5. System Architecture (Simplified)

```
CLI --> Manifest --> Resolver --> Analyzer --> Profiler --> Composer --> Runner
```

- Written in Go, modular packages under `internal/`
- Uses Cobra for CLI
- Go AST + gopsutil for analysis
- Python envs handled via subprocesses + virtualenv + venv

---

## 📄 6. External Interfaces

### 6.1 File System
- Reads from: `substrate.yaml`, `requirements.txt`
- Writes to: `.substrate/`, `substrate.lock`, access_graph.json

### 6.2 Python Interop
- Calls `python` for version check, module import, memray
- Supports injection via `PYTHONPATH`, `sitecustomize.py`

---

## ✅ 7. Future Enhancements
- Remote manifest resolver (cloud registry)
- Docker + remote runner
- GUI for dependency & memory graph
- VS Code extension
- Rust plugin mode for performance-critical analysis

---

## 📌 8. Appendix

### Example Manifest

```yaml
env_name: nlp-env
extends: base.yaml
python_version: "3.11"
memory_budget: 512MB
packages:
  - name: torch
    version: "2.1.*"
    load_strategy: preload
    isolate: true
  - name: transformers
    version: "4.28"
    load_strategy: lazy
```
