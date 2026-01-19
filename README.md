# Kontexto

[![PyPI version](https://img.shields.io/pypi/v/kontexto.svg)](https://pypi.org/project/kontexto/)
[![Python 3.10+](https://img.shields.io/badge/python-3.10+-blue.svg)](https://www.python.org/downloads/)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)
[![PRs Welcome](https://img.shields.io/badge/PRs-welcome-brightgreen.svg)](https://github.com/ferdinandobons/kontexto/pulls)

> **Give your AI coding assistant a map of your codebase, not a blindfold.**

LLMs and coding agents waste tokens and context window navigating code with `ls`, `grep`, and `find`—tools designed for humans, not machines. They return unstructured text, no relationships, no semantics.

**Kontexto** parses your codebase into a navigable graph of classes, functions, and their relationships. One command returns what would take dozens of shell invocations.

```bash
# The old way: 5 commands, hundreds of lines of noise
ls -la src/
grep -r "authenticate" src/
cat src/api/auth.py | head -100
grep -r "class.*Controller" src/
find . -name "*.py" -exec grep -l "BaseModel" {} \;

# Kontexto: structured, semantic, ready for LLMs
kontexto search "authenticate"    # → JSON with signatures, docstrings, call graph
kontexto hierarchy BaseModel      # → all subclasses, instantly
```

## Why Kontexto?

| | `ls` + `grep` + `find` | **Kontexto** |
|---|---|---|
| **Output** | Raw text, needs parsing | Structured JSON |
| **Understands code?** | No—just text matching | Yes—classes, functions, methods |
| **Relationships** | None | Calls, called-by, inheritance |
| **Context efficiency** | Wastes tokens on noise | Minimal, semantic output |
| **Cross-language** | Manual per language | 10 languages, one interface |

## Installation

```bash
pip install kontexto
```

## Quick Start

```bash
# 1. Index your project (builds the code graph)
cd /path/to/your/project
kontexto index

# 2. Explore
kontexto map                              # Project overview with stats
kontexto expand src/api                   # Drill into a directory
kontexto search "authentication"          # Semantic search
kontexto inspect src/api:UserController   # Entity details + relationships
kontexto hierarchy BaseModel              # Find all subclasses
kontexto read src/api/users.py 10 50      # Read specific lines
```

## Commands

### `kontexto index [path]`

Index a project and build the navigation graph. Automatically detects and parses all supported languages.

```bash
kontexto index                    # Index current directory
kontexto index /path/to/project   # Index specific project
kontexto index -i                 # Incremental update (faster)
```

Creates a `.kontexto/index.db` database with:
- File and directory structure
- Classes, methods, functions (and language-specific entities)
- Signatures and docstrings
- Call relationships
- Class inheritance (base classes)
- TF-IDF search index
- Language metadata for each entity

### `kontexto map [path]`

Show a compact map of the project structure.

```bash
$ kontexto map
```

```json
{
  "command": "map",
  "project": "myapp",
  "root": "/path/to/myapp",
  "stats": {"files": 20, "classes": 8, "functions": 45, "methods": 32},
  "children": [
    {"id": "src", "stats": {"files": 12, "classes": 8, "functions": 45}},
    {"id": "tests", "stats": {"files": 8, "classes": 0, "functions": 24}}
  ]
}
```

### `kontexto expand <path>`

Expand a node to see its children.

```bash
$ kontexto expand src/api/users.py
```

```json
{
  "command": "expand",
  "node": {
    "id": "src/api/users.py",
    "name": "users.py",
    "type": "file",
    "line_end": 95
  },
  "children": [
    {
      "id": "src/api/users.py:UserController",
      "name": "UserController",
      "type": "class",
      "line_start": 10,
      "line_end": 89,
      "signature": "class UserController",
      "docstring": "Handles user API endpoints",
      "base_classes": ["BaseController"]
    }
  ]
}
```

### `kontexto inspect <entity>`

Show detailed info about an entity: signature, docstring, relationships.

```bash
$ kontexto inspect src/api/users.py:UserController.get_user
```

```json
{
  "command": "inspect",
  "node": {
    "id": "src/api/users.py:UserController.get_user",
    "name": "get_user",
    "type": "method",
    "file_path": "src/api/users.py",
    "line_start": 15,
    "line_end": 25,
    "signature": "def get_user(self, user_id: int) -> User",
    "docstring": "Retrieve user by ID from database."
  },
  "calls": ["find_by_id"],
  "called_by": ["src/api/routes.py:user_routes"]
}
```

### `kontexto search <query>`

Search for entities by keyword (names, docstrings, signatures).

```bash
$ kontexto search "authentication"
```

```json
{
  "command": "search",
  "query": "authentication",
  "count": 3,
  "results": [
    {
      "node": {
        "id": "src/api/auth.py:require_auth",
        "name": "require_auth",
        "type": "function",
        "signature": "def require_auth(func: Callable) -> Callable"
      },
      "score": 0.8521
    }
  ]
}
```

Options:
- `--limit, -l`: Maximum number of results (default: 10)

### `kontexto hierarchy <base_class>`

Find all classes that inherit from a given base class.

```bash
$ kontexto hierarchy BaseModel
```

```json
{
  "command": "hierarchy",
  "base_class": "BaseModel",
  "count": 5,
  "subclasses": [
    {
      "id": "src/models/user.py:User",
      "name": "User",
      "type": "class",
      "base_classes": ["BaseModel"],
      "signature": "class User(BaseModel)"
    },
    {
      "id": "src/models/product.py:Product",
      "name": "Product",
      "type": "class",
      "base_classes": ["BaseModel"],
      "signature": "class Product(BaseModel)"
    }
  ]
}
```

### `kontexto read <file> [start] [end]`

Read source code from a file. Outputs raw code (not JSON).

```bash
$ kontexto read src/api/users.py 15 20
```

```python
    def get_user(self, user_id: int) -> User:
        """Retrieve user by ID from database."""
        user = self.user_service.find_by_id(user_id)
        if not user:
            raise NotFoundError(f"User {user_id} not found")
        return user
```

Use line ranges from `expand` or `inspect` to read specific functions.

## Supported Languages

All parsers use [tree-sitter](https://tree-sitter.github.io/) for accurate AST-based extraction.

| Language | Extensions | Entities Extracted |
|----------|------------|-------------------|
| Python | `.py` | functions, classes, methods |
| JavaScript | `.js`, `.jsx`, `.mjs` | functions, classes, methods, arrow functions |
| TypeScript | `.ts`, `.tsx` | functions, classes, methods, interfaces, types |
| Go | `.go` | functions, methods, structs, interfaces |
| Rust | `.rs` | functions, structs, enums, traits, impl blocks, methods |
| Java | `.java` | classes, interfaces, enums, methods, constructors |
| C/C++ | `.c`, `.h`, `.cpp`, `.hpp`, `.cc` | functions, structs, enums, classes, methods, typedefs |
| C# | `.cs` | classes, interfaces, structs, enums, methods, constructors, properties |
| PHP | `.php` | functions, classes, interfaces, traits, methods, enums |
| Ruby | `.rb`, `.rake` | classes, modules, methods, singleton methods |

## How It Works

Kontexto uses tree-sitter to parse source files and builds a navigable graph:

```
Project Root
├── Directories
│   └── Files (.py, .js, .ts, .go, .rs, .java)
│       ├── Classes/Structs/Traits (with base_classes)
│       │   └── Methods
│       ├── Functions
│       ├── Interfaces
│       └── Enums
```

The graph is stored in SQLite with:
- **TF-IDF search index** for keyword search
- **Call relationships** tracking who calls what
- **Class inheritance** tracking base classes
- **Language metadata** for each entity
- **Incremental updates** for large codebases (both graph and search index)

### Architecture

```
┌─────────────────┐
│  CLI Commands   │
└────────┬────────┘
         │
┌────────▼────────┐
│   CodeGraph     │
└────────┬────────┘
         │
┌────────▼────────┐
│ ParserRegistry  │
└────────┬────────┘
         │
    ┌────┴────┬────────┬────────┬────────┐
    ▼         ▼        ▼        ▼        ▼
┌───────┐ ┌──────┐ ┌──────┐ ┌──────┐ ┌──────┐
│Python │ │  JS  │ │  Go  │ │ Rust │ │ Java │
│Parser │ │Parser│ │Parser│ │Parser│ │Parser│
└───┬───┘ └──┬───┘ └──┬───┘ └──┬───┘ └──┬───┘
    │        │        │        │        │
    └────────┴────────┴────────┴────────┘
                      │
              ┌───────▼───────┐
              │  tree-sitter  │
              └───────────────┘
```

## Development

```bash
# Install dev dependencies
pip install -e ".[dev]"

# Run tests
pytest

# Run linter
ruff check src/ tests/
```

## License

MIT
