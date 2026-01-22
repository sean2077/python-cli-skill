# Typer Reference

## Table of Contents
- [Basic Structure](#basic-structure)
- [Commands and Subcommands](#commands-and-subcommands)
- [Parameter Types](#parameter-types)
- [Option Configuration](#option-configuration)
- [Callbacks and Hooks](#callbacks-and-hooks)
- [Error Handling](#error-handling)

---

## Basic Structure

### Single Command Script

```python
import typer

def main(name: str):
    print(f"Hello {name}")

if __name__ == "__main__":
    typer.run(main)
```

### Multi-Command Application

```python
import typer

app = typer.Typer()

@app.command()
def hello(name: str):
    print(f"Hello {name}")

@app.command()
def goodbye(name: str):
    print(f"Goodbye {name}")

if __name__ == "__main__":
    app()
```

---

## Commands and Subcommands

### Sub-applications (Command Groups)

```python
import typer

app = typer.Typer()
users_app = typer.Typer()
app.add_typer(users_app, name="users")

@users_app.command()
def create(name: str):
    print(f"Creating user: {name}")

@users_app.command()
def delete(name: str):
    print(f"Deleting user: {name}")

# Usage: python main.py users create alice
```

### Command Help Text

```python
@app.command(help="Create a new user")
def create(name: str):
    """
    Create a new user.

    NAME: The username
    """
    print(f"Creating {name}")
```

---

## Parameter Types

### Argument (Positional)

```python
from typing import Annotated
import typer

@app.command()
def greet(
    name: Annotated[str, typer.Argument(help="Username")],
    times: Annotated[int, typer.Argument()] = 1,
):
    for _ in range(times):
        print(f"Hello {name}")

# Usage: python main.py alice 3
```

### Option (Named)

```python
from typing import Annotated
import typer

@app.command()
def greet(
    name: Annotated[str, typer.Option("--name", "-n", help="Username")],
    formal: Annotated[bool, typer.Option("--formal", "-f")] = False,
):
    greeting = "Good day" if formal else "Hello"
    print(f"{greeting} {name}")

# Usage: python main.py --name alice -f
```

### Common Types

| Type | Example | Description |
|------|---------|-------------|
| `str` | `name: str` | String |
| `int` | `count: int` | Integer |
| `float` | `rate: float` | Float |
| `bool` | `--verbose` | Boolean flag |
| `Path` | `file: Path` | File path |
| `list[str]` | `--item` | Repeatable option |
| `Enum` | `--color` | Enum choice |

### Enum Options

```python
from enum import Enum
from typing import Annotated
import typer

class Color(str, Enum):
    red = "red"
    green = "green"
    blue = "blue"

@app.command()
def paint(
    color: Annotated[Color, typer.Option(help="Choose color")]
):
    print(f"Painting in {color.value}")

# Usage: python main.py --color red
```

### File Paths

```python
from pathlib import Path
from typing import Annotated
import typer

@app.command()
def process(
    input_file: Annotated[Path, typer.Argument(
        exists=True,
        file_okay=True,
        dir_okay=False,
        readable=True,
        help="Input file"
    )],
    output_dir: Annotated[Path, typer.Option(
        exists=True,
        file_okay=False,
        dir_okay=True,
        writable=True,
    )] = Path("."),
):
    print(f"Processing {input_file} -> {output_dir}")
```

---

## Option Configuration

### Prompt Input

```python
@app.command()
def login(
    username: Annotated[str, typer.Option(prompt=True)],
    password: Annotated[str, typer.Option(prompt=True, hide_input=True)],
):
    print(f"Logging in as {username}")
```

### Confirmation Prompt

```python
@app.command()
def delete(
    force: Annotated[bool, typer.Option(
        prompt="Are you sure you want to delete?",
        help="Force delete"
    )] = False,
):
    if force:
        print("Deleted!")
```

### Environment Variables

```python
@app.command()
def connect(
    host: Annotated[str, typer.Option(envvar="DB_HOST")] = "localhost",
    port: Annotated[int, typer.Option(envvar="DB_PORT")] = 5432,
):
    print(f"Connecting to {host}:{port}")
```

---

## Callbacks and Hooks

### Global Options (callback)

```python
import typer

app = typer.Typer()
state = {"verbose": False}

@app.callback()
def main(
    verbose: Annotated[bool, typer.Option("--verbose", "-v")] = False,
):
    """Application description"""
    state["verbose"] = verbose

@app.command()
def run():
    if state["verbose"]:
        print("Verbose mode")
    print("Running...")
```

### Version Option

```python
from typing import Optional
from typing import Annotated
import typer

__version__ = "1.0.0"

def version_callback(value: bool):
    if value:
        print(f"Version: {__version__}")
        raise typer.Exit()

@app.callback()
def main(
    version: Annotated[Optional[bool], typer.Option(
        "--version", "-V",
        callback=version_callback,
        is_eager=True,
    )] = None,
):
    pass
```

---

## Error Handling

### Exit Codes

```python
import typer

@app.command()
def process(file: Path):
    if not file.exists():
        print(f"Error: {file} not found", err=True)
        raise typer.Exit(code=1)
    print(f"Processing {file}")
```

### Abort Execution

```python
@app.command()
def dangerous():
    confirm = typer.confirm("This is dangerous. Continue?")
    if not confirm:
        raise typer.Abort()
    print("Executing...")
```

### Exception Handling

```python
@app.command()
def fetch(url: str):
    try:
        # ... fetch logic
        pass
    except Exception as e:
        print(f"Error: {e}", err=True)
        raise typer.Exit(code=1)
```
