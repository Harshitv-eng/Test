# Colorama Documentation

## Overview

Colorama makes ANSI escape character sequences (for producing colored terminal text and cursor positioning) work under MS Windows. It provides a simple cross-platform API for printing colored terminal text from Python.

## Installation

```bash
pip install colorama
# or
conda install -c anaconda colorama
```

## Usage

### Initialisation

To enable ANSI support on Windows, call `just_fix_windows_console()` (preferred for modern Windows) or `init()`:

```python
from colorama import just_fix_windows_console
just_fix_windows_console()
```

### Colored Output

Use Colorama's constants for foreground, background, and style:

```python
from colorama import Fore, Back, Style
print(Fore.RED + 'some red text')
print(Back.GREEN + 'and with a green background')
print(Style.DIM + 'and in dim text')
print(Style.RESET_ALL)
print('back to normal now')
```

## Project Structure

The repository is organized as follows:

*   **`colorama/`**: Contains the source code for the library.
    *   **`tests/`**: Contains the unit tests for the library.
*   **`demos/`**: Contains example scripts demonstrating various features.

## Development & Testing

### Running Tests

To run tests, you can use the provided `Makefile` (Linux/Mac) or PowerShell scripts (Windows).

**Linux/Mac:**
```bash
make test
```
This runs `python -m unittest discover -p *_test.py`.

**Windows:**
```powershell
.\test.ps1
```

### Makefile Targets

*   `make bootstrap`: Create and populate the virtualenv.
*   `make test`: Run tests.
*   `make build`: Build a release (sdist and wheel).
*   `make test-release`: Test a built release.
*   `make release`: Upload a built release.
*   `make clean`: Remove build artifacts.

### Scripts

*   `test-release`: A bash script to test the currently built release from the `dist/` directory. It uploads to test PyPI and installs it in a sandbox.

## Contributing

See `README-hacking.md` for more details on contributing to Colorama.
