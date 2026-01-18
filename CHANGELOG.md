# Changelog

All notable changes to this project will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [0.1.0] - 2025-01-18

### Added

- **CLI commands** for codebase exploration:
  - `contexto index` - Index a Python project with incremental update support (`-i`)
  - `contexto map` - Show project structure with statistics
  - `contexto expand` - Expand nodes to see children
  - `contexto inspect` - Detailed entity inspection with call relationships
  - `contexto search` - TF-IDF based keyword search
  - `contexto hierarchy` - Find all subclasses of a base class
  - `contexto read` - Read source code with optional line ranges

- **JSON output** for all commands (except `read`) for easy LLM parsing

- **AST parsing** to extract:
  - Functions, classes, and methods
  - Signatures with full argument support (positional-only, keyword-only, defaults)
  - Docstrings
  - Function calls
  - Class inheritance (base classes)

- **SQLite storage** with:
  - WAL mode for concurrent access
  - Optimized indexes for fast queries
  - File hash tracking for incremental updates

- **TF-IDF search engine** with:
  - Tokenization with camelCase/snake_case splitting
  - Stop word filtering
  - Result caching
  - Incremental index updates

- **Performance optimizations**:
  - Batch database inserts
  - Pre-compiled regex patterns
  - Memory-mapped I/O
  - Search result caching

[0.1.0]: https://github.com/ferdinandobons/contexto/releases/tag/v0.1.0
