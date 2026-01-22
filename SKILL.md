---
name: python-cli
description: Build modern Python CLI applications with typer + rich. Use this skill when creating CLI scripts, adding command-line arguments/options, formatting terminal output, displaying progress bars, integrating logging, or showing tables/panels.
---

# Python CLI (typer + rich)

## Dependencies

```bash
pip install typer rich
```

## Single Script Template

For: utility scripts, one-off tasks, simple automation

```python
#!/usr/bin/env python3
"""Brief description of script functionality"""
from pathlib import Path
from typing import Annotated

import typer
from rich.console import Console

console = Console()
err_console = Console(stderr=True)

def main(
    input_file: Annotated[Path, typer.Argument(help="Input file", exists=True)],
    output: Annotated[Path, typer.Option("-o", "--output", help="Output file")] = None,
    verbose: Annotated[bool, typer.Option("-v", "--verbose", help="Verbose output")] = False,
):
    """Main script functionality description"""
    if verbose:
        console.log(f"Processing: {input_file}")

    try:
        result = process(input_file)
        console.print("[green]Done[/]")
    except Exception as e:
        err_console.print(f"[red]Error:[/] {e}")
        raise typer.Exit(1)

if __name__ == "__main__":
    typer.run(main)
```

## Multi-Command Script Template

For: multiple related operations, CRUD tools

```python
#!/usr/bin/env python3
"""Tool description"""
from pathlib import Path
from typing import Annotated, Optional

import typer
from rich.console import Console
from rich.table import Table

app = typer.Typer(help="Tool description")
console = Console()

@app.command()
def list():
    """List all items"""
    table = Table(title="Items")
    table.add_column("ID", style="cyan")
    table.add_column("Name")
    # ...
    console.print(table)

@app.command()
def add(name: Annotated[str, typer.Argument(help="Name")]):
    """Add new item"""
    console.print(f"[green]Added:[/] {name}")

@app.command()
def remove(
    name: Annotated[str, typer.Argument(help="Name")],
    force: Annotated[bool, typer.Option("--force", "-f", help="Force delete")] = False,
):
    """Remove item"""
    if not force and not typer.confirm(f"Delete {name}?"):
        raise typer.Abort()
    console.print(f"[yellow]Removed:[/] {name}")

if __name__ == "__main__":
    app()
```

## Complex Project Structure

For: large CLI tools, multi-module, installable packages

```
mycli/
├── pyproject.toml
├── src/mycli/
│   ├── __init__.py
│   ├── __main__.py      # python -m mycli entry
│   ├── cli.py           # command definitions
│   ├── core.py          # business logic
│   └── utils/
│       └── console.py   # shared Console
```

**`pyproject.toml`** (key section):
```toml
[project.scripts]
mycli = "mycli.cli:app"
```

**`__main__.py`**:
```python
from .cli import app
app()
```

**`utils/console.py`**:
```python
from rich.console import Console
console = Console()
err_console = Console(stderr=True)
```

## Quick Reference

### Typer

| Need | Code |
|------|------|
| Positional arg | `name: Annotated[str, typer.Argument()]` |
| Named option | `name: Annotated[str, typer.Option("--name", "-n")]` |
| Boolean flag | `verbose: Annotated[bool, typer.Option("-v", "--verbose")] = False` |
| File path | `file: Annotated[Path, typer.Argument(exists=True)]` |
| Password input | `typer.Option(prompt=True, hide_input=True)` |
| Confirmation | `typer.confirm("Are you sure?")` |
| Subcommand group | `app.add_typer(sub_app, name="sub")` |
| Exit code | `raise typer.Exit(code=1)` |

See [references/typer.md](references/typer.md)

### Rich

| Need | Code |
|------|------|
| Colored output | `console.print("[bold red]Error[/]")` |
| Debug log | `console.log("msg", log_locals=True)` |
| Table | `Table()` + `add_column()` + `add_row()` |
| Panel | `Panel("content", title="Title")` |
| Simple progress | `for x in track(items): ...` |
| Custom progress | `Progress()` + columns |
| Logging integration | `RichHandler(rich_tracebacks=True)` |

See [references/rich.md](references/rich.md)

## Common Patterns

### Batch Processing with Progress

```python
from rich.progress import track

for file in track(files, description="Processing..."):
    process(file)
```

### Error Handling

```python
try:
    result = process(data)
    console.print(Panel(result, title="[green]Success"))
except Exception as e:
    err_console.print(f"[red]Error:[/] {e}")
    raise typer.Exit(1)
```
