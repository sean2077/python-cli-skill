# python-cli

A Claude Code skill for building modern Python CLI applications with **typer** + **rich**.

## Installation

### Option 1: Clone and link

```bash
git clone https://github.com/sean2077/python-cli-skill.git
cd python-cli-skill

# Link to your Claude Code skills directory
# macOS/Linux:
ln -s "$(pwd)" ~/.claude/skills/python-cli

# Windows (PowerShell as Admin):
New-Item -ItemType SymbolicLink -Path "$env:USERPROFILE\.claude\skills\python-cli" -Target "$(Get-Location)"
```

### Option 2: Copy directly

```bash
git clone https://github.com/sean2077/python-cli-skill.git
cp -r python-cli-skill ~/.claude/skills/python-cli
```

## Usage

The skill triggers automatically when you ask Claude Code to:

- Create CLI scripts
- Add command-line arguments/options
- Display progress bars
- Format terminal output (tables, panels, colors)
- Integrate logging

Or invoke manually: `/python-cli`

## What's Included

| File | Description |
|------|-------------|
| `SKILL.md` | Quick start, patterns cheatsheet, project structure |
| `references/typer.md` | Commands, arguments, options, callbacks, error handling |
| `references/rich.md` | Console, tables, panels, progress bars, logging |

## Example

```python
import typer
from rich.console import Console
from typing import Annotated

app = typer.Typer()
console = Console()

@app.command()
def hello(
    name: Annotated[str, typer.Argument(help="Username")],
    count: Annotated[int, typer.Option("--count", "-c")] = 1,
):
    for _ in range(count):
        console.print(f"[bold green]Hello[/] {name}!")

if __name__ == "__main__":
    app()
```

## Requirements

```bash
pip install typer rich
```

## License

MIT
