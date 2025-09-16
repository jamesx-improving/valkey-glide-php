# AGENTS: PHP Extension Context for Agentic Tools

This file provides AI agents with the minimum but sufficient context to work productively with the Valkey GLIDE PHP extension. It covers build commands, testing, contribution requirements, and essential guardrails specific to the PHP extension implementation.

## Repository Overview

This is the PHP extension for Valkey GLIDE, providing PHPRedis-compatible API for both standalone and cluster Valkey/Redis connections. The PHP extension is implemented in C using the Zend API and communicates with the Rust core via FFI bindings.

**Primary Languages:** C (Zend extension), PHP (API layer)
**Build System:** PHP extension build system (phpize, configure, make)
**Architecture:** C extension with FFI bindings to Rust core library

**Key Components:**
- `*.c` / `*.h` - C extension source files using Zend API
- `*.stub.php` - PHP stub files for argument info generation
- `src/` - PHP source files and utilities
- `tests/` - PHP test suites
- `examples/` - Usage examples
- `valkey-glide/` - Git submodule containing Rust core library

## Architecture Quick Facts

**Core Implementation:** C extension using Zend API with FFI bindings to Rust glide-core
**Client Types:** ValkeyGlide (standalone), ValkeyGlideCluster (cluster)
**API Style:** PHPRedis-compatible synchronous methods
**Protocol:** Protobuf communication with Rust core via C FFI

**Supported Platforms:**
- Linux: Ubuntu 20+ (x86_64, aarch64)
- macOS: 14.7+ (aarch64/Apple Silicon)
- **Note:** Alpine Linux/MUSL not supported

**PHP Versions:** 8.2, 8.3
**Package:** `valkey-io/valkey-glide-php` (PHP extension)

## Build and Test Rules (Agents)

### Preferred (Make Targets)
```bash
# Build commands
make generate-bindings generate-proto    # Generate bindings and protobuf files
make                                     # Build the extension
make install                             # Install extension to PHP
make build-modules-pre                   # Pre-build step (alias for generate targets)

# Testing
make test                                # Run PHP extension tests
composer test                            # Run tests via Composer

# Code quality
composer lint                            # Run PHP CodeSniffer
composer lint:fix                        # Auto-fix PHP code style
composer analyze                         # Run PHPStan static analysis
composer check                           # Run lint + analyze
./lint.sh                               # Run all linters (PHP + C)

# Setup and dependencies
composer install                         # Install PHP dependencies
phpize                                   # Initialize extension build system
./configure --enable-valkey-glide        # Configure build
```

### Raw Equivalents
```bash
# Manual extension build
phpize
./configure --enable-valkey-glide
make

# Manual FFI library build (required dependency)
cd valkey-glide/ffi && cargo build --release && cd ../..

# Manual protobuf generation
protoc --proto_path=./valkey-glide/glide-core/src/protobuf --php_out=./tests/ ./valkey-glide/glide-core/src/protobuf/connection_request.proto

# Manual linting
phpcs --standard=phpcs.xml
clang-format --dry-run --Werror *.c *.h
phpstan analyze

# Manual test execution
php -n -d extension=modules/valkey_glide.so tests/TestValkeyGlide.php
```

### Build Dependencies Setup
```bash
# Build FFI library (required before extension build)
python3 utils/remove_optional_from_proto.py
cd valkey-glide/ffi
cargo build --release
cd ../../

# Initialize submodules
git submodule update --init --recursive

# Generate PHP protobuf classes for testing
protoc --proto_path=./valkey-glide/glide-core/src/protobuf --php_out=./tests/ ./valkey-glide/glide-core/src/protobuf/connection_request.proto
```

### Test Execution Options
```bash
# Run with Valkey servers
cd tests
./start_valkey_with_replicas.sh 
./create-valkey-cluster.sh 
php -n -d extension=../modules/valkey_glide.so TestValkeyGlide.php

# Memory leak detection with Valgrind
valgrind --tool=memcheck --leak-check=full php -n -d extension=modules/valkey_glide.so tests/TestValkeyGlide.php

# AddressSanitizer build (Linux only)
./configure --enable-valkey-glide --enable-valkey-glide-asan
```

## Contribution Requirements

### Developer Certificate of Origin (DCO) Signoff REQUIRED

All commits must include a `Signed-off-by` line:

```bash
# Add signoff to new commits
git commit -s -m "feat(php): add new command implementation"

# Configure automatic signoff
git config --global format.signOff true

# Add signoff to existing commit
git commit --amend --signoff --no-edit

# Add signoff to multiple commits
git rebase -i HEAD~n --signoff
```

### Conventional Commits

Use conventional commit format:

```
<type>(<scope>): <description>

[optional body]
```

**Example:** `feat(php): implement cluster scan with cursor support`

### Code Quality Requirements

