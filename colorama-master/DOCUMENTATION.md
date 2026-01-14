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

## Architecture

Colorama works by intercepting writes to `sys.stdout` and `sys.stderr` and converting ANSI escape sequences into Windows Console API calls.

### Core Components

1.  **`initialise.py`**: The entry point for the library. It handles the replacement of standard streams with Colorama's wrapped streams.
2.  **`ansitowin32.py`**: Contains the core logic for parsing text and handling streams.
    *   **`StreamWrapper`**: A proxy object that replaces `sys.stdout` or `sys.stderr`. It delegates most operations to the original stream but intercepts `write()` calls.
    *   **`AnsiToWin32`**: The text processor. It scans input text for ANSI sequences using regular expressions. If running on Windows (and not stripping), it separates ANSI codes from plain text.
3.  **`winterm.py`**: Provides an abstraction over the low-level Windows Console API. It tracks the current terminal state (colors, cursor position) and translates high-level actions (e.g., "set foreground red") into specific Win32 API calls.
4.  **`win32.py`**: Contains `ctypes` bindings to `kernel32.dll`, exposing necessary functions like `SetConsoleTextAttribute`, `GetConsoleScreenBufferInfo`, and `SetConsoleCursorPosition`.
5.  **`ansi.py`**: Defines the constants for ANSI escape codes (e.g., `Fore.RED` maps to `\033[31m`).

## Code Flow

### Initialization

1.  The user calls `colorama.init()` or `colorama.just_fix_windows_console()`.
2.  **`just_fix_windows_console()`**:
    *   Checks if the platform is Windows.
    *   Attempts to enable native Virtual Terminal (VT) processing (available in Windows 10+).
    *   If native VT is enabled, it does nothing further (letting the OS handle ANSI codes).
    *   If native VT fails, it falls back to the legacy wrapping method used by `init()`.
3.  **`init()`**:
    *   Checks if wrapping is required (based on `autoreset`, `convert`, `strip` arguments and OS).
    *   Creates `AnsiToWin32` instances for `sys.stdout` and `sys.stderr`.
    *   Replaces `sys.stdout` and `sys.stderr` with `StreamWrapper` objects provided by `AnsiToWin32`.
    *   Registers `reset_all` to run `atexit`, ensuring the terminal is reset to its original state when the program ends.

### Writing Output

When a user executes `print(Fore.RED + "Text")`:

1.  **Interception**: The call is routed to `StreamWrapper.write()`.
2.  **Delegation**: `StreamWrapper` calls `AnsiToWin32.write()`.
3.  **Parsing**:
    *   `AnsiToWin32` uses regex (`ANSI_CSI_RE`) to find ANSI sequences.
    *   It splits the string into "plain text" and "ANSI codes".
4.  **Processing**:
    *   **Plain Text**: Written directly to the original `sys.stdout` (or `stderr`).
    *   **ANSI Codes**: Passed to `AnsiToWin32.convert_ansi()`.
5.  **Conversion**:
    *   `convert_ansi` extracts parameters (e.g., `31` for Red).
    *   It maps the command (e.g., `m` for graphics mode) to a handler in `winterm.py` via `call_win32()`.
6.  **Execution**:
    *   `WinTerm.fore()` (or similar) is called.
    *   `WinTerm` calculates the new attribute value.
    *   It calls `win32.SetConsoleTextAttribute` to update the console buffer's state.

## Key Functions

### `colorama.init(**kwargs)`
Initializes Colorama.
*   **`autoreset`** (bool): If True, resets colors after every print statement.
*   **`convert`** (bool): If True, enables conversion of ANSI codes to Win32 calls. Default depends on OS.
*   **`strip`** (bool): If True, strips ANSI codes from output.
*   **`wrap`** (bool): If False, disables stream wrapping.

### `colorama.just_fix_windows_console()`
A simpler, modern alternative to `init()`.
*   Attempts to enable Windows 10 native ANSI support.
*   Only falls back to Colorama's conversion engine if native support is unavailable.
*   Safe to call multiple times.

### `colorama.deinit()`
Restores `sys.stdout` and `sys.stderr` to their original values, disabling Colorama.

### `colorama.reinit()`
Re-applies the wrapping if it was previously disabled by `deinit()`.

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
