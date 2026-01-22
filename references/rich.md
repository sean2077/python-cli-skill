# Rich Reference

## Table of Contents
- [Console Basics](#console-basics)
- [Styles and Colors](#styles-and-colors)
- [Tables](#tables)
- [Panels](#panels)
- [Progress Bars](#progress-bars)
- [Logging Integration](#logging-integration)
- [Other Components](#other-components)

---

## Console Basics

### Creating Console

```python
from rich.console import Console

console = Console()

# Basic print
console.print("Hello, World!")

# Styled print
console.print("Hello", style="bold red")

# Print to stderr
console.print("Error!", style="red", stderr=True)
```

### print vs log

```python
# print: normal output
console.print("Processing data...")

# log: with timestamp and call location, good for debugging
console.log("Starting process")
console.log("Data loaded", log_locals=True)  # show local variables
```

### Output Control

```python
# Disable colors (e.g., redirecting to file)
console = Console(force_terminal=False)

# Specify width
console = Console(width=120)

# Record output
console = Console(record=True)
console.print("Hello")
html = console.export_html()  # export as HTML
```

---

## Styles and Colors

### Inline Styles (markup)

```python
console.print("[bold]Bold[/bold] [italic]Italic[/italic]")
console.print("[red]Red[/red] [green]Green[/green] [blue]Blue[/blue]")
console.print("[bold red on white]Bold red on white[/bold red on white]")
console.print("[link=https://example.com]Click link[/link]")
```

### Common Styles

| Style | Description |
|-------|-------------|
| `bold` | Bold |
| `italic` | Italic |
| `underline` | Underline |
| `strike` | Strikethrough |
| `dim` | Dimmed |
| `reverse` | Reversed |

### Common Colors

| Color | Description |
|-------|-------------|
| `red`, `green`, `blue` | Basic colors |
| `cyan`, `magenta`, `yellow` | Secondary colors |
| `white`, `black` | Black & white |
| `bright_red`, `bright_green` | Bright colors |
| `#ff5733` | Hex color |
| `rgb(255,87,51)` | RGB |

### Style Object

```python
from rich.style import Style

error_style = Style(color="red", bold=True)
success_style = Style(color="green")

console.print("Error!", style=error_style)
console.print("Success!", style=success_style)
```

---

## Tables

### Basic Table

```python
from rich.table import Table

table = Table(title="User List")
table.add_column("ID", justify="right", style="cyan")
table.add_column("Name", style="magenta")
table.add_column("Status", justify="center")

table.add_row("1", "Alice", "[green]Active[/green]")
table.add_row("2", "Bob", "[red]Inactive[/red]")
table.add_row("3", "Charlie", "[yellow]Pending[/yellow]")

console.print(table)
```

### Table Styling

```python
table = Table(
    title="Report",
    title_style="bold magenta",
    header_style="bold cyan",
    border_style="blue",
    show_lines=True,  # show row separators
    expand=True,      # expand to terminal width
)
```

### Column Configuration

```python
table.add_column(
    "Value",
    justify="right",      # left, center, right
    style="green",
    no_wrap=True,         # disable wrapping
    width=20,             # fixed width
    min_width=10,         # minimum width
    max_width=30,         # maximum width
    ratio=1,              # proportional width
)
```

---

## Panels

### Basic Panel

```python
from rich.panel import Panel

console.print(Panel("Hello, World!"))
console.print(Panel("With title", title="Info"))
console.print(Panel("With subtitle", title="Main", subtitle="v1.0"))
```

### Panel Styling

```python
panel = Panel(
    "Content text",
    title="Title",
    title_align="left",    # left, center, right
    border_style="green",
    padding=(1, 2),        # (vertical, horizontal)
    expand=False,          # don't expand to terminal width
)
console.print(panel)
```

### Nested Content

```python
from rich.panel import Panel
from rich.table import Table

table = Table()
table.add_column("Name")
table.add_row("Alice")

console.print(Panel(table, title="User Table"))
```

---

## Progress Bars

### Simple Progress (track)

```python
from rich.progress import track
import time

for item in track(range(100), description="Processing..."):
    time.sleep(0.01)
```

### Custom Progress Bar

```python
from rich.progress import (
    Progress,
    SpinnerColumn,
    TextColumn,
    BarColumn,
    TaskProgressColumn,
    TimeRemainingColumn,
    TimeElapsedColumn,
)

with Progress(
    SpinnerColumn(),
    TextColumn("[bold blue]{task.description}"),
    BarColumn(),
    TaskProgressColumn(),
    TimeRemainingColumn(),
) as progress:
    task1 = progress.add_task("[cyan]Downloading...", total=1000)
    task2 = progress.add_task("[green]Processing...", total=1000)

    while not progress.finished:
        progress.update(task1, advance=5)
        progress.update(task2, advance=3)
        time.sleep(0.01)
```

### Indeterminate Progress

```python
with Progress() as progress:
    task = progress.add_task("[cyan]Loading...", total=None)
    # total=None shows indeterminate progress (spinning animation)
    time.sleep(2)
```

### Manual Control

```python
progress = Progress()
progress.start()

task = progress.add_task("Processing", total=100)
for i in range(100):
    progress.update(task, advance=1, description=f"Processing {i}")
    time.sleep(0.01)

progress.stop()
```

---

## Logging Integration

### Standard Library logging Integration

```python
import logging
from rich.logging import RichHandler

logging.basicConfig(
    level=logging.DEBUG,
    format="%(message)s",
    datefmt="[%X]",
    handlers=[RichHandler(rich_tracebacks=True)]
)

log = logging.getLogger("myapp")

log.debug("Debug message")
log.info("Info message")
log.warning("Warning message")
log.error("Error message")
log.exception("Exception message")  # shows full traceback
```

### RichHandler Configuration

```python
handler = RichHandler(
    show_time=True,
    show_level=True,
    show_path=True,
    rich_tracebacks=True,
    tracebacks_show_locals=True,
    markup=True,  # allow rich markup in logs
)
```

### With Console

```python
from rich.console import Console
from rich.logging import RichHandler

console = Console(stderr=True)
handler = RichHandler(console=console)

logging.basicConfig(handlers=[handler])
```

---

## Other Components

### Syntax Highlighting

```python
from rich.syntax import Syntax

code = '''
def hello(name: str) -> str:
    return f"Hello {name}"
'''

syntax = Syntax(code, "python", theme="monokai", line_numbers=True)
console.print(syntax)

# Read from file
syntax = Syntax.from_path("main.py", theme="monokai")
console.print(syntax)
```

### Markdown Rendering

```python
from rich.markdown import Markdown

md = """
# Title

This is **bold** and *italic* text.

- Item 1
- Item 2
"""

console.print(Markdown(md))
```

### Tree Structure

```python
from rich.tree import Tree

tree = Tree("[bold]Project Structure")
src = tree.add("[blue]src")
src.add("main.py")
src.add("utils.py")
tests = tree.add("[green]tests")
tests.add("test_main.py")

console.print(tree)
```

### Status Indicator

```python
from rich.status import Status

with console.status("[bold green]Processing...") as status:
    time.sleep(2)
    status.update("[bold blue]Phase 2...")
    time.sleep(2)
```

### Group Output

```python
from rich.console import Group
from rich.panel import Panel

group = Group(
    Panel("First block"),
    Panel("Second block"),
)
console.print(Panel(group, title="Combined"))
```
