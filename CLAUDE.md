# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

FidoCadJ is a multiplatform EDA (Electronic Design Automation) tool for drawing electrical schematics and PCB layouts. Written in Java (source/target 16), it uses Swing for the GUI with FlatLaf for modern look-and-feel. Licensed under GPL v3.

## Build Commands

The build system uses Make with shell scripts in `dev_tools/`. No Maven or Gradle.

```bash
make compile       # Compile (javac, output to bin/)
make createjar     # Build jar/fidocadj.jar
make run           # Run the application
make rebuild       # Clean + compile + run
make clean         # Remove compiled classes
make cleanall      # Full cleanup (classes, jar, docs)
make createdoc     # Generate Javadoc
```

Compile options: `make compile ARGS="-debug"` for debug symbols, `ARGS="-mac"` to include VAqua7 library.

## Running Tests

Tests are shell-script-based (not JUnit). They require `jar/fidocadj.jar` to exist first.

```bash
make createjar            # Must build jar first
cd test && ./all_tests.sh # Run all tests
```

Individual test suites:
- `test/export/test_export.sh` — export format validation (PNG, JPG, SVG, EPS, PDF, PGF, SCR)
- `test/messages/test_messages.sh` — translation file consistency
- `test/size/test_size.sh` — geometric primitive size calculations

Tests use headless mode (`-n` flag) and compare output against reference files.

## Code Quality

- **Checkstyle**: `./dev_tools/checkstyle.sh` with rules in `dev_tools/rules.xml`
- **PMD**: `./dev_tools/pmd.sh`
- Style: 4-space indentation, max 80 chars/line, Unix (LF) newlines, no tabs

## Architecture

**Entry point**: `fidocadj.FidoMain` — parses CLI args, launches `FidoFrame` (main window).

**MVC pattern in `circuit/`**:
- **Model**: `circuit/model/DrawingModel` — central drawing data store
- **View**: `circuit/views/Drawing`, `circuit/CircuitPanel` — rendering and editor panel
- **Controllers**: `circuit/controllers/` — `EditorActions`, `SelectionActions`, `UndoActions`, `ParserActions`, `CopyPasteActions`, `AddElements`, etc.

**Graphics abstraction** (`graphic/`): `GraphicsInterface` decouples drawing from Swing. Implementations in `graphic/swing/` (screen rendering) and `graphic/nil/` (headless/export mode).

**Primitives** (`primitives/`): All drawing elements extend `GraphicPrimitive` — lines, beziers, ovals, rectangles, polygons, text, PCB pads/traces, macros (component instances).

**Export system** (`export/`): Strategy pattern — `ExportInterface` implemented by format-specific exporters (PDF, SVG, EPS, PGF/TikZ, Eagle SCR, bitmap via `ExportGraphic`).

**Library system** (`librarymodel/` + `macropicker/`): Component library management with tree-based picker UI.

**Headless mode**: The `-n` CLI flag enables non-GUI operation for batch export and testing, using the nil graphics implementation.

## Coding Conventions

- **Java compatibility**: Java 14+ (source/target currently set to 16 in `dev_tools/compile`)
- **Indentation**: 4 spaces, no tabs
- **Line length**: max 80 characters
- **Newlines**: Unix-style LF only
- **Naming**: PascalCase for classes, camelCase for methods/variables, no underscores
- **Javadoc**: required for all public classes and methods
- **Brace style**: opening brace on same line for methods/control structures, next line for classes
- **Quality gates**: Checkstyle (`dev_tools/rules.xml`) and PMD must pass before commits

## Contributing / Committer Checklist

- Code must build successfully
- Must follow coding style above
- Checkstyle must pass with `dev_tools/rules.xml`
- PMD analysis should be clean
- JAR must be generated and automated tests must pass
- Export tests compare output against reference files in `test/export/*/ref/` — update references with `test/export/update_ref.sh` after verified changes

## Dependencies

- **FlatLaf 3.5.1** (`jar/flatlaf-3.5.1.jar`) — cross-platform Swing L&F
- **VAqua7** (`OSes/mac/VAqua7/`) — macOS-native L&F (auto-copied on Darwin)

## Internationalization

Message bundles in `bin/MessagesBundle_xx.properties`. Loaded via `globals/Utf8ResourceBundle`.