**PHP Linters (via Composer):**
```bash
composer lint                          # Must pass before commit
composer lint:fix                      # Auto-fix code style issues
composer analyze                       # PHPStan static analysis
```

**C Code Linters:**
```bash
clang-format --dry-run --Werror *.c *.h    # C code formatting check
./lint.sh                                  # Run all linters
```

**Individual Tools:**
- `phpcs` - PHP_CodeSniffer with PSR-12 standards
- `phpcbf` - PHP Code Beautifier (auto-fix)
- `phpstan` - Static analysis for PHP
- `clang-format` - C code formatting (Google-based style)
- `valgrind` - Memory leak detection

## Guardrails & Policies

### Generated Outputs (Never Commit)
- `modules/` - Compiled PHP extension files
- `include/glide_bindings.h` - Generated FFI bindings
- `*_arginfo.h` - Generated argument info headers
- `src/` - Generated protobuf files
- `vendor/` - Composer dependencies
- `valkey-glide/ffi/target/` - Rust build artifacts
- `.libs/` - Build artifacts
- `autom4te.cache/` - Autotools cache
- `build/` - PHP extension build files
- `run-tests.php` - Generated test runner

### PHP Extension-Specific Rules
- **FFI Dependency:** Must build `valkey-glide/ffi` Rust library before extension
- **Zend API:** Follow PHP extension development best practices
- **Memory Management:** Use `emalloc`/`efree` for request-scoped memory
- **Error Handling:** Use PHP exception system via `zend_throw_exception`
- **PHPRedis Compatibility:** Maintain API compatibility with PHPRedis where possible
- **Stub Files:** Update `.stub.php` files when adding new methods
- **Protobuf:** Regenerate protobuf files after core changes

### Platform Considerations
- **macOS:** Uses Homebrew paths (`/opt/homebrew/include`, `/opt/homebrew/lib`)
- **Linux:** Supports multiple architectures (x86_64, aarch64)
- **AddressSanitizer:** Available on Linux only (not macOS)
- **Dependencies:** Requires protobuf-c, openssl, PHP dev headers

## Project Structure (Essential)

```
valkey-glide-php/
├── valkey_glide*.c              # Main extension source files
├── valkey_glide*.h              # Extension header files
├── *.stub.php                   # PHP stub files for arginfo generation
├── config.m4                    # Extension build configuration
├── Makefile.frag               # Platform-specific build rules
├── composer.json               # PHP dependencies and scripts
├── phpcs.xml                   # PHP_CodeSniffer configuration
├── .clang-format              # C code formatting rules
├── src/                       # PHP source files and utilities
├── tests/                     # PHP test suites
├── examples/                  # Usage examples
├── utils/                     # Build utilities and scripts
├── valkey-glide/              # Git submodule (Rust core library)
└── lint.sh                    # Comprehensive linting script
```

## Quality Gates (Agent Checklist)

- [ ] FFI library built: `valkey-glide/ffi/target/release/libglide_ffi.a` exists
- [ ] Extension builds: `make` succeeds
- [ ] Extension installs: `make install` succeeds
- [ ] Tests pass: `make test` or `composer test` succeeds
- [ ] PHP linting passes: `composer lint` succeeds
- [ ] C formatting passes: `clang-format --dry-run --Werror *.c *.h` succeeds
- [ ] Static analysis passes: `composer analyze` succeeds
- [ ] No generated outputs committed (check `.gitignore`)
- [ ] DCO signoff present: `git log --format="%B" -n 1 | grep "Signed-off-by"`
- [ ] Conventional commit format used
- [ ] Stub files updated for new methods
- [ ] Memory leaks checked with Valgrind (Linux)

## Quick Facts for Reasoners

**Package:** `valkey-io/valkey-glide-php` (PHP extension)
**API Style:** PHPRedis-compatible synchronous methods
**Client Types:** ValkeyGlide (standalone), ValkeyGlideCluster (cluster)
**Key Features:** C extension with Zend API, FFI bindings to Rust core, PHPRedis compatibility
**Testing:** PHP test framework, Valgrind memory checking, AddressSanitizer support
**Platforms:** Linux (Ubuntu), macOS (Apple Silicon)
**Dependencies:** PHP 8.2+, Rust toolchain, protobuf compiler, C build tools

## If You Need More

- **Getting Started:** [README.md](./README.md)
- **Development Setup:** [DEVELOPER.md](./DEVELOPER.md)
- **Contributing Guidelines:** [CONTRIBUTING.md](./CONTRIBUTING.md)
- **Security:** [SECURITY.md](./SECURITY.md)
- **Examples:** [examples/](./examples/) directory
- **Test Suites:** [tests/](./tests/) directory
- **Build Configuration:** [config.m4](./config.m4) and [Makefile.frag](./Makefile.frag)
- **Linting Script:** [lint.sh](./lint.sh)
- **Rust Core:** [valkey-glide/](./valkey-glide/) submodule
